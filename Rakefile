# -*- ruby -*-

require 'rubygems/package_task'
require 'fileutils'
require 'rake/clean'
require 'rake/testtask'
require_relative 'lib/kramdown/converter/math_engine/sskatex'

task :default => :test
Rake::TestTask.new do |test|
  test.warning = false
  test.libs << 'test'
  test.test_files = FileList['test/test_*.rb']
end

desc "Check for SsKaTeX availability"
task :test_sskatex_deps do
  katexjs = 'katex/katex.min.js'
  raise (<<TKJ) unless File.exists? katexjs
Cannot find file '#{katexjs}'.
You need to download KaTeX e.g. from https://github.com/Khan/KaTeX/releases/
and extract at least '#{katexjs}'.
Alternatively, if you have a copy of KaTeX unpacked somewhere else,
you can create a symbolic link 'katex' pointing to that KaTeX directory.
TKJ
  html = %x{echo '$$a$$' | \
            #{RbConfig.ruby} -Ilib -S kramdown --no-config-file --math-engine sskatex}
  raise (<<KTC) unless $?.success?
Some requirement by SsKaTeX or the employed JS engine has not been satisfied.
Cf. the above error messages.
KTC
  raise (<<XJS) unless / class="katex"/ === html
Some static dependency of SsKaTeX, probably the 'sskatex' or the 'execjs' gem,
is not available.
If you 'gem install sskatex', also make sure that some JS engine is available,
e.g. by installing one of the gems 'duktape', 'therubyracer', or 'therubyrhino'.
XJS
  puts "SsKaTeX is available, and its default configuration works."
end

desc "Update kramdown SsKaTeX test reference outputs"
task update_sskatex_tests: [:test_sskatex_deps] do
  # Not framed in terms of rake file tasks to prevent accidental overwrites.
  Dir['test/testcases/*.text'].each do |f|
    stem = f[0..-6] # Remove .text
    ruby "-Ilib -S kramdown --config-file #{stem}.options #{f} >#{stem}.html"
  end
end

SUMMARY = 'kramdown-math-sskatex uses katex to convert math elements to HTML+MathML on the server side'

PKG_FILES = FileList.new(['COPYING', 'VERSION', 'CONTRIBUTERS', 'lib/**/*.rb', 'test/**/*'])

CLOBBER << "VERSION"
file 'VERSION' do
  puts "Generating VERSION file"
  File.open('VERSION', 'w+') {|file| file.write(Kramdown::Converter::MathEngine::SsKaTeX::VERSION + "\n")}
end

CLOBBER << 'CONTRIBUTERS'
file 'CONTRIBUTERS' do
  puts "Generating CONTRIBUTERS file"
  `echo "  Count Name" > CONTRIBUTERS`
  `echo "======= ====" >> CONTRIBUTERS`
  `git log | grep ^Author: | sed 's/^Author: //' | sort | uniq -c | sort -nr >> CONTRIBUTERS`
end

spec = Gem::Specification.new do |s|
  s.name = 'kramdown-math-sskatex'
  s.version = Kramdown::Converter::MathEngine::SsKaTeX::VERSION
  s.summary = SUMMARY
  s.license = 'MIT'

  s.files = PKG_FILES.to_a

  s.require_path = 'lib'
  s.required_ruby_version = '>= 2.3'
  s.add_dependency 'kramdown', '~> 2.0'
  s.add_dependency 'sskatex', '>= 0.9.37'

  s.has_rdoc = true

  s.author = 'Thomas Leitner'
  s.email = 't_leitner@gmx.at'
  s.homepage = "https://github.com/kramdown/math-sskatex"
end

Gem::PackageTask.new(spec) do |pkg|
  pkg.need_zip = true
  pkg.need_tar = true
end

task :gemspec => ['CONTRIBUTERS', 'VERSION'] do
  print "Generating Gemspec\n"
  contents = spec.to_ruby
  File.write("kramdown-math-sskatex.gemspec", contents, mode: 'w+')
end
CLOBBER << 'kramdown-math-sskatex.gemspec'

desc 'Release version ' + Kramdown::Converter::MathEngine::SsKaTeX::VERSION
task :release => [:clobber, :package, :publish_files]

desc "Upload the release to Rubygems"
task :publish_files => [:package] do
  sh "gem push pkg/kramdown-math-sskatex-#{Kramdown::Converter::MathEngine::SsKaTeX::VERSION}.gem"
  puts 'done'
end
