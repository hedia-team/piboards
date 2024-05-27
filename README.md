# piboards

Contains lists of URLs to be launched on a Raspberry Pi.

## Usage

Most of this takes inspiration from [Building a Raspberry Pi dashboard](https://www.aimeerivers.com/2021/08/15/building-a-raspberry-pi-dashboard.html).

### Recommendations

* Set up [Tab Carousel](https://chromewebstore.google.com/detail/tabcarousel/ddldimidiliclngjipajmjjiakhbcohn) in Chromium
* Install unclutter: `sudo apt-get -y install unclutter`
* Install emoji: `sudo apt-get -y install fonts-noto-color-emoji`

### Chromium launch script

Your Raspberry Pi will need a script like this:

**/home/pi/dashboard1.sh**
```bash
#!/bin/bash

# config options
export ORG=hedia-team
export REPO=piboards
export BRANCH=main
export DASHBOARD=dashboards/backend1.txt

# don't let this display sleep
export DISPLAY=:0
xset s noblank
xset s off
xset -dpms

# hide mouse pointer
unclutter -idle 0.5 -root &

# trick Chromium into believing it shut down successfully last time
sed -i 's/"exited_cleanly":false/"exited_cleanly":true/' /home/pi/custom-chromium-profiles/profile1/Default/Preferences
sed -i 's/"exit_type":"Crashed"/"exit_type":"Normal"/' /home/pi/custom-chromium-profiles/profile1/Default/Preferences

# Launch Chromium 
/usr/bin/chromium-browser \
  --user-data-dir=/home/pi/custom-chromium-profiles/profile1 \
  --window-position=0,0 \
  --start-fullscreen \
  --no-first-run \
  --noerrdialogs \
  --disable-infobars \
  $(curl -s "https://raw.githubusercontent.com/${ORG}/${REPO}/${BRANCH}/${DASHBOARD}") &
```

💡 ***Tip:** If you want to start on a different screen, use a `--window-position` with high enough value that it goes to that screen.*

Set that script to be executable by anyone:

    sudo chmod a+x /home/pi/dashboard1.sh

Try it out:

    /home/pi/dashboard1.sh

Chromium should launch, in full screen, with the tabs loaded from the relevant file in this repo.

### Create the service

The service definition will look something like this:

**/home/pi/dashboard1.sh**
```bash
[Unit]
Description=Dashboard1
After=network.target
After=systemd-user-sessions.service
After=network-online.target
After=graphical.target
Requires=graphical.target
Requires=network-online.target

[Service]
Environment=DISPLAY=:0
Environment=XAUTHORITY=/home/pi/.Xauthority
Type=forking
ExecStartPre=/bin/sh -c 'until ping -c1 raw.githubusercontent.com; do sleep 1; done;'
ExecStart=/home/pi/dashboard1.sh
Restart=on-abort
User=pi
Group=pi

[Install]
WantedBy=graphical.target
```

Enable this service:

    sudo systemctl enable dashboard

Try it out:

    sudo systemctl start dashboard

Now the service should start automatically whenever you turn on the Raspberry Pi.
