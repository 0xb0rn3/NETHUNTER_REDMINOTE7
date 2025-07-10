# Redmi Note 7 NetHunter Kernel Development Guide

## Overview
This comprehensive guide covers kernel development for the Xiaomi Redmi Note 7 (lavender) with NetHunter capabilities, targeting Android 13 Axion AOSP ROM.

## Device Specifications
- **Codename**: lavender
- **SoC**: Qualcomm SDM660 Snapdragon 660
- **Architecture**: ARM64
- **Target Android**: 13 (Axion AOSP)
- **Kernel Version**: 4.19 (CAF-based)

## Prerequisites

### System Requirements
- Linux build environment (Arch Linux, Ubuntu 18.04+, or similar)
- Minimum 16GB RAM, 100GB+ free storage
- Stable internet connection for downloading sources

### Required Tools

#### For Arch Linux (Recommended)
```bash
# Update system
sudo pacman -Syu

# Install base development tools
sudo pacman -S base-devel git ccache automake flex lzop bison \
    gperf zip curl zlib python python-networkx libxml2 bzip2 \
    squashfs-tools pngcrush schedtool lz4 make optipng maven \
    openssl pwgen perl-switch policycoreutils minicom \
    perl-xml-sax-base perl-xml-simple bc lib32-ncurses \
    libx11 lib32-zlib mesa xsltproc unzip fontconfig

# Install multilib support (for 32-bit libraries)
sudo pacman -S lib32-gcc-libs lib32-glibc lib32-zlib lib32-ncurses

# Install AUR helper (yay) for additional packages
sudo pacman -S --needed git base-devel
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
cd ..
rm -rf yay

# Install additional tools from AUR
yay -S android-tools android-udev
```

#### For Ubuntu/Debian
```bash
# Install build dependencies
sudo apt update
sudo apt install -y git ccache automake flex lzop bison \
    gperf build-essential zip curl zlib1g-dev g++-multilib \
    python3-networkx libxml2-utils bzip2 libbz2-dev \
    libbz2-1.0 libghc-bzlib-dev squashfs-tools pngcrush \
    schedtool dpkg-dev liblz4-tool make optipng maven \
    libssl-dev pwgen libswitch-perl policycoreutils \
    minicom libxml-sax-base-perl libxml-simple-perl bc \
    libc6-dev-i386 lib32ncurses5-dev libx11-dev lib32z-dev \
    libgl1-mesa-dev xsltproc unzip fontconfig
```

### Device Prerequisites
- Unlocked bootloader
- Root access (Magisk recommended)
- Custom recovery (TWRP)
- Backup of original kernel/boot image

## Phase 1: Environment Setup

### 1. Create Build Directory
```bash
mkdir -p ~/android/kernel
cd ~/android/kernel
```

### 2. Set up Build Environment (Arch Linux specific optimizations)
```bash
# Configure ccache for faster builds
export USE_CCACHE=1
export CCACHE_DIR="$HOME/.ccache"
ccache -M 50G  # Set cache size to 50GB

# Set up environment variables
export ARCH=arm64
export SUBARCH=arm64
export CROSS_COMPILE=aarch64-linux-gnu-
export KBUILD_BUILD_USER=nethunter
export KBUILD_BUILD_HOST=archlinux

# Add to ~/.bashrc or ~/.zshrc for persistence
echo 'export USE_CCACHE=1' >> ~/.bashrc
echo 'export CCACHE_DIR="$HOME/.ccache"' >> ~/.bashrc
echo 'export ARCH=arm64' >> ~/.bashrc
echo 'export SUBARCH=arm64' >> ~/.bashrc
echo 'export CROSS_COMPILE=aarch64-linux-gnu-' >> ~/.bashrc
```

