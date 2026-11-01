# Test matrix and package contents

Reading time: 4 min

Applies to: `v1.4.8-aix3` (fork commit `a9f05e8`)    Verified: 2026-11-01    Revision: 6

## Package

| File | Built against | Expected to run on |
|---|---|---|
| `btop-<VERSION>-<RELEASE>.aix6.1.ppc.rpm` and the same tree as `.tar.gz`, attached to each [release](https://github.com/mawelo/aix-btop/releases) | AIX 6100-09 sysroot, static libc++ 19 | AIX 6.1 TL7 and later, 7.1, 7.2 and 7.3, every technology level ([design.md](design.md) gives the reasons) |

"aix6.1" in the file name is the minimum release. rpm on 7.1 to 7.3 accepts the package through its table of
compatible OS releases. The release number counts per upstream version: `v1.4.8-aix1` follows `v1.4.7-aix4`, `v1.4.8-aix2` follows `v1.4.8-aix1`, and `v1.4.8-aix3` follows `v1.4.8-aix2`.

## Test runs per AIX level

The matrix lists one test run per AIX level with `v1.4.8-aix3` (`a9f05e8`) on 2026-11-01. The column rpm.rte
gives the level of the installed `rpm.rte` fileset, taken from the software inventory of the hosts on 2026-10-30
and 2026-10-31. "Load" means two single-threaded busy loops next to btop. "Launcher" means that
`opt/btop/bin/btop` was started through a symlink in another directory, without `LANG`.

The runs used the RPM and the tar archive of the CI build before signing, except on 7300-02-01, which got the
signed release package. The released RPM has the same payload
(sha256 of the `rpm2cpio` output) and adds the header signature. The released tar archive is the same file.

| AIX level | rpm.rte | Installed via | Started | Under load | Removed again |
|---|---|---|---|---|---|
| 6100-07-10 | 4.13.0 (not used: test as non-root user) | tar, in a private directory | yes, launcher | yes | yes |
| 6100-09-04 | 4.9.1 (not used: test as non-root user) | tar, in a private directory | yes, launcher | yes | yes |
| 7100-02-01 | 3.0.5 | tar, in a private directory | yes, launcher | yes | yes |
| 7100-05-03 | 4.13.0 | `rpm -ivh`, `rpm -V` | yes, through `/usr/bin/btop` | no | `rpm -e`, no files left |
| 7200-05-05 | 4.15.1 | tar, in a private directory | yes, launcher | yes | yes |
| 7300-02-01 | 4.18.1 | `rpm -Uvh` of the signed release package over `v1.4.8-aix2`, `rpm -V`, no group- or world-writable files | `--version` only | no | no (kept installed) |
| 7300-04-00 | 4.20.1 | `rpm -ivh`, `rpm -V` | yes, through `/usr/bin/btop` | yes | `rpm -e`, no files left |

On every tar host, `sh btop` and `ksh btop` inside `opt/btop/bin` started the binary next to them, although another
`btop` came first in `PATH`. A busy process showed up as `x?]0;PWNED??y`.
Its name contained ESC, BEL and the byte 0x9B (invalid UTF-8); btop replaced each of the three bytes with `?`.

The binary searches libraries only in `/usr/lib:/lib`. The CI build checks this with `dump -X64 -H` for all three
builds, and the build stops otherwise. On 7300-02-01, `dump -X64 -H` showed the same value for the installed binary.

## MCP server runs

`btop --mcp-stdio` ran from the tar archive on 6100-07, 6100-09, 7100-02, 7200-05 and 7300-04, one run each,
without `LANG` and with the default registry of tools. Each run gave 7 answers: initialize, tools/list, three tool
calls (cpu, processes, disks), an unknown tool and ping. No tool call returned an error. btop rejected the
unknown tool with JSON-RPC error -32602.

## Not tested

These levels were not tested: 6.1 TL8, 7.1 TL0/TL1/TL3/TL4, 7.2 TL0 to TL4, and 7.3 TL0/TL1/TL3. With this release, `rpm -K` did not run on AIX. On 7300-02-01, `rpm -Uvh` reported `NOKEY` for the signature, because `KEYS` is not imported there; the signature was therefore not checked on AIX. AIX 6.1 and 7.1 are out of IBM support; the port does not change that.

## Files of the package

| Path | Content |
|---|---|
| `/opt/btop/bin` | launcher `btop` (shell script) and binary `btop.bin` |
| `/opt/btop/share/btop/themes` | themes |
| `/opt/btop/share/btop/mcp` | tool registry of `--mcp-stdio` |
| `/opt/btop/share/doc/btop` | `README.md`, `README.AIX`, `LICENSE`, `THIRD-PARTY-NOTICES.txt` (MIT texts of Lua and nlohmann/json) |
| `/usr/bin/btop` | symlink to the launcher (RPM only) |
| `~/.config/btop` | configuration per user |
| `~/.local/state/btop.log` | log per user |

In this repository, `rpm/rpm-info.txt` lists the header, the requirements and the file list of the package.
`rpm/BUILD.txt` names the commit and the build timestamp.
