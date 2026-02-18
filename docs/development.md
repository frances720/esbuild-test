# Development

Here are some quick notes about how I develop esbuild.

## Primary workflow

My development workflow revolves around the top-level [`Makefile`](../Makefile), which I use as a script runner.

1. **Build**

Assuming you have [Go](https://go.dev/) installed, you can compile esbuild by running `make` in the top-level directory (or `go build ./cmd/esbuild` if you don't have `make` installed). This creates an executable called `esbuild` (or `esbuild.exe` on Windows).

2. **Test**

You can run the tests written in Go by running `make test-go` in the top-level directory (or `go test ./internal/... ./pkg/...` if you don't have `make` installed).

If you want to run more kinds of tests, you can run `make test` instead. This requires installing [node](https://nodejs.org/). And it's possible to run even more tests than that with additional `make` commands (read the [`Makefile`](../Makefile) for details).

3. **Publish**

Here's what I do to publish a new release:

1. Bump the version in [`version.txt`](../version.txt)
2. Copy that version into [`CHANGELOG.md`](../CHANGELOG.md)
3. Run `make publish-all` and follow the prompts

## Running in the browser

If you want to test esbuild in the browser (lets you try out lots of things rapidly), you can:

1. Run `make platform-wasm` to build the WebAssembly version of esbuild
2. Serve the repo directory over HTTP (such as with `./esbuild --servedir=.`)
3. Visit [`/scripts/try.html`](../scripts/try.html) in your browser

## Frequently asked questions

### Why is esbuild written in Go instead of JavaScript?

Performance. JavaScript isn't great for heavy parallelism because of Node's single-threaded nature and garbage collector overhead. Go compiles to native code, has efficient goroutines for parallelism, and a low-overhead garbage collector. This lets esbuild fully utilize all CPU cores during bundling.

### How can I debug esbuild's bundling process?

You can take a CPU trace using the `--trace=[file]` flag. View the resulting trace file with `go tool trace [file]`. This shows exactly what work is happening in parallel and helps identify performance bottlenecks.

### Why does esbuild use only three AST passes?

For better cache locality and performance. Most compilers have many more passes because separating concerns makes code easier to maintain. esbuild combines as much work as possible into three passes:

1. Lexing + parsing + scope setup + symbol declaration
2. Symbol binding + constant folding + syntax lowering + syntax mangling
3. Printing + source map generation

### How does esbuild handle both CommonJS and ES6 modules?

The parser processes a superset of both module systems. You can use CommonJS syntax (`require`, `exports`, `module`) and ES6 syntax (`import`, `export`) in the same file. CommonJS modules are wrapped in closures, while ES6 modules use scope hoisting for better performance and smaller bundle sizes.

### What should I know before modifying the parser?

A few key points:

- The lexer runs on-the-fly during parsing, not ahead of time
- Lookahead is limited to one token in most cases (TypeScript is an exception)
- Import path resolution syscalls have high overhead, so results are cached
- Data structures must remain immutable to support watch mode
