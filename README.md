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

# Find available defconfigs
echo "Searching for available defconfigs..."
find . -name "*defconfig*" -type f
find . -name "*lavender*" -type f

# Check if lavender_defconfig exists
if [ ! -f "arch/arm64/configs/lavender_defconfig" ]; then
    echo "WARNING: lavender_defconfig not found!"
    echo "Available configs:"
    ls -la arch/arm64/configs/ 2>/dev/null || echo "No configs directory found"
    
    # Look for defconfig in other locations
    echo "Searching for defconfig in other locations..."
    find . -name "*defconfig*" -type f
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

### Step 8: Fix Common Build Script Issues

**🚨 CRITICAL: Common build script errors and their fixes**

```bash
cd ~/nethunter-build/kali-nethunter-kernel

# Fix 1: Missing configs directory error
# If you get: "cd: /home/oxbv1/nethunter-build/arch/arm64/configs: No such file or directory"
cd ~/nethunter-build/kernel_lavender
mkdir -p arch/arm64/configs

# Fix 2: Missing nethunter_defconfig
# The build script looks for this file but it doesn't exist initially
# You need to create it based on your device's base defconfig

# First, find your device's base defconfig
echo "Looking for device defconfig..."
find . -name "*lavender*defconfig" -type f
find . -name "*defconfig*" -type f | head -10

# If you find lavender_defconfig, copy it
if [ -f "arch/arm64/configs/lavender_defconfig" ]; then
    cp arch/arm64/configs/lavender_defconfig arch/arm64/configs/nethunter_defconfig
    echo "Created nethunter_defconfig from lavender_defconfig"
elif [ -f "arch/arm64/configs/lavender-perf_defconfig" ]; then
    cp arch/arm64/configs/lavender-perf_defconfig arch/arm64/configs/nethunter_defconfig
    echo "Created nethunter_defconfig from lavender-perf_defconfig"
else
    echo "ERROR: No lavender defconfig found!"
    echo "Available configs:"
    ls -la arch/arm64/configs/ 2>/dev/null || echo "No configs directory"
    echo "You may need to use a different kernel source or create the defconfig manually"
fi
```

### Step 9: Initialize NetHunter Builder

```bash
cd ~/nethunter-build/kali-nethunter-kernel

# Run the build script in interactive mode
./build.sh

# When prompted with the menu, select one of these options:
# Option 35: local.config.examples/local.config.vince (similar device)
# Option 39: local.config.examples/local.config.example (generic example)
# Option 42: local.config.examples/local.config.allinone (comprehensive config)

# Recommended: Select option 39 (local.config.example)
```

**Menu Navigation Guide:**
- When you see the numbered menu (1-144), type `39` and press Enter
- This will use the generic example configuration as a base
- The script will then create the necessary files

### Step 10: Handle Interactive Configuration

```bash
# After selecting option 39, the script may ask additional questions:

# 1. Device name: Enter "lavender"
# 2. Kernel architecture: Enter "arm64"
# 3. Kernel version: Enter "4.19" (based on your current kernel)
# 4. Cross compiler: Enter "aarch64-linux-android-" or leave default

# The script should now create:
# - nethunter_defconfig in your kernel source
# - Updated local.config with your device settings
```

## 🔨 Phase 4: Build Process

### Step 11: Apply NetHunter Patches

```bash
cd ~/nethunter-build/kali-nethunter-kernel

# Apply patches for kernel 4.19
./build.sh --patch-only --device lavender --kernel-version 4.19

# If automatic patching fails, apply manually:
if [ $? -ne 0 ]; then
    echo "Automatic patching failed, applying manually..."
    cd ~/nethunter-build/kernel_lavender
    
    # Apply patches from the patches directory
    patch -p1 < ~/nethunter-build/kali-nethunter-kernel/patches/4.19/add-wifi-injection-4.19.patch
    patch -p1 < ~/nethunter-build/kali-nethunter-kernel/patches/4.19/add-rtl8188eus-to-rtl8xxxu-drivers.patch
    patch -p1 < ~/nethunter-build/kali-nethunter-kernel/patches/4.19/add-rtl88xxau-5.6.4.2-drivers.patch
    
    # Check if patches applied successfully
    echo "Manual patching completed"
fi
```

### Step 12: Build the Kernel

