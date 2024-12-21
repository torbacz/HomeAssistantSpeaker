# Installation process
## OS flash
First download RaspberryPi Imager and flash RaspberryPi OS 64 bit version. Set WiFi and SSH in settings. Offical [guide](https://www.raspberrypi.com/documentation/computers/getting-started.html#installing-the-operating-system).

When flashing finished, insert card into RaspberryPi, connect display and insert HAT into GPIO slots. Then power on RPI. It should boot into GUI, touch screen should work and it should be connected to your network. Make sure the write down IP address of PI. Make sure that you can access your RPI via SSH.

## RPI Settings
From 2024 the RPI OS comes with wayland GUI interface. It's required to switch to old X11 to make it compatible with some of components. To do it:
- Log in into PI via SSH
- Type `sudo raspi-config` the GUI interface should appear
- Go to Advanced options -> A6 Wayland -> switch to X11
- Exit settings saving and reboot using `sudo reboot`

## Screen saver
If you want to hide HA interface after idle time, you can install screen saver. To do so, login via SSH and type:    
```bash
sudo apt-get update
sudo apt-get install xscreensaver
```
After that, the configuration menu should appear under menu settings. Open it via touch screen and then close. It should init default config file. Next, get back to you SSH terminal and type:
```bash
nano ~/.xscreensaver
```
Find and change lines.
```
change: 
- mode: blank
- timeout: 0:01:00
```

Now, reboot your system and wait for "timeout" time. The sceen should go blank.

## HomeAssistant WebPage in kiosk mode
To create autostrat script that will open web browser in kiosk mode with HA web page, login via SSH and type
```bash
sudo nano /etc/xdg/lxsession/LXDE-pi/autostart
```

Add to the end of file and save file

```bash
@xset s off
@xset -dpms
@xset s noblank
@chromium --kiosk --noerrdialogs --disable-infobars --hide-scrollbars http://homeassistant:8123
```

Now, reboot using `sudo reboot`. After restart, the HA web page should appear.

NOTE: You may need to type credentials into RPI. To do so, connect external keyboard.

## Voice assist
Follow [guide](https://github.com/rhasspy/wyoming-satellite/blob/master/docs/tutorial_2mic.md). My suggestion for services config:

### wyoming-satellite.service
```text
[Unit]
Description=Wyoming Satellite
Wants=network-online.target
After=network-online.target
Requires=2mic_leds.service
Requires=wyoming-openwakeword.service

[Service]
Type=simple
ExecStart=/home/Pi/wyoming-satellite/script/run \
        --name 'Satellite' \
        --uri 'tcp://0.0.0.0:10700' \
        --mic-command 'arecord -D plughw:CARD=seeed2micvoicec,DEV=0 -r 16000 -c 1 -f S16_LE -t raw' \
        --snd-command 'aplay -D plughw:CARD=seeed2micvoicec,DEV=0 -r 16000 -c 1 -f S16_LE -t raw' \
        --snd-command-rate 16000 \
        --snd-volume-multiplier 0.75 \
        --mic-auto-gain 5\
        --mic-noise-suppression 2 \
        --vad \
        --awake-wav sounds/awake.wav \
        --done-wav sounds/done.wav \
        --timer-finished-wav 'sounds/timer_finished.wav' \
        --event-uri 'tcp://127.0.0.1:10500' \
        --wake-uri 'tcp://127.0.0.1:10400' \
        --wake-word-name 'hey_jarvis'
WorkingDirectory=/home/pi/wyoming-satellite
Restart=always
RestartSec=1

[Install]
WantedBy=default.target
```

### wyoming-openwakeword.service
```text
[Unit]
Description=Wyoming openWakeWord

[Service]
Type=simple
ExecStart=/home/pi/wyoming-openwakeword/script/run --uri 'tcp://127.0.0.1:10400'
WorkingDirectory=/home/pi/wyoming-openwakeword
Restart=always
RestartSec=1

[Install]
WantedBy=default.target
```

### 2mic_leds.service
```text
[Unit]
Description=2Mic LEDs

[Service]
Type=simple
ExecStart=/home/pi/wyoming-satellite/examples/.venv/bin/python3 2mic_service.py \
        --uri 'tcp://127.0.0.1:10500' \
        --led-brightness 1
WorkingDirectory=/home/pi/wyoming-satellite/examples
Restart=always
RestartSec=1

[Install]
WantedBy=default.target
```