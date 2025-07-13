# NetHunter Kernel Compilation Guide - Xiaomi Redmi Note 7 (Lavender)

Updated build guide for Android 15 with current kernel version 4.19.321-S0NiX  
**By 0xb0rn3 | 0xbv1** - Updated for Android 15 compatibility

## 📱 Device Information (Current)

- **Device**: Xiaomi Redmi Note 7 (lavender)
- **Manufacturer**: Xiaomi
- **Android Version**: 15 (API Level 35)
- **Architecture**: ARM64 (arm64-v8a)
- **Current Kernel**: Linux 4.19.321-S0NiX
- **Compiler**: Android clang version 19.0.1 (r536225)
- **Build Date**: Fri May 16 14:11:11 UTC 2025
- **Bootloader**: Unlocked (Required)
- **Root**: Magisk recommended

## 🚨 Important Notes for Android 15

### Critical Compatibility Issues

- **Kernel Version**: Your device runs 4.19.321-S0NiX, which is compatible with NetHunter patches
- **Clang Version**: Android 15 uses clang 19.0.1 - ensure toolchain compatibility
- **Build Environment**: Requires Android 15 SDK (API 35) and updated NDK
- **Security**: Android 15 has enhanced security features that may affect NetHunter functionality

### Prerequisites Update

- **Minimum RAM**: 16GB (Android 15 build requirements)
- **Storage**: 80GB+ free space
- **Android 15 SDK**: API Level 35
- **NDK**: r26d or newer
- **Clang**: 19.0.1 or compatible

## 🔧 Build Environment Setup

### System Requirements (Updated for 2025)

Recommended Linux distributions for Android 15 builds:
- Ubuntu 22.04 LTS or 24.04 LTS
- Arch Linux (rolling release)
- Fedora 39+
- openSUSE Tumbleweed

### Ubuntu 22.04/24.04 LTS Setup

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Android 15 build dependencies
sudo apt install -y build-essential git python3 python3-pip bc \
                   libssl-dev libncurses-dev libelf-dev bison \
                   flex make cmake ninja-build zip unzip curl wget \
                   clang-19 llvm-19 lld-19 adb fastboot openjdk-21-jdk \
                   python3-distutils python3-setuptools rsync

# Set clang-19 as default
sudo update-alternatives --install /usr/bin/clang clang /usr/bin/clang-19 100
sudo update-alternatives --install /usr/bin/clang++ clang++ /usr/bin/clang++-19 100
```

### Arch Linux Setup

```bash
# Update system
sudo pacman -Syu

# Install dependencies
sudo pacman -S base-devel git python python-pip bc openssl \
               android-tools jdk21-openjdk clang llvm lld \
               make cmake ninja zip unzip curl wget rsync

# Install AUR packages
yay -S android-ndk android-sdk-build-tools
```

## 📦 Source Code and Toolchain

### 1. Android 15 NDK Setup

```bash
# Create build directory
mkdir -p ~/nethunter-android15
cd ~/nethunter-android15

# Download Android NDK r26d (compatible with Android 15)
wget https://dl.google.com/android/repository/android-ndk-r26d-linux.zip
unzip android-ndk-r26d-linux.zip
export ANDROID_NDK_HOME=~/nethunter-android15/android-ndk-r26d

# Set toolchain paths
export ANDROID_NDK_TOOLCHAIN=${ANDROID_NDK_HOME}/toolchains/llvm/prebuilt/linux-x86_64
export PATH=${ANDROID_NDK_TOOLCHAIN}/bin:$PATH
```

### 2. NetHunter Kernel Builder (Latest - GitLab)

```bash
# Clone latest NetHunter project from GitLab (current official repo)
git clone https://gitlab.com/kalilinux/nethunter/build-scripts/kali-nethunter-project.git

# Alternative: Clone the kernel builder specifically
git clone https://gitlab.com/kalilinux/nethunter/build-scripts/kali-nethunter-kernel.git kali-nethunter-kernel-builder
cd kali-nethunter-kernel-builder
chmod +x build.sh
chmod +x bootstrap.sh
```

### 3. Kernel Source for Android 15

```bash
# Use Team-420's proven NetHunter kernel source
git clone https://github.com/Team-420/android_kernel_xiaomi_lavender.git -b Nethunter kernel_lavender

