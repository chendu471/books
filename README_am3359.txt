1.cross-compiler_tool_chain
   linaro>6_version
vim /root/.bashrc
	export PATH=$PATH:/path_for_cross_compiler_tool_chain/bin

	=>Install Device Tree Compiler	
	  ->apt-get install device-tree-compiler	
	=>Install u-boot-tools (mkimage)
	  ->apt-get install u-boot-tools
	  ->apt-get install ncurses-dev
	=>apt-get install libsdl1.2-dev 
	=>apt-get install libssl-dev

2.u-boot compilation
	=>To make source clean  	
	    ->$make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- distclean

	    ->$make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- mrproper
	
	=>$make CROSS_COMPILE=arm-linux-gnueabihf- ARCH=arm config_file

		[Note: search for /u-boot_source/configs/am335x_evm_config or am335x_evm_defconfig and replace config_file with this file in above command]	

	   	output: ".config" file
	=>make CROSS_COMPILE=arm-linux-gnueabihf- ARCH=arm menuconfig
		enable a default value for bootcmd 
		edit next line with  run sdboot
	->edit am335x_evm.h
	vim /include/configs/am335x_evm.h
		

		"optargs=\0" \

	"myfdtfile=am335x-evmsk.dtb\0"\
	"mmcroot=/dev/mmcblk0p2 rw\0" \
	"mmcrootfstype=ext4 rootwait\0" \

	"sdargs=setenv bootargs console=${console} " \
	"${optargs} " \
	"root=${mmcroot} " \
	"rootfstype=${mmcrootfstype}\0" \
	"sdboot=echo Booting from SDCARD ...; " \
		"run sdargs; " \
		"fatload mmc 0 ${loadaddr} ${bootfile};" \
		"fatload mmc 0 ${fdtaddr} ${myfdtfile};" \
		"bootz ${loadaddr} - ${fdtaddr}\0"\
	"run sdboot"


	Input: ".config" file
	=>.$ make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf-
		
              uboot.img,MLO,uboot.bin files are created in uboot source directoty and uboot.spl.bin created in uboot_source_code.spl directory
		output: 
                MLO
		u-boot.bin is the binary compiled U-Boot bootloader.
		u-boot.img contains u-boot.bin along with an additional header to be used by the boot 	ROM to determine how and where to load and    			executeUBoot.	

3.Kernel compilation	
	
	$cd /linux_source
=>To make source clean  	
	$make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- distclean

	$make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- mrproper

	-> $ti_config_fragments/defconfig_builder.sh -t ti_sdk_am3x_release
        	[ Note:if you get error download or copy am335x_evm_defconfig file and copy that file to /arch/arm/configs]

	-> $export ARCH=arm
	-> $make ti_sdk_am3x_release_defconfig
	-> $make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- ti_sdk_am3x_release_defconfig
	-> $ make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig	
	-> $make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- LOADADDR=0x8000 uImage -j4
	-> $make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- zImage -j4
		output:
		zImage,uImage created in kernel_source/arch/arm/boot directory

	->$ make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- am335x-evmsk.dtb -j4
		output:
		am335x_evmsk.dtb created kernel_source/arch/arm/boot/dts

FOR SD CARD BOOT

	1st partition is formatted as FAT32 containing MLO, u-boot.img, uImage files
	2nd partition is formatted as ext3 where the filesystem is extracted in root
	
	$ cp arch/arm/boot/zImage arch/arm/boot/dts/am335x-evmsk.dtb /media/rootfs/boot
	


->Connect the SD memory card using Memory Card reader to the Linux Host
->Note the name allotted for this device. Type "dmesg". The SD/MMC card name should show up near the end,
  usually something like "SDC" (/dev/sdc) or "SDD" (/dev/sdd).
->Navigate to the /home/am335x directory where all the mentioned files are copied
->Ensure that the script mksd-am335x.sh has executable permissions
->This scripts expects arguments in the following format
  ./mksd-am335x.sh <sd-device-name> <sd-1st-stage-bootloader> <sd-2nd-stage-bootloader> <kernel-uImage> <filesystem>

Problem faced:

	1.Missing defconfig & am335x-pm-firmware.bin files.(Taken from internet)
		->copy am335x-pm-firmware.bin to uboot-source/firmware.
	2.kernel loading (need to copy .dtb & zImage in rootfs/boot).
	3.kernel compilation error:openssl/bio.h openssl/... no such file or directory.
		->apt-get install libsdl1.2-dev 
		->apt-get install libssl-dev
	4.booting time error: Could not update .ICEauthority file /var/lib/gdm/.ICEauthority"
		->rm rootfs/var/lib/gdm/.ICEauthority 



