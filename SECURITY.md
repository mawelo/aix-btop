# Security policy

## Reporting a vulnerability

Please report security problems of the AIX port or of the packages privately through
[GitHub private vulnerability reporting](https://github.com/mawelo/aix-btop/security/advisories/new),
not as a public issue. Problems in btop itself that are not specific to AIX belong to the
[upstream project](https://github.com/aristocratos/btop).

## Supported releases

Only the latest release receives fixes.

| Release | Status |
|---|---|
| `v1.4.7-aix4` and later | supported |
| `v1.4.7-aix3` | superseded: control characters in process names reached the terminal of the user running btop (fixed in `v1.4.7-aix4`) |
| `v1.4.7-aix1`, `v1.4.7-aix2` | withdrawn: the themes were group-writable (release 1 tar: also owned by uid/gid 1000); control characters as in release 3 |

If release 1, 2 or 3 is installed, upgrade with `rpm -Uvh` (or unpack the release 4 tar archive); see advisory
[GHSA-p45r-rq5h-mrcj](https://github.com/mawelo/aix-btop/security/advisories/GHSA-p45r-rq5h-mrcj).

For release 1 or 2, the following closes only the file-mode problem, not the control-character problem:

```sh
chown -R root:system /opt/btop
chmod -R go-w /opt/btop
```

## Terminal assumptions

btop writes UTF-8, and the launcher `/opt/btop/bin/btop` adds `--force-utf` when no UTF-8 locale is active. Since
`v1.4.7-aix4`, process names and arguments have C0 control characters, DEL and UTF-8 encoded C1 controls replaced
by `?`. Other bytes from 0x80 to 0x9F pass unchanged (continuation bytes of other UTF-8 characters, or invalid
UTF-8); a terminal in 8-bit mode (for example Latin-1) can read 0x9B as CSI and 0x9D as OSC. Use a UTF-8 terminal, which btop needs anyway.

## Verifying packages

Each release is signed; see "Verifying the download" in the README. Signing key fingerprint:
`4C28 DC73 FAC1 2125 F8E3 6A04 CF43 58E9 DA7A DDD4`, also listed on
[github.com/mawelo.gpg](https://github.com/mawelo.gpg).