# Alternative: Use latest compatible source
# git clone https://github.com/ravi0900/android_kernel_xiaomi_lavender.git kernel_lavender
```

## ⚙️ Configuration for Android 15

### 1. Environment Variables

```bash
# Export Android 15 specific variables
export ANDROID_MAJOR_VERSION=15
export ANDROID_MINOR_VERSION=0
export ANDROID_VERSION=15.0
export ANDROID_API_LEVEL=35
export TARGET_ARCH=arm64
export ARCH=arm64
export SUBARCH=arm64
export CROSS_COMPILE=aarch64-linux-android35-
export CLANG_TRIPLE=aarch64-linux-gnu-
export CC=clang
export CXX=clang++
export AR=llvm-ar
export NM=llvm-nm
export STRIP=llvm-strip
export OBJCOPY=llvm-objcopy
export OBJDUMP=llvm-objdump
export READELF=llvm-readelf
export HOSTCC=clang
export HOSTCXX=clang++
export HOSTAR=llvm-ar
export HOSTLD=ld.lld
```

### 2. Device Configuration

```bash
cd ~/nethunter-android15/kali-nethunter-kernel-builder

# Create local.config for Android 15
cat > local.config << 'EOF'
# NetHunter Configuration for Xiaomi Redmi Note 7 (Android 15)
DEVICE_NAME="lavender"
KERNEL_SOURCE="~/nethunter-android15/kernel_lavender"
KERNEL_ARCH="arm64"
KERNEL_SUBARCH="arm64"
KERNEL_DEFCONFIG="lavender_defconfig"
CROSS_COMPILE="aarch64-linux-android35-"
CLANG_TRIPLE="aarch64-linux-gnu-"
ANDROID_MAJOR_VERSION="15"
ANDROID_MINOR_VERSION="0"
ANDROID_VERSION="15.0"
ANDROID_API_LEVEL="35"
KERNEL_LOCALVERSION="-NetHunter-A15"
THREADS=$(nproc)
CLANG_VERSION="19.0.1"

# NetHunter Features
ENABLE_WIFI_INJECTION=true
ENABLE_USB_HID=true
ENABLE_BLUETOOTH_INJECTION=true
ENABLE_RTL8188EU=true
ENABLE_RTL8812AU=true
ENABLE_RTL8814AU=true
ENABLE_MT7612U=true
ENABLE_ATH9K_HTK=true
ENABLE_USB_SERIAL=true
ENABLE_KALI_WIFI=true

# Android 15 specific
ENABLE_SELINUX_PERMISSIVE=true
ENABLE_ANDROID15_COMPAT=true
MODULES_COMPRESSION=xz
USE_CCACHE=true
EOF
```

### 3. Fix Missing Aircrack Drivers

Based on the error output, the NetHunter patches are partially applied but missing the actual RTL8812AU driver. Here's the fix:

```bash
# Check current state
cd ~/nethunter-android15/kernel_lavender
ls -la drivers/aircrack/

# You should see rtl8188eus, rtl8188fu, rtl8192eu but missing rtl8812au
# Let's add the missing RTL8812AU driver

# Create the missing RTL8812AU driver directory
mkdir -p drivers/aircrack/rtl8812au

# Clone the RTL8812AU driver
cd drivers/aircrack/rtl8812au
git clone https://github.com/aircrack-ng/rtl8812au.git .
git checkout v5.6.4.2  # Use stable version

# Create the missing Kconfig file
cat > Kconfig << 'EOF'
config RTL8812AU
    tristate "Realtek RTL8812AU/RTL8821AU USB AC WiFi Adapter"
    depends on USB && WLAN
    select WIRELESS_EXT
    select WEXT_PRIV
    default m
    help
      This option enables support for Realtek RTL8812AU/RTL8821AU USB AC WiFi adapters.
      
      This driver supports monitor mode and packet injection for NetHunter.
      
      If you want to use this driver, say M here and it will be compiled as a module.
EOF

# Update the Makefile
cat > Makefile << 'EOF'
# Makefile for RTL8812AU driver
obj-$(CONFIG_RTL8812AU) += rtl8812au.o

