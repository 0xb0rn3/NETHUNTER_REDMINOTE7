# NetHunter Kernel Compilation Guide for Xiaomi Redmi Note 7 (Lavender)

> **Complete step-by-step guide for compiling NetHunter kernel on Linux distributions**

## 📋 Device Information

- **Device**: Xiaomi Redmi Note 7 (lavender)
- **Architecture**: ARM64 (arm64-v8a)
- **Android Version**: 15
- **Target Kernel**: Linux 4.19.321 (NetHunter Compatible)
- **Bootloader**: Unlocked (Required)
- **Root**: Required for flashing

## 🔧 Prerequisites

### System Requirements
- **RAM**: Minimum 8GB (16GB recommended)
- **Storage**: At least 50GB free space
- **CPU**: Multi-core processor (8+ cores recommended)
- **Internet**: Stable connection for downloading sources

### Required Tools
- Git
- Make
- GCC/Clang toolchain
- Python 3.x
- Java 8 or 11
- ADB and Fastboot
- Custom recovery (TWRP recommended)

## 🐧 Distribution-Specific Setup

### Arch Linux / Archcraft
```bash
# Update system
sudo pacman -Syu

# Install build dependencies
sudo pacman -S base-devel git python python-pip bc openssl \
               android-tools java-runtime-headless clang llvm \
               lld make cmake ninja zip unzip curl wget

# Install AUR helper if needed (yay)
git clone https://aur.archlinux.org/yay.git
cd yay && makepkg -si
cd ..

# Install additional tools from AUR
yay -S android-sdk-build-tools android-platform-tools
```

### Ubuntu/Debian
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install build dependencies
sudo apt install -y build-essential git python3 python3-pip bc \
                   libssl-dev libncurses5-dev libelf-dev bison \
                   flex make cmake ninja-build zip unzip curl wget \
                   clang llvm lld adb fastboot default-jdk

# Install additional Android tools
sudo apt install -y android-sdk-build-tools-installer
```

### Fedora/RHEL/CentOS
```bash
# Update system
sudo dnf update -y

# Install build dependencies
sudo dnf install -y @development-tools git python3 python3-pip bc \
                   openssl-devel ncurses-devel elfutils-libelf-devel \
                   bison flex make cmake ninja-build zip unzip curl wget \
                   clang llvm lld android-tools java-11-openjdk-devel

# Enable additional repositories if needed
sudo dnf install -y epel-release
```

### openSUSE
```bash
# Update system
sudo zypper update

# Install build dependencies
sudo zypper install -y patterns-devel-base-devel_basis git python3 \
                      python3-pip bc openssl-devel ncurses-devel \
                      libelf-devel bison flex make cmake ninja zip \
                      unzip curl wget clang llvm lld android-tools \
                      java-11-openjdk-devel
```

## 📦 Download Sources

### 1. Clone NetHunter Kernel Builder
```bash
# Create workspace directory
mkdir -p ~/nethunter-build
cd ~/nethunter-build

# Clone the main kernel builder
git clone https://github.com/offensive-security/kali-nethunter-kernel.git
cd kali-nethunter-kernel

# Make scripts executable
chmod +x build.sh
chmod +x bootstrap.sh
```

### 2. Clone Device Kernel Source
```bash
# Clone the Redmi Note 7 kernel source
git clone https://github.com/ImSpiDy/kernel_xiaomi_lavender.git -b lineage-18.1 kernel_source
```

### 3. Download Android NDK and Toolchain
```bash
# Download Android NDK (if not already installed)
cd ~/nethunter-build
wget https://dl.google.com/android/repository/android-ndk-r26d-linux.zip
unzip android-ndk-r26d-linux.zip
export ANDROID_NDK_HOME=~/nethunter-build/android-ndk-r26d

# Alternative: Use system clang
export CC=clang
export CXX=clang++
export AR=llvm-ar
export NM=llvm-nm
export STRIP=llvm-strip
export OBJCOPY=llvm-objcopy
export OBJDUMP=llvm-objdump
export READELF=llvm-readelf
```

## ⚙️ Configuration

### 1. Create Device Configuration
```bash
cd ~/nethunter-build/kali-nethunter-kernel

