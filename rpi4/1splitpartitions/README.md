# Split /boot and / to partition images

This Makefile splits the latest Raspberry Pi 4 image into two raw partition images to be used later for building your own SD card or other image(s).

* Source: https://archlinuxarm.org/platforms/armv8/broadcom/raspberry-pi-4

## FAQ

* Why not directly write the image?
  * Writing directly to SD card is slow
  * This approach gives more flexibility as you have the main partitions splitted, you can also split those even more
    * For example: Also split `/home`, `/usr`, `/var`, `/var/log`, etc ..

## Requirements

* Sudo

## What it does?

* Download
  1. Download latest .tar.gz image
* File and mount point for .tar.gz image
  1. Create empty file 
  1. Format that file as ext4
  1. Create mount point for that file
  1. Extract .tar.gz to that mount point
* Split `/boot`
  1. Create empty file for `/boot`
  1. Format that file as vfat
  1. Create mount point for that file
  1. Move `/boot/*` to created mount point
* Split `/`
  1. Now what is left of the latest .tar.gz mount point and partition image will be left for final phase
* Finally
  1. Clean mount points and temporary files
  1. Rename partition image files and remove write access
  1. Now these files can be used to make your own custom SD card etc image(s)

## Usage:

    make release

After that you should have read-only partition images:

* `bootsrc.raw`
* `rootsrc.raw`

After that is up to you how you will create your own image(s) or split `rootsrc.raw` into even more own partitions..

## Problems during make?
 
    make clean
    
and sometimes:

    rm bootsrc.raw
    rm rootsrc.raw
 
 and sometimes:
 
    rm latest.tar.gz