# Core objects
rtl8812au-objs := \
    core/rtw_cmd.o \
    core/rtw_security.o \
    core/rtw_debug.o \
    core/rtw_io.o \
    core/rtw_ioctl_query.o \
    core/rtw_ioctl_set.o \
    core/rtw_ieee80211.o \
    core/rtw_mlme.o \
    core/rtw_mlme_ext.o \
    core/rtw_mi.o \
    core/rtw_wlan_util.o \
    core/rtw_vht.o \
    core/rtw_pwrctrl.o \
    core/rtw_rf.o \
    core/rtw_recv.o \
    core/rtw_sta_mgt.o \
    core/rtw_ap.o \
    core/rtw_xmit.o \
    core/rtw_p2p.o \
    core/rtw_tdls.o \
    core/rtw_br_ext.o \
    core/rtw_iol.o \
    core/rtw_sreset.o \
    core/rtw_btcoex.o \
    core/rtw_beamforming.o \
    core/rtw_odm.o \
    core/efuse/rtw_efuse.o

# HAL objects
rtl8812au-objs += \
    hal/hal_intf.o \
    hal/hal_com.o \
    hal/hal_com_phycfg.o \
    hal/hal_phy.o \
    hal/hal_dm.o \
    hal/hal_btcoex.o \
    hal/hal_mp.o \
    hal/hal_mcc.o \
    hal/hal_hci/hal_usb.o

# Platform objects
rtl8812au-objs += \
    os_dep/osdep_service.o \
    os_dep/linux/os_intfs.o \
    os_dep/linux/usb_intf.o \
    os_dep/linux/usb_ops.o \
    os_dep/linux/ioctl_linux.o \
    os_dep/linux/xmit_linux.o \
    os_dep/linux/mlme_linux.o \
    os_dep/linux/recv_linux.o \
    os_dep/linux/ioctl_cfg80211.o \
    os_dep/linux/rtw_cfgvendor.o \
    os_dep/linux/wifi_regd.o \
    os_dep/linux/rtw_android.o \
    os_dep/linux/rtw_proc.o \
    os_dep/linux/rtw_rhashtable.o

# Chip specific objects
rtl8812au-objs += \
    hal/rtl8812a/rtl8812a_hal_init.o \
    hal/rtl8812a/rtl8812a_phycfg.o \
    hal/rtl8812a/rtl8812a_rf6052.o \
    hal/rtl8812a/rtl8812a_dm.o \
    hal/rtl8812a/rtl8812a_rxdesc.o \
    hal/rtl8812a/rtl8812a_cmd.o \
    hal/rtl8812a/usb/usb_halinit.o \
    hal/rtl8812a/usb/rtl8812au_led.o \
    hal/rtl8812a/usb/rtl8812au_xmit.o \
    hal/rtl8812a/usb/rtl8812au_recv.o

# PHY objects
rtl8812au-objs += \
    hal/phydm/phydm_debug.o \
    hal/phydm/phydm_antdiv.o \
    hal/phydm/phydm_soml.o \
    hal/phydm/phydm_smt_ant.o \
    hal/phydm/phydm_antdect.o \
    hal/phydm/phydm_interface.o \
    hal/phydm/phydm_phystatus.o \
    hal/phydm/phydm_hwconfig.o \
    hal/phydm/phydm.o \
    hal/phydm/phydm_dig.o \
    hal/phydm/phydm_pathdiv.o \
    hal/phydm/phydm_rainfo.o \
    hal/phydm/phydm_dynamictxpower.o \
    hal/phydm/phydm_adaptivity.o \
    hal/phydm/phydm_dfs.o \
    hal/phydm/phydm_ccx.o \
    hal/phydm/phydm_psd.o \
    hal/phydm/phydm_primary_cca.o \
    hal/phydm/phydm_cfotracking.o \
    hal/phydm/phydm_adc_sampling.o \
    hal/phydm/phydm_kfree.o \
    hal/phydm/phydm_cleancenter.o \
    hal/phydm/phydm_noisemonitor.o \
    hal/phydm/phydm_beamforming.o \
    hal/phydm/phydm_direct_bf.o \
    hal/phydm/phydm_auto_dbg.o \
    hal/phydm/phydm_math_lib.o \
    hal/phydm/phydm_api.o \
    hal/phydm/phydm_pow_train.o \
    hal/phydm/phydm_lna_sat.o \
    hal/phydm/phydm_pmac_tx_setting.o \
    hal/phydm/phydm_mp.o \
    hal/phydm/phydm_pre_defined.o \
    hal/phydm/halrf/halrf.o \
    hal/phydm/halrf/halrf_debug.o \
    hal/phydm/halrf/halphyrf_ce.o \
    hal/phydm/halrf/halrf_powertracking_ce.o \
    hal/phydm/halrf/halrf_powertracking.o \
    hal/phydm/halrf/halrf_kfree.o \
    hal/phydm/halrf/halrf_psd.o \
    hal/phydm/rtl8812a/halhwimg8812a_bb.o \
    hal/phydm/rtl8812a/halhwimg8812a_mac.o \
    hal/phydm/rtl8812a/halhwimg8812a_rf.o \
    hal/phydm/rtl8812a/phydm_regconfig8812a.o \
    hal/phydm/rtl8812a/phydm_rtl8812a.o \
    hal/phydm/rtl8812a/halrf/halrf_8812a_ce.o

