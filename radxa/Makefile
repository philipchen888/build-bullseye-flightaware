export BOARD=radxa
export LB_BOOTSTRAP_INCLUDE="apt-transport-https gnupg"
export OUT=$(PWD)/out
export BOOT=$(OUT)/boot.img
export ARCH=arm64
export CROSS_COMPILE=/usr/local/gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu/bin/aarch64-linux-gnu-
export BL31=$(PWD)/rkbin/bin/rk33/rk3399_bl31_v1.26.elf

all: u-boot kernel rootfs mksdcardimg

cleanall:
	cd u-boot; \
	make clean
	cd kernel; \
	sudo make ARCH=arm64 clean
	sudo lb clean --purge
	\rm -rf config linaro-rootfs.img

rootfs: clean prebuild buildrootfs postbuild mkimage

clean:
	sudo lb clean --purge
	\rm -rf config

prebuild:
	lb config \
	--apt-indices false \
	--apt-recommends false \
	--apt-secure false \
	--architectures arm64 \
	--archive-areas 'main contrib non-free' \
	--backports false \
	--binary-filesystem ext4 \
	--binary-images tar \
	--bootappend-live "hostname=linaro-alip username=linaro" \
	--bootstrap-qemu-arch arm64 \
	--bootstrap-qemu-static /usr/bin/qemu-aarch64-static \
	--cache false \
	--chroot-filesystem none \
	--compression gzip \
	--debootstrap-options "--variant=minbase --include=apt-transport-https,gnupg" \
	--distribution bullseye \
	--gzip-options '-9 --rsyncable' \
	--iso-publisher 'Linaro; http://www.linaro.org/; linaro-dev@lists.linaro.org' \
	--iso-volume 'Linaro Bullseye $(date +%Y%m%d-%H:%M)' \
	--linux-flavours none \
	--linux-packages none \
	--mode debian \
	--security true \
	--system normal \
	--updates true

buildrootfs:
	\cp -rf $(PWD)/../customization/* config/
	sudo lb build

postbuild:
	ARCH=arm64 $(PWD)/../mk-rootfs-bullseye.sh

mkimage:
	$(PWD)/../mk-image.sh

u-boot: cleanuboot builduboot postbuilduboot
kernel: buildkernel postbuildkernel

cleanuboot:
	\rm -f $(OUT)/idbloader.img
	\rm -f $(OUT)/rk3399_loader_v1.20.119.bin
	\rm -f $(OUT)/trust.img
	\rm -f $(OUT)/uboot.img

builduboot:
	cd u-boot; \
	make clean; \
	make ARCH=arm64 rock-pi-4-rk3399_defconfig; \
	make all

postbuilduboot:
	cd u-boot; \
	../rkbin/tools/loaderimage --pack --uboot ./u-boot-dtb.bin uboot.img 0x200000 --size 1024 1; \
	./tools/mkimage -n rk3399 -T rksd -d ../rkbin/bin/rk33/rk3399_ddr_800MHz_v1.20.bin idbloader.img; \
	cat ../rkbin/bin/rk33/rk3399_miniloader_v1.19.bin >> idbloader.img; \
	cp idbloader.img ${OUT}/; \
	cp ../rkbin/bin/rk33/rk3399_loader_v1.20.119.bin ${OUT}/; \
	../rkbin/tools/trust_merger --size 1024 1 ../trust.ini; \
	cp uboot.img ${OUT}/; \
	cp trust.img ${OUT}/

buildkernel:
	cd kernel; \
	make ARCH=arm64 clean; \
	make ARCH=arm64 rockchip_linux_defconfig; \
	make ARCH=arm64 -j4; \
	sudo make ARCH=arm64 INSTALL_MOD_PATH=./tmp modules_install

postbuildkernel:
	cd kernel; \
	\rm -f $(BOOT); \
	mkfs.vfat -n "boot" -S 512 -C ${BOOT} 512000; \
	mmd -i ${BOOT} ::/extlinux; \
	mcopy -i ${BOOT} -s ../rk3399.conf ::/extlinux/extlinux.conf; \
	mcopy -i ${BOOT} -s ./arch/arm64/boot/Image ::; \
	mcopy -i ${BOOT} -s ./arch/arm64/boot/dts/rockchip/rockpi-4b-linux.dtb ::

mksdcardimg:
	./mksdcardimg.sh
