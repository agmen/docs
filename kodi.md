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
=======
#randmon .xsession errors
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