ccflags-y += -DCONFIG_RTL8812AU=1
ccflags-y += -DCONFIG_IOCTL_CFG80211=1
ccflags-y += -DCONFIG_CONCURRENT_MODE=1
ccflags-y += -DCONFIG_PLATFORM_I386_PC=1
ccflags-y += -DCONFIG_WIFI_MONITOR=1
ccflags-y += -DCONFIG_WIFI_INJECT_FRAME=1
ccflags-y += -DRTW_USE_CFG80211_STA_EVENT=1
ccflags-y += -Wno-unused-variable
ccflags-y += -Wno-unused-value
ccflags-y += -Wno-unused-label
ccflags-y += -Wno-unused-parameter
ccflags-y += -Wno-unused-function
ccflags-y += -Wno-unused
ccflags-y += -Wno-uninitialized
ccflags-y += -I$(src)/include
EOF

# Go back to kernel root
cd ~/nethunter-android15/kernel_lavender
```

### 4. Alternative: Quick Fix (Temporary)

If you want to quickly bypass the issue temporarily:

```bash
cd ~/nethunter-android15/kernel_lavender

# Comment out the problematic line temporarily
sed -i 's/source "drivers\/aircrack\/rtl8812au\/Kconfig"/# source "drivers\/aircrack\/rtl8812au\/Kconfig"/' drivers/Kconfig
sed -i 's/obj-y.*+= aircrack\/rtl8812au\//# obj-y += aircrack\/rtl8812au\//' drivers/Makefile

# Now try making the defconfig
make lavender_defconfig
```

## 🔨 Build Process

### 1. Apply NetHunter Patches (Proper Method)

```bash
cd ~/nethunter-android15/kali-nethunter-kernel-builder

# Initialize build environment
./bootstrap.sh

# Apply patches for kernel 4.19.321
./build.sh --patch-only --device lavender --kernel-version 4.19
```

### 2. Build Using NetHunter Kernel Builder

```bash
# Build NetHunter kernel for Android 15
./build.sh --device lavender --kernel-only --android-version 15
```

### 3. Manual Build Alternative

```bash
cd ~/nethunter-android15/kernel_lavender

# Set environment
export ARCH=arm64
export SUBARCH=arm64
export CROSS_COMPILE=aarch64-linux-android35-
export CC=clang
export CXX=clang++
export CLANG_TRIPLE=aarch64-linux-gnu-

# Generate configuration
make lavender_defconfig

# Build kernel
make -j$(nproc) \
    ARCH=arm64 \
    SUBARCH=arm64 \
    CROSS_COMPILE=aarch64-linux-android35- \
    CC=clang \
    CXX=clang++ \
    CLANG_TRIPLE=aarch64-linux-gnu- \
    LLVM=1 \
    LLVM_IAS=1 \
    Image.gz-dtb modules
```

### 4. Create Flashable Package

```bash
cd ~/nethunter-android15/kali-nethunter-kernel-builder

# Build complete NetHunter package for Android 15
./build.sh --device lavender --full --android-version 15

# Package will be in releases/
ls -la releases/nethunter-*-lavender-android15*.zip
```

## 🚀 Installation (Android 15)

### 1. Device Preparation

```bash
# Enable Developer Options
# Enable USB Debugging
# Enable OEM Unlocking

# Boot to fastboot
adb reboot bootloader

