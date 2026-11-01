# Install btop on AIX

Reading time: 3 min

Applies to: `v1.4.8-aix1` and later, AIX 6.1 TL7 to 7.3    Verified: 2026-11-01, see [test-matrix.md](test-matrix.md)    Revision: 2

## Goal

At the end, `btop` runs on the AIX host from the signed package of a release. Three ways lead there: the RPM, the
tar archive as root, or the tar archive without root in any directory.

## Prerequisites

- `bos.perf.libperfstat` is installed.
- For the RPM: `rpm.rte` 4.9.1.3 or newer (AIX Toolbox) and root. `rpm.rte` 3.0.5 cannot install the package,
  because the package needs `rpmlib(PayloadFilesHavePrefix)`. On such hosts, use the tar archive.
- The RPM refuses to install over a `/usr/bin/btop` that is not its own symlink.
- No UTF-8 locale is needed. The launcher `/opt/btop/bin/btop` adds `--force-utf` when no UTF-8 locale is active.
  It sets `TERM=xterm-256color` when `TERM` is empty.
- Download from the [latest release](https://github.com/mawelo/aix-btop/releases/latest):
  `btop-<VERSION>-<RELEASE>.aix6.1.ppc.rpm` or `.tar.gz`, `SHA256SUMS`, `SHA256SUMS.asc` and `KEYS`.

The examples use `btop-1.4.8-2`. Replace it with the name of the downloaded file.

## Steps

### 1. Verify the download

The RPM carries an OpenPGP signature in its header. `SHA256SUMS.asc` is a detached signature over `SHA256SUMS`.
The signing key is `aix-btop package signing <mawelo@users.noreply.github.com>`, RSA 4096, with the fingerprint
`4C28 DC73 FAC1 2125 F8E3 6A04 CF43 58E9 DA7A DDD4`. The public key is in `KEYS`. Compare the fingerprint also
with the one on the GitHub profile (https://github.com/mawelo.gpg), not only with this file.

On AIX with `rpm.rte` 4.15 or newer (tested on 2026-10-13: 4.15.1 on 7200-05, 4.20.1 on 7300-04; not run with `v1.4.8-aix2`):

```sh
rpm --import KEYS
rpm -K btop-1.4.8-2.aix6.1.ppc.rpm
```

Success: `rpm -K` prints `digests signatures OK`.

On any host with GnuPG:

```sh
gpg --import KEYS
gpg --verify SHA256SUMS.asc SHA256SUMS
sha256sum -c SHA256SUMS
```

Success: `gpg` reports a good signature, and `sha256sum` prints `OK` for each file.

### 2a. Install the RPM

```sh
rpm -ivh btop-1.4.8-2.aix6.1.ppc.rpm
btop          # /usr/bin/btop -> /opt/btop/bin/btop -> /opt/btop/bin/btop.bin
```

Success: `rpm -V btop` prints nothing, and `btop --version` prints the version and a commit of the `aix-public`
branch of the fork.

### 2b. Install the tar archive as root

The archive contains only `opt/btop`.

```sh
cd / && gzip -dc btop-1.4.8-2.aix6.1.ppc.tar.gz | tar -xf -
ln -s /opt/btop/bin/btop /usr/bin/btop   # optional
```

Success: `/opt/btop/bin/btop --version` prints the version.

### 2c. Install the tar archive without root

```sh
mkdir ~/btop && cd ~/btop && gzip -dc /path/to/btop-1.4.8-2.aix6.1.ppc.tar.gz | tar -xf -
~/btop/opt/btop/bin/btop
```

The launcher starts the `btop.bin` next to it and follows symlinks.

Success: btop draws its screen.

## Verify

`btop --version` prints the commit of the fork that the binary was built from. In this repository,
`rpm/BUILD.txt` names the same commit and the build timestamp.

## Remove btop

- RPM: `rpm -e btop`
- tar as root: `rm -rf /opt/btop /usr/bin/btop`
- tar without root: `rm -rf ~/btop`

## If it fails

| Message | Cause | Fix |
|---|---|---|
| `rpm -K` reports "NOT OK", but `rpm -Kv` shows "Header V4 RSA/SHA512 Signature, key ID da7addd4: OK" | `rpm.rte` 4.13 (tested on 7100-05) cannot evaluate the additional signature tag | verify `SHA256SUMS.asc` with GnuPG on another host and compare the SHA-256 sum |
| `rpm-info.txt` shows `Signature: (none)` | the build host wrote the file before signing | none needed; `SHA256SUMS` is the authoritative checksum list |
| `rpm_share: 0645-007 ATTENTION: whichfs() returned an unexpected result` on `rpm -ivh` or `rpm -e`; `rpm -ivh` returns 1 | a mount layout of the host that `rpm_share` cannot map to a file system (seen once with `v1.4.7-aix3` on 7200-05) | none needed if `rpm -V btop` is clean; the files were installed and removed correctly |
| rpm reports a missing `rpmlib(PayloadFilesHavePrefix)` | `rpm.rte` 3.0.5 | use the tar archive (step 2b or 2c) |

## Related

- [test-matrix.md](test-matrix.md): AIX levels on which a release was tested
- [design.md](design.md): why one package runs on AIX 6.1 TL7 to 7.3
- [SECURITY.md](../SECURITY.md): withdrawn releases and how to report a vulnerability
