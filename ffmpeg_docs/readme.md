# Resources on how to configure various encoders

## General Info
ffmpeg codecs documentation:

https://ffmpeg.org/ffmpeg-codecs.html

## Video

### nvenc

Nvidia Encoder/Decoder Capability Matrix

https://developer.nvidia.com/video-encode-and-decode-gpu-support-matrix-new

#### av1_nvenc

Requires Nvidia RTX 40-series (Ada Lovelace) or newer.

https://docs.nvidia.com/video-technologies/video-codec-sdk/12.0/ffmpeg-with-nvidia-gpu/index.html

Run `ffmpeg -h encoder=av1_nvenc` for full options.

Key differences from hevc_nvenc:
- B-frames (`-bf`) are **not supported** — omit entirely
- `-cq` range: 1-51 (same as hevc_nvenc)
- `-rc vbr -rc-lookahead 32` still supported
- `-spatial-aq 1 -aq-strength 8` still supported
- `-preset p1-p7` still supported (p7 = slowest/best quality)

#### hevc_nvenc (reference, no longer used)

https://docs.nvidia.com/video-technologies/video-codec-sdk/11.1/ffmpeg-with-nvidia-gpu/index.html

https://goughlui.com/2023/12/29/video-codec-round-up-2023-part-9-hevc_nvenc-h-265-nvidia-nvenc/

### qsv

Intel Quick Sync Encoder/Decoder Compatibility Matrix

https://en.wikipedia.org/wiki/Intel_Quick_Sync_Video#Hardware_decoding_and_encoding

#### av1_qsv

Requires Intel Arc GPU or 12th gen+ Intel CPU iGPU.

https://trac.ffmpeg.org/wiki/Hardware/QuickSync

#### hevc_qsv (reference, no longer used)

https://goughlui.com/2023/12/28/video-codec-round-up-2023-part-7-hevc_qsv-h-265-intel-quick-sync-video/

https://nelsonslog.wordpress.com/2022/08/22/ffmpeg-and-hevc_qsv-intel-quick-sync-settings/

### amf

AMD Encoder/Decoder Compatibility Matrix

https://en.wikipedia.org/wiki/Video_Core_Next

https://en.wikipedia.org/wiki/Video_Coding_Engine

https://github.com/GPUOpen-LibrariesAndSDKs/AMF

#### av1_amf

Requires AMD RDNA3+ GPU.

#### hevc_amf (reference, no longer used)

https://goughlui.com/2023/12/31/video-codec-round-up-2023-part-11-hevc_amf-h-265-amd-advanced-media-framework/

### libsvtav1

CPU AV1 encoder (SVT-AV1). Fastest software AV1 encoder.

https://gitlab.com/AOMediaCodec/SVT-AV1

Key parameters:
- `-preset 0-13` — 0 = slowest/best quality, 13 = fastest/lowest quality. Default: 10. Use 6 for a medium quality balance.
- `-crf 1-63` — lower = better quality. ~28-35 is roughly equivalent to x265 CRF 18-22.
- `-svtav1-params key=val:key=val` — additional SVT-AV1 parameters

#### libx265 (reference, no longer used)

https://goughlui.com/2023/12/26/video-codec-round-up-2023-part-3-libx265-mpeg-h-part-2-h-265-hevc/

### vaapi
#### av1_vaapi

https://trac.ffmpeg.org/wiki/Hardware/VAAPI

## Audio

### opus