### 3. Install Cross-Compilation Toolchain
```bash
# For Arch Linux - install from official repos
sudo pacman -S aarch64-linux-gnu-gcc aarch64-linux-gnu-binutils

# Alternative: Download Google's prebuilt toolchain
mkdir -p ~/toolchains
cd ~/toolchains
wget https://android.googlesource.com/platform/prebuilts/gcc/linux-x86/aarch64/aarch64-linux-android-4.9/+archive/refs/heads/master.tar.gz
tar -xzf master.tar.gz
export PATH="$HOME/toolchains/bin:$PATH"
```

### 4. Clone NetHunter Kernel Builder
```bash
git clone https://gitlab.com/kalilinux/nethunter/build-scripts/kali-nethunter-kernel-builder.git
cd kali-nethunter-kernel-builder
```

### 3. Clone Kernel Source
```bash
# Clone the Aeoniixx kernel source
git clone https://github.com/Aeoniixx/kernel_xiaomi_lavender-4.19.git -b main
cd kernel_xiaomi_lavender-4.19

# Verify the defconfig exists
ls arch/arm64/configs/ | grep lavender
```

## Phase 2: Kernel Configuration

### 1. Create Build Configuration
```bash
cd ~/android/kernel/kali-nethunter-kernel-builder
cp config local.config
nano local.config
```

### 2. Configure Build Settings
Edit `local.config` with the following parameters:

```bash
# Device Configuration
KERNEL_DIR="/home/$USER/android/kernel/kernel_xiaomi_lavender-4.19"
DEFCONFIG="lavender_defconfig"
DEVICE_NAME="lavender"
CODENAME="lavender"

# Architecture Settings
ARCH="arm64"
SUBARCH="arm64"
CROSS_COMPILE="aarch64-linux-android-"

# Build Configuration
JOBS=$(nproc)
CCACHE_DIR="/tmp/ccache"
USE_CCACHE=1

# NetHunter Specific
KERNEL_LOCALVERSION="-NetHunter"
KERNEL_PATCHES="y"
HID_KEYBOARD="y"
WIRELESS_INJECTION="y"
```

### 3. NetHunter Kernel Patches
Essential patches for NetHunter functionality:

#### USB HID Keyboard Support
```c
// Enable in kernel config
CONFIG_USB_GADGET=y
CONFIG_USB_CONFIGFS=y
CONFIG_USB_CONFIGFS_F_HID=y
CONFIG_HIDRAW=y
CONFIG_USB_GADGET_VBUS_DRAW=500
```

#### Wireless Injection Support
```c
// Enable monitor mode and packet injection
CONFIG_MAC80211_MESH=y
CONFIG_CFG80211_WEXT=y
CONFIG_WIRELESS_EXT=y
CONFIG_WEXT_PRIV=y
CONFIG_NETFILTER_XT_MATCH_STATE=y
```

## Phase 3: Advanced Kernel Modifications

### 1. Security Enhancements
```c
// Add to defconfig for enhanced security
CONFIG_SECURITY_SELINUX_BOOTPARAM=y
CONFIG_SECURITY_SELINUX_DISABLE=y
CONFIG_SECURITY_SELINUX_DEVELOP=y
CONFIG_SECURITY_SELINUX_AVC_STATS=y
```

### 2. Performance Optimizations
```c
// CPU Governor and Scheduler tweaks
CONFIG_CPU_FREQ_DEFAULT_GOV_SCHEDUTIL=y
CONFIG_CPU_FREQ_GOV_PERFORMANCE=y
CONFIG_CPU_FREQ_GOV_ONDEMAND=y
CONFIG_SCHED_WALT=y
CONFIG_SCHED_TUNE=y
```

### 3. Memory Management
```c
// Enhanced memory management
CONFIG_MEMCG=y
CONFIG_MEMCG_SWAP=y
CONFIG_MEMCG_KMEM=y
CONFIG_CGROUP_HUGETLB=y
CONFIG_TRANSPARENT_HUGEPAGE=y
```

## Phase 4: Compilation Process

