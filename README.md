# kramdown math engine for conversion to HTML

This is a converter for [kramdown](https://kramdown.gettalong.org) that uses
[KaTeX](https://khan.github.io/KaTeX/) and the [sskatex
gem](https://github.com/ccorn/sskatex/) to convert math formulas to HTML+MathML
on the server side.

Note: Until kramdown version 2.0.0 this math engine was part of the kramdown
distribution.


## Installation

~~~ruby
gem install kramdown-math-sskatex
~~~


## Usage

~~~ruby
require 'kramdown'
require 'kramdown-math-sskatex'

Kramdown::Document.new(text, math_engine: :sskatex).to_html
~~~


## Documentation

This math engine uses a server-side installation of [KaTeX] to convert TeX math formulas into
HTML+MathML. This eliminates the need for client-side math-rendering Javascript. Consider this a
flexible for-trusted-users-only alternative to the [KaTeX math engine] and a lightweight efficient
alternative to [Mathjax-Node].

Your HTML templates need no longer include Javascripts for Math (neither `katex.js` nor any
search-and-replace script). Your HTML templates should continue referencing the KaTeX CSS. If you
host your own copy of the CSS, also keep hosting the fonts.

Requirements for running kramdown with math engine SsKaTeX:

- Ruby gem `kramdown-math-sskatex`
- Ruby gem [`sskatex`][sskatex],
- Ruby gem [`execjs`][execjs],
- A Javascript engine supported by ExecJS, e.g. via one of
  - Ruby gem [`therubyracer`][therubyracer],
  - Ruby gem [`therubyrhino`][therubyrhino],
  - Ruby gem [`duktape`][duktape],
  - [Node.js],
- `katex.min.js` from [KaTeX].

All these requirements need to be met only if SsKaTeX is actually used.

A typical SsKaTeX configuration looks like this:

~~~ yaml
math_engine: sskatex
math_engine_opts:
  katex_js: 'path-to-katex/katex.min.js'
~~~
The complete list of available options in `math_engine_opts` follows. Most options, with the notable
exception of `katex_opts`, do not affect usage nor output, but may be needed to make SsKaTeX work
with all the external parts (JS engine and KaTeX). Admins should read the [security
notes](#security).

katex_js
: The path to your copy of `katex.min.js`. Defaults to `'katex/katex.min.js'`. For a relative path,
  the starting point is the current working directory.

katex_opts
: A dictionary with general KaTeX options such as `throwOnError`, `errorColor`, `colorIsTextColor`,
  and `macros`. See the [KaTeX documentation][KaTeXopts] for details. Use `throwOnError: false` if
  you want parse errors highlighted in the HTML output rather than raised as exceptions when
  compiling. Note that `displayMode` is computed dynamically and should not be specified here.

js_run
: An identifier for the Javascript engine to use. Recognized identifiers include: `RubyRacer`,
  `RubyRhino`, `Duktape`, `MiniRacer`, `Node`, `JavaScriptCore`, `Spidermonkey`, `JScript`, `V8`,
  and `Disabled`; that last one would raise an error on first use. Which engines are actually
  available depends on your installation.

  If **js_run** is not defined, the contents of the environment variable `EXECJS_RUNTIME` will be
  considered instead; and if that is not defined, an automatic choice will be made. For more
  information, use the **verbose** option and consult the [execjs] documentation.

js_dir
: The path to a directory with Javascript helper files. Defaults to the subdirectory `js` in the
  data directory of the `sskatex` gem. There is no need to change that setting unless you want to
  experiment with Javascript details.

js_libs
: A list of UTF-8-encoded Javascript helper files to load. Relative paths are interpreted relative
  to **js_dir**. The default setting is

  ~~~ yaml
  js_libs:
    - escape_nonascii_html.js
    - tex_to_html.js
  ~~~
  And there is no need to change that unless you want to experiment with Javascript details.

  Files available in the default **js_dir** are:

  escape_nonascii_html.js
  : defines a function `escape_nonascii_html` that converts non-ASCII characters to HTML numeric
    character references. Intended as postprocessing filter.

  tex_to_html.js
  : defines a function `tex_to_html`(*tex*, *display_mode*, *katex_opts*) that takes a LaTeX math
    string, a boolean display mode (`true` for block display, `false` for inline), and a dict with
    general KaTeX options, and returns a string with corresponding HTML+MathML output. The
    implementation is allowed to set `katex_opts.displayMode`. SsKaTeX applies `tex_to_html` to each
    math fragment encountered. The implementation given here uses `katex.renderToString` and
    postprocesses the output with `escape_nonascii_html`.

verbose
: Whether to log the engine configuration. Defaults to `false`. When set to something else, logs
  (by converter warnings) the identifiers of the available Javascript engines, the selected engine
  (cf. option **js_run**), the names of the Javascript files loaded, and the contents of
  **katex_opts**. That happens only once per new configuration.

debug
: For debugging. Defaults to `false`. When set to something else, prints (by `warn`) every
  information that the **verbose** option would log, and every Javascript expression used after
  engine initialization, immediately to `stderr`.

### Security

The options with `js` in their names can be used to achieve arbitrary Javascript code execution with
the privileges of the kramdown process. If kramdown is part of a service that allows file uploads
and user-specified math engine selection and options, the service should therefore be sandboxed
(which is recommended practice), or the user-specified options filtered, or SsKaTeX kept unavailable
e.g. by not installing the `sskatex` gem. Consider using the [KaTeX math engine] instead.

### Differences to the KaTeX math engine

Both the KaTeX and the SsKaTeX engine operate in similar ways with similar efficiency and produce
equivalent output if the underlying `katex.min.js` versions are the same. Differences are mostly in
configuration and usage scenarios. The following table gives an overview.

| Usage aspect | KaTeX | SsKaTeX |
|:-|:-|:-|
| Usability | easy to use | all JS details configurable |
| Target users | untrusted users | trusted users |
| Target usage | web services | personal pages |
| Required Ruby gem | `katex` | `sskatex` |
| KaTeX JS/CSS/fonts | included | not included |
| Math language depends on | `katex` gem version | `katex_js` option |
| KaTeX options | in `math_engine_opts` | in `katex_opts` sub-dict |
| Default error handling | catches errors | throws on errors |

Therefore:

* If in doubt, try the [KaTeX math engine]. It should work out of the box.
* If you need more control and are trusted (i.e. you are allowed to run arbitrary code), try
  SsKaTeX. Example uses:
  - Work around issues in the current KaTeX version or in the default JS interpreter
  - Try a more recent KaTeX version when you want it, and not until then
  - Try interfacing with other JS-based math renderers
  - Try another Javascript interpreter. E.g. `Duktape` is fast, but `Spidermonkey` may give better
    error diagnostics and backtraces.
* Web services processing kramdown input from untrusted users should not make the `sskatex` gem
  available, as explained in the [Security](#security) section.

[duktape]: https://github.com/judofyr/duktape.rb#duktaperb
[execjs]: https://github.com/rails/execjs#execjs
[KaTeX math engine]: https://github.com/kramdown/math-katex
[Mathjax-Node]: https://github.com/kramdown/math-mathjaxnode
[KaTeX]: https://khan.github.io/KaTeX/
[KaTeXopts]: https://github.com/Khan/KaTeX#rendering-options
[sskatex]: https://github.com/ccorn/sskatex/
[therubyracer]: https://github.com/cowboyd/therubyracer#therubyracer
[therubyrhino]: https://github.com/cowboyd/therubyrhino#therubyrhino
[Node.js]: https://nodejs.org/


## Development

Clone the git repository and you are good to go. You probably want to install
`rake` so that you can use the provided rake tasks.


## License

MIT - see the **COPYING** file.
