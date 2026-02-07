# Upgrading Guide

## v1.3.0 - Variable Name Changes

### Overview

All library and global variable names have been renamed from `snake_case` to
`camelCase` for consistency. **This is a breaking change** - you must update your
Tdarr global variables and library variables to match the new names.

### Required Changes

After importing the updated flows, update your variable names in the Tdarr UI.

#### Global Variables

| Old Name             | New Name           |
| -------------------- | ------------------ |
| `single_node_mode`   | `singleNodeMode`   |
| `remux_output_path`  | `remuxOutputPath`  |

All other global variables (`url_plex`, `plex_token`, API keys, paths, etc.)
are **unchanged** since they reference external service names.

#### Library Variables

| Old Name                 | New Name               |
| ------------------------ | ---------------------- |
| `enable_audio_cleaning`  | `enableAudioCleaning`  |
| `enable_subs_cleaning`   | `enableSubsCleaning`   |
| `enable_audio_transcoding` | `enableAudioTranscoding` |
| `enable_video_transcoding` | `enableVideoTranscoding` |
| `enable_notifications`   | `enableNotifications`  |
| `enable_control_flow`    | `enableControlFlow`    |
| `quality_level`          | `qualityLevel`         |
| `use_nvenc`              | `useNvenc`             |
| `ffmpeg_preset`          | `ffmpegPreset`         |
| `use_foreign`            | `useForeign`           |
| `is_remux`               | `isRemux`              |
| `is_animated`            | `isAnimated`           |
| `check_hardlinks`        | `checkHardlinks`       |
| `file_size_lower_bound`  | `fileSizeLowerBound`   |
| `file_size_upper_bound`  | `fileSizeUpperBound`   |

The following variables were **already camelCase** and are unchanged:
- `useCheckpoints`
- `name`

#### New Variables

| Variable          | Type    | Default | Flow              | Description                                   |
| ----------------- | ------- | ------- | ----------------- | --------------------------------------------- |
| `enableLoudnorm`  | Library | `false` | Audio Transcoding | Apply EBU R128 loudnorm audio normalization   |

### Step-by-Step Upgrade Process

1. **Export your current variable values** - Before upgrading, note down all your
   current global and library variable values.

2. **Import the updated flows** - Import all 7 updated JSON flow files into
   Tdarr. This will overwrite the existing flows.

3. **Update global variables** - Go to `Tools > Global Variables` and rename:
   - `single_node_mode` → `singleNodeMode`
   - `remux_output_path` → `remuxOutputPath`

4. **Update library variables** - For **each** Tdarr library, update the
   variable names using the table above. The values stay the same, only the
   names change.

5. **Test with a single file** - Run a test file through the flow to verify
   everything works before batch processing.

### Notes

- The old variable names will **not** work with the updated flows. If a variable
  is not found, Tdarr will treat it as empty/unset, which may cause unexpected
  behavior (e.g., features being skipped or wrong encoding paths being taken).
- If you have automation scripts that set Tdarr variables, update those scripts
  to use the new names as well.
