# CHANGELOG

## v1.3.0

### Breaking Changes

- **All variable names renamed from `snake_case` to `camelCase`.** See
  [UPGRADING.md](UPGRADING.md) for the full migration guide and variable
  mapping.

### Added

- `enableLoudnorm` library variable for optional EBU R128 loudnorm audio
  normalization in the audio transcoding flow
- `singleNodeMode` global variable with `checkFlowVariable` bypass nodes for
  worker type tags across controller, video transcoding, and cleanup flows
- `enableNotifications` library variable gate in notification flow
- `fileSizeLowerBound` and `fileSizeUpperBound` library variables for
  configurable file size checks in video transcoding flow
- Checkpoint and hardlink documentation sections in README
- Loudnorm documentation section in README
- Getting Started guide with requirements, required plugins, and quick start
- [UPGRADING.md](UPGRADING.md) migration guide for breaking changes

### Fixed

- Standardized checkpoint variable to `useCheckpoints` (was inconsistently
  `use_checkpoints` in one location)
- Fixed extra `{` in CPU faster preset variable template in video transcoding
  flow
- Replaced hardcoded NAS path with `remuxOutputPath` global variable in cleanup
  flow
- Fixed `url_plex` typo in README global variables table

## v1.2.0 - March 4, 2025

### Added

- Emojis to subtitles flow
- Comments indicating which plugins to bypass for single-node setups
- Better error handling. Multiple 'reset flow errors' have been added so
  progress can continue even if some steps fail.
- File size check for movies/kids movies. Tdarr will now hold for review if a
  movie is already under 4.25gb.

### Removed

- Disabled 'Add local subtitles' plugin from the subtitles cleaning flow. It
  errors out no matter what, so removing it for now.

## v1.0.1 - March 2, 2025

### Added

- `useCheckpoints` variable for overwriting source files after each flow is
  complete
- `checkHardlinks` variable to filter out any files that may be used other
  places that we don't want to touch
- Created the changelog file you're reading right now
- Updated README to include info about new variables

### Changed

- Updated flow titles to include emoji
- Added indicator emojis to controller, notification, and cleanup flow plugins
- Moved cleanup logic to its own flow
-

### Fixed

- Minor grid layout cleanup