```bash
cd ~/nethunter-build/kali-nethunter-kernel

# Method 1: Automated build (recommended)
./build.sh --device lavender --kernel-only

# Method 2: Full build (kernel + modules + anykernel package)
./build.sh --device lavender --full

# Method 3: If automated build fails, use manual build (see Step 13)
```

### Step 13: Manual Build (Fallback Method)

```bash
cd ~/nethunter-build/kernel_lavender

# Set environment variables
export ARCH=arm64
export SUBARCH=arm64
export CROSS_COMPILE=aarch64-linux-android-
export CC=clang
export CXX=clang++
export CLANG_TRIPLE=aarch64-linux-gnu-

# Clean previous builds
make clean
make mrproper

# Create defconfig
if [ -f "arch/arm64/configs/lavender_defconfig" ]; then
    make lavender_defconfig
elif [ -f "arch/arm64/configs/lavender-perf_defconfig" ]; then
    make lavender-perf_defconfig
else
    echo "ERROR: No suitable defconfig found!"
    exit 1
fi

# Enable NetHunter features manually
make menuconfig
# Navigate to and enable:
# - Device Drivers → Network device support → Wireless LAN → Enable RTL8188EU
# - Device Drivers → USB support → Enable USB HID
# - Security options → Enable permissive SELinux
# - Networking support → Wireless → Enable monitor mode support

# Build kernel and modules
make -j$(nproc) Image.gz-dtb modules

# Check if build was successful
if [ $? -eq 0 ]; then
    echo "Kernel build successful!"
    ls -la arch/arm64/boot/Image.gz-dtb
else
    echo "Kernel build failed!"
    exit 1
fi

# Install modules
make -j$(nproc) INSTALL_MOD_PATH=./modules modules_install
```

### Step 14: Create Flashable Package

```bash
cd ~/nethunter-build/kali-nethunter-kernel

# If automated build completed successfully:
ls -la releases/
# Should show: nethunter-*-lavender-*.zip

# If manual build was used, package with AnyKernel3:
if [ ! -f "releases/nethunter-*-lavender-*.zip" ]; then
    echo "Creating flashable package manually..."
    
    cd ~/nethunter-build
    git clone https://github.com/osm0sis/AnyKernel3.git
    cd AnyKernel3
    
    # Configure AnyKernel3 for lavender
    cat > anykernel.sh << 'EOF'
# AnyKernel3 Ramdisk Mod Script
# osm0sis @ xda-developers

## AnyKernel setup
# begin properties
properties() { '
kernel.string=NetHunter Kernel for Redmi Note 7 (lavender)
do.devicecheck=1
do.modules=1
do.systemless=1
do.cleanup=1
do.cleanuponabort=0
device.name1=lavender
device.name2=Redmi Note 7
device.name3=
device.name4=
device.name5=
supported.versions=
supported.patchlevels=
'; } # end properties

# shell variables
block=/dev/block/bootdevice/by-name/boot;
is_slot_device=0;
ramdisk_compression=auto;

## AnyKernel methods (DO NOT CHANGE)
# import patching functions/variables - see for reference
. tools/ak3-core.sh;

## AnyKernel file attributes
# set permissions/ownership for included ramdisk files
set_perm_recursive 0 0 755 644 $ramdisk/*;
set_perm_recursive 0 0 750 750 $ramdisk/init* $ramdisk/sbin;

## AnyKernel boot install
dump_boot;
write_boot;
EOF
    
    # Copy kernel
    cp ~/nethunter-build/kernel_lavender/arch/arm64/boot/Image.gz-dtb ./
    
    # Copy modules if they exist
    if [ -d "~/nethunter-build/kernel_lavender/modules" ]; then
        cp -r ~/nethunter-build/kernel_lavender/modules ./
    fi
    
    # Create flashable zip
    zip -r9 nethunter-lavender-$(date +%Y%m%d).zip . -x .git README.md .gitignore
    
    echo "Flashable package created: nethunter-lavender-$(date +%Y%m%d).zip"
fi
```

## 📱 Phase 5: Installation

### Step 15: Backup Current System

**⚠️ CRITICAL: Always backup before flashing custom kernels!**

```bash
# Boot to TWRP recovery
adb reboot recovery

# Wait for device to boot into recovery
sleep 30

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

# Verify backup integrity
if [ -f "./kernel_backup/boot_stock.img" ] && [ -s "./kernel_backup/boot_stock.img" ]; then
    echo "✅ Boot backup successful"
else
    echo "❌ Boot backup failed - DO NOT PROCEED!"
    exit 1
fi
```

