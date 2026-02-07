# Modular Tdarr Flows

The idea is that the entire system is 'modular.' That meaning that I can tweak a
handful of library variables to best suit the media I want to transcode. For
example, some pieces of media I might only want to clean the subtitles on and
that's it, and others I might want to go ham on and run a full transcode. A few
notes:

- I separate my libraries out per TV show. Some shows I don't want to transcode
  because I have great downloads for, and others I want a different quality
  level. i.e RuPaul's Drag Race is going to get a 24 ql vs Arcane getting a 19
- The end goal is to have my entire library be x265. I don't care what Trash
  says- 1080p h265 looks fine.
- I have 15 users on my plex server all streaming from a range of different
  devices, so compatibility is key. For example, each audio transcode gets a 2ch
  AAC version.
- The server is ran on my Mac Studio via docker. I have a node on the Mac Studio
  and a node on my PC. Since my Mac Studio is also my plex server, the only
  thing I have that node do is move/save files.
- NVENC logic isn't fully integrated yet
- PC has a RTX 4080 Ti and a AMD Ryzen 5 5600X in it
- This is all trial and error and I'm sure I'm doing things wrong.
- I still need to route general animation through this (kids shows, Midnight
  Gospel, etc)
- I like really detailed logs, so I try to add as many comments as possible to
  my flows to help get a more clear picture of what's happening. todo: requires
  more emojis.
- Tuning for anime content is:

`-tune animation -bf 3 -maxrate 2000k -bufsize 4400k -x265-params "frame-threads=6:deblock=-2:-2:psy-rd=1:psy-rdoq=0.5:aq-mode=1:aq-strength=0.80:ipratio=1.4:pbratio=1.3:qpmax=69:qpmin=10:no-cu-lossless=1:no-amp=1:no-rect=1:rskip=1:ctu-info=0:limit-refs=1:no-aq-motion=1:scenecut=40:scenecut-bias=0.1:no-sao=1:strong-intra-smoothing=0:no-sao=1"`

- Tuning for non-anime content is:

`-x265-params "frame-threads=6:deblock=-2:-2:psy-rd=1.1:psy-rdoq=1:aq-mode=1:aq-strength=0.80:ipratio=1.4:pbratio=1.3:qpmax=69:qpmin=10:no-cu-lossless=1:no-amp=1:no-rect=1:rskip=1:ctu-info=0:limit-refs=1:no-aq-motion=1:no-dynamic-refine=1:scenecut=40:scenecut-bias=0.1"`

## Getting Started

### Requirements

- Tdarr v2.00.20+ (with flow support and community plugins)
- Docker (recommended) or native installation
- For GPU encoding: NVIDIA GPU with NVENC support (optional)

### Required Community Plugins

The flows use the following third-party community plugins (not included in a
standard Tdarr install). Install these from the Tdarr community plugin list:

- `Tdarr_Plugin_a9he_New_file_size_check` - File size validation
- `Tdarr_Plugin_a9hf_New_file_duration_check` - Duration validation
- `Tdarr_Plugin_henk_Keep_Native_Lang_Plus_Eng` - Language-based audio
  retention (requires Sonarr/Radarr/TMDB API keys)
- `Tdarr_Plugin_x7ac_Remove_Closed_Captions` - Closed caption removal
- `Tdarr_Plugin_MC93_Migz5ConvertAudio` - Audio codec conversion
- `Tdarr_Plugin_MP01_MichPasCleanSubsAndAudioCodecs` - Subtitle codec cleaning

### Quick Start

1. **Import the flows** into Tdarr via the Flows page. Import all 7 JSON files:
   - `TDARR_0_controllerFlow.json` - Main orchestrator
   - `TDARR_1_subtitleCleaningFlow.json` - Subtitle processing
   - `TDARR_2_audioCleaningFlow.json` - Audio filtering
   - `TDARR_3_audioTranscodingFlow.json` - Audio codec conversion
   - `TDARR_4_videoTranscodingFlow.json` - Video transcoding
   - `TDARR_5_cleanupFlow.json` - File replacement
   - `TDARR_6_notificationFlow.json` - Notifications