# Create local.config for lavender
cat > local.config << 'EOF'
# Device configuration for Xiaomi Redmi Note 7 (lavender)
DEVICE_NAME="lavender"
KERNEL_SOURCE="~/nethunter-build/kernel_source"
KERNEL_ARCH="arm64"
KERNEL_DEFCONFIG="lavender_defconfig"
CROSS_COMPILE="aarch64-linux-android-"
CLANG_TRIPLE="aarch64-linux-gnu-"
ANDROID_MAJOR_VERSION="11"
ANDROID_MINOR_VERSION="0"
ANDROID_VERSION="11.0"
KERNEL_LOCALVERSION="-NetHunter"
THREADS=$(nproc)
ENABLE_WIFI_INJECTION=true
ENABLE_USB_HID=true
ENABLE_BLUETOOTH_INJECTION=true
ENABLE_RTL8188EU=true
ENABLE_RTL8812AU=true
ENABLE_RTL8814AU=true
ENABLE_MT7612U=true
ENABLE_ATH9K_HTK=true
ENABLE_USB_SERIAL=true
MODULES_COMPRESSION=xz
EOF
```

### 2. Configure Kernel Options
```bash
# Navigate to kernel source
cd ~/nethunter-build/kernel_source

# Set environment variables
export ARCH=arm64
export SUBARCH=arm64
export CROSS_COMPILE=aarch64-linux-android-
export CC=clang
export CXX=clang++
export CLANG_TRIPLE=aarch64-linux-gnu-

# Generate .config from defconfig
make lavender_defconfig

# Optional: Manual configuration
make menuconfig
```

## 🔨 Compilation Process

### 1. Apply NetHunter Patches
```bash
cd ~/nethunter-build/kali-nethunter-kernel

# Run bootstrap to prepare environment
./bootstrap.sh

# Apply NetHunter patches
./build.sh --patch-only --device lavender
```

### 2. Build Kernel
```bash
# Build the kernel with NetHunter patches
./build.sh --device lavender --kernel-only

# Alternative manual build
cd ~/nethunter-build/kernel_source
make -j$(nproc) \
    ARCH=arm64 \
    SUBARCH=arm64 \
    CROSS_COMPILE=aarch64-linux-android- \
    CC=clang \
    CXX=clang++ \
    CLANG_TRIPLE=aarch64-linux-gnu- \
    Image.gz-dtb
```

### 3. Build Modules
```bash
# Build kernel modules
make -j$(nproc) \
    ARCH=arm64 \
    SUBARCH=arm64 \
    CROSS_COMPILE=aarch64-linux-android- \
    CC=clang \
    CXX=clang++ \
    CLANG_TRIPLE=aarch64-linux-gnu- \
    modules

# Install modules to temporary directory
mkdir -p ~/nethunter-build/modules
make -j$(nproc) \
    ARCH=arm64 \
    SUBARCH=arm64 \
    CROSS_COMPILE=aarch64-linux-android- \
    CC=clang \
    CXX=clang++ \
    CLANG_TRIPLE=aarch64-linux-gnu- \
    INSTALL_MOD_PATH=~/nethunter-build/modules \
    modules_install
```

### 4. Create Flashable ZIP
```bash
cd ~/nethunter-build/kali-nethunter-kernel

# Create complete NetHunter package
./build.sh --device lavender --full

# Output will be in releases/ directory
ls -la releases/
```

## 📱 Device Preparation

### 1. Unlock Bootloader
```bash
# Enable Developer Options and USB Debugging
# Enable OEM unlocking in Developer Options
# Boot to fastboot mode
adb reboot bootloader

# Unlock bootloader (THIS WILL WIPE DATA)
fastboot oem unlock
# OR
fastboot flashing unlock
```

### 2. Install Custom Recovery
```bash
# Download TWRP for lavender
wget https://dl.twrp.me/lavender/twrp-3.7.0_9-0-lavender.img

