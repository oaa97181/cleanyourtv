# Screensaver timeout pill on a TCL G08

A small fork of [cleanyourtv](https://github.com/Gathin23/cleanyourtv)'s launcher, tried on a TCL Smart TV Pro
(G08), Google TV 14.

## What it changes

- **A pill at the far left of the top bar**, before Inputs, showing the TV's screen-off timeout
  (`Settings.System.SCREEN_OFF_TIMEOUT`) as **10s** or **5 min**. Press it to flip between the two; the label
  updates at once and again whenever Home comes back. 10s is for seeing the screensaver without waiting out the
  default; 5 min is for everyday use. Any other value (the TV's own 10-minute default, say) shows as 5 min, and
  the first press sets 10s. D-pad Left from Inputs reaches it.
- **One new permission, `WRITE_SETTINGS`.** It is app-op gated, not signature-only, so it needs one grant over
  ADB after installing (below). Without it the pill shows the value but cannot change it.
- **`themes/glass/config.json`**: its `dock_apps` point at the apps installed on this TV. Put your own there.

No network permission, no background service. Everything else is upstream as of
[`2041430`](https://github.com/Gathin23/cleanyourtv/commit/2041430).

Code: `MainActivity.java` (`btnScreensaverTimeout`, `refreshScreensaverTimeoutLabel()`,
`toggleScreensaverTimeout()`), `activity_main.xml`, `ic_clock.xml`, `AndroidManifest.xml`. The strings are in
English only.

## Build and install

As upstream: JDK 17, Android SDK platform 34 and build-tools 34.0.0, and the tools venv
(`python3 -m venv tools/venv && tools/venv/bin/pip install -r tools/requirements.txt`). Then:

```bash
git clone -b tcl-g08-screensaver-timeout https://github.com/oaa97181/cleanyourtv.git
cd cleanyourtv/launcher
TV_HOST=192.168.1.50 ./build-and-install.sh
```

On Windows, `launcher/build-and-install.ps1` does the same. To use the glass theme:
`TV_HOST=192.168.1.50 tools/theme.sh themes/glass`.

## Grant the permission (once)

```bash
adb shell appops set com.gathin.tvhome WRITE_SETTINGS allow
```

In testing the grant survived a reinstall (`pm install -r`), but run it again after a rebuild if the pill stops
changing the value. To undo: `adb shell appops set com.gathin.tvhome WRITE_SETTINGS default`.

## Notes

- This sets the screen-off timeout, which is what starts the screensaver. It does not touch `sleep_timeout`
  (deeper standby). Making that one short can put the TV into standby before the screensaver shows, dropping
  Wi-Fi and ADB until the remote wakes it.
- Starting the screensaver on demand, as a button, needs `WRITE_DREAM_STATE`, a signature permission, and
  does not work from this launcher on the G08. The timeout pill is the workaround.
