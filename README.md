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
		--append-to-version=-cw-net6501-120813-1 --revision=1 \
		kernel-image kernel-headers kernel-debug


