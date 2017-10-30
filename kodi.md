basic bits
#get root, needed to pretty much do anything.
sudo -i

#get software, will pull in over a Gb. See notes.
apt-get install kodi xorg xserver-xorg-legacy dbus-x11 alsa-utils openssh-server usbmount lirc lightdm-gtk-greeter

#needed to get alsa (sound) and proper graphics card drivers to be seen/used by kodi.
usermod -a -G audio,video kodi

#set these two options for xwrapper else wont start.
echo -e "allowed_users=anybody\nneeds_root_rights=yes" >> /etc/X11/Xwrapper.config

#add ntfs ability to usbmount so can auto mount
sed -i "s/vfat/ntfs vfat/" /etc/usbmount/usbmount.conf
Notes
lirc not needed if not using a remote. Installing lirc will bring up a ncurses menu , have to manually select your IR remote. Wihout lirc it uses kernel only which makes remote emulate a keyboard, not much use.

openssh-server not needed but makes setting up easier as can cut n paste below bits in more easily via an ssh connection.

usbmount only needed if wanting to mount media via usb.

dbus-x11 provides dbus-launch needed with systemd unit file.

create systemd service (unit file)
Save below to this file, this was lifted from kodi wiki. Note I removed the 'network.target' from 'After' as despite systemd docs to contrary this will hang boot indefinitely if network interface doesn't get an IP.

/etc/systemd/system/kodi.service

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
Enable the new systemd service
systemctl enable kodi
Add polkit entry for systemd's logind
Create this polkit file, (this allows the poweroff/shutdown menu to be available in kodi)

/etc/polkit-1/localauthority/50-local.d/kodi.pkla

[kodi user]
Identity=unix-user:kodi
Action=org.freedesktop.login1.*
ResultAny=yes
ResultInactive=no
ResultActive=yes

Using latest PPA version
At time of writing Kodi 15.2 comes with Ubuntu 16.04, to update to version 16

Note For me I had to apt-get remove and add - gets in a dependency loop mess if you just try and upgrade.
sudo -i
apt-get install software-properties-common
add-apt-repository ppa:team-xbmc/ppa
apt-get update
apt-get remove kodi kodi-bin
apt-get autoremove
apt-get install kodi kodi-bin
If you have already got unattended-upgrades running and want to keep it updated automatically add "LP-PPA-team-xbmc:${distro_codename}"; to Allowed-Origins so it looks something like this;

/etc/apt/apt.conf.d/50unattended-upgrades

Unattended-Upgrade::Allowed-Origins {
        "${distro_id}:${distro_codename}-security";
        "${distro_id}:${distro_codename}-updates";
        "${distro_id}:${distro_codename}-proposed";
//      "${distro_id}:${distro_codename}-backports";
        "LP-PPA-team-xbmc:${distro_codename}";

Configure Xserver to start Kodi on startup

Create the file /etc/lightdm/lightdm.conf with following content:

 

[SeatDefaults]
autologin-user=kodi
autologin-user-timeout=0
autologin-session=kodi
greeter-session=lightdm-gtk-greeter

 

Check the file /usr/share/xsessions/kodi.desktop which should be:

 

[Desktop Entry]
Name=Kodi
Comment=This session will start Kodi Media Center
Exec=kodi-standalone
TryExec=kodi-standalone
Type=Application




