# Build btop for AIX from the patches

Reading time: 3 min

Applies to: the patch series on top of upstream `v1.4.8`, AIX 7.2 TL5 and 7.3, IBM Open XL C/C++ 17.1.3
Verified: 2026-11-01 (`git am --keep-non-patch` on Linux; the CI built the same tree on AIX 7.2 TL5)    Revision: 2

## Goal

At the end, a btop binary for AIX 7.2 TL5 or 7.3 exists, built from the upstream source and the patch series.
For AIX 7.1 and 6.1 a different build applies; see [If the target is AIX 7.1 or 6.1](#if-the-target-is-aix-71-or-61).

## Prerequisites

- AIX 7.2 TL5 or 7.3 with IBM Open XL C/C++ 17.1.3 (LLVM 19, libc++ 19).
- CMake 3.25 or newer (the C++26 build was tested with 4.2.0), and Ninja or GNU make.
- `git` and a copy of this repository (for `patches/`).

## Steps

### 1. Get the source with the patches

Apply the series on top of the upstream base commit:

```sh
git clone https://github.com/aristocratos/btop
cd btop
git checkout cf1da5900b1e9eaa1c20fd7148dd812b43340b8a
git am --keep-non-patch /path/to/aix-btop/patches/*.patch
```

Success: `git am` applies all five patches without conflict, and `git log --format=%s -5` shows `[AI generated]`
at the start of each subject.

The result is identical to the `aix-public` branch of the fork, which can be used instead:

```sh
git clone -b aix-public https://github.com/mawelo/btop
```

### 2. Build

```sh
export PATH=/opt/freeware/bin:$PATH
OBJECT_MODE=64 cmake --preset aix-openxl && cmake --build --preset aix-openxl
```

The compiler paths are absolute (`/opt/IBM/openxlC/17.1.3/bin/ibm-clang++_r`), see
`cmake/Toolchain-aix-openxl.cmake`. For another level of Open XL, pass `-DBTOP_OPENXL_ROOT=<PATH>`.

Success: `build/aix-openxl/btop --version` prints the version and the commit.

## Verify

`git rev-parse HEAD^{tree}` in the source tree prints the same tree as `git rev-parse aix-public^{tree}` in a clone
of the fork. `btop --version` names the commit of the source tree.

## If it fails

| Message | Cause | Fix |
|---|---|---|
| The commit subjects lack `[AI generated]` | `git am` without `--keep-non-patch` removes every bracketed prefix of the subject, not only `[PATCH n/5]` | `git reset --hard cf1da5900b1e9eaa1c20fd7148dd812b43340b8a`, then apply again with `git am --keep-non-patch`; the tree is the same either way |
| `git am` reports `trailing whitespace` and `new blank line at EOF` | the two LLVM patch files in `tools/aix/sysroot71` have empty context lines with one space, which the diff format needs; the unchanged Lua 5.4.8 sources end with a blank line | none needed; the patches apply |

## If the target is AIX 7.1 or 6.1

AIX 7.1 and 6.1 have no C++26 compiler. The IBM runtime of 7.2 does not load there, so a binary that uses it does not start.
The binaries for these releases are therefore built on a 7.2 host against a sysroot of the target release and a
libc++ that is built statically against that sysroot. The presets are `aix71-openxl` and `aix61-openxl`.
The steps are in `docs/aix/howto/build-for-older-aix.md` of the source tree. The released package is the
`aix61-openxl` build.

## Related

- [patches.md](patches.md): content of each patch
- [design.md](design.md): why one package runs on AIX 6.1 TL7 to 7.3
- README of the source tree, section "Compilation AIX"