### Step 16: Install Custom Recovery (If Needed)

```bash
# Download TWRP for lavender
wget https://dl.twrp.me/lavender/twrp-3.7.0_12-0-lavender.img

# Verify TWRP download
if [ -f "twrp-3.7.0_12-0-lavender.img" ]; then
    echo "TWRP downloaded successfully"
else
    echo "TWRP download failed"
    exit 1
fi

# Flash TWRP
adb reboot bootloader
sleep 10
fastboot flash recovery twrp-3.7.0_12-0-lavender.img
fastboot boot twrp-3.7.0_12-0-lavender.img
```

### Step 17: Flash NetHunter Kernel

```bash
# Find the NetHunter package
NETHUNTER_ZIP=$(find ~/nethunter-build -name "nethunter-*-lavender-*.zip" | head -1)

if [ -z "$NETHUNTER_ZIP" ]; then
    echo "NetHunter package not found!"
    exit 1
fi

echo "Found NetHunter package: $NETHUNTER_ZIP"

# Copy NetHunter package to device
adb push "$NETHUNTER_ZIP" /sdcard/

# Verify file was copied
adb shell ls -la /sdcard/nethunter-*.zip

echo "📱 Now in TWRP, perform the following steps:"
echo "1. Wipe → Advanced Wipe → Select Dalvik/ART Cache and Cache"
echo "2. Swipe to Wipe"
echo "3. Back to main menu"
echo "4. Install → Select the nethunter-*.zip file"
echo "5. Swipe to confirm flash"
echo "6. Reboot System"
echo ""
echo "⚠️ If boot fails, restore backup:"
echo "   - Boot to fastboot mode"
echo "   - fastboot flash boot kernel_backup/boot_stock.img"
echo "   - fastboot reboot"
```

### Step 18: Install Magisk and NetHunter App

```bash
# Download and install Magisk
wget https://github.com/topjohnwu/Magisk/releases/latest/download/Magisk-v27.0.apk
adb install Magisk-v27.0.apk

# Download and install NetHunter App
wget https://store.nethunter.com/packages/com.offsec.nethunter.apk
adb install com.offsec.nethunter.apk

# Verify installations
adb shell pm list packages | grep -E "(magisk|nethunter)"
```

## 🔍 Phase 6: Verification and Testing

### Step 19: Verify Installation

```bash
# Check kernel version
echo "Checking kernel version..."
adb shell cat /proc/version
# Should show: Linux version 4.19.xxx-NetHunter

# Check NetHunter modules
echo "Checking NetHunter modules..."
adb shell lsmod | grep -i nethunter

# Check HID support
echo "Checking HID support..."
adb shell ls /dev/hidg*

# Check wireless capabilities
echo "Checking wireless capabilities..."
adb shell iwconfig 2>/dev/null || echo "iwconfig not available"
adb shell iw list 2>/dev/null || echo "iw not available"

# Check SELinux status
echo "Checking SELinux status..."
adb shell getenforce
# Should show: Permissive (for NetHunter to work properly)

# Check if NetHunter terminal is working
echo "Testing NetHunter terminal..."
adb shell su -c "echo 'NetHunter kernel test successful'"
```

### Step 20: Test NetHunter Features

```bash
# Test script to verify NetHunter functionality
echo "🧪 Testing NetHunter Features..."

# Test 1: HID Attack capability
echo "Testing HID attacks..."
adb shell su -c "ls /dev/hidg*" && echo "✅ HID support available" || echo "❌ HID support missing"

# Test 2: Monitor mode capability
echo "Testing monitor mode..."
adb shell su -c "iw dev" && echo "✅ Wireless tools available" || echo "❌ Wireless tools missing"

# Test 3: Packet injection
echo "Testing packet injection capability..."
adb shell su -c "which aircrack-ng" && echo "✅ Aircrack-ng available" || echo "❌ Aircrack-ng missing"

# Test 4: NetHunter app permissions
echo "Testing NetHunter app..."
adb shell su -c "pm list packages | grep nethunter" && echo "✅ NetHunter app installed" || echo "❌ NetHunter app missing"

echo "🔧 Manual testing required:"
echo "1. Open NetHunter App"
echo "2. Go to HID Attacks → Test PowerSploit or Windows CMD"
echo "3. Connect USB wireless adapter → Check monitor mode"
echo "4. Install Kali Chroot → Download ARM64 chroot"
```

