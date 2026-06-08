# _package

This directory is the Acumatica customization package staging area. It mirrors the structure that Acumatica expects when importing a customization project — screens, pages, project metadata, compiled DLLs, and frontend sources all live here in the layout the platform requires.

The `px` script (at the repo root) packages the contents of this directory, uploads the resulting `.zip` to the Acumatica site, and publishes the customization.

## Why tsconfig.json is here

The TypeScript files under `InventoryItemExt/FrontendSources/` need to eventually land in `Site/FrontendSources/screen/src/development/` on the Acumatica site — that is their real home at build/publish time. From that location, the site's own `tsconfig.json` covers them, so IntelliSense and type checking work correctly.

While editing from inside `_package` (before publishing), those same files sit outside the site's TypeScript project and lose all type information. The `tsconfig.json` here exists solely to restore that IntelliSense without interfering with the site's build:

- **`noEmit: true`** — TypeScript never compiles or outputs anything; it only serves the language server.
- **`baseUrl` → `../../Site/FrontendSources/screen`** — points module resolution at the site's source root, so non-relative imports like `"src/screens/..."` resolve to the same files they would at runtime.
- **`paths.client-controls`** — redirects the `client-controls` package import to `Site/FrontendSources/screen/node_modules/client-controls`, where the actual Acumatica client library lives.
- **`include: ["InventoryItemExt/FrontendSources"]`** — scopes type checking strictly to this package's own files; the site's files are referenced as dependencies only.
- **Compiler options** (`experimentalDecorators`, `lib`, `target`, etc.) match the site's `tsconfig.json` exactly, so the same code that compiles cleanly there shows no false errors here.
