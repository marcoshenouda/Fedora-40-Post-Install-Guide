 # Fedora 40 Post Install Guide
Things to do after installing Fedora 40

## Faster Updates
* `sudo nano /etc/dnf/dnf.conf` 
* Copy and replace the text with the following:
```
[main] 
gpgcheck=1 
installonly_limit=3 
clean_requirements_on_remove=True 
best=False 
skip_if_unavailable=True 
max_parallel_downloads=20 
``` 
* Note: The `fastestmirror=1` and `deltarpm=true` arguments were removed. Avoid using these even if you find them in other guides. They are counterproductive at best.

## RPM Fusion
* Fedora has disabled the repositories for a lot of free and non-free .rpm packages by default. Follow this if you want to use non-free software like Steam, Discord and some multimedia codecs etc. As a general rule of thumb it is advised to do this to get access to many mainstream useful programs.
* If you forgot to enable third party repositories during the initial setup window, enable them by pasting the following into the terminal: 
* `sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm`
* also while you're at it, install app-stream metadata by
* `sudo dnf update @core`

## Update 
* Go into the software center and click on update. Alternatively, you can use the following commands:
* `sudo dnf -y update`
* `sudo dnf -y upgrade --refresh`
* Reboot

## Firmware
* If your system supports firmware update delivery through lvfs, update your device firmware by:
```
sudo fwupdmgr refresh --force
sudo fwupdmgr get-devices # Lists devices with available updates.
sudo fwupdmgr get-updates # Fetches list of available updates.
sudo fwupdmgr update
```
## Flatpak
* Fedora doesn't include all non-free flatpaks by default. The command below enables access to all the flathub flatpaks. Particularly useful for users of Fedora KDE and other spins since they do not get the "Enable Third Party Repositories" option on initial boot.
* `flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo`

## NVIDIA Drivers
* Only follow this if you have a NVIDIA gpu. Also, don't follow this if you have a gpu which has dropped support for newer driver releases i.e. anything earlier than nvidia GT/GTX 600, 700, 800, 900, 1000, 1600 and RTX 2000, 3000, 4000 series. Fedora comes preinstalled with NOUVEAU drivers which may or may not work better on those remaining older GPUs. This should be followed by Desktop and Laptop users alike.
* Disable Secure Boot.
* `sudo dnf update` #To make sure you're on the latest kernel and then reboot.
* Enable RPM Fusion Nvidia non-free repository in the app store and install it from there,
* or alternatively
* `sudo dnf install akmod-nvidia`
* Install this if you use applications that can utilise CUDA i.e. Davinci Resolve, Blender etc.
* `sudo dnf install xorg-x11-drv-nvidia-cuda`
* Wait for atleast 5 mins before rebooting, to let the kermel module get built.
* `modinfo -F version nvidia` #Check if the kernel module is built.
* Reboot

