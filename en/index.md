---
slug: /
sidebar_position: 1
---

# OpenWrt

A Board Support Package (BSP) based on native OpenWrt 23.05 (*bl-v2.0.y based on master*) that integrates SpacemiT's Stone series chips. It includes, among others, the supervisor program interface (OpenSBI), bootloader (U-Boot/UEFI), Linux kernel, root file system (including various middleware and libraries), and examples applications. Its goal is to provide customers with support for soft routing and NAS scheme soluitions, and to enable the development of drivers or applications.

## Main components

The components of OpenWrt are:

- OpenSBI
- U-Boot
- Linux Kernel
- mpp: Media Process Platform
- FFmpeg (with Hardware Accelerated)

More components are being adapted, in particular:

- onnxruntime (with Hardware Accelerated)
- ai-support: AI demo
- k1x-vpu-firmware: Video Process Unit firmware
- k1x-vpu-test: Video Process Unit test program
- k1x-jpu: JPEG Process Unit API
- GStreamer (with Hardware Accelerated)

## Quick Guide

- [Download and Compile](openwrt_quickstart.md)
- [Device Management](openwrt_device_management.md)
- [Solution Management](openwrt_solution_management.md)

## Precompile

[SpacemiT OpenWrt source site](https://archive.spacemit.com/openwrt/releases/)

[BPI-F3、MUSE-Pi image](https://archive.spacemit.com/openwrt/releases/23.05.2/targets/spacemit/DEVICE_debX/openwrt-spacemit-k1-sbc-debX-ext4-pack.zip)

## Supported Device List

The following hardware devices are currently supported:

- BPI-F3
- [MUSE Pi](supported_devices/muse_pi.md)