## 🚨 Troubleshooting

### Common Build Issues

**Issue 1: Build script fails with "configs directory not found"**
```bash
# Solution: Create missing directories and fix paths
cd ~/nethunter-build/kernel_lavender
mkdir -p arch/arm64/configs
find . -name "*defconfig*" -type f
# Copy appropriate defconfig to arch/arm64/configs/
```

**Issue 2: "nethunter_defconfig not found" error**
```bash
# Solution: Create nethunter_defconfig from device defconfig
cd ~/nethunter-build/kernel_lavender
if [ -f "arch/arm64/configs/lavender_defconfig" ]; then
    cp arch/arm64/configs/lavender_defconfig arch/arm64/configs/nethunter_defconfig
fi
```

**Issue 3: Compilation errors with clang**
```bash
# Solution: Use specific clang version and flags
export CC=clang
export CXX=clang++
export CLANG_TRIPLE=aarch64-linux-gnu-
export CROSS_COMPILE=aarch64-linux-android-
```

**Issue 4: Missing kernel modules**
```bash
# Solution: Build and install modules separately
make modules
make INSTALL_MOD_PATH=./modules modules_install
```

### Runtime Issues

**Issue 5: Bootloop after flashing**
```bash
# Emergency solution: Restore stock kernel
fastboot flash boot kernel_backup/boot_stock.img
fastboot reboot

# Alternative: Boot to recovery and restore from backup
```

**Issue 6: NetHunter features not working**
```bash
# Solution: Check kernel config and load modules
adb shell su -c "zcat /proc/config.gz | grep -i CONFIG_USB_HID"
# Should show: CONFIG_USB_HID=y

# Load modules manually if needed
adb shell su -c "modprobe rtl8188eu"
adb shell su -c "modprobe rtl8812au"
```

**Issue 7: Wireless injection not working**
```bash
# Solution: Check wireless adapter support
adb shell su -c "lsusb"
adb shell su -c "dmesg | grep -i wireless"
adb shell su -c "iwconfig"

# Try loading specific drivers
adb shell su -c "modprobe rtl8188eu"
adb shell su -c "modprobe rtl8812au"
```

**Issue 8: SELinux blocking NetHunter**
```bash
# Solution: Set SELinux to permissive
adb shell su -c "setenforce 0"
adb shell su -c "getenforce"
# Should show: Permissive
```

### Emergency Recovery

**If device won't boot:**
1. **Boot to fastboot**: Hold Volume Down + Power
2. **Flash stock kernel**: `fastboot flash boot kernel_backup/boot_stock.img`
3. **Reboot**: `fastboot reboot`

**If TWRP is broken:**
1. **Boot to fastboot**: Hold Volume Down + Power
2. **Flash stock recovery**: `fastboot flash recovery kernel_backup/recovery_stock.img`
3. **Or reflash TWRP**: `fastboot flash recovery twrp-3.7.0_12-0-lavender.img`

**If system is unstable:**
1. **Boot to TWRP**
2. **Wipe**: Dalvik/ART Cache, Cache, Data
3. **Restore**: Nandroid backup or flash stock ROM

## 📚 Additional Resources

