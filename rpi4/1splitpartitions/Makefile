# Split Raspberry Pi 4 Arch Linux .tar.gz image into two raw partition images:
# vfat /boot into bootsrc.raw
# ext4 / into rootsrc.raw
# And later you can use these to build SD card etc..

# Stupid automount
UDISKS_AUTOMOUNT_HINT:=never
UDISKS_PRESENTATION_NOPOLICY:=1
UDISKS_SYSTEM_INTERNAL:=1
UDISKS_AUTO:=0
UDISKS_IGNORE:=1

define chown_file =
	if [[ -n "$(SUDO_UID)" ]]; then \
		sudo chown --verbose --from=0:0 $(SUDO_UID):$(SUDO_GID) "$(1)"; \
	fi
	sync
	exit 0
endef

define unmount_if_mounted =
	if [[ -n "$(shell findmnt --noheadings --output SOURCE --mountpoint "$(1)")" ]]; then \
		sudo umount --verbose "$(1)"; \
	fi
	sync
	exit 0
endef

.NOTPARALLEL:
.EXPORT_ALL_VARIABLES:

all: release

require_root:
	if [[ $(shell id -u) -ne 0 ]]; then \
		echo "Please run as root"; \
  		exit 1; \
	fi

release: require_root latest.tar.gz bootsrc.raw
	$(call unmount_if_mounted,rootdir)
	sudo rm -rf rootdir
	sync

	$(call unmount_if_mounted,bootdir)
	sudo rm -rf bootdir
	sync

	@echo Now use PARTITIONS rootsrc.raw and bootsrc.raw as READ-ONLY source for your own images.

latest.tar.gz:
	wget --output-document "$@" "http://os.archlinuxarm.org/os/ArchLinuxARM-rpi-4-latest.tar.gz"
	$(call chown_file,$@)
	@echo Downloaded.

root.raw: latest.tar.gz 
	mkdir -p rootdir
	$(call chown_file,rootdir)

	sync

	$(call unmount_if_mounted,rootdir)
	sync

	if [[ -f "$@" ]]; then rm "$@"; fi

	sync

	# /
	truncate -s 10G "$@"
	sync
	$(call chown_file,$@)

	mkfs.ext4 -v -L "rawimgsrc" -m 0 -M "/" -e remount-ro -U "3fe1752a-cae3-11e9-8504-74d02b2c6ed7" "$@"
	sync

	sudo mount --verbose --make-private --no-mtab --types ext4 -o loop,noexec,nouser,suid,noauto "$@" rootdir
	sync
	$(call chown_file,rootdir)

	sudo bsdtar -x --preserve-permissions --file latest.tar.gz --directory rootdir
	sync
	@echo File "$@" built.


bootsrc.raw rootsrc.raw: root.raw rootdir/boot/cmdline.txt
	$(call unmount_if_mounted,bootdir)
	sync

	mkdir -p bootdir

	if [[ -f "boot.raw" ]]; then rm "boot.raw"; sync; fi

	# /boot
	truncate -s 256M boot.raw
	sync

	$(call chown_file,boot.raw)

	mkfs.vfat -v -n "BOOT" -i "DB2DF092" boot.raw
	sync

	sudo mount --verbose --make-private --no-mtab --types vfat -o user,noauto,loop boot.raw bootdir
	sync

	sudo mv rootdir/boot/* bootdir
	sync

	$(call unmount_if_mounted,bootdir)
	$(call unmount_if_mounted,rootdir)

	du -hs root.raw
	e2fsck -v -f root.raw
	sync

	resize2fs -p -M root.raw
	sync

	du -hs root.raw

	mv root.raw rootsrc.raw
	sudo chmod a-w rootsrc.raw

	mv boot.raw bootsrc.raw
	sudo chmod a-w bootsrc.raw

clean:
	$(call unmount_if_mounted,rootdir)
	$(call unmount_if_mounted,bootdir)

	sync

	sudo rm -rf rootdir
	sudo rm -rf bootdir

	sync

	sudo rm -rf root.raw
	sudo rm -rf boot.raw

	sync

	sudo rm -rf rootsrc.raw
	sudo rm -rf bootsrc.raw

	@echo Cleaned.

listmounts:
	find . -maxdepth 1 -type d -exec findmnt "{}" \;

.PHONY: require_root all clean test release listmounts
.DEFAULT: all
.NOTPARALLEL: require_root bootsrc.raw rootsrc.raw root.raw boot.raw all release clean
