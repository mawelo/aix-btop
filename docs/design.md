# Why one package runs on AIX 6.1 TL7 to 7.3

Reading time: 3 min

Applies to: `v1.4.8-aix1` and later
Verified: 2026-11-01 (imports of the 6.1 and 7.1 builds against the exports of 6100-09 and 7100-05; runs, see
[test-matrix.md](test-matrix.md)); the accepted sizes were measured on 2026-10-12    Revision: 3

The releases ship one package for every AIX level from 6.1 TL7 to 7.3. It is the build against the AIX 6.1
headers and libraries. The build for 7.1 stays in the source tree as a build check but is not shipped. This
document explains why one binary is expected to cover all these levels. The levels that were actually tested
are in [test-matrix.md](test-matrix.md).

## The fields that btop reads have the same offsets everywhere

btop reads six structures from libperfstat: `perfstat_cpu_total_t`, `perfstat_cpu_t`, `perfstat_memory_total_t`,
`perfstat_disk_t`, `perfstat_netinterface_t` and `perfstat_logicalvolume_t`. The fields that btop reads sit at
the same offsets in every `libperfstat.h` from AIX 5.3 to 7.3.

The structure sizes do not only grow. `perfstat_netinterface_t` has 240 bytes in 6.1.8, 224 in 7.1.0 and 7.1.1,
and 240 again from 7.2 on. The table gives the sizes in bytes per header version.

| Structure | Sizes |
|---|---|
| `perfstat_cpu_t` | 528 (5.3, 6.1.2), 616 (6.1.7), 624 (6.1.8, 7.1), 640 (7.2, 7.3) |
| `perfstat_netinterface_t` | 208 (5.3, 6.1.2), 224 (6.1.7, 7.1.0, 7.1.1), 240 (6.1.8, 7.2, 7.3) |
| `perfstat_cpu_total_t` | 624, 696 |
| `perfstat_memory_total_t` | 208, 352 (`real_avail` exists since 6.1.7) |
| `perfstat_disk_t` | 472, 496 |
| `perfstat_logicalvolume_t` | 216, 224 |

btop reads up to byte 96 in `perfstat_cpu_t` and up to byte 184 in `perfstat_netinterface_t` (field offset plus
field size). It reads up to byte 264 in `perfstat_disk_t`, 216 in `perfstat_logicalvolume_t`, 224 in
`perfstat_cpu_total_t`, and 336 in `perfstat_memory_total_t` (`real_avail`; without it 136). Each lies within the
smallest size of its structure. The exception is `real_avail`, which the older headers do not have and which then
reads as zero.

The header versions are the `libperfstat.h` levels 5.3.9.7, 6.1.2.8, 6.1.7.0, 6.1.8.0, 7.1.0.1, 7.1.1.1, 7.2.5.201
and 7.3.1.0, shortened to three digits above. They are not technology levels. The table is
`tools/aix/probes/perfstat_layout/offsets_5.3-7.3.txt` in patch 0004; the generator in the same directory produced
it. The IBM headers themselves are not included.

## btop probes the size that libperfstat accepts

libperfstat rejects a `sizeof_struct` that is larger than the structure that the library was built with
(`EINVAL`). The binary is built against the 6100-09 sysroot, whose header has the 6.1.8 sizes. On `EINVAL` btop
shrinks the size in steps of 8 bytes until the call succeeds. Fields beyond the accepted size read as zero.

Measured: 6100-07 accepts 616 instead of 624 for `perfstat_cpu_t` and 224 instead of 240 for
`perfstat_netinterface_t`; 7100-02 accepts 616. The library accepts the size unchanged if its structure is at
least as large as the structure of the binary. The same probe handles a smaller structure. For
`perfstat_netinterface_t` this also covers 7.1 TL0 and TL1 with 224 bytes, which was not measured. The probe
applies to all six calls above.

## What the C library of 6.1 lacks is replaced

- `aligned_alloc` is missing before 6.1 TL9; `posix_memalign` replaces it.
- The POSIX 2008 locale calls are missing. The statically linked C++ runtime gets a shim with one global locale.
  `std::locale("xx_XX")` therefore does not create a separate locale. btop uses only the "C" locale.
- `O_CLOEXEC` is missing and defined as 0. The flag only takes effect when the process starts another
  program with `exec`, and btop on AIX starts none.
- btop calls `_Exit` instead of `quick_exit`.

6.1 TL7 is the oldest level that was examined: its C library exports every symbol that the binary imports. TL6
and older were not examined. The binary imports only `libc.a`, `libpthreads.a` and `libperfstat.a`.

## Related

- [test-matrix.md](test-matrix.md): tested levels
- `docs/aix/` in the [`aix-public` branch of the fork](https://github.com/mawelo/btop/tree/aix-public/docs/aix):
  how the 7.1 and 6.1 builds work and which AIX pitfalls the port met