2. **Set global variables** in `Tools > Global Variables`:
   - **Required:** `url_plex`, `plex_token`, at least one set of Plex
     library paths/keys
   - **Required if using Arr:** `url_radarr`/`url_sonarr` and corresponding
     API keys
   - **Required:** `single_node_mode` - set to `true` for single-node setups,
     `false` for multi-node
   - **Optional:** `remux_output_path` - only needed if `is_remux` is used

3. **Set library variables** for each Tdarr library. At minimum:
   - `name` - Library identifier (MOVIES, TV, ANIME, etc.)
   - `enable_audio_cleaning` - true/false
   - `enable_subs_cleaning` - true/false
   - `enable_audio_transcoding` - true/false
   - `enable_video_transcoding` - true/false
   - `enable_notifications` - true/false
   - `quality_level` - CRF value (18-25, lower = higher quality)
   - `ffmpeg_preset` - slower, slow, medium, fast, or faster
   - `use_nvenc` - true/false (requires NVIDIA GPU)
   - `file_size_lower_bound` - Lower bound for file size check (default: 8)
   - `file_size_upper_bound` - Upper bound for file size check (default: 200)

4. **Assign the controller flow** to your Tdarr library as the processing flow.

5. **Test with a single file** before running batch processing.

## Variables

### Global Variables

`Tools > Global Variables`

I use global variables as a defacto dotenv and store any sensitive or easily
forgettable info here. I have a few groupings: Plex library IDs, my Plex token,
volume mapping paths, Arr URLs, and Arr Tokens. The Tdarr paths vs plex paths
are because Tdarr uses bind mounts via docker and Plex uses system paths.

