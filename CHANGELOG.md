> `Changelog:`
> - All significant changes to this project will be documented here.
---

> [8.5.0]
>
> - License Changes.
> - Updated module banner.
> - Updated `README.md` with simplified description and updated feature list.
> - Updated `verify.sh` — added `#!/system/bin/sh`, `clean_path()`, numbered comments, sub-comments, and improved `abort_verify` message.
> - Updated `uninstall.sh` — added `sm fstrim`, `fstrim` per partition, `sync`, and removed echo output.
> - Improved `service.sh` — root detection now reads from `.method` file, removed Android version and emoji.
> - Improved `customize.sh` — removed `SKIPUNZIP=1`, `DEBUG=false`, `MODPATH`/`TMPDIR` fallback, and old simple root detection.
> - Removed `notif` — notification now handled inside `main()`.
> - Removed `tune_io` — replaced by dedicated I/O and scheduler functions.
> - Removed `setup_zram` — replaced by `zram_optimize` with simpler lz4-only approach.
> - Removed `tune_memory` — replaced by `memory_daemon` and `tune_activity_manager`.
> - Removed `run_fstrim` — replaced by `storage_trim` with blacklist-aware folder loop.
> - Removed `ui_optimize` — all setprop removed, function was empty.
> - Removed `game_opt` — replaced by `game_mode` and `game_boost_daemon`.
> - Removed `disable_thermal` — merged into `game_mode` and `disable_tracing`.
> - Removed `gpu_optimize` — replaced by `configure_renderer` and `apply_driver_settings`.
> - Removed `touch_optimize` (old) — replaced by simpler settings-based `touch_optimize`.
> - Removed `entropy_optimize` — removed due to limited real-world impact.
> - Removed `kernel_optimize` — partially merged into `disable_tracing`.
> - Removed `optimize_art` — removed entirely from both `optimize` and `service.sh`.
> - Rewrote `optimize` — renamed from `optimize.sh`, restructured into 21 modular functions called from a single `main()`.
> - Added `module_remove_check` to `customize.sh` — removes old module before installation.
> - Added full `detect_root_all` to `customize.sh` — replaces simple 3-line root detection with comprehensive multi-variant detection.
> - Added `zram_optimize` — dynamic ZRAM sizing based on RAM with lz4 compression.
> - Added `memory_daemon` — real-time background RAM monitor with NORMAL/MODERATE/LOW/CRITICAL thresholds.
> - Added `restrict_background` — system app background restrictions via `appops` with OEM-aware whitelist.
> - Added `restrict_gapps` — Google app background and standby bucket restrictions.
> - Added `wm_logging_off` — Window Manager logging and tracing disabled.
> - Added `disable_logcat` — silent all log.tag and persist.log.tag properties and clear log buffer.
> - Added `storage_trim` — fstrim across all root folders and via `sm fstrim`.
> - Added `touch_optimize` — reduced multi-press and long-press timeout for faster response.
> - Added `apply_driver_settings` — Game Driver and ANGLE applied to 271 game packages.
> - Added `lock_refresh_rate` — auto-detect and lock display to max refresh rate with OEM support.
> - Added `configure_renderer` — auto-select Skia Vulkan or Skia GL based on hardware.
> - Added `preload_apps` — preload all APK and native libraries to RAM via cat and dd.
> - Added `game_mode` — OEM-aware performance mode activation (MIUI, ColorOS, XOS, MagicOS).
> - Added `disable_tracing` — disable kernel tracing, atrace, service logs, and MIUI migard.
> - Added `compile_dex` — DEX compile with speed mode for games and speed-profile for others.
> - Added `configure_game_mode` — Android Game Mode with auto-detected downscale and loadingBoost.
> - Added `tune_activity_manager` — Activity Manager constants tuned for gaming profile.
> - Added `game_boost_daemon` — foreground game detection daemon with boost/idle mode switching.
> - Added `system_cleanup` — thermal inject, battery stats, connectivity, logging, and junk cleanup.
> - Added `gms_optimize` — aggressive GMS restriction with appops, doze, standby, and network policy.
> - Added `optimize_sqlite` — SQLite WAL, VACUUM, REINDEX, ANALYZE with toybox-compatible size check.
---