# Unlock bootloader (if not already unlocked)
fastboot flashing unlock
```

### 2. Backup Current Kernel (CRITICAL STEP)

**Method 1: Using dd command (Recommended)**

```bash
# Boot to TWRP recovery first
adb reboot recovery

# Create backup directory
adb shell mkdir -p /sdcard/kernel_backup

# Backup boot partition (contains kernel)
adb shell dd if=/dev/block/bootdevice/by-name/boot of=/sdcard/kernel_backup/boot_stock.img bs=4096

# Backup recovery partition (safety backup)
adb shell dd if=/dev/block/bootdevice/by-name/recovery of=/sdcard/kernel_backup/recovery_stock.img bs=4096

# Backup dtbo partition (Device Tree Blob Overlay)
adb shell dd if=/dev/block/bootdevice/by-name/dtbo of=/sdcard/kernel_backup/dtbo_stock.img bs=4096

# Copy backups to PC
adb pull /sdcard/kernel_backup/ ./kernel_backup/

# Verify backup integrity
ls -la ./kernel_backup/
# Should show: boot_stock.img, recovery_stock.img, dtbo_stock.img
```

**Method 2: Using TWRP Built-in Backup**

In TWRP Recovery:
1. Tap "Backup"
2. Select "Boot" partition
3. Select "Recovery" partition  
4. Select "System" partition (optional but recommended)
5. Swipe to backup
6. Wait for completion

```bash
# Copy TWRP backup to PC
adb pull /sdcard/TWRP/BACKUPS/ ./twrp_backup/
```

### 3. Custom Recovery (TWRP)

```bash
# Download latest TWRP for lavender
wget https://dl.twrp.me/lavender/twrp-3.7.0_12-0-lavender.img

# Flash TWRP
fastboot flash recovery twrp-3.7.0_12-0-lavender.img
fastboot boot twrp-3.7.0_12-0-lavender.img

# IMPORTANT: Perform kernel backup BEFORE proceeding to NetHunter installation
```

### 4. Install NetHunter

```bash
# VERIFY BACKUP COMPLETED SUCCESSFULLY BEFORE PROCEEDING
ls -la kernel_backup/
# Should show: boot_stock.img, recovery_stock.img, dtbo_stock.img

# Copy NetHunter package to device
adb push releases/nethunter-*-lavender-android15*.zip /sdcard/
```

In TWRP:
1. Wipe > Advanced Wipe > Dalvik/ART Cache, Cache
2. Install > Select NetHunter ZIP
3. Flash and reboot system
4. If bootloop occurs, use emergency recovery methods

### 5. Install Magisk (Required for Android 15)

```bash
# Download latest Magisk
wget https://github.com/topjohnwu/Magisk/releases/latest/download/Magisk-v27.0.apk

# Install Magisk
adb install Magisk-v27.0.apk

# Install NetHunter App
wget https://kali.org/get-kali/kali-nethunter/images/nethunter-app.apk
adb install nethunter-app.apk
```

## 🔍 Verification

### 1. Check Kernel

```bash
# Verify NetHunter kernel is loaded
adb shell cat /proc/version
# Should show: Linux version 4.19.321-NetHunter-A15

# Check NetHunter modules
adb shell lsmod | grep -i nethunter
```

### 2. Test NetHunter Features

```bash
# Check HID support
adb shell ls /dev/hidg*

# Check wireless injection
adb shell iwconfig
adb shell iw dev

# Test in NetHunter App
# - HID attacks should be available
# - Wireless tools should work
# - Chroot should install properly
```

## 🐛 Troubleshooting (Android 15 Specific)

### Emergency Recovery Procedures

**1. Bootloop After NetHunter Installation**

Immediate Steps:
1. Force reboot: Hold Power + Volume Up for 10 seconds
2. Boot to TWRP: Hold Volume Up + Power during boot
3. Flash stock kernel restore ZIP
4. Or restore TWRP backup

If TWRP is inaccessible:
1. Boot to fastboot: Hold Volume Down + Power
2. `fastboot flash boot kernel_backup/boot_stock.img`
3. `fastboot reboot`

**2. Kernel Panic or Boot Issues**

```bash
# Check boot logs in TWRP
adb shell dmesg | grep -i "panic\|error\|fail"

