# ESP32 FLASH LAB

A professional, static browser flasher and serial debugging workbench for ESP32-family boards. It uses the [Web Serial API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Serial_API) and [esptool-js](https://github.com/espressif/esptool-js) to communicate directly with the board from Chrome or Edge.

> Firmware files are processed locally in your browser and are not uploaded to our server. Serial data is not stored remotely.

## Features

- Real ESP bootloader connection and chip detection
- ESP32, ESP32-S2, ESP32-S3, ESP32-C3, ESP32-C6, and ESP32-H2 support through esptool-js
- Local `.bin` file selection and drag-and-drop
- Configurable flash address, defaulting to `0x0`
- Real flash progress from the esptool-js write callback
- Complete flash erase with confirmation
- Hard reset after flashing and a separate reset control
- Live serial monitor with baud-rate selection, pause/resume, timestamps, send, clear, and copy
- Optional profiles for Generic ESP32, ESP32-S3, and XiaoZhi ESP32-S3
- Three-column developer-tool dashboard with activity log, device details, settings, help, light/dark/system-ready theming, and responsive mobile stacking
- No backend, database, upload endpoint, API key, or paid service

## Download the project

Either download this repository as a ZIP from GitHub (**Code → Download ZIP**) or clone it:

```bash
git clone YOUR_GITHUB_REPOSITORY_URL
cd esp32-firmware-flasher
```

The project intentionally contains only static files:

```text
/
├── index.html
├── style.css
├── app.js
├── README.md
└── assets/
```

## Create a GitHub repository and publish

1. Sign in to GitHub and select **New repository**.
2. Choose a repository name, such as `esp32-firmware-flasher`.
3. Create the repository. A README is optional because this project already has one.
4. From the project directory, run:

```bash
git init
git add .
git commit -m "Add ESP32 Web Flasher"
git branch -M main
git remote add origin YOUR_GITHUB_REPOSITORY_URL
git push -u origin main
```

5. In GitHub, open **Settings → Pages**.
6. Under **Build and deployment**, choose **Deploy from a branch**.
7. Select branch **main** and folder **/ (root)**, then press **Save**.
8. Wait for the deployment check to finish and open the published URL.

GitHub Pages serves this project over HTTPS, which is required by Web Serial except for localhost development.

## Use the flasher

### Supported browser

Use the latest desktop version of **Google Chrome** or **Microsoft Edge**. Web Serial does not work in Safari or Firefox. The page detects unsupported browsers and displays a warning.

### Put an ESP32 board in bootloader mode

For many ESP32-S3 boards with **BOOT** and **RESET** buttons:

1. Hold **BOOT**.
2. Press and release **RESET**.
3. Release **BOOT**.
4. Select the board's USB serial port in Chrome or Edge.

The exact procedure varies by board. Some boards enter the ROM bootloader automatically when the browser opens the port; others require the manual sequence above.

### Connect

1. Connect the board with a data-capable USB cable.
2. Click **Connect device**.
3. Select the USB serial port in the browser dialog.
4. The interface will attempt to connect to the ROM bootloader and show the chip, revision, MAC address, and flash size when available.

### Select a BIN

Choose a local `.bin` file or drag it into the firmware area. The file is read into browser memory only. Select a profile if useful, then confirm the flash address:

- `0x0` is the default requested by this tool for complete images.
- Common factory/bootloader layouts may use `0x1000`, `0x8000`, or `0x10000` depending on the image.
- Use the address supplied by the firmware vendor. This page does not guess or rewrite a binary.

### Erase flash

Click **Erase flash** and confirm the warning. This calls the ESP bootloader's full `eraseFlash()` operation. It permanently removes the current contents of the device flash.

### Flash firmware

Click **Flash firmware**. The tool calls esptool-js to write the selected local image, reports the actual write callback progress, and sends a hard reset after a successful write. A successful result is shown in both the progress area and serial console.

### Serial monitor

After connecting, the serial monitor can read live output at the selected baud rate. The default is `115200`. Available rates are `9600`, `19200`, `38400`, `57600`, `115200`, `230400`, `460800`, and `921600`.

- **Pause / Resume** controls rendering of incoming output.
- **TIMESTAMPS** adds local timestamps to displayed lines.
- Enter text and press **Send** or Enter to transmit it with CRLF.
- **Clear** removes the visible console.
- **Copy output** copies the visible console to the clipboard.
- **Reset device** sends a hard reset through esptool-js when the bootloader session is active.

## Windows notes

- Install the USB-to-serial driver required by your board, commonly CP210x, CH340, or FTDI. Some native USB ESP32-S2/S3 boards do not need an extra driver.
- Use a USB data cable, not a charge-only cable.
- Close Arduino Serial Monitor, PlatformIO monitor, PuTTY, ESP-IDF monitor, and any vendor flasher before connecting; only one application can own a serial port at a time.
- If the port appears and disappears, try another USB port or cable and check **Device Manager → Ports (COM & LPT)**.
- Chrome and Edge must be opened from a normal desktop session; Web Serial is not available in most embedded webviews.

## Troubleshooting

| Symptom | What to try |
| --- | --- |
| Unsupported browser warning | Use current Chrome or Edge on Windows, macOS, or Linux. |
| No port appears | Use a data cable, install the board's driver, and reconnect the board. |
| Bootloader connection failed | Hold BOOT, tap RESET, release BOOT, then retry. |
| Port already in use | Close every other serial monitor/flasher and retry. |
| Flash write failure | Verify the image and flash address, then retry in bootloader mode. |
| Device disconnects during flashing | Try a shorter cable, direct USB port, or lower USB/flash settings on the board. |
| Serial output is unreadable | Match the firmware's baud rate, usually 115200. |
| XiaoZhi profile confusion | The profile is a UI label only; select the correct XiaoZhi `.bin` supplied for your board. No firmware URL is hard-coded. |
| Page loads but esptool-js is unavailable | Check internet access to `unpkg.com`, or vendor a pinned esptool-js bundle locally. |

## Security and privacy

This is a static site. The `.bin` file is selected with a browser file picker, converted to an in-memory `Uint8Array`, and passed directly to esptool-js. There is no server upload flow. The serial monitor is rendered in the current browser tab and is not sent to a database or analytics service.

The only runtime dependency is the pinned esptool-js browser bundle loaded from `https://unpkg.com/esptool-js@0.6.1/bundle.js`. If you need an offline deployment, download that bundle into `assets/`, update the script tag in `index.html`, and remove the CDN dependency.

## Local testing

Because the project is static, any static server works. For example:

```bash
python3 -m http.server 8080
```

Open `http://localhost:8080` in Chrome or Edge. Opening `index.html` with a `file://` URL is not recommended because browser security policies can block scripts or serial access.

## Important hardware safety note

Erasing and flashing can permanently remove firmware and configuration data. Confirm the board model, image type, flash address, and power connection before starting an operation. The project performs real hardware operations; its progress is not simulated.
