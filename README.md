# Automerge JSPM Provider

This package is a JSPM provider which allows hosting of ES6 modules out of an Automerge-backed registry.

The basic approach is to translate URLs following a particular pattern (using a pseudo-URL with a made-up, invalid hostname) into queries against one or more registry documents.

If the package is found in the local registry document, it will be resolved to a module document. The module document holds metadata, essentially the contents of package.json as an Automerge document, as well as an additional field `fileContents` which has all the actual file contents.

The Provider also includes an NPM module cacher which relies on the accuracy of the `files` array found in `package.json`. This array is generally inaccurate for projects found randomly on the internet, but the JSPM.io provider promises its accuracy for its mirrored packages and anecdotally it seems to work so far.

This package also pairs nicely with runtime import map rewriting thanks to `es-module-shims`.

# Usage

TBD

# Acknowledgements / Credits

Originally authored by Peter van Hardenberg (Ink & Switch).

Thanks to Guy Bedford for his assistance in understanding JSPM guts and taking upstream pull requests to make this a reality (and of course, his many years of development on JSPM / es-module-shims that made it possible to build this.)
