# Security policy

## Reporting a vulnerability

Please report security problems of the AIX port or of the packages privately through
[GitHub private vulnerability reporting](https://github.com/mawelo/aix-btop/security/advisories/new),
not as a public issue. Problems that are not specific to AIX belong to the
[upstream project](https://github.com/aristocratos/btop).

## Supported releases

Only the latest release receives fixes.

| Release | Status |
|---|---|
| `v1.4.8-aix3` | supported |
| `v1.4.8-aix1`, `v1.4.8-aix2` | superseded: the search path for libraries contains directories that do not belong there (see "Library search path"; fixed in `v1.4.8-aix3`) |
| `v1.4.7-aix4` | superseded: invalid UTF-8 in process names passed unchanged (matters only on a terminal in 8-bit mode); on AIX 6.1 and 7.1 the main loop spun for up to one second per update cycle (both fixed in `v1.4.8-aix1`) |
| `v1.4.7-aix3` | superseded: control characters in process names reached the terminal of the user running btop (fixed in `v1.4.7-aix4`) |
| `v1.4.7-aix1`, `v1.4.7-aix2` | withdrawn: the themes were group-writable (`v1.4.7-aix1` tar: also owned by uid/gid 1000); control characters as in `v1.4.7-aix3` |

If `v1.4.7-aix1`, `-aix2` or `-aix3` is installed, upgrade with `rpm -Uvh` (or unpack the tar archive of the latest release); see advisory
[GHSA-p45r-rq5h-mrcj](https://github.com/mawelo/aix-btop/security/advisories/GHSA-p45r-rq5h-mrcj).

For `v1.4.7-aix1` or `-aix2`, the following closes only the file-mode problem, not the control-character problem:

```sh
chown -R root:system /opt/btop
chmod -R go-w /opt/btop
```

## Terminal assumptions

btop writes UTF-8, and the launcher `/opt/btop/bin/btop` adds `--force-utf` when no UTF-8 locale is active. Since
`v1.4.7-aix4`, process names and arguments have C0 control characters, DEL and UTF-8 encoded C1 controls replaced
by `?`. Since `v1.4.8-aix1`, invalid UTF-8 is replaced by `?` as well, so a lone byte 0x9B or 0x9D no longer reaches
a terminal in 8-bit mode (for example Latin-1), where it would read as CSI or OSC. Up to `v1.4.7-aix4` such bytes
passed unchanged. Use a UTF-8 terminal, which btop needs anyway.

## Library search path

The binary of the releases `v1.4.7-aix1` to `v1.4.8-aix2` has a search path for libraries. This search path contains
directories that do not belong there. The AIX loader searches them before `/usr/lib` for `libc.a`, `libpthreads.a` and `libperfstat.a`.
Since `v1.4.8-aix3`, the search path is `/usr/lib:/lib`.

Upgrade with `rpm -Uvh` (or unpack the tar archive of the latest release). `dump -X64 -H /opt/btop/bin/btop.bin`
shows the search path as path 0 under "Import File Strings"; after the upgrade it is `/usr/lib:/lib`.

## Upstream status

Upstream btop has the same problem in its process list: [aristocratos/btop#1846](https://github.com/aristocratos/btop/issues/1846)
(escape sequences through the process name). The issue was closed as "not planned" on 2026-10-18 without a
comment; upstream `main` has no fix as of 2026-10-31 (last checked commit `fd1d292`, after the release v1.4.8). This port closes the paths
listed there: the process name (`v1.4.7-aix4`), the shortened command in the narrow tree view, DEL and UTF-8 encoded
C1 controls (`v1.4.7-aix4`), and invalid UTF-8 (`v1.4.8-aix1`). Upstream builds do not include these fixes, on AIX or
any other platform.

## Verifying packages

Each release is signed; `docs/install.md`, step 1, shows how to verify it. Signing key fingerprint:
`4C28 DC73 FAC1 2125 F8E3 6A04 CF43 58E9 DA7A DDD4`, also listed on
[github.com/mawelo.gpg](https://github.com/mawelo.gpg).