### 1. Initialize Build Environment
```bash
cd ~/android/kernel/kali-nethunter-kernel-builder
./build.sh
```

### 2. Setup Environment (First Time)
```bash
# Select option 'S' for setup
# This installs required toolchains
```

### 3. Configure and Compile
```bash
# Select option '2' - Configure & compile kernel from scratch
# Choose your defconfig when prompted
# Apply NetHunter patches when asked
```

### 4. Manual Compilation (Arch Linux Optimized)
```bash
cd ~/android/kernel/kernel_xiaomi_lavender-4.19

# Set environment variables (if not in bashrc)
export ARCH=arm64
export SUBARCH=arm64
export CROSS_COMPILE=aarch64-linux-gnu-
export KBUILD_BUILD_USER=nethunter
export KBUILD_BUILD_HOST=archlinux

# Configure kernel
make lavender_defconfig
make menuconfig  # For custom configuration

# Compile kernel with optimal settings for Arch
make -j$(nproc --all) \
    ARCH=arm64 \
    SUBARCH=arm64 \
    CROSS_COMPILE=aarch64-linux-gnu- \
    KCFLAGS="-O3 -pipe -fno-plt" \
    KCPPFLAGS="-O3 -pipe -fno-plt" \
    2>&1 | tee build.log

# Alternative with clang (if available)
make -j$(nproc --all) \
    ARCH=arm64 \
    SUBARCH=arm64 \
    CROSS_COMPILE=aarch64-linux-gnu- \
    CC=clang \
    HOSTCC=clang \
    CLANG_TRIPLE=aarch64-linux-gnu- \
    2>&1 | tee build_clang.log
```

## Phase 5: Custom Kernel Features

### 1. NetHunter HID Implementation
Create custom HID functionality:

```c
// drivers/hid/hid-nethunter.c
#include <linux/hid.h>
#include <linux/module.h>
#include <linux/usb.h>

// HID keyboard descriptor for NetHunter
static const char keyboard_descriptor[] = {
    0x05, 0x01,  // Usage Page (Generic Desktop)
    0x09, 0x06,  // Usage (Keyboard)
    0xa1, 0x01,  // Collection (Application)
    // ... (full HID descriptor)
};

// NetHunter HID driver implementation
static struct hid_driver nethunter_hid_driver = {
    .name = "nethunter-hid",
    .id_table = nethunter_hid_devices,
    .probe = nethunter_hid_probe,
    .remove = nethunter_hid_remove,
};

module_hid_driver(nethunter_hid_driver);
```

### 2. Wireless Monitor Mode
Enable monitor mode for wireless interfaces:

```c
// net/wireless/nl80211.c modifications
static int nl80211_set_interface(struct sk_buff *skb, struct genl_info *info)
{
    // Add monitor mode support
    if (type == NL80211_IFTYPE_MONITOR) {
        // Enable monitor mode functionality
        return cfg80211_change_iface(rdev, dev, type, flags, &params);
    }
    // ... rest of function
}
```

### 3. Packet Injection Support
```c
// drivers/net/wireless/*/cfg80211.c
static int nethunter_inject_frame(struct wiphy *wiphy,
                                  struct wireless_dev *wdev,
                                  struct cfg80211_mgmt_tx_params *params,
                                  u64 *cookie)
{
    // Custom packet injection implementation
    // Bypass normal wireless stack restrictions
    return 0;
}
```

## Phase 6: AnyKernel Integration

### 1. Create AnyKernel Configuration
```bash
cd ~/android/kernel/kali-nethunter-kernel-builder
./build.sh
# Select option '8' - Edit Anykernel config
```

### 2. Configure AnyKernel Settings
```bash
# Edit anykernel.sh
device.name1=lavender
device.name2=Redmi Note 7
device.name3=
device.name4=
device.name5=

supported.versions=13
supported.patchlevels=

block=/dev/block/bootdevice/by-name/boot
is_slot_device=0
ramdisk_compression=auto
```

