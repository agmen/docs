Minimanl install of:
```
Distributor ID: Ubuntu
Description:    Ubuntu 16.04.3 LTS
Release:        16.04
Codename:       xenial
```

`apt-get install kodi xorg xserver-xorg-legacy dbus-x11 alsa-utils openssh-server usbmount lirc lightdm-gtk-greeter`

#needed to get alsa (sound) and proper graphics card drivers to be seen/used by kodi.
`usermod -a -G audio,video kodi`

#set these two options for xwrapper else wont start.
`echo -e "allowed_users=anybody\nneeds_root_rights=yes" >> /etc/X11/Xwrapper.config`

#add ntfs ability to usbmount so can auto mount
`sed -i "s/vfat/ntfs vfat/" /etc/usbmount/usbmount.conf`

#create systemd service (unit file)

`vim /etc/systemd/system/kodi.service`

```
[Unit]
Description = Kodi Media Center

After = systemd-user-sessions.service sound.target

[Service]
User = kodi
Group = kodi
Type = simple
PAMName = login
ExecStart = /usr/bin/xinit /usr/bin/dbus-launch --exit-with-session /usr/bin/kodi-standalone -- :0 -nolisten tcp vt7
Restart = on-abort
RestartSec = 5

[Install]
WantedBy = multi-user.target
```

#Enable the new systemd service
`systemctl enable kodi`

#Add polkit entry for systemd's logind (this allows the poweroff/shutdown menu to be available in kodi)

`vim /etc/polkit-1/localauthority/50-local.d/kodi.pkla`

```
[kodi user]
Identity=unix-user:kodi
Action=org.freedesktop.login1.*
ResultAny=yes
ResultInactive=no
ResultActive=yes
```

#Configure Xserver to start Kodi on startup

`vim /etc/lightdm/lightdm.conf`

```
[SeatDefaults]
autologin-user=kodi
autologin-user-timeout=0
autologin-session=kodi
greeter-session=lightdm-gtk-greeter
```

`vim /usr/share/xsessions/kodi.desktop`

```
[Desktop Entry]
Name=Kodi
Comment=This session will start Kodi Media Center
Exec=kodi-standalone
TryExec=kodi-standalone
Type=Application
```

#Allow desktop sessions
```
cat /usr/share/xsessions/custom-desktop << EOF
[Desktop Entry]
Name=Xsession
Exec=/etc/X11/Xsession
EOF
```

```
cat /home/user/.xsession << EOF
#!/usr/bin/env bash
exec startfluxbox
EOF
```

#random .xsession errors
https://forum.kodi.tv/showthread.php?tid=304441


find /media/T2 -type d -exec setfacl -m d:g:kodi:rwx {} \;
setfacl -Rm g:kodi:rwX /media/T2/

cat /usr/lib/tmpfiles.d/sickbeard.conf
```
d /var/run/sickbeard 0755 sickbeard sickbeard -
```

```
systemd-tmpfiles --create
```

#With openbox to allow external programs to launch
```
wget -O openbox-kodi-master.zip https://github.com/lufinkey/kodi-openbox/archive/master.zip
```

```
unzip openbox-master.zip
```

```
cd kodi-openbox-master
bash ./build.sh
sudo dpkg -i kodi-openbox.deb
```

```
cat << EOF > /etc/lightdm/lightdm.conf
[Seat:*]
autologin-user=kodi
autologin-session=kodi-openbox
sudo reboot
```

#Configure Advanced Launcher

```
https://github.com/SpiralCut/plugin.program.advanced.launcher/archive/master.zip -O advanced-launcher-master.zip
```

In Kodi, add the Advanced Launcher add-on. How to install an add-on from a .zip is explained on the Kodi Wiki. The .zip is located in the Home folder and is named advanced-launcher-master.zip.

Navigate to your program add-ons in the Aeon Nox skin. It is located on the Home Screen under the title Apps.

To add a launcher for Steam do the following:
* Open the add-on
* Open the Default folder
* Bring up the Context Menu and select Standalone Launcher
* In the Path dialogue that appers select Root and navigate to: /usr/bin/kodi-openbox-runprogram. Press Done.
* In the Arguments dialogue enter the argument: steam -bigpicture
* In the Title dialogue enter the name Steam
* In the Platform dialogue select Linux
* Leave the next two dialogues blank.
* Mark the Steam launcher and bring up the Context Menu and add it to Favorites.

Create a launcher for EmulationStation:
* Open the add-on
* Open the Default folder
* Bring up the Context Menu and select Standalone Launcher
* In the Path dialogue that appers select Root and navigate to: /usr/bin/kodi-openbox-runprogram. Press Done.
* In the Arguments dialogue enter the argument: emulationstation
* In the Title dialogue enter the name Emulation
* In the Platform dialogue select Linux
* Leave the next two dialogues blank.
* Mark the Emulation launcher and bring up the Context Menu and add it to Favorites.
