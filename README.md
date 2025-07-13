# Complete NetHunter Kernel Build Guide - Xiaomi Redmi Note 7 (Lavender)

A comprehensive, step-by-step guide to building NetHunter kernel for Android 15 on Xiaomi Redmi Note 7.

**By 0xb0rn3 | 0xbv1** - Updated for Android 15 compatibility

## 📋 Prerequisites

### System Requirements
- **OS**: Ubuntu 22.04 LTS or newer (recommended)
- **RAM**: 16GB minimum (32GB recommended)
- **Storage**: 100GB+ free space
- **CPU**: Multi-core processor (build time: 2-4 hours)

### Device Requirements
- **Device**: Xiaomi Redmi Note 7 (lavender)
- **Bootloader**: Unlocked
- **Current Android**: 15 (API Level 35)
- **Current Kernel**: 4.19.321-S0NiX
- **Root Access**: Required (Magisk recommended)

## 🛠️ Phase 1: Environment Setup

### Step 1: Install Build Dependencies

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install essential build tools
sudo apt install -y \
    build-essential git python3 python3-pip bc curl wget \
    libssl-dev libncurses-dev libelf-dev bison flex \
    make cmake ninja-build zip unzip rsync \
    clang-19 llvm-19 lld-19 adb fastboot \
    openjdk-21-jdk python3-distutils python3-setuptools

# Set clang as default compiler
sudo update-alternatives --install /usr/bin/clang clang /usr/bin/clang-19 100
sudo update-alternatives --install /usr/bin/clang++ clang++ /usr/bin/clang++-19 100

# Verify installation
clang --version
java --version
```

### Step 2: Create Build Directory

```bash
# Create main build directory
mkdir -p ~/nethunter-build
cd ~/nethunter-build

# Set up environment variables
export NETHUNTER_BUILD_HOME=~/nethunter-build
export ARCH=arm64
export SUBARCH=arm64
export CROSS_COMPILE=aarch64-linux-android-
export CC=clang
export CXX=clang++
```

### Step 3: Download Android NDK

```bash
# Download Android NDK r26d (compatible with Android 15)
cd ~/nethunter-build
wget https://dl.google.com/android/repository/android-ndk-r26d-linux.zip
unzip android-ndk-r26d-linux.zip

# Set NDK environment
export ANDROID_NDK_HOME=~/nethunter-build/android-ndk-r26d
export PATH=$ANDROID_NDK_HOME/toolchains/llvm/prebuilt/linux-x86_64/bin:$PATH
```

## 🔧 Phase 2: Source Code Preparation

### Step 4: Clone NetHunter Build Scripts

```bash
cd ~/nethunter-build

# Clone the official NetHunter kernel builder
git clone https://gitlab.com/kalilinux/nethunter/build-scripts/kali-nethunter-kernel.git
cd kali-nethunter-kernel

# Make scripts executable
chmod +x build.sh
chmod +x bootstrap.sh 2>/dev/null || echo "bootstrap.sh not found - will use build.sh directly"
```

### Step 5: Clone Kernel Source

```bash
cd ~/nethunter-build

# Clone kernel source for Xiaomi Redmi Note 7
git clone https://github.com/Team-420/android_kernel_xiaomi_lavender.git -b Nethunter kernel_lavender

# Alternative sources (if Team-420 doesn't work):
# git clone https://github.com/ravi0900/android_kernel_xiaomi_lavender.git kernel_lavender
# git clone https://github.com/xiaomi-lavender-devs/android_kernel_xiaomi_lavender.git kernel_lavender
```

### Step 6: Prepare Kernel Directory Structure

```bash
cd ~/nethunter-build/kernel_lavender

# Create missing directories that the build script expects
mkdir -p arch/arm64/configs
mkdir -p kernel-out
mkdir -p modules-out

# Check if lavender_defconfig exists
if [ ! -f "arch/arm64/configs/lavender_defconfig" ]; then
    echo "ERROR: lavender_defconfig not found!"
    echo "Available configs:"
    ls -la arch/arm64/configs/
    echo "You may need to copy from a different location or use a different kernel source"
fi
```

## 🎯 Phase 3: NetHunter Configuration

### Step 7: Create NetHunter Configuration

```bash
cd ~/nethunter-build/kali-nethunter-kernel

# Create local.config file
cat > local.config << 'EOF'
# NetHunter Build Configuration for Xiaomi Redmi Note 7
DEVICE_NAME="lavender"
KERNEL_SOURCE="~/nethunter-build/kernel_lavender"
KERNEL_ARCH="arm64"
KERNEL_SUBARCH="arm64"
KERNEL_DEFCONFIG="lavender_defconfig"
CROSS_COMPILE="aarch64-linux-android-"
CLANG_TRIPLE="aarch64-linux-gnu-"
ANDROID_MAJOR_VERSION="15"
ANDROID_MINOR_VERSION="0"
KERNEL_LOCALVERSION="-NetHunter"
THREADS=$(nproc)
CLANG_VERSION="19"