You can find instructions on how to get your Plex library IDs
[here](https://support.plex.tv/articles/201638786-plex-media-server-url-commands/).

| Variable               | Key Example                          | Category             | Arg                                                      |
| ---------------------- | ------------------------------------ | -------------------- | -------------------------------------------------------- |
| plex_path_movies       | /Volumes/Drive_01/data/media/movies/ | Plex Path            | `{{{args.userVariables.global.plex_path_movies}}}`       |
| plex_path_tv           | /Volumes/Drive_02/data/media/tv/     | Plex Path            | `{{{args.userVariables.global.plex_path_tv}}}`           |
| plex_path_anime        | /Volumes/Drive_03/data/media/anime/  | Plex Path            | `{{{args.userVariables.global.plex_path_anime}}}`        |
| plex_libraryKey_movies | 1                                    | Plex Library Key     | `{{{args.userVariables.global.plex_libraryKey_movies}}}` |
| plex_libraryKey_tv     | 2                                    | Plex Library Key     | `{{{args.userVariables.global.plex_libraryKey_tv}}}`     |
| plex_libraryKey_anime  | 3                                    | Plex Library Key     | `{{{args.userVariables.global.plex_libraryKey_anime}}}`  |
| tdarr_path_movies      | /mnt/01/media/movies/                | TDARR Volume Mapping | `{{{args.userVariables.global.tdarr_path_movies}}}`      |
| tdarr_path_tv          | /mnt/02/media/movies/                | TDARR Volume Mapping | `{{{args.userVariables.global.tdarr_path_tv}}}`          |
| tdarr_path_anime       | /mnt/03/media/movies/                | TDARR Volume Mapping | `{{{args.userVariables.global.tdarr_path_anime}}}`       |
| plex_token             | xxxxxxxxxxxxx                        | Plex Token           | `{{{args.userVariables.global.plex_token}}}`             |
| url_plex               | 192.168.1.xx                         | Plex IP              | `{{{args.userVariables.global.url_plex}}}`               |
| url_radarr             | http://192.168.1.xx:8989             | Arr URL              | `{{{args.userVariables.global.url_radarr}}}`             |
| url_sonarr             | http://192.168.1.xx:8989             | Arr URL              | `{{{args.userVariables.global.url_sonarr}}}`             |
| url_sonarrAnime        | http://192.168.1.xx:8989             | Arr URL              | `{{{args.userVariables.global.url_sonarrAnime}}}`        |
| api_key_radarr         | xxxxxxxxxxxxx                        | API Key              | `{{{args.userVariables.global.api_key_radarr}}}`         |
| api_key_sonarr         | xxxxxxxxxxxxx                        | API Key              | `{{{args.userVariables.global.api_key_sonarr}}}`         |
| api_key_sonarrAnime    | xxxxxxxxxxxxx                        | API Key              | `{{{args.userVariables.global.api_key_sonarrAnime}}}`    |
| api_key_tmdb           | xxxxxxxxxxxxx                        | API Key              | `{{{args.userVariables.global.api_key_tmdb}}}`           |
| remux_output_path      | /mnt/syno_a/media/remuxing/output    | Remux Output Path    | `{{{args.userVariables.global.remux_output_path}}}`      |
| single_node_mode       | true                                 | Node Config          | `{{{args.userVariables.global.single_node_mode}}}`       |

### Library Variables

| Variable                 | Key Example | Used By Flow                   | Arg                                                         | Note                                                                              |
| ------------------------ | ----------- | ------------------------------ | ----------------------------------------------------------- | --------------------------------------------------------------------------------- |
| name                     | MOVIES      | All                            | `{{{args.userVariables.library.name}}}`                     | Must be all uppercase                                                             |
| enable_audio_cleaning    | true        | Controller & Audio Cleaning    | `{{{args.userVariables.library.enable_audio_cleaning}}}`    | lowercase, true/false                                                             |
| enable_subs_cleaning     | true        | Controller & Subs Cleaning     | `{{{args.userVariables.library.enable_subs_cleaning}}}`     | lowercase, true/false                                                             |
| enable_audio_transcoding | true        | Controller & Audio Transcoding | `{{{args.userVariables.library.enable_audio_transcoding}}}` | lowercase, true/false                                                             |
| enable_video_transcoding | true        | Controller & Video Transcoding | `{{{args.userVariables.library.enable_video_transcoding}}}` | lowercase, true/false                                                             |
| enable_notifications     | true        | Controller & Notification      | `{{{args.userVariables.library.enable_notifications}}}`     | lowercase, true/false                                                             |
| enable_control_flow      | true        | All                            | `{{{args.userVariables.library.enable_control_flow}}}`      | Prevents returning to the controller                                              |
| quality_level            | 21          | Video Transcoding              | `{{{args.userVariables.library.quality_level}}}`            | 18 to 25 recommended. lower = higher quality                                      |
| use_nvenc                | false       | Video Transcoding              | `{{{args.userVariables.library.use_nvenc}}}`                | lowercase, true/false                                                             |
| ffmpeg_preset            | slower      | Video Transcoding              | `{{{args.userVariables.library.ffmpeg_preset}}}`            | lowercase. Options: slower, slow, medium, fast, faster. NVENC cannot use 'slower' |
| use_foreign              | false       | Subtitle                       | `{{{args.userVariables.library.use_foreign}}}`              | Optional                                                                          |
| useCheckpoints           | false       | Controller                     | `{{{args.userVariables.library.useCheckpoints}}}`           | Optional. Should we overwrite the source file after each flow?                    |
| check_hardlinks          | true        | Controller                     | `{{{args.userVariables.library.check_hardlinks}}}`          | Optional. Filter check to see if video is hard linked anywhere.                   |
| file_size_lower_bound    | 8           | Video Transcoding              | `{{{args.userVariables.library.file_size_lower_bound}}}`    | Optional. Lower bound for video file size check (percentage).                     |
| file_size_upper_bound    | 200         | Video Transcoding              | `{{{args.userVariables.library.file_size_upper_bound}}}`    | Optional. Upper bound for video file size check (percentage).                     |

## Audio

I have several plex users that have older TVs, so each video should have a base
aac 2ch regardless of quality. Additional qualities might be added, depending on
the source audio. A few example scenarios:

- Source: 8ch FLAC | Outcome: 8ch AC3, no 6ch, 2ch AAC
- Source: 6ch AAC | Outcome: 6ch AAC, 2ch AAC
- Source: 2ch AC3 | Outcome: 2ch AAC
- Source: 2ch AAC | Outcome: 2ch AAC

The only allowed audio formats are AAC and AC3, all other formats will be
discarded. The following are discarded:
dca,dts,flac,mp2,mp3,truehd,vorbis,wav,wma,eac3. Commentary is not removed.

If {{{args.userVariables.library.is_remux}}} is set to `true`, we will only
transcode a single AAC audio channel at the highest channel count. Commentary is
not removed.

## Customizing for Your Personal Setup

### Single-Node vs Multi-Node

The flows support both single-node and multi-node setups via the
`single_node_mode` global variable.

- **Single-node setup:** Set `single_node_mode` to `true` in your global
  variables. All worker type tags will be automatically bypassed, allowing any
  worker to handle any task.
- **Multi-node setup:** Set `single_node_mode` to `false` (or leave it unset).
  Worker type tags will be enforced, routing tasks to the appropriate node based
  on tags like `pc-only`, `media-server-only`, etc. Make sure your Tdarr nodes
  have matching tags configured.

The following worker type tags are affected:

| Flow | Plugin | Node Tag |
|------|--------|----------|
| Controller | Tags: Worker Type (MEDIA SERVER) | media-server-only |
| Controller | Tags: Worker Type (PC) | pc-only |
| Video Transcoding | Tags: Worker Type GPU | pc-only (GPU:nvenc) |
| Video Transcoding | Tags: Worker Type CPU | pc-only (CPU) |
| Video Transcoding | Tags: Worker Type ANY | CPUorGPU |
| Cleanup | Tags: Worker Type (ANY) | any |

### Subtitle Flow

- No adjustments needed for single or multi-node setups.

### Audio Cleaning Flow

- No adjustments needed for single or multi-node setups.

### Audio Transcoding Flow

- No adjustments needed for single or multi-node setups.

### Notification Flow

- No adjustments needed for single or multi-node setups.

## ToDo

- Add support for commentary filtering (low priority)
- ~~Add optional library variables for FileSizeLowerBoundsCheck and
  FileSizeUpperBoundsCheck~~ ✅ Done!
- Add additional emoji indicators unique to each flow state (high priority)
- Add a way to specify the target audio format (e.g., AC3, AAC) (low priority)
  or something like KeepAc3
- Add variable to set the maximum number of audio channels to keep (low
  priority)
- Add optional variable to enable Loudnorm processing (medium priority)
- ~~Add 'checkpoint' file overwrites. If enabled, the controller flow will
  overwrite the original file after each module. For example, if audio
  transcoding is successful but video transcoding fails, the audio transcoding
  will have already overwritten the original file. When we retry the flow, it
  will bypass all the work needed for audio transcoding and go directly to video
  transcoding, saving time/resources.~~ ✅ Done!
- ~~Wire up `enable_notifications` library variable~~ ✅ Done!
- ~~Add `single_node_mode` global variable to replace manual tag bypassing~~ ✅
  Done!
- Simplify notification flow by adding `arr_type` library variable (radarr,
  sonarr, sonarr_anime, none) to reduce per-library branching (medium priority)
- Change variable names to camelCase (low priority)
- Replace images in /docs with updated screenshots (high priority)
- Add additional documentation for checkpoint and hardlink variables/logic
- Add setup validation flow to verify configuration (medium priority)
- Add example docker-compose configuration (low priority)
