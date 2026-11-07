> `Changelog:`
> - All significant changes to this project will be documented in this file.
---

> [10.0.0]
>
> - Added `tune_surfaceflinger` — pins SurfaceFlinger to maximum CPU/IO priority (renice -20, SCHED_FIFO 99, ionice realtime), OOM-kill protection, and the `top-app` cgroup; also sets SurfaceFlinger properties for context priority, phase offsets, and HWC-forced composition.
> - Added `tune_network` — TCP tuning (reordering tolerance, low latency mode, fast open, no slow-start after idle, reduced notsent buffer), lower `txqueuelen` across network interfaces, WiFi forced low-latency mode, disabled scan throttling, and disabled WiFi power saving.
> - Added `tune_vm` — kernel memory management tuning (`vfs_cache_pressure`, `dirty_ratio`, `dirty_background_ratio`) with a dynamic `min_free_kbytes` scaled to 1% of total RAM (minimum 64MB); skipped entirely on devices with less than 4GB RAM.
> - Added `fix_full_screen_intent` — scans all installed apps and re-enables `USE_FULL_SCREEN_INTENT` for any app where it was silently rejected, fixing missed call/VoIP notifications (found affecting WhatsApp and the default Dialer app).
> - Added `speed_up_animations` — sets window, transition, and animator duration scale to 0.5x for a snappier UI.
> - Added `whitelist_games` — adds every installed game from `GAME_LIST` to the doze whitelist so background connections and sync aren't interrupted by battery restrictions.
> - Added `boost_saturation` — increases display color saturation to 1.3x via SurfaceFlinger for more vivid visuals.
> - Added `compile_system_ui` — AOT-compiles `com.android.systemui` and the `android` framework package in `speed --full` mode for a more responsive system UI.
> - Added `tune_sdcard_readahead` — dynamically detects any mounted external SD card and raises its block device read-ahead to 2MB for smoother large file transfers.
> - Added `boost_graphics_services` — raises CPU/IO priority and `top-app` cgroup placement for the core graphics pipeline (GPU service, SurfaceFlinger, display composer, display allocator) and the active keyboard app, for smoother rendering without introducing input lag.
> - Added support for Aster (`me.yuki.aster`), FolkSU (`me.yuki.folksu`), and FinaSU (`me.yuki.fina`), all APatch forks, and BakaSU, a KernelSU fork sharing its package ID with ReSukiSU and distinguished automatically, to `detect_root_all` in `customize.sh`.
> - Changed `game_mode` — added `cmd power set-fixed-performance-mode-enabled` to request maximum performance through the Power HAL on devices that support it; also added `POWER_SAVE_PRE_HIDE_MODE` and `touch_pressure_sensitivity` to the universal high performance settings.
> - Changed `lock_refresh_rate` — added dynamic detection of any additional `refresh_rate` settings key exposed via `dumpsys content`, catching OEM-specific keys not already covered by the hardcoded list.
> - Changed `preload_apps` — fixed `job_count` not persisting across subshell boundary in `while read`; replaced with tmpfile-based counter in `/data/local/tmp`; `mktemp` path also fixed for `compile_dex`.
> - Changed `preload_apps` — now targets only the top 20 most frequently opened apps (ranked via `usagestats`), falling back to every installed app if usage history is unavailable, instead of preloading every installed app unconditionally; also now preloads each app's compiled `.odex`/`.vdex` files alongside its APK and native libraries, so apps already AOT-compiled by `compile_dex` launch straight from RAM.
> - Changed `game_boost_daemon` — added 12-hour idle shutdown; timer resets on every game/idle transition so the daemon only stops after 12 hours with no activity, not from a fixed runtime since boot.
> - Changed `device_info` in `customize.sh` — added `device_capitalized` to local declaration; refactored RAM label logic into a reusable `ram_label()` function; added SDK format validation before minimum version check; sub-comments renumbered 3a–3f.
> - Changed `verify_module_id` in `verify.sh` — added `module_author` and `original_author` check alongside existing id and name verification.
> - Changed `detect_root_all` in `customize.sh` — simplified `KSUD_OUTPUT` spoofed case from 5 conditions to 3, removing separate VortexSU/ReSukiSU labels in favor of a generic "KernelSU Spoofed" fallback.
> - Changed `print_random_devil_message` in `customize.sh` — replaced Lucius with Baal in the list of devil names shown during installation.
> - Fixed `zram_optimize` — swapoff/mkswap/swapon used `/dev/$dev_name` which doesn't exist on this device; corrected to `/dev/block/$dev_name` with fallback to `/dev/$dev_name` for devices with a different layout.
> - Fixed `memory_daemon` — `cmd activity compact background` is not a valid subcommand and always failed silently; replaced with `cmd activity compact system`. LOW threshold trim now skips the foreground app (`send-trim-memory COMPLETE` throws for foreground processes). Sub-comments renumbered 2a–2i.
> - Fixed `wm_logging_off` — `cmd window logging stop` is deprecated on newer Android versions and always failed; replaced with `cmd protolog_configuration logcat disable`. Removed dead `dumpsys window | grep Proto/Logcat` loop that matched nothing on tested devices.
> - Fixed `storage_trim` — `fstrim` was attempted on read-only partitions (`/`, `/vendor`) and always failed with `FITRIM: Inappropriate ioctl for device`; added an `rw` mount check before trimming each partition.
> - Fixed `configure_renderer` — forcing `debug.hwui.renderer`/`debug.renderengine.backend` to Skia Vulkan (`skiavk`) caused severe video playback corruption (horizontal artifacting) in apps like Facebook; now explicitly forces Skia OpenGL (`skiagl`) instead, confirmed stable across all tested content.
> - Fixed `apply_driver_settings` — ANGLE step was gated behind `if settings get angle_gl_driver_selection_pkgs`, which is always true (returns `null` with exit 0 for unset keys, not a failure); replaced with a meaningful `-n "$packages"` check.
> - Fixed `preload_apps` — native library preload never worked because it looked up `lib/$abi` (e.g. `lib/arm64-v8a`), but Android's actual APK lib folder naming is shortened (e.g. `lib/arm64`); added an abi-to-folder-name mapping before the lookup.
> - Fixed `game_mode` — `inject-temperature` used the wrong argument order and passed the sensor name as the type, which always failed with "Invalid temperature type"; corrected to `TYPE STATUS NAME VALUE` with type mapped from each sensor's actual `mType` (CPU/GPU/BATTERY/SKIN).
> - Fixed `disable_tracing` — `cmd activity clear-watch-heap` was called with no argument and always failed; now passes an empty string. Removed `cmd companiondevice clear-association-memory-cache`, which is not a recognized command on tested Android versions.
> - Fixed `compile_dex` — compilation never worked at all: `run_compile` was a shell function defined in the parent process and called from an `xargs ... sh -c` subshell, which cannot see it in POSIX sh (`export -f` is unsupported), always failing with "inaccessible or not found". Rebuilt without the function/template indirection — each `sh -c` subshell now builds and runs its own compile command directly.
> - Fixed `system_cleanup` — `inject-temperature` had the same wrong argument order bug as `game_mode`; applied the same fix (type mapped from `mType`, correct `TYPE STATUS NAME VALUE` order).
> - Fixed `main` — `tune_activity_manager` was called before `system_cleanup`, but `system_cleanup` deletes every `*_constants` setting (including `activity_manager_constants` and `app_ops_constants`) without restoring them, silently wiping out all of `tune_activity_manager`'s settings on every run. Swapped the call order so `system_cleanup` runs first.
> - Fixed `system_cleanup` — UID parsing from `pm list packages -U` used `cut -f3 -d:`, which only worked by coincidence on package names without extra colons; replaced with `grep -oE 'uid:[0-9]+'` for reliable extraction.
> - Fixed `gms_optimize` — the `grep "gms"` package filter had no exclusion, so it matched and force-stopped, killed, and hibernated `com.google.android.gms` itself (core Google Play Services), risking broken push notifications, sign-in, and Play Store. Added an explicit exclude for `com.google.android.gms` and `com.google.android.gsf`, matching the whitelist already used in `restrict_gapps`.
> - Fixed `uninstall.sh` — synced the ZRAM revert and `fstrim` logic with the fixes applied to `optimize`: ZRAM path now resolves `/dev/block/$dev_name` with fallback, and `fstrim` no longer targets read-only partitions (`/`, `/vendor`, `/odm`, `/system_ext`, `/product`), replaced with the same `rw`-checked partition list plus SD card detection.
> - Removed `cmd otadexopt cleanup` from `system_cleanup` — the `otadexopt` service no longer exists on tested Android versions.
> - Removed VortexSU, Kokoro Mask, and FolkLite from `detect_root_all` in `customize.sh`.
---

