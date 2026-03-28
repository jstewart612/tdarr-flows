## Tdarr - One Flow - To Rule Them All

Goal to have One Flow (set of flows) to Rule all your Media!

I accomplish this by using Library Variables.  This allows us to change our quality and encoding settings in the library.  This is much easier than trying to edit the flow every time we want to encode differently.
I have a library for low quality, high quality, Animation, Movies.  Each has their own quality settings.  Then I just move the files I'm processing into the corresponding library folder and tdarr will process as needed.

- While I have ran thousands of files through this flow, please consider this beta! Do not trust it with your media library until you have ran a bunch of various files through it and understand how it operates!  Let me know if you run into unexpected behavior!
- CPU & QSV work but need more testing, as I don't use them often.  NVENC is well tested.
- Uses the -vbr method to obtain a predictable bitrate.  With cq as a fallback method, or when we decide the bitrate is too low for -vbr to work well.

> **Fork note:** This is a fork of [samssausages/Tdarr-One-Flow](https://github.com/samssausages/Tdarr-One-Flow) with the following changes:
> - **AV1 encoding** — all encoders switched from HEVC (H.265) to AV1 (av1_nvenc / av1_qsv / av1_amf / av1_vaapi / libsvtav1)
> - **Dolby Vision support** — DV files are detected via fresh ffprobe and video-copied as-is to preserve DV metadata
> - Various bug fixes (x265-params flag order, VAAPI lookahead, AMF lookahead, variable namespace, stream reorder variable names, audio codec detection, NVENC hardware encoder detection)

# Features
- Uses Library Variables for Quality Settings. This way you can have different libraries for different quality settings
- Uses Centralized Flow Variables for configurables in one location (1 - Input) No need to hunt the entire flow for configurables
- We calculate things like -maxrate based on your target bitrate. Simplifying user input
- Lots of notes & documentation in the flow
- Extensive logging and use of icons to make tracking down failures a breeze
- Works with Nvidia (av1_nvenc), Intel QuickSync (av1_qsv), AMD (av1_amf), VAAPI (av1_vaapi), and CPU (libsvtav1)
- Dolby Vision files are detected and video-copied (not re-encoded) to preserve DV metadata
- Strip audio to where only the tracks you want remain
- If a lossless audio track exists, encode in opus (can disable, still needs refinement)
- Deinterlace .ts files (tv DVR broadcasts)
- Export Embedded Subtitles (Could use more testing and refinement)

I broke it down into 5 steps/flows:

1 - Input (Define Flow Variables & Configurables.  Tags files that may need special processing down the stack)

2 - Prep (Standardizes the File so it is less likely to fail encoding later)

3 - Audio (Clean audio and encode to Opus, if enabled)

4 - Video (Define desired bitrate by resolution, fall back on cq)

5 - Save (final checks and move operations)

# Installation
1. Create a new flow for each of the above steps (1-5) by:

    a. Go to Tdarr Flows

    b. Click "add flow"

    c. Scroll to bottom and copy/paste json into "Import JSON Template"

3. Create a new Library with the Variables listed below (Make Sure your library has an input folder defined & output folders exist)
4. Profit

# In-Place Replacement Setup

To transcode files and replace them at their original path:

- Set `output_dir_done` to the same root as your library (e.g. `/media/Movies`)
- Enable `keepRelativePath` in library output settings
- Set `test_mode = true` until you have validated the output quality

# Known Limitations
  - .ts files in 720p often end up with an unexpected bitrate.  Have not been able to figure out why yet.
  - AV1 hardware encoding (av1_nvenc) requires an Nvidia RTX 40-series (Ada Lovelace) or newer GPU. Older GPUs will fall back to CPU (libsvtav1).
  - av1_qsv requires Intel Arc or 12th gen+ Intel CPU with iGPU. av1_amf requires AMD RDNA3+.

# Tweaks
- All the configurable Flow Settings can be edited in flow 1 - Input
- You can disable audio processing with library variable do_audio = false

# Variable Notes:
Audio bitrates and cutoff are set PER CHANNEL.  We use that to calculate based on number of channels in the audio stream.

# Required Library Variables
```
test_mode true # true = will not delete source file.  False = will delete source file.  Without source files won't delete.

output_dir_done /media/4_done # path from within tdarr

output_dir_review /media/4_done_review # if something didn't go right, we move to review folder.

v_cq 20 # quality setting for cq fallback method.
         # av1_nvenc CQ scale: 1-51 (lower = better). 20 is high quality.
         # libsvtav1 CRF scale: 1-63 (lower = better). 28-35 is comparable to x265 CRF 20. 20 will be very high quality / large files.

bitrate_480p 1000k # target bitrate for given resolution (AV1 is ~30-40% more efficient than HEVC, so lower targets are appropriate)

bitrate_576p 1200k

bitrate_720p 1600k

bitrate_1080p 2000k

bitrate_1440p 3000k

bitrate_4k 8000k

bitrate_4k_hdr 10000k

do_audio_clean - true - Remove Commentary Audio, Remove Languages not listed in "audio_language", Keep only the Audio Track with Highest Channel Count

do_audio_encode - true - Encode 1st Audio track to Opus

bitrate_audio 160k # Output audio bitrate we will encode to.  This is PER CHANNEL.

bitrate_audio_cutoff 192k # will not encode if source audio bitrate is under this value.  This is PER CHANNEL

audio_language und,un,eng,en,ger,deu,de,zho,zh,chi,jpn,ja,kor,ko,spa,es,cpe,  # languages that you want to keep, if blank it's skipped

```

# Optional Library Variable Fields

```
If not defined, the default is "false" (disabled)

disable_vbr = true # Disable Primary VBR encoding Method

disable_cq = true # Disable Fallback encoding method

disable_video = true # Optional - Only needed if you want to disable video processing - Set to True

do_av1 = true # Re-encode files that are already AV1? Default: skip AV1 source files.

encoder = nvenc/qsv/amf/vaapi/cpu - Override Encoder Autodetect and manually set what encoder to use

```

# Quality Examples

Peoples opinions vary greatly on this.  Also content will have a big impact on what is optimal.  If you think you have good settings, share them!

**Note on AV1 quality settings:** AV1 achieves similar visual quality to H.265 at roughly 30-40% lower bitrate. If you were using H.265 bitrates, you can reduce them by about 30%.

For `v_cq` (CQ/CRF fallback):
- `av1_nvenc` uses CQ scale 1-51. 20 is high quality, 28 is medium-high.
- `libsvtav1` uses CRF scale 1-63. 28-35 is roughly equivalent to x265 CRF 18-22.

## Low Quality:

```
v_cq 30

bitrate_480p 800k
bitrate_576p 1000k
bitrate_720p 1400k
bitrate_1080p 1800k
bitrate_1440p 2800k
bitrate_4k 7000k
bitrate_4k_hdr 9000k

bitrate_audio 160k
bitrate_audio_cutoff 192k
```

## Mid-High Quality:

```
v_cq 24

bitrate_480p 1250k
bitrate_576p 1500k
bitrate_720p 2200k
bitrate_1080p 3000k
bitrate_1440p 4500k
bitrate_4k 12500k
bitrate_4k_hdr 15000k

bitrate_audio 256k
bitrate_audio_cutoff 384k
```

## FAQ

Q: It was working but now it isn't.  The Variables aren't working anymore, but I do have them configured.  Log has message:  "Variable of value does not match condition == "

A: Sometimes a Tdarr update can break the existing Variables that you configured in the Library.  When this happens you need to re-save them to update the database.  You could delete them and re-add them, but the easiest way I have found is to simply add a space, " ", after the variable, then click another variable and it will save it.  After adding spaces to everything, go back and click on the variables and delete the spaces.  This will re-save the variables to the database and should correct the issue.

Q:  Why is my video not encoding?  Why is it just copying the video?

A:  The most common reason for this is that your desired Output Bitrate is higher (or close to) your Input bitrate.  It wouldn't make sense to encode a 4000k video to 6000k.  When we encounter this we do go to the fallback cq encoding method, but this also checks to make sure a reasonable size savings can be obtained.  You can lower the quality by raising the cq value.
But keep in mind, it may not be worth it to encode such videos.  One-Flow probably won't encode if you only save 5% of space, the loss in quality wouldn't be worth a 5% space savings.  Keep it original, or lower the output bitrate.

Q:  Why are my AV1 videos not encoding?

A:  By default we skip AV1 source files, to force re-encoding add the library variable: do_av1 = true

Q:  My GPU doesn't support AV1 encoding.  What happens?

A:  The flow autodetects available hardware encoders. If av1_nvenc/av1_qsv/av1_amf/av1_vaapi are not available, it falls back to CPU encoding via libsvtav1.  You can also force a specific encoder with the `encoder` library variable.

Q:  What about Dolby Vision content?

A:  Dolby Vision files are detected in Flow 1 using ffprobe (checks for `dovi_configuration_record` in stream data).  In Flow 4, DV files take a copy path (`-c:v copy`) so the DV metadata is fully preserved.  Audio and other processing still runs normally.

## Release Notes

Changelog:

V0.95 (fork)

- **BREAKING:** Switched all video encoders from HEVC to AV1
  - av1_nvenc (Nvidia RTX 40-series+)
  - av1_qsv (Intel Arc / 12th gen iGPU+)
  - av1_amf (AMD RDNA3+)
  - av1_vaapi (VA-API capable hardware)
  - libsvtav1 (CPU fallback)
- **NEW:** Dolby Vision support — DV files detected via ffprobe and video-copied
- Library variable `do_hevc` renamed to `do_av1`
- Removed B-frames flag (not supported by av1_nvenc)
- Removed x265-params from CPU encoder nodes (not applicable to libsvtav1)
- CPU preset updated: `-preset medium` (x265) → `-preset 6` (libsvtav1 scale 0-13)
- Reduced suggested bitrate targets by ~30% to match AV1 efficiency
- Fixed: x265-params flag order bug (could cause "Unable to choose output format" error)
- Fixed: VAAPI lookahead flag (`-look_ahead 32` → `-async_depth 4`)
- Fixed: AMF lookahead flag (`-look_ahead 32` → `-look_ahead_depth 32`)
- Fixed: QSV lookahead flag removed (not supported)
- Fixed: NVENC hardware encoder detection (empty inputsDB)
- Fixed: Variable namespace for is_hdr, is_dv (missing user. prefix)
- Fixed: Stream reorder variable names (config_reorder_streams_* → fl_reorder_streams_*)
- Fixed: DV bypass in Flow 2 (skip MP4 remux and force-conform for DV files)
- Fixed: DV video copy path in Flow 4
- Fixed: Audio codec detection (removed wmav1/wmav2 from lossless list, added DTS profile check)

V0.94

Not too many front end changes this release, much is to prepare for future features.  Also squashed a few bugs.

 - Fixed bug that resulted in wrong bitrate during certain circumstances
 - Backend changes to prepare for future features
 - Added better notifications for when encoding is skipped due to incompatible HDR

V0.93

BREAKING CHANGES:

Split "do_audio" into:

do_audio_clean - true/false - Remove Commentary Audio, Remove Languages not listed in "audio_language", Keep only the Audio Track with Highest Channel Count

do_audio_encode - true/false - Encode 1st Audio track to Opus


ACTION TO TAKE IF UPGRADING FROM OLD VERSION:

Remove "do_audio" Library Variable and create "do_audio_clean" and "do_audio_encode" Library Variables


OTHER CHANGES:

- is_audio_lossless custom JS - Expanding to capture more lossless audio codecs through the custom JS, for more durable lossless codec identification. - Still refining this
- Added optional library variable: "disable_vbr" - true/false - to disable vbr encoding method and force cq encoding. (if you have disable_cq enabled as well, then video encoding is skipped)
- Added optional library variable: "encoder" - nvenc/qsv/amf/vaapi/cpu - Override Autodetect and manually set what encoder to use (currently only nvenc/qsv/cpu work)
- Moved "lookahead=32" from 4 - Video to 1 - Input - Flow Variable "fl_cpu_main"

BUG FIX:

- Added missing error handling to audio cleaning process
- Improved Audio encode error reporting