# Restore stock kernel
fastboot flash boot kernel_backup/boot_stock.img
fastboot flash dtbo kernel_backup/dtbo_stock.img
fastboot reboot
```

**3. Recovery Mode Issues**

```bash
# Re-flash TWRP if corrupted
fastboot flash recovery twrp-3.7.0_12-0-lavender.img
fastboot boot twrp-3.7.0_12-0-lavender.img
```

### Common Issues

**1. SELinux Denials**

```bash
# Set SELinux to permissive (temporary)
adb shell su -c "setenforce 0"

# Check denials
adb shell dmesg | grep -i selinux
```

**2. Magisk Module Issues**

Reinstall Magisk if NetHunter doesn't work - Flash Magisk.zip in TWRP after NetHunter installation

**3. Build Errors**

```bash
# Clean build
cd ~/nethunter-android15/kernel_lavender
make clean
make mrproper

# Rebuild with verbose output
make -j$(nproc) V=1
```

**4. Android 15 Compatibility**

```bash
# Check for Android 15 specific patches
grep -r "ANDROID_VERSION" kernel_lavender/
grep -r "API_LEVEL" kernel_lavender/
```

## 📱 Android 15 Chroot Setup

### 1. Download ARM64 Chroot

```bash
# Download minimal ARM64 chroot for Android 15
wget https://kali.org/get-kali/kali-nethunter/images/kalifs-arm64-minimal.tar.xz

# Copy to device
adb push kalifs-arm64-minimal.tar.xz /sdcard/
```

### 2. Install via NetHunter App

1. Open NetHunter App
2. Navigate to Kali Chroot Manager
3. Select "Install Chroot"
4. Choose downloaded chroot file
5. Wait for installation to complete

## 🔧 Advanced Configuration

### 1. Custom Kernel Modules

```bash
# Build additional wireless modules
cd ~/nethunter-android15/kernel_lavender
make -j$(nproc) \
    ARCH=arm64 \
    CROSS_COMPILE=aarch64-linux-android35- \
    CC=clang \
    M=drivers/net/wireless/rtl8188eu \
    modules

# Install modules
make -j$(nproc) \
    ARCH=arm64 \
    CROSS_COMPILE=aarch64-linux-android35- \
    CC=clang \
    M=drivers/net/wireless/rtl8188eu \
    INSTALL_MOD_PATH=./modules \
    modules_install
```

### 2. Performance Optimization

```bash
# Enable performance governor
echo "performance" > /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor

# Set CPU frequency
echo "1804800" > /sys/devices/system/cpu/cpu0/cpufreq/scaling_max_freq
```

## 📚 Resources

### Official Documentation
- [Kali NetHunter Docs](https://www.kali.org/docs/nethunter/)
- [Android 15 Developer Guide](https://developer.android.com/about/versions/15)
- [NetHunter GitLab Project](https://gitlab.com/kalilinux/nethunter)
- [NetHunter Kernel Builder](https://gitlab.com/kalilinux/nethunter/build-scripts/kali-nethunter-kernel)

### Community Resources
- [Team-420 NetHunter Kernel](https://github.com/Team-420/android_kernel_xiaomi_lavender)
- [XDA Developers](https://forum.xda-developers.com/c/xiaomi-redmi-note-7.8434/)
- [Kali Forums](https://forums.kali.org/forumdisplay.php?30-Kali-NetHunter)

## ⚠️ Disclaimer

- **Android 15 Compatibility**: This guide is for Android 15 - older guides may not work
- **Security**: Android 15 has enhanced security features that may affect NetHunter
- **Warranty**: This process voids your warranty
- **Risk**: Custom kernels can brick your device
- **Legal**: Use NetHunter only for authorized penetration testing

## 🤝 Contributing

1. Fork the repository
2. Test on Android 15
3. Submit pull request with Android 15 compatibility notes
4. Update documentation for current versions

## 📄 License

- **Linux Kernel**: GPL v2
- **NetHunter Patches**: GPL v2
- **Build Scripts**: GPL v3

---

**Updated for Android 15 compatibility**  
**Created by 0xb0rn3 | 0xbv1**  
**Last Updated**: July 2025  
**Device Tested**: Xiaomi Redmi Note 7 (lavender) - Android 15 - Kernel 4.19.321-S0NiX