# Build options
ANYKERNEL_BRANCH="master"
KERNEL_BRANCH="Nethunter"
USE_CCACHE=false
BUILD_MODULES=true
MODULES_COMPRESSION="xz"

# NetHunter features
ENABLE_WIFI_INJECTION=true
ENABLE_USB_HID=true
ENABLE_BLUETOOTH_INJECTION=true
ENABLE_RTL8188EU=true
ENABLE_RTL8812AU=true
ENABLE_SELINUX_PERMISSIVE=true
EOF

echo "Configuration created successfully!"
```

### Step 8: Initialize NetHunter Builder

```bash
cd ~/nethunter-build/kali-nethunter-kernel

# Run the build script in interactive mode
./build.sh

# When prompted, you'll see a menu like this:
# 1) anykernel3/tools/magiskpolicy
# 2) anykernel3/tools/busybox
# ... etc ...
# 144) Cancel

# Select option 39 (local.config.example) or similar
# This will create the nethunter_defconfig
```

**Important**: When you see the configuration menu:
1. Look for `local.config.example` (usually around option 39)
2. Select that number and press Enter
3. This will create the base configuration

### Step 9: Configure for Your Device

```bash
# After selecting the base config, the script will continue
# If it asks for device-specific settings, choose:
# - Architecture: arm64
# - Device: lavender
# - Kernel version: 4.19

# The script should now create nethunter_defconfig
ls -la arch/arm64/configs/
# Should show: lavender_defconfig and nethunter_defconfig
```

## 🔨 Phase 4: Build Process

### Step 10: Apply NetHunter Patches

```bash
cd ~/nethunter-build/kali-nethunter-kernel

# Apply patches for kernel 4.19
./build.sh --patch-only --device lavender --kernel-version 4.19

# If this fails, try manual patching:
# cd ~/nethunter-build/kernel_lavender
# Apply patches from patches/4.19/ directory manually
```

### Step 11: Build the Kernel

```bash
cd ~/nethunter-build/kali-nethunter-kernel

# Build NetHunter kernel
./build.sh --device lavender --kernel-only

# Alternative: Full build (kernel + modules + anykernel package)
./build.sh --device lavender --full

# Monitor build progress
# Build time: 1-4 hours depending on your system
```

### Step 12: Manual Build (If Automated Build Fails)

```bash
cd ~/nethunter-build/kernel_lavender

# Set environment
export ARCH=arm64
export SUBARCH=arm64
export CROSS_COMPILE=aarch64-linux-android-
export CC=clang
export CXX=clang++

# Clean previous builds
make clean
make mrproper

# Create defconfig
make lavender_defconfig

# Enable NetHunter features manually
make menuconfig
# Navigate to:
# - Device Drivers → Network device support → Wireless LAN → Enable RTL8188EU
# - Device Drivers → USB support → Enable USB HID
# - Security options → Enable permissive SELinux

# Build kernel and modules
make -j$(nproc) Image.gz-dtb modules

# Install modules
make -j$(nproc) INSTALL_MOD_PATH=./modules modules_install
```

### Step 13: Create Flashable Package

```bash
cd ~/nethunter-build/kali-nethunter-kernel

# If automated build completed successfully:
ls -la releases/
# Should show: nethunter-*-lavender-*.zip

# If manual build was used, package with AnyKernel3:
cd ~/nethunter-build
git clone https://github.com/osm0sis/AnyKernel3.git
cd AnyKernel3

# Copy kernel
cp ~/nethunter-build/kernel_lavender/arch/arm64/boot/Image.gz-dtb ./

# Create flashable zip
zip -r9 nethunter-lavender-$(date +%Y%m%d).zip . -x .git README.md .gitignore
```

## 📱 Phase 5: Installation

### Step 14: Backup Current System

**⚠️ CRITICAL: Always backup before flashing custom kernels!**

```bash
# Boot to TWRP recovery
adb reboot recovery

# Create backup directory
adb shell mkdir -p /sdcard/kernel_backup

# Backup boot partition (contains kernel)
adb shell dd if=/dev/block/bootdevice/by-name/boot of=/sdcard/kernel_backup/boot_stock.img

# Backup recovery partition
adb shell dd if=/dev/block/bootdevice/by-name/recovery of=/sdcard/kernel_backup/recovery_stock.img

# Backup dtbo partition
adb shell dd if=/dev/block/bootdevice/by-name/dtbo of=/sdcard/kernel_backup/dtbo_stock.img

# Copy backups to PC
adb pull /sdcard/kernel_backup/ ./kernel_backup/

# Verify backups
ls -la ./kernel_backup/
# Should show: boot_stock.img, recovery_stock.img, dtbo_stock.img
```

### Step 15: Install Custom Recovery (If Needed)

```bash
# Download TWRP for lavender
wget https://dl.twrp.me/lavender/twrp-3.7.0_12-0-lavender.img

