# Photonicat 2 OpenWrt

<div align="center">
<img src="https://photonicat.com/images/thumb/d/d7/Pcat2-wiki.webp/1200px-Pcat2-wiki.webp.png?20250822110422" alt="Photonicat 2" width="350" height="350"/>
</div>

Photonicat 2 is a portable smart router platform based on the Rockchip RK3678 SoC. This fork keeps the Photonicat 2 hardware support while moving the default firmware closer to a stock OpenWrt experience.

## Hardware Highlights

- **CPU upgrade**: 8-core RK3678 SoC built on an 8 nm process.
- **Memory upgrade**: LPDDR5 with On-Die ECC support.
- **Antenna system**: 7 internal enhanced antennas, plus support for 4 external antennas.
- **Battery upgrade**: Integrated 4 x 18650 battery pack.
- **Power system**: Supports batteryless DC operation and 30 W bidirectional fast charging.
- **Storage expansion**: eMMC, SD card, and NVMe SSD support.
- **Display**: Built-in screen for device and network status.
- **Sensors**: Coulomb meter for current/capacity measurement and a G-sensor for motion/orientation.
- **Software**: OpenWrt support, with Debian, Ubuntu, and Android firmware options available separately.

## Supported Devices

This source tree supports Photonicat v1 and Photonicat 2 devices.

- https://photonicat.com/
- https://photonicat.com/wiki

## About This Fork

This fork removes the Chinese-focused default packages, Chinese LuCI translations, and the custom Photonicat web UI from the Photonicat 2 image while keeping the hardware-specific support packages:

- `pcat-manager`
- `pcat2-display-mini`
- Photonicat power-management and watchdog drivers

The Photonicat 2 mini display package is pinned to this fork:

- https://github.com/Defiant-Engineer/photonicat2_mini_display

## Default Settings

The Photonicat 2 image is intended to behave closer to stock OpenWrt defaults:

- Default LAN IP: `192.168.1.1`
- Default LAN netmask: `255.255.255.0`
- DHCP server: enabled on LAN
- Default Wi-Fi SSID prefix: `OpenWrt`
- Default Wi-Fi encryption: open, matching normal first-boot OpenWrt behavior
- LuCI runs on the normal web interface instead of the removed Photonicat custom port-80 page

Set a root password after first login.

## Build Notes

1. Do not build as `root`.
2. Use a case-sensitive filesystem.
3. A clean Debian 13 or Ubuntu 24.04 LTS build host is recommended.
4. The first build can take a long time and may need significant disk space.

## Install Build Dependencies

```bash
sudo apt update -y
sudo apt full-upgrade -y
sudo apt install -y ack antlr3 asciidoc autoconf automake autopoint binutils bison build-essential \
  bzip2 ccache clang cmake cpio curl device-tree-compiler flex gawk gcc-multilib g++-multilib gettext \
  genisoimage git gperf haveged help2man intltool libc6-dev-i386 libelf-dev libfuse-dev libglib2.0-dev \
  libgmp3-dev libltdl-dev libmpc-dev libmpfr-dev libncurses5-dev libncursesw5-dev libpython3-dev \
  libreadline-dev libssl-dev libtool llvm lrzsz msmtp ninja-build p7zip p7zip-full patch pkgconf \
  python3 python3-pyelftools python3-setuptools qemu-utils rsync scons squashfs-tools subversion \
  swig texinfo uglifyjs upx-ucl unzip vim wget xmlto xxd zlib1g-dev
```

## Build Photonicat 2 Firmware

```bash
git clone https://github.com/Defiant-Engineer/photonicat_openwrt.git
cd photonicat_openwrt
git switch photonicat2-openwrt-cleanup-only

./scripts/feeds update -a
./scripts/feeds install -a
cp configs/photonicat2_base_defconfig .config
make defconfig

make download -j"$(nproc)"
make V=s -j"$(nproc)"
```

Build outputs are written to:

```txt
bin/targets
```

## Rebuild After Updating

```bash
cd photonicat_openwrt
git pull --ff-only
./scripts/feeds update -a
./scripts/feeds install -a
make defconfig
make download -j"$(nproc)"
make V=s -j"$(nproc)"
```

## Reconfigure From Scratch

```bash
rm -rf .config tmp
cp configs/photonicat2_base_defconfig .config
make menuconfig
make V=s -j"$(nproc)"
```

## WSL/WSL2 Notes

WSL can include Windows paths with spaces in `PATH`, which may break the OpenWrt build. If needed, run builds with a clean Linux-style path:

```bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin make V=s -j"$(nproc)"
```

OpenWrt also requires a case-sensitive filesystem. NTFS paths mounted into WSL are commonly case-insensitive and may fail with:

```txt
Build dependency: OpenWrt can only be built on a case-sensitive filesystem
```

Create a repository directory and enable case sensitivity before cloning:

```powershell
# Run from an elevated Windows terminal.
fsutil.exe file setCaseSensitiveInfo <your_local_photonicat_openwrt_path> enable
git clone https://github.com/Defiant-Engineer/photonicat_openwrt.git <your_local_photonicat_openwrt_path>
```

Enabling case sensitivity after cloning may not fix files that already exist.

## macOS Native Build Notes

Linux is recommended. If building on macOS, install Xcode from the App Store and install Homebrew:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Install the required GNU tools:

```bash
brew unlink awk
brew install coreutils diffutils findutils gawk gnu-getopt gnu-tar grep make ncurses pkg-config wget quilt xz
brew install gcc@11
```

For Intel Macs:

```bash
echo 'export PATH="/usr/local/opt/coreutils/libexec/gnubin:$PATH"' >> ~/.bashrc
echo 'export PATH="/usr/local/opt/findutils/libexec/gnubin:$PATH"' >> ~/.bashrc
echo 'export PATH="/usr/local/opt/gnu-getopt/bin:$PATH"' >> ~/.bashrc
echo 'export PATH="/usr/local/opt/gnu-tar/libexec/gnubin:$PATH"' >> ~/.bashrc
echo 'export PATH="/usr/local/opt/grep/libexec/gnubin:$PATH"' >> ~/.bashrc
echo 'export PATH="/usr/local/opt/gnu-sed/libexec/gnubin:$PATH"' >> ~/.bashrc
echo 'export PATH="/usr/local/opt/make/libexec/gnubin:$PATH"' >> ~/.bashrc
```

For Apple Silicon Macs:

```bash
echo 'export PATH="/opt/homebrew/opt/coreutils/libexec/gnubin:$PATH"' >> ~/.bashrc
echo 'export PATH="/opt/homebrew/opt/findutils/libexec/gnubin:$PATH"' >> ~/.bashrc
echo 'export PATH="/opt/homebrew/opt/gnu-getopt/bin:$PATH"' >> ~/.bashrc
echo 'export PATH="/opt/homebrew/opt/gnu-tar/libexec/gnubin:$PATH"' >> ~/.bashrc
echo 'export PATH="/opt/homebrew/opt/grep/libexec/gnubin:$PATH"' >> ~/.bashrc
echo 'export PATH="/opt/homebrew/opt/gnu-sed/libexec/gnubin:$PATH"' >> ~/.bashrc
echo 'export PATH="/opt/homebrew/opt/make/libexec/gnubin:$PATH"' >> ~/.bashrc
```

Reload your shell configuration, then build from a Bash shell:

```bash
source ~/.bashrc
bash
```

## License

This tree is based on Lean's LEDE/OpenWrt source tree and Photonicat's OpenWrt work. Packages under `package/lean` are provided under their respective licenses, including GPLv3 where applicable.
