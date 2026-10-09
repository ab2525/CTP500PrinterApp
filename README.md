# CTP500 Printer App

[![CI](https://github.com/thirtythreedown/CTP500PrinterApp/actions/workflows/ci.yml/badge.svg)](https://github.com/thirtythreedown/CTP500PrinterApp/actions/workflows/ci.yml)

A desktop app and command-line tool for the CTP500 Bluetooth thermal printer, the little receipt printer sold at Walmart. Print text, images and live "ticker tape" log output from Windows, Linux or macOS.

The app was originally written by Mel at ThirtyThreeDown Studio. Read about how the printer protocol was figured out in [PC app for the Walmart thermal printer](https://thirtythreedown.com/2025/11/02/pc-app-for-walmart-thermal-printer/).

## Features

- **GUI:** type or load text and choose a font, size (small, medium or large) and alignment (left, center or right). You can also load an image. Every print opens a preview showing exactly what will come out of the printer.
- **CLI:** print from scripts and pipes. `--follow` prints each line as it arrives (a ticker tape) and reconnects if the printer drops. `--no-feed` stops the printer wasting paper between lines.
- **Cross-platform:** works on Windows, Linux and macOS. macOS talks to the printer through Apple's IOBluetooth framework, because Python on macOS has no Bluetooth sockets.

## Download for Windows

You don't need Python. Get these from the [latest release](../../releases/latest):

- **`CTP500Printer.exe`:** the app. Double-click it to run.
- **`ctp500.exe`:** the command-line version, for scripts. It takes the same options as the CLI described under [Command line](#command-line).

The first time you run one, Windows may say "Windows protected your PC", because the exes aren't code-signed. Click **More info**, then **Run anyway**.

## Install from source

Use this on macOS or Linux, or on Windows if you'd rather run the Python script. You need Python 3.10 or newer, with Tkinter.

```sh
git clone https://github.com/thirtythreedown/CTP500PrinterApp.git
cd CTP500PrinterApp
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt        # Windows: .venv\Scripts\pip install -r requirements.txt
```

On macOS, `requirements.txt` also installs `pyobjc-framework-IOBluetooth`. With Homebrew's Python, get Tkinter with `brew install python-tk@3.13`, using your Python version.

## Set up your printer

1. Pair the printer in your system's Bluetooth settings.
2. Find its Bluetooth address:
   - **macOS:** `system_profiler SPBluetoothDataType`
   - **Linux:** `bluetoothctl devices`
   - **Windows:** in Device Manager, open your printer under **Bluetooth**, then go to **Properties** → **Details** and pick **Bluetooth device address**. Windows shows it without colons, so add one between each pair, like `12:34:56:78:9A:BC`.
   - **Anywhere:** the printer's phone app also shows it.
3. Type the address into the **Printer address** box next to **Connect**. The app remembers the last printer that connected, in both the GUI and the CLI, so you only do this once.
   - To set it ahead of time, put it in `mac_address` at the top of the script, or pass `--address` to the CLI.

## GUI

```sh
.venv/bin/python CTP500_GUI_app_Github_Export.py
```

Check the printer address and click **Connect**. Type some text or load a text file, then pick a font, size and alignment. **Print your text!** opens a preview; click **Print** there to send it. Images work the same way through the image tools.

## Command line

Running the script with any arguments uses the CLI instead of the GUI. It connects and disconnects on its own.

```sh
python CTP500_GUI_app_Github_Export.py "Hello world"
echo "build finished" | python CTP500_GUI_app_Github_Export.py -
python CTP500_GUI_app_Github_Export.py --image todo.png
tail -F /var/log/apache2/error.log | python CTP500_GUI_app_Github_Export.py --follow --no-feed --size small
```

| Option | What it does |
| --- | --- |
| `text...` | Text to print. Leave it out, or use `-`, to read stdin instead. |
| `-f`, `--follow` | Ticker tape: stays connected and prints each stdin line as it arrives. It connects on the first line, and if the printer drops it reconnects and retries that line once. |
| `-n`, `--no-feed` | Skips the paper feed after each printout, so the next one prints right below it. With `--follow`, the paper feeds once when it stops. |
| `-i`, `--image FILE` | Prints an image instead of text. |
| `--font NAME` | A font name from `--list-fonts`, or a path to a font file. |
| `--size small/medium/large` | Text size. The default is medium. |
| `--align left/center/right` | Text alignment. The default is left. |
| `--address ADDRESS` | The printer's Bluetooth address. The default is the last printer that connected, or `mac_address` if none has yet. |
| `--list-fonts` | Lists the fonts that can print text. |

Status messages go to stderr, so stdout stays clean for scripts. Exit codes:
- `0`: printed.
- `1`: printing failed, for example because the printer couldn't be reached.
- `2`: bad arguments.

If the program feeding the pipe is a Python script, print with `flush=True` or run it with `python -u`. Otherwise its lines arrive in bursts instead of one at a time.

## macOS notes

- **Bluetooth permission:** macOS asks for Bluetooth permission for the app that runs the script, usually your terminal. If something else launches it, such as `launchd`, `cron` or another app, that app needs Bluetooth permission in System Settings → Privacy & Security.
- **Braille service conflict:** macOS treats the CTP500 as a Braille display. When you connect the printer from the Bluetooth menu, the Braille service grabs its only serial channel. Let the app open the connection instead. If **Connect** fails, disconnect the printer in the Bluetooth menu and try again.

## Development

The tests use only the standard library. They don't need a printer: the Bluetooth connection is swapped for a fake one that records what would have been sent.

```sh
python -m unittest discover -s tests -v
```

The GUI tests skip themselves when there's no display. CI runs the tests on Linux, macOS and Windows with Python 3.10 and 3.13.

### Releases

To publish a release, push a version tag:

```sh
git tag v1.0.0
git push origin v1.0.0
```

The Release workflow then:
1. runs the tests;
2. builds both Windows exes with PyInstaller;
3. checks that they start;
4. publishes a GitHub release with the exes attached and notes generated from the merged pull requests.

Every pull request builds the exes too. You can download them from the workflow run's artifacts to try a build before releasing it.

## Credits

The original app is by Mel at [ThirtyThreeDown Studio](https://thirtythreedown.com/2025/11/02/pc-app-for-walmart-thermal-printer/). Thanks to Bitflip, Tsathoggualware, Reid and everyone whose research into the printer made it possible.