### Documentation
- [Official NetHunter Documentation](https://www.kali.org/docs/nethunter/)
- [NetHunter GitLab Repository](https://gitlab.com/kalilinux/nethunter)
- [Android Kernel Build Guide](https://source.android.com/docs/setup/build/building-kernels)
- [Xiaomi Redmi Note 7 XDA Forum](https://forum.xda-developers.com/c/xiaomi-redmi-note-7.8434/)

### Community Support
- [Kali Forums - NetHunter](https://forums.kali.org/forumdisplay.php?30-Kali-NetHunter)
- [XDA Developers - Redmi Note 7](https://forum.xda-developers.com/c/xiaomi-redmi-note-7.8434/)
- [Reddit - r/Kalilinux](https://www.reddit.com/r/Kalilinux/)
- [Telegram - NetHunter Community](https://t.me/nethunterofficial)

### Useful Commands

```bash
# System Information
getprop ro.product.device                    # Device codename
getprop ro.build.version.release            # Android version
getprop ro.build.version.sdk                # API level
cat /proc/version                            # Kernel version

# Kernel and Module Information
lsmod                                        # List loaded modules
zcat /proc/config.gz | grep -i CONFIG_       # Check kernel config
dmesg | tail -f                              # Monitor kernel messages
cat /proc/cmdline                            # Kernel command line

# Wireless and Network
iwconfig                                     # Wireless interfaces
iw dev                                       # Wireless devices
iw list                                      # Wireless capabilities
lsusb                                        # USB devices
ip addr show                                 # Network interfaces

# NetHunter Specific
ls /dev/hidg*                               # HID gadget devices
getenforce                                  # SELinux status
which aircrack-ng                           # Check aircrack-ng
pm list packages | grep nethunter           # NetHunter apps

# Debugging
logcat | grep -i nethunter                  # NetHunter logs
dmesg | grep -i error                       # Kernel errors
cat /proc/mounts                            # Mounted filesystems
```

### File Locations

```bash
# Important file paths
/proc/config.gz                             # Kernel configuration
/system/lib/modules/                        # Kernel modules
/dev/hidg*                                  # HID gadget devices
/sys/kernel/debug/                          # Kernel debug info
/data/local/nhsystem/                       # NetHunter system files
```

## ⚠️ Important Notes

### Legal Disclaimer
- **Use NetHunter only for authorized penetration testing**
- **Respect local laws and regulations**
- **Only test on systems you own or have explicit permission to test**
- **The authors are not responsible for any illegal use**

### Warranty and Risk
- **This process voids your device warranty**
- **Custom kernels can permanently damage your device**
- **Always maintain current backups**
- **The authors are not responsible for device damage**
- **Proceed at your own risk**

### Android 15 Compatibility
- **This guide is specifically for Android 15**
- **Older guides may not work with current Android versions**
- **Some features may be limited due to Android security enhancements**
- **Regular updates may break NetHunter functionality**

### Security Considerations
- **NetHunter requires root access**
- **SELinux must be set to permissive**
- **Device security is reduced with custom kernel**
- **Regular security updates may not be available**

### Performance Impact
- **Custom kernel may affect battery life**
- **Some optimizations may be lost**
- **Performance may vary compared to stock kernel**
- **Monitor device temperature during intensive tasks**

## 🤝 Contributing

Found an issue or improvement? Contributions are welcome!

### How to Contribute
1. **Test the guide** on your device
2. **Document any issues** or improvements
3. **Create detailed bug reports** with logs
4. **Submit pull requests** with clear descriptions
5. **Help others** in community forums

### Reporting Issues
When reporting issues, please include:
- Device model and Android version
- Kernel source and version used
- Build environment details
- Complete error logs
- Steps to reproduce

### Improvement Suggestions
- Better error handling
- Additional device support
- Performance optimizations
- Documentation improvements
- Automated testing scripts

## 📈 Changelog

### Version 2.1 (Current)
- ✅ Added comprehensive troubleshooting for build script issues
- ✅ Fixed missing configs directory problem
- ✅ Added manual defconfig creation steps
- ✅ Enhanced error handling and recovery procedures
- ✅ Added build verification steps
- ✅ Improved emergency recovery procedures
- ✅ Added detailed testing and verification steps

### Version 2.0
- ✅ Updated for Android 15 compatibility
- ✅ Added NDK r26d support
- ✅ Updated toolchain to Clang 19
- ✅ Enhanced build process
- ✅ Added comprehensive testing procedures

### Version 1.0
- ✅ Initial release for Android 14
- ✅ Basic kernel build process
- ✅ Simple installation guide

## 🏆 Credits

### Development Team
- **0xb0rn3** - Main developer and guide author
- **0xbv1** - Testing and documentation
- **Kali Linux Team** - NetHunter development
- **XDA Community** - Device-specific support

### Special Thanks
- **Team-420** - Kernel source maintenance
- **OmniROM Team** - TWRP development
- **topjohnwu** - Magisk development
- **osm0sis** - AnyKernel3 development

---

**Guide Version**: 2.1  
**Last Updated**: July 2025  
**Target Device**: Xiaomi Redmi Note 7 (lavender)  
**Target Android**: 15 (API 35)  
**Kernel Version**: 4.19.321+  
**Build Status**: ✅ Tested and Working  
**Created by**: 0xb0rn3 | 0xbv1

---

**⚠️ Disclaimer**: This guide is provided for educational and legitimate penetration testing purposes only. The authors are not responsible for any damage to devices or illegal use of the tools described herein. Always ensure you have proper authorization before testing on any systems.
