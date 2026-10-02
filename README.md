
# XtremeBS (Xtreme Battery Saver)

**Maximize your Android device’s battery life with highly configurable power-saving tools.**

**XtremeBS** is a Magisk/KernelSU module designed for rooted Android devices, offering aggressive battery optimization through dynamic, event-driven settings. It allows advanced users to fine-tune CPU cores, apps, WiFi, Doze mode, and more to extend battery life significantly—potentially up to 5x stock uptime. While powerful, it requires careful configuration to avoid lag, missed notifications, or device instability.

---

## Features

- **App Management**: Kill, suspend, or reprioritize apps with allowlists and denylists for user and system apps.
- **CPU Optimization**: Set CPU cores to powersave mode or disable high-power cores automatically or manually.
- **System Tweaks**: Force Doze mode (light/deep), disable WiFi, enable low RAM mode, or manage Google Mobile Services (GMS) and process priorities.
- **Event-Driven Control (v2)**: Apply settings based on triggers like `boot`, `charging`, `screen_off`, `low_power`, or custom events.
- **User-Friendly Tools**: Control via `XBSctl` commands, monitor with logs and status files, and configure via a web UI (v1.0.6+).
- **Safety Features**: Safe mode to recover from misconfigurations and sanity checks to prevent system crashes.

---

## Supported Root Managers

- **Magisk** (Confirmed)
- **KernelSU** (Confirmed)
- **APatch** (Reports show incompatiblity; Unknown still)

---

## Disclaimer

XtremeBS is an advanced tool that modifies system behavior and requires root access. Misconfiguration may cause lag, missed notifications, alarms, or SystemUI crashes. **Use at your own risk**. I is not responsible for damages or data loss. Always back up your config and test settings incrementally.

Tested primarily on a **Pixel 5** running **ProtonAOSP**. Compatibility varies by device and ROM.

---

## Installation

