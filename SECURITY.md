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
| latest (`v1.4.7-aix3` and later) | supported |
| `v1.4.7-aix1`, `v1.4.7-aix2` | withdrawn: the themes were group-writable (release 1 tar: also owned by uid/gid 1000) |

If release 1 or 2 is installed, upgrade (`rpm -Uvh`) or fix the modes and owner:

```sh
chown -R root:system /opt/btop
chmod -R go-w /opt/btop
```

## Verifying packages

Each release is signed; see "Verifying the download" in the README. Signing key fingerprint:
`4C28 DC73 FAC1 2125 F8E3 6A04 CF43 58E9 DA7A DDD4`, also listed on
[github.com/mawelo.gpg](https://github.com/mawelo.gpg).
