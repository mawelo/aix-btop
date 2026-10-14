# Packages

The packages are not stored in this directory. They are attached to the GitHub release
[v1.4.7-aix4](https://github.com/mawelo/aix-btop/releases/tag/v1.4.7-aix4) (all releases: [Releases](https://github.com/mawelo/aix-btop/releases)).

Direct downloads of the current release:

- [`btop-1.4.7-4.aix6.1.ppc.rpm`](https://github.com/mawelo/aix-btop/releases/download/v1.4.7-aix4/btop-1.4.7-4.aix6.1.ppc.rpm)
- [`btop-1.4.7-4.aix6.1.ppc.tar.gz`](https://github.com/mawelo/aix-btop/releases/download/v1.4.7-aix4/btop-1.4.7-4.aix6.1.ppc.tar.gz)
- [`KEYS`](https://github.com/mawelo/aix-btop/releases/download/v1.4.7-aix4/KEYS)
- [`SHA256SUMS`](https://github.com/mawelo/aix-btop/releases/download/v1.4.7-aix4/SHA256SUMS)
- [`SHA256SUMS.asc`](https://github.com/mawelo/aix-btop/releases/download/v1.4.7-aix4/SHA256SUMS.asc)

This directory holds the metadata of that release:

- `SHA256SUMS`, `SHA256SUMS.asc`: checksums of the release files and their signature (key in `../KEYS`)
- `rpm-info.txt`: `rpm -qpi`, requirements and file list of the package
- `BUILD.txt`: commit and build timestamp

Verify: `gpg --verify SHA256SUMS.asc SHA256SUMS && sha256sum -c SHA256SUMS`
