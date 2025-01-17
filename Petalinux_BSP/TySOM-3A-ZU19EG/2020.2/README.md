# Petalinux 2020.2 for [TySOM-3A-ZU19EG](https://www.aldec.com/en/products/emulation/tysom_boards/zynq_ultrascale_mpsoc/tysom_3a_zu19eg) board

## Table of Content
- [Provided files](#provided_files)
- [Petalinux 2020.2 instruction](#petalinux_instruction)
- [Vivado 2020.2 instruction](#vivado_instruction)
- [How to update Petalinux project and BSP](#update_instruction)

<a name="provided_files"/>

## Provided files

This package is composed of the following files:
- TySOM-3A-ZU19EG.bsp - Petalinux BSP file
- TySOM-3A-ZU19EG_vivado - Vivado project

<a name="petalinux_instruction"/>

## Petalinux 2020.2 instruction

This step by step instruction allows to rebuild a Linux kernel and a Petalinux filesystem based on a provided Petalinux BSP file.

1. Source a Petalinux configuration file
```
source <petalinux_installation_path>/petalinux-2020.2/settings.sh
```

2. Create a project with using the BSP.
```
petalinux-create -t project -s <path>/TySOM-3A-ZU19EG.bsp
```

3. Go to the project directory
```
cd ./TySOM-3A-ZU19EG
```

4. Build the project (required before the step 5 as a workaround for proper kernel patching)
```
petalinux-build
```

5. If any changes in the Linux kernel configuration are required then run the command:
```
petalinux-config -c kernel
```

6. If any changes in the root filesystem configuration are required then run the command:
```
petalinux-config -c rootfs
```

7. Build the project
```
petalinux-build
```

8. If the building process is successfully finished then created files should be placed at **./images/linux** folder.
Find **Image** and **rootfs.cpio.u-boot.gz** files.

9. Generate the BOOT.BIN file
```
petalinux-package --boot --force --fpga --u-boot
```

10. Copy files to a micro SD card.
The micro SD card should contain the following set of files:
- BOOT.BIN (**images/linux/BOOT.BIN**)
- boot.scr (**images/linux/boot.scr**)
- devicetree.dtb (renamed **images/linux/system.dtb**)
- Image (**images/linux/Image**)
- uramdisk.image.gz (renamed **images/linux/rootfs.cpio.u-boot.gz**)
- uEnv.txt (**pre-built/linux/images/uEnv.txt**)

Above files are also available in the pre-built directory (included in the BSP).

<a name="vivado_instruction"/>

## Vivado 2020.2 instruction

Build a project with using a TCL script.

1. Go to **TySOM-3A-ZU19EG_vivado** directory (directory with TCL script for the example design).

2. Source Vivado 2020.2 settings64.sh file
```
source <vivado_2020_2_path>/settings64.sh
```

3. Launch Vivado tool and run the script:
```
vivado -source ./TySOM-3A-ZU19EG.tcl
```

4. Run "Generate Bitstream"

<a name="update_instruction"/>

## How to update Petalinux project and BSP

If a user modified the reference Vivado design then it is possible to update a hardware platform for the Petalinux project. When the PetaLinux project is created then it is possible to update the hardware platform and rebuild the project. Follow below steps:

1. Export a hardware platform from Vivado design.
In Vivado 2020.2 tool: **File -> Export -> Export Hardware -> Next -> Include bitstream -> Select a name and a directory -> Finish**
A new XSA file will be created.

2. Clean the whole Petalinux project to its initial state
```
petalinux-build -x mrproper
```

3. Copy a new XSA to the PetaLinux project directory

4. Load the new XSA to the project and configure autogeneration options in menu
```
petalinux-config --get-hw-description=./
```

5. Save and exit configuration menu. Build the project
```
petalinux-build
```

6. If the new hardware platform is completely different than previous one then it may be needed to update the device tree structure as well. Without any changes an error can occur during the build process.

After a hardware platform update a device tree structure is autogenerated in the **./components/plnx_workspace/device-tree/device-tree/** directory.
User modifications should be placed in **./project-spec/meta-user/recipes-bsp/device-tree/files/system-user.dtsi**. Add required changes and rebuild the project.

7. It is also possible to crate an updated Petalinux BSP file by executing:
```
petalinux-package
```