### 3. Generate Flashable ZIP
```bash
# Select option '6' - Create Anykernel zip
# This creates the flashable kernel zip
```

## Phase 7: Installation and Testing

### 1. Pre-Installation Backup
```bash
# Boot into TWRP
# Backup current boot partition
# Create NANDroid backup
```

### 2. Flash Custom Kernel
```bash
# Transfer kernel zip to device
# Flash via TWRP
# Wipe cache/dalvik
# Reboot system
```

### 3. Verification
```bash
# Check kernel version
cat /proc/version

# Verify NetHunter features
ls /sys/kernel/config/usb_gadget/
dmesg | grep -i nethunter

# Test HID functionality
echo "echo 'NetHunter Test' > /dev/hidg0" > /data/local/tmp/test_hid.sh
chmod 755 /data/local/tmp/test_hid.sh
```

## Arch Linux Specific Advantages

### 1. Rolling Release Benefits
```bash
# Always get latest build tools
sudo pacman -Syu

# Access to cutting-edge development tools
sudo pacman -S clang lld llvm

# Latest kernel headers for reference
sudo pacman -S linux-headers
```

### 2. AUR Package Benefits
```bash
# Install Android development tools from AUR
yay -S android-studio android-sdk android-sdk-platform-tools
yay -S repo-git  # For AOSP development

# Install additional cross-compilation tools
yay -S arm-linux-gnueabihf-gcc
yay -S aarch64-linux-gnu-gdb  # For debugging
```

### 3. Optimized Build Configuration
```bash
# Create optimized makepkg.conf for kernel building
sudo cp /etc/makepkg.conf /etc/makepkg.conf.backup
sudo nano /etc/makepkg.conf

# Add these optimizations:
# CFLAGS="-march=native -O3 -pipe -fno-plt"
# CXXFLAGS="${CFLAGS}"
# MAKEFLAGS="-j$(nproc)"
# BUILDENV=(!distcc !color ccache check !sign)
```

### 4. Performance Monitoring
```bash
# Install performance monitoring tools
sudo pacman -S htop iotop nethogs powertop

# Monitor build process
htop &
iotop -o &
# Then run your kernel build
```

## Phase 8: Troubleshooting

### Arch Linux Specific Issues

#### 1. Missing 32-bit Libraries
```bash
# Enable multilib repository
sudo nano /etc/pacman.conf
# Uncomment these lines:
# [multilib]
# Include = /etc/pacman.d/mirrorlist

sudo pacman -Syu
sudo pacman -S lib32-gcc-libs lib32-glibc lib32-zlib
```

#### 2. Toolchain Issues
```bash
# If aarch64-linux-gnu-gcc not found
sudo pacman -S aarch64-linux-gnu-gcc

# Alternative: Use Android NDK
yay -S android-ndk
export CROSS_COMPILE=$ANDROID_NDK_ROOT/toolchains/llvm/prebuilt/linux-x86_64/bin/aarch64-linux-android29-
```

#### 3. Build Optimization Issues
```bash
# If build fails with optimization flags
export KCFLAGS="-O2"  # Reduce optimization level
export KCPPFLAGS="-O2"

# Disable specific optimizations that might cause issues
export KCFLAGS="-O2 -fno-strict-aliasing"
```

#### 1. Compilation Errors
```bash
# Check build log
tail -n 100 build.log

# Common fixes
make clean
make mrproper
make lavender_defconfig
```

#### 2. Boot Issues
```bash
# Check dmesg for errors
dmesg | grep -i error

# Verify kernel modules
lsmod | grep -i nethunter
```

#### 3. NetHunter Features Not Working
```bash
# Check USB gadget configuration
ls /sys/kernel/config/usb_gadget/

# Verify HID devices
ls /dev/hidg*

# Check wireless capabilities
iw list | grep -i monitor
```

## Advanced Customization

