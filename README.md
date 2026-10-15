# 🌙 GyroRest

Orientation-sensor-based periodic inactivity alerts for iPhone and Android phones.

GyroRest watches your phone's orientation sensor. When the phone stays still for a while, it plays a soft chime, vibrates (where supported) and shows a notification. If the phone moves, the timers reset.

**Live demo:** https://gyrorest.netlify.app

## Features

- Detects "phone at rest" using the `deviceorientation` API (no external libraries)
- Four alerts per stillness period: at 10 s, 80 s, 160 s and 240 s
- Synthesised chime via Web Audio API, so no audio files are needed
- Vibration and notifications where the browser supports them
- Pauses monitoring when the page is hidden
- Battery level display (where the Battery API exists)
- Single file, no build step, no dependencies

## Usage

1. Open the page on your phone.
2. Tap **Start Monitoring** and accept the terms.
3. Allow motion-sensor access (required on iOS) and notifications.
4. Put the phone down, **keeping the page open and the screen on**.

To run locally:

```bash
git clone https://github.com/mykytakolenko-alt/GyroRest.git
cd GyroRest
npx serve .
```

Motion sensors require HTTPS (or `localhost`), so open it over HTTPS when testing on a phone, for example via the Netlify demo.

## Configuration

The timings and sensitivity are constants at the top of the `<script>` in `index.html`:

| Constant | Default | Meaning |
|---|---|---|
| `ALERT_TIMES_SEC` | `[10, 80, 160, 240]` | Seconds of stillness before each alert |
| `STILL_THRESHOLD_DEG` | `3` | Rotation (degrees) that counts as movement |

## Limitations

This is a web page, not a native app:

- It only works **while the page is open and visible**. Browsers suspend sensors and timers in the background or when the screen locks.
- iOS requires motion permission, granted by a tap. Web notifications on iOS only work if the page is added to the Home Screen (iOS 16.4+).
- Some Android browsers do not allow `new Notification()` from a page. Sound and vibration still work in that case.
- Sensor support and accuracy vary by device.

## Privacy

Sensor data is processed locally in your browser. Nothing is sent to or stored on any server.

## Disclaimer

Provided "as is", without warranty. The app may encourage habitual phone checking, so use it responsibly. The author is not liable for battery consumption or missed notifications.

## License

Licensed under the [Apache License 2.0](LICENSE). Copyright © 2026 Mykyta Kolenko.
