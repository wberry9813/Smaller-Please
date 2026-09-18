# Third-Party Notices

This file distinguishes **Smaller, Please Core's own license** from the licenses of the
separate **Smaller, Please Media Pack** and its system dependencies. It is engineering
compliance information, not legal advice.

## Smaller, Please Core

Smaller, Please Core (the `smaller` CLI, the `contextslim` legacy alias, the native host, the
lifecycle, and the browser extension) is licensed under the **MIT License** — see `LICENSE`.
The Media Pack is a **separate program and a separate distribution artifact**
(`smaller-media-<version>-macos-arm64.tar.gz`); it is **not** covered by Core's MIT license, is
never linked into Core, and the Core release archive does not embed it. Core executes it only as
a subprocess after validating the payload; Core does not grow because of the Media Pack.

## Smaller, Please Media Pack (FFmpeg)

The Media Pack (`smaller-media-<version>-macos-arm64.tar.gz`) contains an unmodified build of
**FFmpeg 7.1**:

- License: **GNU Lesser General Public License, version 2.1 or later** (LGPL-2.1-or-later).
- No GPL, nonfree, or (L)GPLv3 component is enabled: the build uses `--disable-gpl`,
  `--disable-nonfree`, `--disable-version3`, and no external library. There is **no
  libx264/libx265/xvid** or other GPL library. H.264 encoding uses Apple's VideoToolbox;
  AAC uses FFmpeg's native `aac` encoder.
- The full LGPL v2.1 text ships inside the artifact at `licenses/COPYING.LGPLv2.1`, and
  FFmpeg's own license overview at `licenses/LICENSE.md`.
- Copyright: "FFmpeg is free software ... Copyright (c) 2000–2024 the FFmpeg developers."
  FFmpeg is a trademark of Fabrice Bellard.

### Attribution

> This software uses code of [FFmpeg](https://ffmpeg.org) licensed under the
> [LGPLv2.1](https://www.gnu.org/licenses/old-licenses/lgpl-2.1.html), and its source can be
> obtained as described below.

### Corresponding source

The Media Pack is a statically linked pair; the LGPL requires the **exact corresponding
source** of the binaries and the freedom to modify/relink them. Distribution of the Media Pack
must be accompanied by — or make a **written offer, valid for at least three years** (LGPL 2.1
§6(c)), to supply — the following corresponding source package:

- the unmodified FFmpeg 7.1 source tarball
  (`https://ffmpeg.org/releases/ffmpeg-7.1.tar.xz`,
  SHA-256 `40973d44970dbc83ef302b0609f2e74982be2d85916dd2ee7472d30678a7abe6`);
- the build scripts and configuration used to produce the binaries, available on request from
  the distributor (the exact configure flags are also recorded in the artifact's
  `build/configure.txt` / `build/configure-flags.txt`);
- the modifications — there are **none**; the corresponding diff is empty (only configure
  flags differ from upstream);
- the tools/data needed to reproduce the binaries (FFmpeg's own build system, the pinned
  source, and the documented toolchain).

The artifact records the source URL, tag, SHA-256, signature status, and toolchain in
`build/source-info.json`. Requests for the corresponding source should be made to the
distributor of the Media Pack; the source is also publicly available at the FFmpeg release URL
above.

Nothing in Smaller, Please's distribution may restrict modifying, running, or relinking the
LGPL Media Pack, and the Media Pack must remain replaceable by the user.

### Independent JPEG Group notice

FFmpeg includes `libavcodec/jfdctfst.c`, `libavcodec/jfdctint_template.c`, and
`libavcodec/jrevdct.c`, which are based in part on the work of the **Independent JPEG Group**.
The Media Pack therefore includes the statement:

> This software is based in part on the work of the Independent JPEG Group.

No changes were made to those files.

## System dependencies (not redistributed)

The Media Pack links only against macOS system libraries and frameworks:

- `/usr/lib/libSystem.B.dylib`
- `/usr/lib/libz.1.dylib` (zlib)
- `VideoToolbox`, `CoreMedia`, `CoreVideo`, `CoreFoundation`, `CoreServices` frameworks

These are part of macOS and are **not** redistributed by Smaller, Please. VideoToolbox is
Apple's H.264 implementation; using it introduces no GPL component, but framework availability
is not proof of H.264 patent rights.

## Patents

Copyright compliance does not settle H.264/AAC patent-pool licensing. The FFmpeg project's own
legal page warns that commercial use of patented standards can attract licensing fees. Review
the H.264/AAC patent position with qualified counsel before monetization. Smaller, Please does
not grant any patent license.
