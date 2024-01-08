# NEPI-Engine
This repository contains documentation and tools for getting started with NEPI Engine, a full-featured edge-AI and automation software platform for embedded edge-compute hardware.

**[Learn more about NEPI Engine](https://numurus.com/software-nepi-engine/)**

## Hardware and O/S Requirements
NEPI Engine runs on lots of embedded Linux devices, though certain hardware and O/S configurations are more well-suited than others. In general, NVIDIA Jetson platforms are well tested and supported. The scripts and documentation in this repository are tailored for modern Debian-based Linux distributions (e.g., Ubuntu 18.04+); if that is not your situation, then NEPI _may_ still be appropriate wholly or in part, albeit with additional setup legwork.

**Numurus provides commercially licensed pre-built root filesystem images and complete hardware solutions for select platforms -- speed up your development time considerably by [exploring one of these options](https://numurus.com/wp-content/uploads/NEPI-Engine-Pre-Built-Image-Installation-Instructions.pdf). Or [contact us](mailto:nepi@numurus.com) to discuss professional support options for other platforms**

The following sections describe the architecture of the NEPI Engine and provide tools and guidance for getting NEPI running on your device.

## NEPI Engine Architecture

The following diagram provides a high-level overview of the functional components of NEPI Engine:

<img src="https://numurus.com/wp-content/uploads/NEPI_Engine_Architecture_2000x1700.png" width="600px">

A NEPI-enabled device provides the complete NEPI Engine suite of tools and applications. Most of these components can be enabled and disabled through system configuration, and many can also be started and stopped at run-time as needed.

NEPI Engine setup and source code is distributed across a few top-level repositories:

- [nepi_rootfs_tools](https://github.com/numurus-nepi/nepi_rootfs_tools) - Collection of scripts and documentation for preparing a base system. This includes setting up for the NEPI fully redundant system image software update scheme and installing all dependencies (including platform-specific dependencies where applicable). 

- [nepi_base_ws](https://github.com/numurus-nepi/nepi_base_ws) - Superproject for primary NEPI Engine SDK, including sensor and motion control drivers, system configuration tools, ROS nodes and interfaces, and test-and-development utilities. This is the nuts-and-bolts of the NEPI Engine. Most of the operative code is distributed to Git submodules within this top-level repo.

- [nepi_rui](https://github.com/numurus-nepi/nepi_rui) - Node.js project for the browser-based, NEPI-device-hosted Resident User Interface for setup, configuration, and observation of NEPI-enabled devices.

- [nepi-bot](https://github.com/numurus-nepi/nepi_rui) - Python application for NEPI-device cloud/server connectivity, configuration, and control. The primitive interface to this application is fully file-based, but see nepi_edge_sdk_link for an alternative programmatic interface

- [nepi_edge_sdk_link](https://github.com/numurus-nepi/nepi_edge_sdk_link) - C and Python library that allows on-device applications to configure and control nepi-bot. The NEPI Engine ROS interface to nepi-bot (nepi_link_ros_bridge) leverages this library, and custom applications can, too.

## Typical System Bring-up
The following workflow describes how NEPI Engine is first deployed to a new hardware platform. It assumes
- Your target device has a Debian-based Board Support Package (BSP) and/or Operating System (O/S) available and accessible as a binary image (_img.raw_, _.bin_), compressed archive (_.tar.gz_), etc.
- Your target device has a storage media installed that allows you to add NEPI-specific partitions (typically requires 32GB for each of the NEPI A/B partitions, and ideally a large leftover space to be dedicated to the _nepi_storage_ partition)
- Your target device can be configured for internet access. 

Workflow:
1. Install the base Board Support Package (BSP) and/or Operating System (O/S) on the platform according to manufacturers instructions. In some cases, this step is already completed by the manufacturer or reseller and you can skip. A minimal install is typically sufficient, since this BSP will serve only as the NEPI _INIT_ rootfs.

1. Boot, log in, and configure the target device for internet connectivity. 

1. Follow the instructions for [installing the INIT Rootfs](https://github.com/numurus-nepi/nepi_rootfs_tools#installing-the-init-rootfs) from the nepi_rootfs_tools README.

1. Follow the instructions for [installing the Main A/B Rootfs from complete image](https://github.com/numurus-nepi/nepi_rootfs_tools?tab=readme-ov-file#installing-the-main-ab-rootfs-from-complete-image) from the same nepi_rootfs_tools README. For the "complete image," use the same initial BSP ROOTFS image as in step 1 above.

1. At this point, you should have rebooted/power cycled the target device and it should have booted up into a pristine copy of the BSP rootfs mounted from your NEPI_ROOTFS_A partition. You can verify with
    ```
    $ df
    ```
    ensuring that the NEPI_ROOTFS_A device is listed as "Mounted on" /

1. Now begin converting the BSP rootfs into a full-fledged NEPI rootfs by following the instructions for [constructing the main a/b rootfs from a base image](https://github.com/numurus-nepi/nepi_rootfs_tools?tab=readme-ov-file#constructing-the-main-ab-rootfs-from-a-base-image) from the nepi_rootfs_tools README. You should stop after running the setup_nepi_<xyz>_rootfs.sh script and proceed from here to build NEPI s/w from source.

1. At this point you have a rootfs that has all dependencies and build-tools installed to begin building the NEPI Engine software. Those build steps are individually covered in the repositories that constitute the full NEPI Engine as enumerated above in NEPI Engine Architecture(#nepi-engine-architecture) section. Go through the build/install steps in each of these repo READMEs individually:
- [nepi_base_ws](https://github.com/numurus-nepi/nepi_base_ws)

- [nepi_rui](https://github.com/numurus-nepi/nepi_rui)

- [nepi-bot](https://github.com/numurus-nepi/nepi-bot)

- [nepi_edge_sdk_link](https://github.com/numurus-nepi/nepi_edge_sdk_link)

## Alternatives
This section details approaches to overcoming some target platform limitations. In most cases these alternatives have been successfully employed by Numurus in porting NEPI to specific platforms.

### Target device has no internet capability
If you cannot run setup scripts that install NEPI Engine dependencies because the target device has no internet capability, it is often possible to mount the BSP root filesystem on a Linux development host PC using the relevant _qemu_ package to provide an emulator engine and then _chroot_ to the mountpoint and install external dependencies, etc. via the host PC internet connection using the regular nepi_rootfs_tools setup scripts.

Because there is different preparation for the NEPI _init_ and _main_ rootfs, you should maintain a pristine copy of the BSP rootfs at all times and make copies that you modify into. Beware, this approach usually requires considerable disk space on the development machine to maintain the various rootfs images.

### Target device does not have user-partionable installation media
In cases where there is a strictly prescribed partition layout for the sole storage device, it is sometimes possible to reconfigure the partition tables such that the BSP deployment process creates the necessary NEPI_ROOTFS_<A,B> and DATA partitions. This is how NEPI is installed on [Jetson Orin-NX on the SSD](https://github.com/numurus-nepi/nepi_rootfs_tools/blob/master/dev_host_tools/jetson_initrd_flash_support/README_nepi_initrd_flash.txt) alongside all the BSP partitions. See manufacturer documentation to determine if this is a viable path for your platform.

It is also possible to bypass the NEPI A/B filesystem support and convert the sole BSP rootfs into a NEPI main rootfs by running the appropriate _setup_nepi_rootfs.sh_ script right after deploying the BSP rootfs. Note that this lead to some error messages in NEPI logs and an inability to use the NEPI full system image update capabilities, but generally provides a well-featured NEPI Engine deployment.

### BSP Rootfs is a .tar.gz, not a binary image file
This will only impact the way you write the initial contents to the NEPI_ROOTFS_A and NEPI_ROOTFS_B partitions. In general, you'll replace the _dd_ call with a _tar -xf_ call.