> [7.0.0]
>
> - Updated `README.md` structure and content for a more professional impression.
> - Updated `customize.sh` with improved `arch` detection and minor code changes.
> - Updated `uninstall.sh` with minor removal code improvements.
> - Improved `tune_io` with adaptive scheduler selection, affinity, readahead, and mount options.
> - Improved `setup_zram` with dual device support, adaptive compression, and RAM-based sizing.
> - Improved `tune_memory` with new sysctl parameters and better lowmemorykiller settings.
> - Improved `ui_optimize` with more SurfaceFlinger, HWUI, EGL, and rendering properties.
> - Improved `game_opt` with stricter CPU/GPU tuning and kernel scheduler tweaks.
> - Added module banner for KernelSU Next.
> - Added system notification on successful optimization completion.
> - Added `disable_thermal` — disables throttling, trip points, cooling devices, and thermal services.
> - Added `gpu_optimize` — governor, max/min frequency, bus governor, and throttling control.
> - Added `touch_optimize` — sensitivity, boost level, polling tweaks, and 240Hz report rate.
> - Added `entropy_optimize` — rng selection, jitterentropy support, and kernel entropy parameters.
> - Added `kernel_optimize` — disables unnecessary logging, panic triggers, and improves OOM handling.
> - Added `sleep 5` delays between modules for stability during execution.
> - Tidied up `service.sh` code and moved `optimize_sqlite` into it for better optimization flow.
> - General refactoring and code cleanup for stability and maintainability.
---

> [6.0.0]
>
> - Fixed minor code issues in `uninstall.sh`.
> - Fixed `chmod 660` being forced when permissions were already correct in `optimize_sqlite`.
> - Improved memory optimization with `page-cluster`, `dirty_writeback_centisecs`, and KSM control.
> - Improved `optimize_sqlite` with `[ -w ... ]` fail-safe checks before applying all sysctl values.
> - Improved game tuning with `sched_child_runs_first` and `sched_idle_min_granularity_ns`.
> - Improved GPU max frequency detection to support more device paths.
> - Removed direct `optimize_art` call from `main` — now managed by `service.sh`.
> - Added availability checks for `sqlite3`, `setprop`, and other commands.
> - Moved `optimize_art` into `service.sh` for faster execution on boot.
> - Separated ZRAM logic from `tune_io` into its own `setup_zram` function.
> - General code cleanup for safety and efficiency.
---

> [5.0.0]
>
> - Migrated from `bash` to `sh` for broader compatibility.
> - Improved SQLite optimization with size filtering — skip files larger than 100MB.
> - Improved I/O tuning with scheduler, read-ahead, noatime, and ZRAM setup.
> - Improved RAM-based memory tuning with drop caches, swappiness, and lowmemorykiller.
> - Removed logging and remount logic for lighter runtime optimization.
> - Removed legacy `.rc` file optimization and background logging.
> - Added lockfile system to prevent duplicate execution.
> - Added UI optimization via SurfaceFlinger system properties.
> - Added gaming mode with CPU/GPU frequency scaling and kernel scheduler tuning.
> - Added ART optimization via `cmd package compile`.
> - Added automatic fstrim on core partitions.
---

> [4.0.0]
>
> - Updated documentation to reflect new features.
> - Fixed a bug causing errors when inserting data.
> - Improved validation and error checking.
> - Removed unused and irrelevant code.
> - Added new optimization modules and functions.
> - Reorganized file and directory structure for easier maintenance.
> - Refactored code for performance and stability improvements.
---

> [3.5.0]
>
> - Version update.
---

> [1.0.0]
>
> - Initial release.
---