# Flash TWRP
adb reboot bootloader
fastboot flash recovery twrp-3.7.0_12-0-lavender.img
fastboot boot twrp-3.7.0_12-0-lavender.img
```

### Step 16: Flash NetHunter Kernel

```bash
# Copy NetHunter package to device
adb push nethunter-lavender-*.zip /sdcard/

# In TWRP:
# 1. Wipe → Advanced Wipe → Dalvik/ART Cache, Cache
# 2. Install → Select NetHunter ZIP
# 3. Swipe to flash
# 4. Reboot System
```

### Step 17: Install Magisk and NetHunter App

```bash
# Download and install Magisk
wget https://github.com/topjohnwu/Magisk/releases/latest/download/Magisk-v27.0.apk
adb install Magisk-v27.0.apk

# Download and install NetHunter App
wget https://store.nethunter.com/packages/com.offsec.nethunter.apk
adb install com.offsec.nethunter.apk
```

## 🔍 Phase 6: Verification and Testing

### Step 18: Verify Installation

```bash
# Check kernel version
adb shell cat /proc/version
# Should show: Linux version 4.19.321-NetHunter

# Check NetHunter modules
adb shell lsmod | grep -i nethunter

# Check HID support
adb shell ls /dev/hidg*

# Check wireless capabilities
adb shell iwconfig
adb shell iw list
```

### Step 19: Test NetHunter Features

1. **Open NetHunter App**
2. **Test HID Attacks**: 
   - Go to HID Attacks
   - Try PowerSploit or Windows CMD
3. **Test Wireless Injection**:
   - Connect USB wireless adapter
   - Check if monitor mode works
4. **Install Kali Chroot**:
   - Download ARM64 chroot
   - Install via NetHunter App

## 🚨 Troubleshooting

### Common Issues and Solutions

**Issue 1: Build fails with "configs directory not found"**
```bash
# Solution: Create missing directories
mkdir -p arch/arm64/configs
# Copy defconfig from device or use generic one
```

**Issue 2: Bootloop after flashing**
```bash
# Solution: Restore stock kernel
fastboot flash boot kernel_backup/boot_stock.img
fastboot reboot
```

**Issue 3: NetHunter features not working**
```bash
# Solution: Check kernel config
zcat /proc/config.gz | grep -i CONFIG_USB_HID
# Should show: CONFIG_USB_HID=y
```

**Issue 4: Wireless injection not working**
```bash
# Solution: Load modules manually
adb shell su -c "modprobe rtl8188eu"
adb shell su -c "modprobe rtl8812au"
```

### Emergency Recovery

**If device won't boot:**
1. Boot to fastboot: Hold Volume Down + Power
2. Flash stock kernel: `fastboot flash boot kernel_backup/boot_stock.img`
3. Reboot: `fastboot reboot`

**If TWRP is broken:**
1. Boot to fastboot
2. Flash stock recovery: `fastboot flash recovery kernel_backup/recovery_stock.img`
3. Or reflash TWRP: `fastboot flash recovery twrp-3.7.0_12-0-lavender.img`

## 📚 Additional Resources

### Documentation
- [Official NetHunter Documentation](https://www.kali.org/docs/nethunter/)
- [NetHunter GitLab Repository](https://gitlab.com/kalilinux/nethunter)
- [Android Kernel Build Guide](https://source.android.com/docs/setup/build/building-kernels)

### Community Support
- [Kali Forums - NetHunter](https://forums.kali.org/forumdisplay.php?30-Kali-NetHunter)
- [XDA Developers - Redmi Note 7](https://forum.xda-developers.com/c/xiaomi-redmi-note-7.8434/)
- [Reddit - r/Kalilinux](https://www.reddit.com/r/Kalilinux/)

### Useful Commands

```bash
# Check kernel config
zcat /proc/config.gz | grep -i CONFIG_

# List loaded modules
lsmod

# Check device info
getprop ro.product.device
getprop ro.build.version.release

# Monitor kernel messages
dmesg | tail -f

# Check wireless interfaces
iwconfig
iw dev
```

## ⚠️ Important Notes

### Legal Disclaimer
- Use NetHunter only for authorized penetration testing
- Respect local laws and regulations
- Only test on systems you own or have explicit permission to test

### Warranty and Risk
- This process voids your device warranty
- Custom kernels can permanently damage your device
- Always maintain current backups
- The authors are not responsible for device damage

### Android 15 Compatibility
- This guide is specifically for Android 15
- Older guides may not work with current Android versions
- Some features may be limited due to Android security enhancements

## 🤝 Contributing

Found an issue or improvement? Contributions are welcome!

1. Test the guide on your device
2. Document any issues or improvements
3. Submit pull requests with detailed descriptions
4. Help others in the community forums

---

**Guide Version**: 2.0  
**Last Updated**: July 2025  
**Target Device**: Xiaomi Redmi Note 7 (lavender)  
**Target Android**: 15 (API 35)  
**Kernel Version**: 4.19.321  
**Created by**: 0xb0rn3 | 0xbv1
