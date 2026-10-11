# ironrdp-egfx

Graphics Pipeline Extension ([MS-RDPEGFX]) implementation for IronRDP.

Provides PDU types and client/server processors for the Display Pipeline Virtual
Channel Extension, including H.264/AVC420 and AVC444 video streaming support.

## AVC444 V1 frame submission

The server sends a V1 main view as LC 1 and its auxiliary view as LC 2 inside
one `StartFrame`/`EndFrame` pair. Both views keep one frame ID and share
the existing acknowledgement limit. This avoids a combined LC 0 wrapper that
Windows App 11.4.3 rejects while preserving the encoder's picture sequence.
The serializer borrows each coded payload until it encodes the final PDU.
AVC444v2 keeps its existing combined wrapper.

Each received main rectangle has YUV420 color until an auxiliary rectangle
restores YUV444, as defined in [MS-RDPEGFX stream combination]. A caller that
updates the full main view must send the corresponding auxiliary update to
keep full chroma detail.

## OpenH264 Integration

This crate contains optional integration support for OpenH264 through the
[openh264](https://crates.io/crates/openh264) crate. When that integration
is enabled by downstream consumers, the applicable BSD license notices for
OpenH264 / openh264 must be preserved. See
[`THIRD_PARTY_NOTICES`](THIRD_PARTY_NOTICES) for the full license texts.

The OpenH264 integration is optional and disabled by default. Redistribution
scenarios using Cisco's prebuilt binary may also require compliance with
Cisco's separate binary-license conditions (documented in THIRD_PARTY_NOTICES).

### Feature Flags

- **`openh264-bundled`** -- Compiles OpenH264 from C source at build time
  (requires a C compiler and NASM). Source-compiled binaries do not carry
  H.264 patent coverage from Cisco's MPEG LA license. This is not the
  recommended path for redistribution.

- **`openh264-libloading`** -- Loads a prebuilt OpenH264 shared library at
  runtime via `dlopen`/`LoadLibrary`. When using Cisco's official prebuilt
  binaries (downloaded separately by the end user), those binaries carry
  patent coverage under Cisco's OpenH264 license. This is the recommended
  path for applications distributed via package managers.

### Choosing between bundled and libloading

Applications that distribute binaries to end users should use
`openh264-libloading` and arrange for the end user to download the Cisco
binary separately. Consumers are responsible for Cisco notice and EULA
placement as described in `THIRD_PARTY_NOTICES`. See the
[openh264 crate documentation](https://docs.rs/openh264) for details on
library discovery and hash verification.

Applications in controlled environments (CI, development, testing) can
use `openh264-bundled` for a simpler build with no external dependencies
beyond a C toolchain.

[MS-RDPEGFX]: https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-rdpegfx/da5c75f9-cd99-450c-98c4-014a496942b0

[MS-RDPEGFX stream combination]: https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-rdpegfx/8131c1bc-1af8-4907-a05a-f72f4581160f
