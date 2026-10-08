# Flashing from Windows 10/11

Total time about 20 minutes, mostly downloads. You need: a USB **data** cable (many cables are charge-only),
the board, and internet for the first build.

## 1. Install the tools (once)
1. **Git**: https://git-scm.com/download/win (defaults are fine).
2. **Python 3.10+**: https://www.python.org/downloads/ — tick **"Add python.exe to PATH"** on the first screen.
3. Open **PowerShell** (Start menu) and install PlatformIO:
   ```powershell
   python -m pip install --user platformio
   python -m platformio --version
   ```
   If `pio` is not found afterwards, use `python -m platformio` wherever this guide says `pio`.
   (Alternative: VS Code + the "PlatformIO IDE" extension; open the project folder and use the checkmark/arrow buttons.)

## 2. Get the code
```powershell
git clone https://github.com/vitorhpsousa/DPFSentinel-CYD35.git
cd DPFSentinel-CYD35
copy include\secrets.example.h include\secrets.h
notepad include\secrets.h
```
While the repo is private, Git for Windows opens a browser window to sign in to GitHub the first time you clone.
Optional: put your WiFi name and password in `secrets.h` (2.4 GHz only). Leave empty for a first test — logging to SD
works without WiFi, and the `DPF-Sentinel` hotspot is always available.

## 3. Connect the board and find its port
Plug the board in and run `pio device list`; look for a `COMx` entry (Device Manager > Ports shows the same). The board
uses the ESP32-S3's native USB and Windows 10/11 installs the driver itself. No port: try another data-capable cable.

## 4. Back up the factory firmware (once, before the first flash)
```powershell
python -m pip install --user esptool
python -m esptool --chip esp32s3 --port COM5 read_flash 0 0x1000000 cyd_factory_backup.bin
```
(Replace COM5 with your port.) Keep the file; restoring is `write_flash 0 cyd_factory_backup.bin`.

## 5. Build and flash
Insert a FAT32 microSD card first if you want to see logging work immediately.
```powershell
pio run -t upload        # CYD screen board
```
If PlatformIO picks the wrong port add `--upload-port COM5`.
**If upload says "Failed to connect":** hold **BOOT**, tap **RESET** (or plug in while holding BOOT), release BOOT, then run
the upload again. Press RESET afterwards to start the program.

## 6. Check it works
```powershell
pio device monitor -b 115200
```
Expected lines: `Boot: reset reason`, then `Logging to SD as session_N.csv`. If it says `flash` instead of `SD`, the card
is missing or not FAT32. Without the car you will see
`Connecting to BLE ELM327 adapter...` repeating — that is normal. Close the serial monitor (Ctrl+C) before re-flashing.
Note: opening the serial port resets the board and starts a new session file.

## 7. Next steps
- Plug the adapter into the car with the ignition on, power the board, and watch the monitor for `ELM327 adapter ready`.
- Set the clock: join the `DPF-Sentinel` WiFi on your phone, open http://192.168.4.1.
- Read the card on the PC: files are in the `obd` folder.
- Developers: `pio test -e native` needs a compiler on PATH (MinGW-w64/MSYS2); skip it if you only flash.

## Troubleshooting
| Symptom | Fix |
|---|---|
| `pio` / `python` not recognised | reopen PowerShell; reinstall Python with "Add to PATH" |
| Build downloads fail | check internet/proxy; rerun, PlatformIO resumes |
| No COM port | data cable, other USB port, driver (CH340/CP210x) |
| Failed to connect | BOOT + RESET sequence above |
| Antivirus blocks esptool | allow `esptool` / python in your AV |
| Garbled monitor output | baud must be 115200 |
