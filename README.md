# pgvector Windows binaries (unofficial)

> [!IMPORTANT]
> **This is an unofficial project.** It is not affiliated with, maintained by, or
> endorsed by the [pgvector](https://github.com/pgvector/pgvector) project or its
> authors. It simply automates a native Windows build of pgvector and republishes
> the result. Use at your own risk.

pgvector doesn't ship prebuilt Windows binaries, so building it means installing
Visual Studio and running `nmake` yourself. This repo does that for you: a GitHub
Actions workflow watches [pgvector/pgvector](https://github.com/pgvector/pgvector)
for new version tags, compiles the extension natively on Windows against every
supported PostgreSQL major version, and publishes the packaged `.dll` + SQL files
to this repo's [Releases](../../releases).

## Downloads

Grab the archive matching your PostgreSQL major version from the
[Releases page](../../releases). Each release corresponds to an upstream pgvector
version tag (e.g. `v0.8.0`) and contains one zip per PostgreSQL major:

```
pgvector-v0.8.0-pg18-windows-x64.zip
pgvector-v0.8.0-pg17-windows-x64.zip
pgvector-v0.8.0-pg16-windows-x64.zip
...
```

## Installation

1. Download the archive that matches your PostgreSQL major version.
2. Extract it and copy the contents into your PostgreSQL install directory,
   merging the folders:
   - `vector.dll` → `<PostgreSQL>\lib\`
   - `vector.control` and `vector--*.sql` → `<PostgreSQL>\share\extension\`
3. In `psql`, enable the extension:

   ```sql
   CREATE EXTENSION vector;
   ```

Match the archive to your server's **major** version (the binaries are built
against a specific PostgreSQL minor, but the extension ABI is stable within a
major line).

## How it works

The workflow ([`.github/workflows/build-pgvector-windows.yml`](.github/workflows/build-pgvector-windows.yml))
runs three jobs:

1. **check** — resolves the target pgvector version (latest upstream tag, or
   a version you pass manually) and skips the build if this repo already has a
   matching release. It also queries
   [`postgresql.org/versions.json`](https://www.postgresql.org/versions.json) to
   discover every currently supported PostgreSQL major and its latest minor, so
   the build matrix stays current without hardcoding version numbers.
2. **build** — for each PostgreSQL major, downloads the official EnterpriseDB
   Windows binaries (for the headers/libs), sets `PGROOT`, and compiles pgvector
   with MSVC via `nmake /F Makefile.win` — the same native toolchain described in
   pgvector's Windows install docs. Packages the result into a zip.
3. **release** — collects all the zips and publishes them to a single release in
   this repo, tagged with the pgvector version.

### Can GitHub Actions compile this natively for free?

Yes. GitHub's `windows-latest` runners are real Windows Server VMs with Visual
Studio 2022 and the MSVC C++ toolchain preinstalled — the same environment as the
"x64 Native Tools Command Prompt" the pgvector docs call for. This is a genuine
native build, not cross-compilation. GitHub Actions is free for public
repositories, so no extra cost for this workflow.

## Triggers

- **Scheduled** — checks daily for a new upstream tag and builds it
  automatically. (Scheduled runs only fire from the repo's default branch.)
- **Manual** — via the Actions tab → *Build pgvector for Windows* → *Run
  workflow*, with optional inputs:
  - `version` — a specific pgvector tag to build (e.g. `v0.8.0`). Defaults to the
    latest upstream tag.
  - `force` — rebuild even if a release for that version already exists.

## License

pgvector itself is distributed under its own license; each built archive includes
the upstream `LICENSE`. The workflow and scripts in this repository are provided
as-is.
