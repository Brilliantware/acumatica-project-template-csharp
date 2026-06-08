# px — Acumatica Customization Publish Script

`px` is a Node.js wrapper around `Px.CommandLine.exe` that automates the init → copy → build → upload → publish pipeline for Acumatica customization projects.

---

## Prerequisites

- **Node.js** available on `PATH` (or `node.exe` in the solution root as a fallback).
- **`Site\` symlink** pointing at the local Acumatica instance — see [SETUP.md](SETUP.md).
- **`.env`** file in the project folder (see [Configure .env](#configure-env) below).

---

## Usage

Run from the solution root (where `px` lives):

```sh
node px <projectName> [flags]
```

`projectName` is optional when run from a directory that contains a `.csproj` — `px` will auto-detect it.

### Flags

| Flag | Short | Description |
|---|---|---|
| `--init` | `-i` | Create `_package/<name>/Bin/` and `_package/<name>/_project/ProjectMetadata.xml` if missing |
| `--copy` | `-c` | Copy the compiled `<projectName>.dll` into `_package/<name>/Bin/` |
| `--build` | `-b` | Zip the package source into `_package/<name>.zip` using `Px.CommandLine.exe` |
| `--upload` | `-u` | Upload the zip to the Acumatica instance via the Customization API |
| `--publish` | `-p` | Publish the uploaded package inside Acumatica |
| `--name <n>` | `-n` | Override the package name (folder, zip, Acumatica project name). Defaults to `PACKAGE_NAME` in `.env`, then `projectName`. The `.csproj` and compiled DLL always use `projectName`. |
| `--help` | `-h` | Print usage |

Flags compose freely. The full pipeline:

```sh
node px MyProject -icbup
```

Upload only (skip publish for faster iteration):

```sh
node px MyProject -icbu
```

Build locally without touching the server:

```sh
node px MyProject -icb
```

Rebuild with a different Acumatica package name:

```sh
node px MyProject -icbup --name "MyProject_Client"
```

---

## Configure .env

Create a `.env` file in the project folder (gitignored — never commit it):

```env
ACUMATICA_INSTANCE_PATH=../Site
ACUMATICA_BASE_URL=http://localhost/<InstanceName>
ACUMATICA_USERNAME=Admin
ACUMATICA_PASSWORD=<password>
ACUMATICA_COMPANY=<CompanyCD>

# Optional: override the generated package name
# PACKAGE_NAME=SomethingElse
```

`px` also accepts these via environment variables or CLI flags (`--instancePath`, `--pxCommandLine`, `--targetPath`) for scripted use.

### `Px.CommandLine.exe` resolution order

1. `--pxCommandLine` CLI flag
2. `PX_COMMAND_LINE` environment variable
3. `Px.CommandLine.exe` next to the `px` script
4. `<ACUMATICA_INSTANCE_PATH>\Bin\Px.CommandLine.exe`

### Compiled DLL resolution order (`--copy`)

Searches in this order for `<projectName>.dll`:

1. `--targetPath` CLI flag / `TARGET_PATH` env var
2. `bin/Debug/net4.8/`
3. `bin/Debug/net48/`
4. `bin/Debug/`
5. `bin/net4.8/`
6. `bin/net48/`
7. `bin/`

---

## .csproj PostBuild integration

The standard `PostBuild` target in the project's `.csproj` runs `px` automatically after every successful build:

```xml
<Target Name="PostBuild" AfterTargets="PostBuildEvent">
  <Exec Command="node ../px $(ProjectName) -icbup" />
</Target>
```

Adjust the flags to taste:

- Drop `-p` to skip publish (publish is the slow step).
- Drop `-u` to work entirely offline.
- Comment out the whole `<Target>` block to publish manually.

If `node` is not on `PATH`, place `node.exe` in the solution root and reference it as `$(SolutionDir)node.exe`.

---

## _package folder structure

`px --init` creates this layout under the project folder:

```
_package/
  <packageName>/
    Bin/                          ← compiled DLL lands here (--copy)
    _project/
      ProjectMetadata.xml         ← Acumatica project name/description
  <packageName>.zip               ← built package (--build)
```

The `_package/` folder also stores customization project elements exported from Acumatica (screens, DAC extensions, SQL table scripts, site map entries). These are excluded from the C# build and committed to git.