## ~~Battery Life (Deprecated)~~
* ~~Follow this if you have a Laptop and are facing sub optimal battery backup.~~
* ~~power-profiles-daemon which come pre-configured on fedora works well on a great majority of systems but still in case you're facing sub-optimal battery backup you try installing tlp by:~~
* ~~`sudo dnf install tlp tlp-rdw`~~
* ~~and mask power-profiles-daemon by:~~
* ~~`sudo systemctl mask power-profiles-daemon`~~
* ~~Also install powertop by:~~
* ~~`sudo dnf install powertop`~~
* ~~`sudo powertop --auto-tune`~~
* Edit: Fedora comes preinstalled with PPD(power-profiles-daemon) which works well on its own now and all the aforementioned changes are now unnecessary. Just follow [HW video acceleration](https://github.com/devangshekhawat/Fedora-40-Post-Install-Guide/blob/main/README.md#hw-video-acceleration) for better battery backup. 

## Media Codecs
* Install these to get proper multimedia playback.
````
sudo dnf swap ffmpeg-free ffmpeg --allowerasing # Switch to full FFMPEG.
sudo dnf group install Multimedia
sudo dnf update @multimedia --setopt="install_weak_deps=False" --exclude=PackageKit-gstreamer-plugin # Installs gstreamer components. Required if you use Gnome Videos and other dependent applications.
sudo dnf group install 'sound-and-video' # All packages in this group are listed as optional.
sudo dnf update @sound-and-video # Installs useful Sound and Video complement packages.
````
* ~~dnf group info 'sound-and-video' # Shows all optional packages included.~~
* ~~sudo dnf group install --with-optional --skip-broken 'sound-and-video' # Installs all optional packages in the group.~~

* Install these if you want DVD Playback
````
sudo dnf install rpmfusion-free-release-tainted # Tainted free is dedicated for FLOSS packages where some usages might be restricted in some countries.
sudo dnf install libdvdcss
````

## H/W Video Acceleration
* Helps decrease load on the CPU when watching videos online by alloting the rendering to the dGPU/iGPU. Quite helpful in increasing battery backup on laptops.

### H/W Video Decoding with VA-API 
* `sudo dnf install ffmpeg ffmpeg-libs libva libva-utils`

<details>
<summary>Intel</summary>
 
* If you have a recent Intel chipset (5th Gen and above) after installing the packages above., Do:
* `sudo dnf swap libva-intel-media-driver intel-media-driver --allowerasing`
</details>

<details>
<summary>AMD</summary>No need to do this for intel integrated graphics. Mesa drivers are for AMD graphics, who lost support for h264/h245 in the fedora repositories in f38 due to legal concerns.
 
* If you have an AMD chipset, after installing the packages above do:
```
sudo dnf swap mesa-va-drivers mesa-va-drivers-freeworld
sudo dnf swap mesa-vdpau-drivers mesa-vdpau-drivers-freeworld
```

* If using i686 compat libraries (for steam or alikes):
```
sudo dnf swap mesa-va-drivers.i686 mesa-va-drivers-freeworld.i686
sudo dnf swap mesa-vdpau-drivers.i686 mesa-vdpau-drivers-freeworld.i686
```
</details>

<details>
<summary>Nvidia</summary>
 
* If you have an Nvidia GPU after installing the packages above, install this:
* `sudo dnf install libva-nvidia-driver`
* This library requires that the `nvidia_drm kernel module` is configured with the parameter `nvidia-drm.modeset=1`
* ~~sudo nano /etc/default/grub~~
* ~~Add `nvidia-drm.modeset=1` to `GRUB_CMDLINE_LINUX=`~~
* ~~Example: `GRUB_CMDLINE_LINUX="rhgb quiet rd.driver.blacklist=nouveau modprobe.blacklist=nouveau nvidia-drm.modeset=1"`~~
* ~~Next run `sudo grub2-mkconfig -o /boot/grub2/grub.cfg`~~
* `sudo grubby --update-kernel=ALL --args="nvidia-drm.modeset=1"`
* Run `sudo nano /etc/environment` then add `LIBVA_DRIVER_NAME=nvidia`
</details>





### Potential Fix for Audio Crackling/Latency
* `sudo grubby --args="preempt=full" --update-kernel=ALL`
* After this, reboot your PC. Then run this command to check if it worked:
* `sudo dmesg | grep Preempt`
* You should see a line that says something like: `Dynamic Preempt: full`.

### OpenH264 for Firefox
* `sudo dnf config-manager --set-enabled fedora-cisco-openh264`
* `sudo dnf install -y openh264 gstreamer1-plugin-openh264 mozilla-openh264`
* Afterwards you need open Firefox, go to menu -> Add-ons -> Plugins and enable OpenH264 plugin.

## Set Hostname
* `hostnamectl set-hostname YOUR_HOSTNAME`

## Custom DNS Servers
* For people that want to setup custom DNS servers for better privacy
```
sudo mkdir -p '/etc/systemd/resolved.conf.d' && sudo -e '/etc/systemd/resolved.conf.d/99-dns-over-tls.conf'

[Resolve]
DNS=1.1.1.2#security.cloudflare-dns.com 1.0.0.2#security.cloudflare-dns.com 2606:4700:4700::1112#security.cloudflare-dns.com 2606:4700:4700::1002#security.cloudflare-dns.com
DNSOverTLS=yes
```
## Set UTC Time
* Used to counter time inconsistencies in dual boot systems
* `sudo timedatectl set-local-rtc '0'`

## Optimizations
* The tips below can allow you to squeeze out a little bit more performance from your system. 

### Disable Mitigations 
* Increases performance in multithreaded systems. The more cores you have in your cpu the greater the performance gain. 5-30% performance gain varying upon systems. Do not follow this if you share services and files through your network or are using fedora in a VM. 
* Modern intel CPUs (above 10th gen) do not gain noticeable performance improvements upon disabling mitigations. Hence, disabling mitigations can present some security risks against various attacks, however, it still _might_ increase the CPU performance of your system.
* `sudo grubby --update-kernel=ALL --args="mitigations=off"`

### Modern Standby
* Can result in better battery life when your laptop goes to sleep.
* `sudo grubby --update-kernel=ALL --args="mem_sleep_default=s2idle"`
* If "s2idle" doesn't work for you i.e. people with alder lake CPUs, then you might want to refer to [this](https://www.reddit.com/r/linuxhardware/comments/ng166t/s3_deep_sleep_not_working/)

### Enable nvidia-modeset 
* Useful if you have a laptop with an Nvidia GPU. Necessary for some PRIME-related interoperability features.
* `sudo grubby --update-kernel=ALL --args="nvidia-drm.modeset=1"`

### Disable `NetworkManager-wait-online.service`
* Disabling it can decrease the boot time by at least ~15s-20s:
* `sudo systemctl disable NetworkManager-wait-online.service`

### Disable Gnome Software from Startup Apps
* Gnome software autostarts on boot for some reason, even though it is not required on every boot unless you want it to do updates in the background, this takes at least 100MB of RAM upto 900MB (as reported anecdotically). You can stop it from autostarting by:
* `sudo rm /etc/xdg/autostart/org.gnome.Software.desktop`

## Gnome Extensions
* Don't install these if you are using a different spin of Fedora.
* Pop Shell - run `sudo dnf install -y gnome-shell-extension-pop-shell xprop` to install it.
* [GSconnect](https://extensions.gnome.org/extension/1319/gsconnect/) - run `sudo dnf install nautilus-python` for full support.
* [Gesture Improvements](https://extensions.gnome.org/extension/4245/gesture-improvements/)
* [Quick Settings Tweaker](https://github.com/qwreey75/quick-settings-tweaks)
* [User Themes](https://extensions.gnome.org/extension/19/user-themes/)
* [Compiz Windows Effect](https://extensions.gnome.org/extension/3210/compiz-windows-effect/)
* [Just Perfection](https://extensions.gnome.org/extension/3843/just-perfection/)
* [Rounded Windows Corners](https://extensions.gnome.org/extension/5237/rounded-window-corners/)
* [Dash to Dock](https://extensions.gnome.org/extension/307/dash-to-dock/)
* [Quick Settings Tweaker](https://extensions.gnome.org/extension/5446/quick-settings-tweaker/)
* [Blur My Shell](https://extensions.gnome.org/extension/3193/blur-my-shell/)
* [Bluetooth Quick Connect](https://extensions.gnome.org/extension/1401/bluetooth-quick-connect/)
* [App Indicator Support](https://extensions.gnome.org/extension/615/appindicator-support/)
* [Clipboard Indicator](https://extensions.gnome.org/extension/779/clipboard-indicator/)
* [Legacy (GTK3) Theme Scheme Auto Switcher](https://extensions.gnome.org/extension/4998/legacy-gtk3-theme-scheme-auto-switcher/)
* [Caffeine](https://extensions.gnome.org/extension/517/caffeine/)
* [Vitals](https://extensions.gnome.org/extension/1460/vitals/)
* [Wireless HID](https://extensions.gnome.org/extension/4228/wireless-hid/)
* [Logo Menu](https://extensions.gnome.org/extension/4451/logo-menu/)
* [Space Bar](https://github.com/christopher-l/space-bar)

## Apps [Optional]
* Packages for Rar and 7z compressed files support:
 `sudo dnf install -y unzip p7zip p7zip-plugins unrar`
* These are Some Packages that I use and would recommend:
```
file-roller
gthumb
gnome-tweaks
gnome-extensions-app
wine
winetricks
vkBasalt
gamemode
gamescope
mangohud
goverlay
vlc
mpv
celluloid
smplayer
audacious
lollypop
strawberry
g4music
yt-dlp
Piper
Solaar
Steam
Lutris
Heroic Launcher (Local RPM file)
bottles
Protonup-Qt (Flatpak)
Protontricks
Flatseal
Spotify (Flatpak)
Dolphin-emu
PCSX2 (Flatpak)
Duckstation (Flatpak)
melonDS (Flatpak)
PPSSPP (Flatpak)
Ryujinx (Flatpak)
Cemu (Flatpak)
Chromium
Discord
Calibre 
Visual Studio Code (Enable Microsoft VSCode repo)
innoextract
lgogdownloader
fastfetch
cmatrix
```

## App Links [Optional]
* These are Some Flathub Packages that I use and would recommend:
* [file-roller](https://packages.fedoraproject.org/pkgs/file-roller/file-roller/)
* [gthumb](https://packages.fedoraproject.org/pkgs/gthumb/gthumb/)
* [gnome-tweaks](https://packages.fedoraproject.org/pkgs/gnome-tweaks/gnome-tweaks/)
* [gnome-extensions-app](https://packages.fedoraproject.org/pkgs/gnome-extensions-app/gnome-extensions-app/)
* [wine](https://packages.fedoraproject.org/pkgs/wine/wine/)
* [winetricks](https://packages.fedoraproject.org/pkgs/winetricks/winetricks/)
* [vkBasalt](https://packages.fedoraproject.org/pkgs/vkBasalt/vkBasalt/index.html)
* [gamemode](https://packages.fedoraproject.org/pkgs/gamemode/gamemode/)
* [gamescope](https://packages.fedoraproject.org/pkgs/gamescope/gamescope/)
* [mangohud](https://packages.fedoraproject.org/pkgs/mangohud/mangohud/)
* [goverlay](https://packages.fedoraproject.org/pkgs/goverlay/goverlay/)
* [vlc](https://packages.fedoraproject.org/pkgs/vlc/vlc/)
* [mpv](https://packages.fedoraproject.org/pkgs/mpv/mpv/)
* [celluloid](https://packages.fedoraproject.org/pkgs/celluloid/celluloid/)
* [smplayer](https://packages.fedoraproject.org/pkgs/smplayer/smplayer/)
* [audacious](https://packages.fedoraproject.org/pkgs/audacious/audacious/)
* [lollypop](https://packages.fedoraproject.org/pkgs/lollypop/lollypop/)
* [strawberry](https://packages.fedoraproject.org/pkgs/strawberry/strawberry/)
* [g4music](https://packages.fedoraproject.org/pkgs/g4music/g4music/)
* [yt-dlp](https://packages.fedoraproject.org/pkgs/yt-dlp/yt-dlp/)

## Flatpaks [Optional]
* These are Some Flathub Packages that I use and would recommend:
* [Amberol](https://flathub.org/apps/io.bassi.Amberol)
* [Spotify](https://flathub.org/apps/com.spotify.Client)
* [ProtonUp-Qt](https://flathub.org/apps/net.davidotek.pupgui2)
* [Heroic Games Launcher](https://flathub.org/apps/com.heroicgameslauncher.hgl)

## Theming [Optional]

### GTK Themes
* Don't install these if you are using a different spin of Fedora.
* https://github.com/lassekongo83/adw-gtk3
* https://github.com/vinceliuice/Colloid-gtk-theme
* https://github.com/EliverLara/Nordic
* https://github.com/vinceliuice/Orchis-theme
* https://github.com/vinceliuice/Graphite-gtk-theme

### Use themes in Flatpaks
* `sudo flatpak override --filesystem=$HOME/.themes`
* `sudo flatpak override --env=GTK_THEME=my-theme` 

### Icon Packs
* https://github.com/vinceliuice/Tela-icon-theme
* https://github.com/vinceliuice/Colloid-gtk-theme/tree/main/icon-theme

### Wallpaper
* https://github.com/manishprivet/dynamic-gnome-wallpapers

### Firefox Theme
* Install Firefox Gnome theme by: `curl -s -o- https://raw.githubusercontent.com/rafaelmardojai/firefox-gnome-theme/master/scripts/install-by-curl.sh | bash`

### Starship (terminal theme)
* Configure starship to make your terminal look good (refer https://starship.rs)

### Grub Theme
* https://github.com/vinceliuice/grub2-themes
