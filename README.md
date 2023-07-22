# Automerge JSPM Provider

This package is a JSPM provider which allows hosting of ES6 modules out of an Automerge-backed registry.

The basic approach is to translate URLs following a particular pattern (using a pseudo-URL with a made-up, invalid hostname) into queries against one or more registry documents.

If the package is found in the local registry document, it will be resolved to a module document. The module document holds metadata, essentially the contents of package.json as an Automerge document, as well as an additional field `fileContents` which has all the actual file contents.

The Provider also includes an NPM module cacher which relies on the accuracy of the `files` array found in `package.json`. This array is generally inaccurate for projects found randomly on the internet, but the JSPM.io provider promises its accuracy for its mirrored packages and anecdotally it seems to work so far.

This package also pairs nicely with runtime import map rewriting thanks to `es-module-shims`.

# Usage

## Note that this is a brand-new prototype and APIs are subject to change without warning!

```
import { Generator } from "@jspm/generator"
import { Repo } from "@automerge/automerge-repo"
import * as Automerge from "@automerge/automerge"
import { IndexedDBStorageAdapter } from "@automerge/automerge-repo-storage-indexeddb"
import { BrowserWebSocketClientAdapter } from "@automerge/automerge-repo-network-websocket"
import { AutomergeRegistry } from "./automerge-provider.js"

const PRECOOKED_BOOTSTRAP_DOC_ID = "441f8ea5-c86f-49a7-87f9-9cc60225e15e"
const PRECOOKED_REGISTRY_DOC_ID = "6b9ae2f8-0629-49d1-a103-f7d4ae2a31e0"

// Step one: Set up an automerge-repo.
const repo = new Repo({
  storage: new IndexedDBStorageAdapter(),
  network: [new BrowserWebSocketClientAdapter("wss://sync.inkandswitch.com")],
})

// put it on the window to reach it from the fetch command elsewhere (this is a hack)
window.repo = repo
window.automerge = Automerge

function bootstrap(key, initialDocumentFn) {
  const docId = localStorage.getItem(key)
  if (!docId) {
    const handle = initialDocumentFn(repo)
    localStorage.setItem(key, handle.documentId)
    return handle
  } else {
    const handle = repo.find(docId)
    return handle
  }
}

await registryDocHandle.value()
const registry = new AutomergeRegistry(repo, registryDocHandle))

// Intercept fetch calls and return our data instead. Is this evil? Maybe.
registry.installFetch()

// We stash the registry on the window so we can access it from the debugger for convenience and hackery
window.registry = registry

// Only now can we load the es-module-shims: if we do it before now, the fetch won't be trapped.
window.esmsInitOptions = {
  shimMode: true,
  mapOverrides: true,
  fetch: window.fetch,
}
await import("https://ga.jspm.io/npm:es-module-shims@1.8.0/dist/es-module-shims.js")

/** Remove the close comment to re-run the bootstrapping process and repopulate the registry
 * eventually, this should move into the module editor. Feel free to do so!
 * /

// This code imports the bootstrap document and its dependencies to the registry
const packageName = "@trail-runner/bootstrap"
registry.linkPackage(packageName, "0.0.1", `${PRECOOKED_BOOTSTRAP_DOC_ID}`)
await registry.update(packageName)

// This code creates a reusable importMap based on the current registry you have
// and stores it in the bootstrap document.
const generator = (window.generator = new Generator({
  resolutions: { "@automerge/automerge-wasm": "./web/" },
  defaultProvider: "automerge",
  customProviders: {
    automerge: registry.jspmProvider(),
  },
}))
await generator.install(packageName) // this should load the package above
bootstrapDocHandle.change((doc) => {
  doc.importMap = generator.getMap()
})
/**/

const importMap = (await bootstrapDocHandle.value()).importMap
if (!importMap) {
  throw new Error("No import map found in bootstrap document! Run the code above.")
}

console.log("Bootstrapping...")
// registry.updateImportMap("@trail-runner/bootstrap")
// this (does the below) we could maybe do this in the importShim??? or in fetch?
importShim.addImportMap(importMap)
const Bootstrap = await import("@trail-runner/bootstrap")
```

# Acknowledgements / Credits

Originally authored by Peter van Hardenberg (Ink & Switch).

Thanks to Guy Bedford for his assistance in understanding JSPM guts and taking upstream pull requests to make this a reality (and of course, his many years of development on JSPM / es-module-shims that made it possible to build this.)
