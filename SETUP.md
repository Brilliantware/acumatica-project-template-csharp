# Acumatica Customization Project — Build & Publish Setup

## One-time: Create the `Site` symlink

From an **elevated (Administrator) command prompt**, run `Setup.cmd` from the solution root, or run the command directly:

```cmd
mklink /D Site "C:\Program Files\Acumatica ERP\<InstanceName>"
```

Replace `<InstanceName>` with the folder name of your local Acumatica instance (e.g. `Marble`).

`Site\` is a directory symlink pointing at the local instance. It is used for two purposes:
- **Library references** — all `PX.*` and other Acumatica DLLs are resolved from `Site\Bin\` in the `.csproj`, so no NuGet packages are needed.
- **Source reference** — partial Acumatica source code is available at `Site\App_Data\CodeRepository` and can be used as local documentation.

> `Site\` must exist before building. The symlink is gitignored; every developer creates it locally.

---

## One-time: Place the `px` script in the solution root

The post-build publish step requires `px` — the custom publish script — in the solution root. This is not part of the repository; obtain it from the team.

### `node.exe` in the solution root (optional)

The post-build command references `$(SolutionDir)node.exe` directly. A local copy of `node.exe` was added as a version-pinning fallback when the system Node.js caused problems.

If your system Node.js works correctly with `px`, you can delete the local `node.exe` and update the post-build command in the `.csproj` to use the system `node` instead:

```xml
<Exec Command="node $(SolutionDir)\px $(ProjectName) -icbup --online" />
```

If you experience issues with the `px` script, try placing a known-good `node.exe` back in the solution root and reverting the command to `$(SolutionDir)node.exe`.

---

## Configure `.env`

Each project folder contains a `.env` file that tells the publish tool where to deploy:

```env
ACUMATICA_INSTANCE_PATH=../Site
ACUMATICA_BASE_URL=http://localhost/<InstanceName>
ACUMATICA_USERNAME=Admin
ACUMATICA_PASSWORD=<password>
ACUMATICA_COMPANY=<CompanyCD>
```

`.env` is gitignored. Copy from `.env.sample` if present and fill in the values for your local instance.

---

## Post-build publish

The `.csproj` `PostBuild` target runs automatically after every successful build:

```xml
<Target Name="PostBuild" AfterTargets="PostBuildEvent">
  <Exec Command="$(SolutionDir)node.exe $(SolutionDir)\px $(ProjectName) -icbup --online" />
</Target>
```

This calls the `px` script, which runs the following steps in order:

| Flag | Step | Description |
|---|---|---|
| `-i` | Initialize | Creates the `_package/<ProjectName>/` basic structure |
| `-c` | Copy | Copies the compiled DLL to `_package/<ProjectName>/Bin/` |
| `-b` | Build | Zips the package into `_package/<ProjectName>.zip` |
| `-u` | Upload | Uploads the zip to the Acumatica instance (requires `--online`) |
| `-p` | Publish | Publishes the uploaded package in Acumatica |

`--online` is required for upload; an offline mode existed previously but no longer works.

### Skipping publish for faster iteration

Drop `-p` from the flags to package and upload without publishing:

```xml
<Exec Command="$(SolutionDir)node.exe $(SolutionDir)\px $(ProjectName) -icbu --online" />
```

Publishing is the slow step. The package is always rebuilt regardless. You can also comment out the entire `PostBuild` target and publish manually when needed.

---

## `app_pool.cmd` — IIS pool helper

Stops all IIS application pools and starts only the one for this instance. Run from an elevated prompt when you need to recycle the app pool after publishing:

```cmd
app_pool.cmd
```

Edit the pool name inside the file (`Marble`) to match your instance.

---

## Project folder conventions

These subfolders exist alongside the `.csproj` but are **excluded from the C# build**:

| Folder | Purpose |
|---|---|
| `_package\` | Exported Acumatica customization project elements (screens, DAC extensions, SQL table scripts, site map, access rights). Not compiled; committed to git. |
| `sql\` | Documentation-only SQL scripts. Not required for build or publish. |
| `docs\` | Developer notes. Not compiled. |