### 1. CPU Frequency Scaling
```c
// Add custom governor
static struct cpufreq_governor nethunter_gov = {
    .name = "nethunter",
    .governor = cpufreq_governor_nethunter,
    .owner = THIS_MODULE,
};
```

### 2. I/O Schedulers
```c
// Custom I/O scheduler for performance
static struct elevator_type nethunter_iosched = {
    .ops = {
        .elevator_merge_fn = nethunter_merge,
        .elevator_dispatch_fn = nethunter_dispatch,
        .elevator_add_req_fn = nethunter_add_request,
        .elevator_former_req_fn = nethunter_former_request,
        .elevator_latter_req_fn = nethunter_latter_request,
        .elevator_init_fn = nethunter_init_queue,
        .elevator_exit_fn = nethunter_exit_queue,
    },
    .elevator_name = "nethunter",
    .elevator_owner = THIS_MODULE,
};
```

### 3. Security Bypass Features
```c
// SELinux permissive mode toggle
static ssize_t selinux_enforce_write(struct file *file, const char __user *buf,
                                     size_t count, loff_t *ppos)
{
    // Allow runtime SELinux mode changes
    if (new_value == 0) {
        selinux_enforcing = 0;
        printk(KERN_INFO "NetHunter: SELinux set to permissive\n");
    }
    return count;
}
```

## Performance Optimization

### 1. Kernel Parameter Tuning
```bash
# Add to init.d script
echo "performance" > /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
echo "1" > /proc/sys/vm/drop_caches
echo "60" > /proc/sys/vm/swappiness
```

### 2. Memory Management
```c
// Custom memory allocator
static void *nethunter_alloc(size_t size, gfp_t flags)
{
    void *ptr = kmalloc(size, flags | __GFP_ZERO);
    if (ptr) {
        // Custom memory management logic
    }
    return ptr;
}
```

## Kernel Debugging

### 1. Enable Kernel Debugging
```c
// Add to defconfig
CONFIG_DEBUG_KERNEL=y
CONFIG_DEBUG_INFO=y
CONFIG_FRAME_POINTER=y
CONFIG_KGDB=y
CONFIG_KGDB_SERIAL_CONSOLE=y
```

### 2. Debugging Tools
```bash
# Use kernel debugging
echo "file nethunter.c +p" > /sys/kernel/debug/dynamic_debug/control

# Monitor kernel messages
dmesg -w | grep -i nethunter

# Use systrace for performance analysis
systrace.py --time=10 -o trace.html sched gfx view wm
```

## Security Considerations

### 1. Secure Boot Bypass
```c
// Disable secure boot verification
static int verify_signature(const void *data, size_t len)
{
    // Always return success for NetHunter
    return 0;
}
```

### 2. Root Detection Bypass
```c
// Hide root from detection
static int hide_root_check(void)
{
    // Modify system calls to hide root access
    return 0;
}
```

## Maintenance and Updates

### 1. Regular Updates
```bash
# Update kernel source
cd ~/android/kernel/kernel_xiaomi_lavender-4.19
git pull origin main

# Rebuild kernel
cd ~/android/kernel/kali-nethunter-kernel-builder
./build.sh
```

### 2. Version Management
```bash
# Tag releases
git tag -a v1.0-nethunter -m "NetHunter kernel v1.0"
git push origin v1.0-nethunter
```

## Conclusion

This comprehensive guide provides the foundation for developing a custom NetHunter kernel for the Redmi Note 7. The kernel includes essential NetHunter features like HID keyboard support, wireless monitor mode, and packet injection capabilities.

Key success factors:
- Proper build environment setup
- Correct kernel source and patches
- Thorough testing and verification
- Regular maintenance and updates

For additional support, refer to:
- Kali NetHunter documentation
- XDA Developers forums
- GitHub repositories and issue trackers
- Community Discord/Telegram groups

Remember to always backup your device before flashing custom kernels and test thoroughly in a safe environment.
