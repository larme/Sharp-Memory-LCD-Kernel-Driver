# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a DRM (Direct Rendering Manager) kernel driver for 2.7" 400x240 Sharp memory LCD panels, designed for Raspberry Pi systems. The driver provides a modern DRM interface for the Sharp memory LCD, replacing older fbdev approaches.

## Build Commands

Build the kernel module and device tree overlay:
```bash
make
```

Install the driver (requires root):
```bash
sudo make install
```

This will:
- Install the kernel module to the system
- Install the device tree overlay to `/boot/overlays/`
- Add `dtoverlay=sharp` to `/boot/config.txt`
- Add the module to auto-load in `/etc/modules`
- Update module dependencies

Remove the driver:
```bash
sudo make uninstall
```

Clean build artifacts:
```bash
make clean
```

For Buildroot integration, the driver uses the kernel-module infrastructure defined in `sharp.mk`.

## Architecture

The driver is structured as a modular kernel driver with three main interface components:

### Core Module (`src/main.c`)
- Entry point that coordinates the three interface modules
- Handles SPI device probe/remove lifecycle
- Manages module loading/unloading

### DRM Interface (`src/drm_iface.c`, `src/drm_iface.h`)
- Implements the DRM driver framework
- Handles display refresh operations
- Manages framebuffer updates and display pipeline
- Provides `drm_refresh()` and `drm_set_indicator()` functions

### Parameters Interface (`src/params_iface.c`, `src/params_iface.h`)
- Exposes kernel module parameters for runtime configuration
- Manages `mono_cutoff`, `mono_invert`, and `indicators` settings
- Provides `params_set_mono_invert()` for runtime parameter changes

### IOCTL Interface (`src/ioctl_iface.c`, `src/ioctl_iface.h`)
- Provides userspace control interface
- Allows applications to interact with the driver beyond standard DRM

### Display Indicators (`src/indicators.h`)
- Defines bitmap patterns for 6 keyboard modifier indicators (14x14 pixels each)
- Supports shift, physical alt, control, alt, altgr, and meta key indicators
- Used to display keyboard state on the Sharp LCD

## Hardware Configuration

The device tree overlay (`sharp.dts`) configures:
- SPI interface on spi0 with chip select high
- GPIO pins 22 (display enable) and 23 (VCOM)
- SPI frequency up to 8MHz
- Disables default spidev interfaces

## System Integration

The driver integrates with Buildroot through:
- `Config.in`: Buildroot package configuration
- `sharp.mk`: Build rules for kernel module and device tree compilation
- `init/S01sharp`: Init script for loading the module at boot

## Development Notes

- The driver requires Linux kernel headers for compilation
- Uses standard DRM framework for modern graphics stack integration
- SPI communication with the Sharp memory LCD panel
- Supports both monochrome and indicator display modes
- Debug output available via kernel parameter `debug=1`