> [9.0.0]
>
> - Added revert logic to `uninstall.sh` — resets `settings`, `appops`, `device_config`, standby bucket, game mode, ZRAM, and GMS components changed by `optimize`.
> - Added daemon cleanup to `uninstall.sh` — kills `memory_daemon` and `game_boost_daemon` background processes on uninstall.
> - Changed `optimize` — `GAME_LIST` refactored into a single global variable instead of being redefined in each function.
> - Changed `zram_optimize` — now loops all `/sys/block/zram*` devices instead of only `zram0`.
> - Changed `memory_daemon` — MODERATE threshold no longer trims all third-party apps, reduced to `compact background` only.
> - Changed `configure_renderer` — added fallback Vulkan detection via `libvulkan` in `/vendor/lib*` and `/system/lib*`.
> - Changed `compile_dex` — replaced process substitution `<(...)` with `mktemp` in `/data/local/tmp` for POSIX sh compatibility; replaced unbounded `&` loop with `xargs -P 4` for controlled parallelism; added storage check — skips if `/data` has less than 1GB free.
> - Changed `game_boost_daemon` — `is_game_foreground` now uses `dumpsys window | grep mCurrentFocus` instead of `dumpsys activity top` for lighter foreground detection.
> - Changed `storage_trim` — added `/`, `/vendor`, `/mnt/vendor/persist`, and SD card detection via `/mnt/media_rw/*`.
> - Changed `detect_root_all` in `customize.sh` — refactored from if-elif chain to loop-based detection; added aapt-based spoofed build fallback; changed detection order to APatch → KernelSU → Magisk.
> - Changed `service.sh` — root detection order updated to APatch → KernelSU → Magisk; APatch now reads 5 fields from `.method` including `APATCH_APP_VER` and `APATCH_KERNEL_VER_CODE`.
> - Changed `apply_driver_settings` — added Game Driver availability check via `/vendor/lib*/egl` path before applying to any package.
> - Changed `configure_game_mode` — added SDK level check, returns early if device is below Android 12 (SDK 31).
> - Fixed `system_cleanup` — removed `find -empty -delete` that could delete user storage folders unintentionally.
> - Removed `optimize_sqlite` — removed entirely from `optimize` and `uninstall.sh`.
> - Removed `zip=$(clean_path "$zip")` from `extract()` in `verify.sh`.
---