1. **Download the Module**:
   - Grab the latest release from [GitHub Releases](https://github.com/DethByte64/Xtreme-Battery-Saver/releases/latest).
   - Alternatively, install via [MMRL](https://mmrl.dev/) for easy updates:  
     [![MMRL](https://mmrl.dev/assets/badge.svg)](https://mmrl.dev/repository/zguectZGR/Xtreme-Battery-Saver)

2. **Install**:
   - Flash the module in your root manager (Magisk/KernelSU).
   - Reboot your device.

3. **Configure**:
   - A default config file is created at `/data/local/tmp/XtremeBS/XtremeBS.conf`.
   - Edit the config manually or use the web UI (http://127.0.0.1:8081, launched via `action.sh` in v1.0.6+).
   - After changes, reload the config with `XBSctl reload` or reboot.

---

## Configuration

XtremeBS uses a configuration file (`/data/local/tmp/XtremeBS/XtremeBS.conf`) to control its behavior. It supports two formats:

- **v1 (Legacy)**: Simple `key=value` pairs. Suitable for basic setups but less flexible.
- **v2 (Recommended)**: Event-driven blocks (e.g., `screen_off={...}`) for dynamic control based on device states or custom triggers.

> [!TIP]
> Set `version=2` in the config to enable v2 mode. The module automatically migrates v1 configs to v2 if detected.

### v2 Configuration (Recommended)

v2 uses **event blocks** to apply settings for specific triggers:
- **Hardcoded Events**: `boot`, `charging`, `screen_off`, `low_power`, `manual` (triggered by device states or `XBSctl`).
- **Custom Events**: User-defined (e.g., `my_event`), triggered manually via `XBSctl start my_event`.

Each block contains settings like `disable_cores` or `handle_apps`. Example:

```bash
version=2
delay=3
log_file=/sdcard/XtremeBS.log
log_level=3

screen_off={
  disable_cores=cpu6 cpu7
  handle_apps=nice
  allowlist=/data/local/tmp/XtremeBS/apps.allow
}

low_power={
  disable_cores=cpu2 cpu3 cpu4 cpu5
  doze=light
  kill_wifi=true
}

my_event={
  handle_gms=nice
  low_ram=true
}
```

**Rules**:
- Each block starts with `event_name={` and ends with `}` on separate lines.
- Use alphanumeric characters, underscores, or dashes for custom event names (no spaces, `$`, `=`, `{`, or `}`).
- Empty blocks (e.g., `boot={
  }`) do nothing.
- Multiple events can stack (e.g., `screen_off` and `low_power` disabling different cores) these work in a Last on, First off method, plan accordingly.

### v1 Configuration (Legacy)

v1 uses a single `trigger` to apply settings globally. Example:

```bash
version=1
trigger=auto
delay=3
keep_on_charge=true
handle_cores=auto
disable_cores=false
handle_apps=suspend
allowlist=/data/local/tmp/XtremeBS/apps.allow
```

> [!NOTE]
> v1 is backward compatible but will be deprecated in future releases. Consider switching to v2 for advanced features.

### Config Options

| Option | Description | Values | Default | Notes |
|--------|-------------|--------|---------|-------|
| `version` | Config format | `1`, `2` | `2` | Set to `2` for event-driven mode. |
| `trigger` (v1 only) | When to apply settings | `auto` (Battery Saver), `boot`, `manual` | `auto` | Ignored in v2. |
| `delay` | Polling interval (seconds) | Integer | `3` | Higher values reduce CPU usage, but may takr longer to detect events and commands. Lower values may use more CPU cycles, but provide faster detection. |
| `keep_on_charge` | Keep settings active while charging | `true`, `false` | `true` | Only useful with `trigger=auto` (v1) or `low_power` (v2). |
| `handle_apps` | Manage app behavior | `false`, `kill`, `nice`, `suspend` | `false` | `suspend` requires a valid allowlist. |
| `allowlist` | File with allowed app packages | Path (e.g., `/data/local/tmp/XtremeBS/apps.allow`) | `/data/local/tmp/XtremeBS/apps.allow` | Create manually; list one package per line (e.g., `com.termux`). |
| `denylist` | File with system apps to manage | Path (e.g., `/data/local/tmp/XtremeBS/apps.deny`) | `/data/local/tmp/XtremeBS/apps.deny` | Optional; for system apps. |
| `handle_cores` | Set CPU governors to powersave | `false`, `auto`, Space-separated cores (e.g., `cpu4 cpu5`) | `false` | `auto` targets low-power cores. |
| `disable_cores` | Disable CPU cores | `false`, `auto`, Space-separated cores (e.g., `cpu6 cpu7`) | `false` | `auto` disables high-power cores; avoid on Samsung devices. |
| `handle_gms` | Manage Google Mobile Services | `false`, `nice`, `kill` | `false` | `kill` breaks Google apps and SafetyNet/Play Integrity. |
| `handle_proc` | Reprioritize system processes | `true`, `false` | `false` | Use with `proc_file`; may delay messages/alarms. |
| `proc_file` | File with processes to reprioritize | Path (e.g., `/data/local/tmp/XtremeBS/proc.list`) | `/data/local/tmp/XtremeBS/proc.list` | Format: `process_name nice_level` (e.g., `netd 19`). |
| `low_ram` | Enable low RAM mode | `true`, `false` | `false` | Avoid on OnePlus devices; may cause random reboots. |
| `doze` | Force Doze mode | `false`, `light`, `deep` | `false` | May break alarms; test carefully. |
| `kill_wifi` | Disable WiFi | `true`, `false` | `false` | Saves power but disables WiFi toggle in Settings. |
| `notify` | Show notifications | `true`, `false` | `true` | Disable to not use notifications. |
| `log_file` | Log file path | Path (e.g., `/sdcard/XtremeBS.log`) | `/sdcard/XtremeBS.log` | Set `log_level` for verbosity. |
| `log_level` | Logging verbosity | `1` (INFO), `2` (VERBOSE), `3` (DEBUG) | `2` | Higher levels aid debugging. |

**Allowlist Example** (`apps.allow`):
```bash
com.termux
com.google.android.inputmethod.latin
com.topjohnwu.magisk
```

**Process File Example** (`proc.list`):
```bash
netd 19
system_server 10
```

> [!CAUTION]
> Always include essential apps (e.g., keyboard, terminal) in `apps.allow` when using `handle_apps=suspend`. Without a valid allowlist, apps may become unusable, requiring `XBSctl safe` via ADB.

---

## XBSctl Commands

Control XtremeBS with the `XBSctl` command-line tool (run as root via `su`):

| Command | Description | Usage |
|---------|-------------|-------|
| `start` | Start XtremeBS (v1) or an event (v2) | `XBSctl start` (v1/manual) or `XBSctl start my_event` (v2) |
| `stop` | Stop XtremeBS (v1) or an event (v2) | `XBSctl stop` (v1/manual) or `XBSctl stop my_event` (v2) |
| `reload` | Reload the config | `XBSctl reload` |
| `pause` | Pause trigger handling | `XBSctl pause` |
| `resume` | Resume trigger handling or exit safe mode | `XBSctl resume` |
| `safe` | Enter safe mode (stops XtremeBS, unsuspends apps) | `XBSctl safe` |

> [!TIP]
> Use `XBSctl safe` via ADB (`adb shell XBSctl safe`) if the device becomes unresponsive due to misconfiguration.

---

## Usage Tips

1. **Start Slow**:
   - Enable one option at a time (e.g., `handle_apps=nice`) and test for 24 hours to ensure stability.
   - Avoid aggressive settings like `disable_cores` initially, especially on Samsung or OnePlus devices.

2. **Debugging**:
   - Check `/data/local/tmp/XtremeBS/XtremeBS.status` for CPU, WiFi, and Doze states.
   - Set `log_level=3` and review `/sdcard/XtremeBS.log` for detailed logs if issues occur.
   - Run `su -c ps -eo "%cpu pid cmd" | sort -n -k1,1` to identify high-CPU processes for `proc_file`.

3. **Device-Specific Notes**:
   - **Samsung Devices**: Avoid `disable_cores` and `handle_cores` to prevent reboots or SystemUI crashes.
   - **OnePlus Devices**: Disable `low_ram` to avoid random reboots.

4. **Security**:
   - Restrict config file permissions: `chmod 600 /data/local/tmp/XtremeBS/*`.
   - Avoid sharing configs, as they may include sensitive app data or cause instability on different devices.

---

## FAQ

**Q: My device soft-loops or SystemUI crashes. What do I do?**  
**A**: You likely enabled too many aggressive options. Enter safe mode with `adb shell XBSctl safe`, disable risky settings (e.g., `disable_cores`, `low_ram`), and test incrementally. Check logs for clues.

**Q: Will XtremeBS brick my device?**  
**A**: No, it won’t cause hard bootloops. However, misconfigurations can cause lag, missed alarms, or crashes. Always back up your device.

**Q: Is XtremeBS plug-and-play?**  
**A**: No, it requires manual configuration. Start with the default config and adjust based on your device’s needs. An app is in development to simplify this.

**Q: How effective is XtremeBS?**  
**A**: With proper tuning, it can extend battery life significantly. Effectiveness depends on your config and device.

**Q: Why does my config not take effect?**  
**A**: Ensure you run `XBSctl reload` or reboot after changes. Verify event names (v2) are valid and check logs for errors. Use the web UI to avoid syntax issues.

**Q: Can I use XtremeBS with other battery-saving modules?**  
**A**: Yes, but conflicts may occur (e.g., with L Speed, Naptime). Disable overlapping features in other modules and test thoroughly.

---

## Troubleshooting

- **Unresponsive Device**: Boot into recovery, edit `/data/local/tmp/XtremeBS/XtremeBS.conf` to set `safemode=1`, reboot, and fix the config.
- **No Battery Improvement**: Verify active events (`XBSctl status` in future releases or check logs). Try `handle_apps=suspend` with a robust allowlist.
- **Missed Alarms/Notifications**: Disable `doze` or `handle_proc`, as they may delay background tasks.
- **Report Issues**: Open a [GitHub Issue](https://github.com/DethByte64/Xtreme-Battery-Saver/issues) with your config, device details, and logs.

---

## Contributing

We welcome contributions! To contribute:

1. Fork the repository: [DethByte64/Xtreme-Battery-Saver](https://github.com/DethByte64/Xtreme-Battery-Saver).

2. Submit pull requests with bug fixes, features, or documentation improvements.

3. Report bugs or suggest features via [GitHub Issues](https://github.com/DethByte64/Xtreme-Battery-Saver/issues).

---

## Acknowledgments

- Thanks to the Magisk and KernelSU communities for root support.

- [DerGoogler](https://github.com/DerGoogler) for MMRL and their work in MMAR

- Special thanks to NanKillBro for KernelSU testing.

- Gratitude to XDA and Reddit users for feedback and testing.

---

## Star History

<a href="https://star-history.dera.page/DethByte64/Xtreme-Battery-Saver?compare=Magisk-Modules-Alt-Repo/Xtreme-Battery-Saver">
 <picture>
   <source media="(prefers-color-scheme: dark)" srcset="https://star-history.dera.page/svg?repos=DethByte64/Xtreme-Battery-Saver,Magisk-Modules-Alt-Repo/Xtreme-Battery-Saver&theme=dark" />
   <source media="(prefers-color-scheme: light)" srcset="https://star-history.dera.page/svg?repos=DethByte64/Xtreme-Battery-Saver,Magisk-Modules-Alt-Repo/Xtreme-Battery-Saver" />
   <img alt="Star History Chart" src="https://star-history.dera.page/svg?repos=DethByte64/Xtreme-Battery-Saver,Magisk-Modules-Alt-Repo/Xtreme-Battery-Saver" />
 </picture>
</a>

---

## License

XtremeBS is released under the [GPLv3 License](LICENSE.md).
