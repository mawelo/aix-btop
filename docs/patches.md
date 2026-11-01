# Patch series

Reading time: 2 min

Applies to: the patch series on top of upstream `v1.4.8` (`cf1da5900b1e9eaa1c20fd7148dd812b43340b8a`)
Verified: 2026-11-01 (`git am --keep-non-patch` of the series gives the tree of `aix-public`)    Revision: 3

`patches/` holds the series as `git am` files. `patches/BASE-COMMIT.txt` names the upstream base commit. The
`aix-public` branch of the fork [`mawelo/btop`](https://github.com/mawelo/btop/tree/aix-public) carries the same
patches as commits on top of that base. How to apply them is in [build.md](build.md).

## Content of each patch

| Patch | Content |
|---|---|
| 0001 | Platform layer `src/aix/`: libperfstat (CPU, memory, disks, network), `mntctl`/`statvfs` (file systems), `getprocs64`/`getargs` (processes), `getifaddrs` replacement, executable path lookup; small shared-code changes (`pthread_join`, `_Exit`, AIX signal table, `select` instead of `pselect` on AIX); two shared fixes (terminal-size error logged once, initialization order of `Fx::reset`) |
| 0002 | Build system: CMake branch, presets `aix-openxl`, `aix71-openxl`, `aix61-openxl`, IBM Open XL toolchain files, Makefile platform `aix`, README section, `docs/aix/` |
| 0003 | Tooling to build for AIX 7.1 and 6.1 on a 7.2 host: sysroot preparation, `extern "C"` header wrappers, static libc++/libc++abi/libunwind 19.1.7 with two LLVM patches, compatibility headers |
| 0004 | RPM spec and launcher, `THIRD-PARTY-NOTICES.txt` of the package, probes and check tools |
| 0005 | Scripting API: core API, Lua 5.4.8 in a sandbox, `btop --mcp-stdio` (server for the Model Context Protocol, MCP, over stdio; tools in `mcp/tools.lua`), shared headers in `libs/reuse` (Lua sandbox, tool registry, MCP protocol and stdio transport, JSON bridge, nlohmann/json), GTest suites |

## Effect on other platforms

Patches 0001 to 0004 are meant to change no behaviour on other platforms. Their code changes outside `src/aix/`,
`cmake/` and `tools/aix/` are behind `_AIX` or `CMAKE_SYSTEM_NAME STREQUAL "AIX"`. The exceptions are the two
shared fixes named for 0001.

Two build-system changes are unconditional but neutral. The Makefile collects sources with `$(wildcard ...)`
instead of `find -maxdepth`. CMake gains the cache variable `BTOP_GIT_COMMIT`, which is used only when set.

Patch 0005 applies to every platform. CMake always builds Lua and the scripting layer, and the menu sends signals
through `Core::send_signal`. Without `--mcp-stdio` btop behaves as before.

## Marking

Every commit subject carries `[AI generated]`, the marking that the upstream `CONTRIBUTING.md` asks for in the
section "AI generated code". The `aix-public` branch keeps the marking. Each release commit of the branch carries
a tag (`v1.4.7-aix2` to `v1.4.7-aix4`, `v1.4.8-aix1`, `v1.4.8-aix2`), because the branch is rebuilt for each release.

## Related

- [build.md](build.md): apply the series and build btop
- Documentation of the code in the fork: `docs/aix/README.md`, `docs/btop_core.md`, `docs/btop_lua.md`,
  `docs/btop_mcp.md`, `docs/lua-api.md` and `docs/mcp.md` in the
  [`aix-public` branch](https://github.com/mawelo/btop/tree/aix-public/docs)
