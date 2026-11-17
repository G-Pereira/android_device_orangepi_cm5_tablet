# Orange Pi CM5 Tablet — Android 17

Based on [dvab-sarma's Android 17 work](https://github.com/dvab-sarma/android_device_opi_opi5_pro), retaining his history and shared HAL implementations. This product targets the existing CM5 with the tablet carrier, using `aosp_opi_cm5-cp2a-userdebug`. It does not inherit the Pro product or BoardConfig. Existing `opi5` HAL names are retained as interface identifiers.

## Building

Follow the two-workspace checkout instructions in the [CM5 manifest repository](https://github.com/G-Pereira/android_manifest_orangepi_cm5_tablet). This repository belongs at `device/opi/cm5` in the Android workspace. The companion kernel workspace defaults to the sibling directory `cm5-kernel`.

Install normal AOSP host dependencies, plus `bc`, `libelf-dev`, `libdw-dev`, `gcc-aarch64-linux-gnu`, `swig`, and Python setuptools. Supply Rockchip BL31 and DDR firmware from the stock BSP: `rk3588_bl31_v1.45.elf` and `rk3588_ddr_lp4_2112MHz_lp5_2400MHz_v1.16.bin`. These vendor binaries are not redistributed here.

```sh
export BL31=/absolute/path/to/rk3588_bl31_v1.45.elf
export ROCKCHIP_TPL=/absolute/path/to/rk3588_ddr_lp4_2112MHz_lp5_2400MHz_v1.16.bin
./device/opi/cm5/opi-cm5-tablet/build_android.sh
```

The wrapper builds Linux, applies the included CM5 patch to the pinned U-Boot source, builds Android using `nproc` jobs, and assembles an SD image. `JOBS` overrides parallelism; `CM5_KERNEL_ROOT` overrides the sibling kernel workspace. Build kernel and bootloader separately with `build_kernel.sh` and `build_bootloader.sh` in this directory. Generated kernel artifacts are ignored by Git. The default is the standard AOSP boot animation; no Orange Pi Pro animation is copied.

## Validation and current limits

The development installation has booted from SD into Android, with HDMI 1080p60, USB-C ADB, AP6256 Wi-Fi scanning, Bluetooth controller initialization, and wiringOP GPIO input/table reads. The factored device tree has compiled. **The newly separated product configuration and portable build wrappers have not yet completed a fresh full-image build and boot test.** These are development sources, not a validated release image.

LCD/touch, onboard audio, cameras, charging/battery management, wired bus loopback, and electrical GPIO output/interrupt tests are not validated. This is a userdebug configuration with permissive SELinux. wiringOP MMIO stays root-only. `CM5_WIRINGOP_DEBUG=0` omits the optional kernel MMIO/debug fragment.

## wiringOP

Native source is vendored from [orangepi-xunlong/wiringOP](https://github.com/orangepi-xunlong/wiringOP), `next` commit `0bb861127a77864775a306bd7cfc2e36836a66d1`, with Android/Bionic adaptation and CM5 tablet detection. Upstream license notices are retained.

`wiringop_app/` contains the new JNI/socket adapter for the original Android 13 GUI. The original vendor APK is not included: a redistribution license has not been supplied. Native `gpio` works without it. Rebuilding and importing the GUI still requires the original APK and local repackaging; an automated public extraction/repackaging helper is not yet provided. The GUI adapter was built and tested with the original UI on the development installation; daemon startup after a power cycle is not yet verified. Its current init policy is intended for userdebug only.

## Provenance

The root shared Android HALs and original Pro product remain dvab-sarma's work. `README.upstream.md` preserves his documentation. Board configuration and calibration were adapted from the Orange Pi Android 13 CM5 tablet BSP. Firmware provenance and regulatory database licensing are in `firmware/`. U-Boot changes and its pinned source base are documented in `patches/u-boot/`. Linux source and the ACK merge history are in the [companion kernel fork](https://github.com/G-Pereira/android_kernel_orangepi_cm5_tablet).
