# iSchool Soekris Net6501 Kernel

This repo contains the the Linux kernel customisations for the
Soekris Net6501 platform.

## Build instructions

	git clone git@github.com:aptivate/ischool-net6501-kernel.git
	apt-get source linux-image-3.2.0-0.bpo.2-486
	cp -a linux-3.2.20 linux-3.2.20-ischool
	cd linux-3.2.20-ischool
	ln -s .config ../kernel.config
	patch -p1 --dry-run < ../*.patch || echo "Patch failed! STOP HERE!"
	patch -p1 < ../*.patch
	PATH=/usr/lib/ccache:$PATH time ionice -c3 nice fakeroot \
		make-kpkg -j 5 --initrd \
		--append-to-version=-net6501-120813-1cw --revision=1 \
		kernel-image kernel-headers kernel-debug

Please use a different version number in `--append-to-version`,
e.g. using your initials and the current date.

The Grub installation on the Soekris will automatically boot to
the latest installed kernel if you run `update-grub` after
installation.

You can also install these kernels in the chroot environment and
then copy the updated image to a school server like this:

	./copy-soekris.sh ./root-working 192.168.128.201: \
		--exclude media/hdb1/ischool/ischool.zm \
		--exclude 'media/hdb1/system-imager/image.*' \
		--exclude /etc/hostname

Sources for the patches:

* [Atop 2.6.33](http://www.atoptool.nl/download/atoppatch-kernel-2.6.33.tar.gz),
  partially applied by hand to fit 3.2.20
* `ie6xx_wdt` (watchdog driver): backported `drivers/mfd/lpc_sch.c` and 
  `drivers/watchdog/ie6xx_wdt.c` from `linux-next`
* `soekris-net6501` and `leds-net6501`: written by me, submitted to
  `platform-driver-x86` and `linux-leds@vger.kernel.org`
* watchdog panic bugfix: written by me, submitted to
  `linux-watchdog@vger.kernel.org`

Other config customisations that I remember (compare kernel.config
to kernel.config-3.2.20-1~bpo60+1 for the full monty):

* Optimise the kernel for Intel Atom
* Enable SMP support
* Disable loads of unnecessary drivers to speed up compilation
* Enable [LatencyTOP](http://lwn.net/Articles/266153/)
* Enable [Atop](http://www.atoptool.nl/)
* Set the default hostname to `soekris` instead of `(none)`
* Enable `/proc/config.gz` for checking the actual configuration
  of a running kernel

If you want to compare the built kernel source with the original,
for example to rebuild patches after making local changes, you can
use a command like this:

	diff -Nru linux-3.2.20 linux-3.2.20-ischool \
		--exclude '.*.cmd' \
		--exclude '.*.d' \
		--exclude '*.inc' \
		--exclude '*.lds' \
		--exclude '*.mod.c' \
		--exclude '*.o' \
		--exclude '*.orig' \ 
		--exclude '*.rej' \
		--exclude '*.s' \
		--exclude '*.S' \
		--exclude '.config*' \
		--exclude .tmp_versions \
		--exclude .version \
		--exclude Module.symvers \
		--exclude modules.builtin \
		--exclude modules.order \
		--exclude '*System.map' \
		--exclude arch \
		--exclude config \
		--exclude debian \
		--exclude generated \
		--exclude lib \
		--exclude scripts \
		--exclude scsi \
		--exclude security \
		--exclude vt \
		--exclude config_data.h \
		--exclude devlist.h \
		--exclude gen-kdb_cmds.c \
		--exclude timeconst.h \
		--exclude version.h \
		| grep -v '^Binary files' \
		| tee linux-3.2.20-ischool.patch