> [8.5.0]
>
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
> - Changed license from GNU General Public License to Apache License 2.0.
> - Changed module banner — replaced old banner with a new design.
> - Changed `README.md` with simplified description and updated feature list.
> - Changed `verify.sh` — added `#!/system/bin/sh`, `clean_path()`, numbered comments, sub-comments, and improved `abort_verify` message.
> - Changed `uninstall.sh` — added `sm fstrim`, `fstrim` per partition, `sync`, and removed echo output.
> - Changed `service.sh` — root detection now reads from `.method` file, removed Android version and emoji.
> - Changed `customize.sh` — removed `SKIPUNZIP=1`, `DEBUG=false`, `MODPATH`/`TMPDIR` fallback, and old simple root detection.
> - Changed `optimize` — renamed from `optimize.sh`, restructured into 21 modular functions called from a single `main()`.
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
---

> [7.0.0]
>
> - Added module banner for KernelSU Next.
> - Added system notification on successful optimization completion.
> - Added `disable_thermal` — disables throttling, trip points, cooling devices, and thermal services.
> - Added `gpu_optimize` — governor, max/min frequency, bus governor, and throttling control.
> - Added `touch_optimize` — sensitivity, boost level, polling tweaks, and 240Hz report rate.
> - Added `entropy_optimize` — rng selection, jitterentropy support, and kernel entropy parameters.
> - Added `kernel_optimize` — disables unnecessary logging, panic triggers, and improves OOM handling.
> - Added `sleep 5` delays between modules for stability during execution.
> - Changed `README.md` structure and content for a more professional impression.
> - Changed `customize.sh` with improved `arch` detection and minor code changes.
> - Changed `uninstall.sh` with minor removal code improvements.
> - Changed `tune_io` with adaptive scheduler selection, affinity, readahead, and mount options.
> - Changed `setup_zram` with dual device support, adaptive compression, and RAM-based sizing.
> - Changed `tune_memory` with new sysctl parameters and better lowmemorykiller settings.
> - Changed `ui_optimize` with more SurfaceFlinger, HWUI, EGL, and rendering properties.
> - Changed `game_opt` with stricter CPU/GPU tuning and kernel scheduler tweaks.
> - Changed `service.sh` — tidied up code and moved `optimize_sqlite` into it for better optimization flow.
> - Changed general code — refactoring and cleanup for stability and maintainability.
---

