# Security

## Reporting a vulnerability
Open a private security advisory on this repository, or email the maintainer. Please do not open a public issue for a vulnerability until it has been addressed.

## What this software does and does not do
- **No telemetry.** The application does not phone home, track usage, or send analytics.
- **Network access is explicit.** The only outbound network activity is the data collection you start (web/site crawling, ingest of URLs you provide) and the on-demand download of optional dependencies into `.datatypexxl_deps` from PyPI / official package sources.
- **Third-party code and data** are fetched at collection time for your dataset — they are not redistributed inside the application.
- **Downloaded files are treated as data, never executed.** The site-file harvester blocks executables/scripts by extension *and* by magic bytes (a `.pdf` that is actually a PE/ELF binary is rejected), and skips ad/tracker URLs.

## Code signing & integrity
- The Windows executable is **Authenticode-signed with a locally generated (self-signed) certificate**. This proves the file has not been altered since it was built on the author's machine, but it is **not** a paid CA (DigiCert/Sectigo) certificate — so Windows SmartScreen may still warn on first run until the publisher builds reputation or a commercial certificate is used.
- **Verify integrity with the published SHA-256 checksums** (`CHECKSUMS.txt` in each release) before running a download:
  - Windows: `Get-FileHash .\<file> -Algorithm SHA256`
  - Linux/macOS: `sha256sum <file>`
  If the hash matches the release's `CHECKSUMS.txt`, the file is byte-for-byte the published artifact.
- **Linux:** distributed as source; build it yourself with `build.sh` (Python 3.14) and verify against the checksums. No pre-signed Linux binary is shipped.

## Building from source (most trustworthy path)
The whole toolchain is open. Build the exe yourself with `build.ps1` (Windows) / `build.sh` (Linux) under Python 3.14; the build runs the full test suite and code-signs locally. Nothing in the binary that isn't in this repository.

## Honest limitations
- A self-signed certificate establishes *integrity* (unchanged since build), not *identity* attested by a trusted authority. For distribution without SmartScreen warnings, a commercial code-signing certificate is required — that is a purchase the project owner must make; it cannot be faked.