# Flash TWRP
fastboot flash recovery twrp-3.7.0_9-0-lavender.img

# Boot to recovery
fastboot boot twrp-3.7.0_9-0-lavender.img
```

## 🚀 Installation

### 1. Flash NetHunter Kernel
```bash
# Copy the built ZIP to device
adb push releases/kali-nethunter-kernel-lavender-*.zip /sdcard/

# In TWRP:
# 1. Wipe -> Advanced Wipe -> Dalvik/ART Cache
# 2. Install -> Select the NetHunter kernel ZIP
# 3. Flash and reboot
```

### 2. Install NetHunter App
```bash
# Download NetHunter App
wget https://kali.org/get-kali/kali-nethunter/images/nethunter-app.apk

# Install via ADB
adb install nethunter-app.apk
```

### 3. Install Kali Chroot
```bash
# Download appropriate chroot
wget https://kali.org/get-kali/kali-nethunter/images/kalifs-arm64-minimal.tar.xz

# Copy to device
adb push kalifs-arm64-minimal.tar.xz /sdcard/

# Install through NetHunter App
# Open NetHunter App -> Kali Chroot Manager -> Install Chroot
```

## 🔍 Verification

### 1. Check Kernel Version
```bash
# Check if NetHunter kernel is loaded
adb shell cat /proc/version

# Should show NetHunter in version string
```

### 2. Test NetHunter Features
```bash
# Check wireless injection capability
adb shell iwconfig

# Check USB HID support
adb shell ls /dev/hidg*

# Test in NetHunter App
# HID Attacks should be available
# Wireless tools should work
```

## 🐛 Troubleshooting

### Common Build Errors

#### 1. Missing Dependencies
```bash
# Error: command not found
# Solution: Install missing packages for your distribution

# For Arch Linux
sudo pacman -S missing-package

# For Ubuntu/Debian
sudo apt install missing-package
```

#### 2. Toolchain Issues
```bash
# Error: cross-compiler not found
# Solution: Set correct paths
export PATH=$ANDROID_NDK_HOME/toolchains/llvm/prebuilt/linux-x86_64/bin:$PATH
export CROSS_COMPILE=aarch64-linux-android30-
```

#### 3. Build Fails
```bash
# Clean build directory
cd ~/nethunter-build/kernel_source
make clean
make mrproper

# Rebuild
make lavender_defconfig
make -j$(nproc)
```

### Boot Issues

#### 1. Bootloop
- Flash stock kernel
- Check if bootloader is properly unlocked
- Verify TWRP compatibility

#### 2. No NetHunter Features
- Verify kernel patches were applied
- Check module loading
- Reinstall NetHunter App

## 📚 Additional Resources

### Official Documentation
- [Kali NetHunter Documentation](https://www.kali.org/docs/nethunter/)
- [NetHunter Kernel Builder](https://github.com/offensive-security/kali-nethunter-kernel)
- [Android Kernel Development](https://source.android.com/docs/core/architecture/kernel)

### Community Resources
- [XDA Developers Forum](https://forum.xda-developers.com/c/xiaomi-redmi-note-7.8402/)
- [Kali NetHunter Community](https://forums.kali.org/forumdisplay.php?f=108)
- [GitHub Issues](https://github.com/offensive-security/kali-nethunter-kernel/issues)

## ⚠️ Disclaimer

- **Warranty**: This process will void your device warranty
- **Risk**: Flashing custom kernels can brick your device
- **Legal**: Use NetHunter only for authorized testing
- **Backup**: Always create full device backups before proceeding

## 🤝 Contributing

1. Fork the repository
2. Create feature branch
3. Test thoroughly
4. Submit pull request
5. Update documentation

## 📄 License

This project follows the same license as the original kernel sources:
- Linux Kernel: GPL v2
- NetHunter Patches: GPL v2
- Build Scripts: GPL v3

---

**Created for educational and authorized security testing purposes only.**