> [6.0.0]
>
> - Added availability checks for `sqlite3`, `setprop`, and other commands.
> - Changed memory optimization with `page-cluster`, `dirty_writeback_centisecs`, and KSM control.
> - Changed `optimize_sqlite` with `[ -w ... ]` fail-safe checks before applying all sysctl values.
> - Changed game tuning with `sched_child_runs_first` and `sched_idle_min_granularity_ns`.
> - Changed GPU max frequency detection to support more device paths.
> - Changed `optimize_art` — moved from `main()` to `service.sh` for faster execution on boot.
> - Changed ZRAM logic — separated from `tune_io` into its own `setup_zram` function.
> - Changed general code for safety and efficiency.
> - Fixed minor code issues in `uninstall.sh`.
> - Fixed `chmod 660` being forced when permissions were already correct in `optimize_sqlite`.
---

> [5.0.0]
>
> - Added lockfile system to prevent duplicate execution.
> - Added UI optimization via SurfaceFlinger system properties.
> - Added gaming mode with CPU/GPU frequency scaling and kernel scheduler tuning.
> - Added ART optimization via `cmd package compile`.
> - Added automatic fstrim on core partitions.
> - Changed shell from `bash` to `sh` for broader compatibility.
> - Changed SQLite optimization with size filtering — skip files larger than 100MB.
> - Changed I/O tuning with scheduler, read-ahead, noatime, and ZRAM setup.
> - Changed RAM-based memory tuning with drop caches, swappiness, and lowmemorykiller.
> - Removed logging and remount logic for lighter runtime optimization.
> - Removed legacy `.rc` file optimization and background logging.
---

> [4.0.0]
>
> - Added new optimization modules and functions.
> - Changed documentation to reflect new features.
> - Changed validation and error checking.
> - Changed file and directory structure for easier maintenance.
> - Changed code — refactored for performance and stability improvements.
> - Fixed a bug causing errors when inserting data.
> - Removed unused and irrelevant code.
---

> [3.5.0]
>
> - Changed overall module structure — major overhaul covering all aspects of the module.
---

> [1.0.0]
>
> - Initial release.
---