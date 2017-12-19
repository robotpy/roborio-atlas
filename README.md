ATLAS prebuilt binaries (NI LabVIEW 17.0)
=========================================

ATLAS is tricky to build, so the binaries are provided here (in the releases
directory) instead of providing makefiles to build them. However, if you need
to build it, here's how I did it.

Versions:

* libgfortran 5.3.0
* ATLAS 3.10.3
* LAPACK 3.7.1

There are two pieces that need to be built -- a fortran compiler (cross compiled
from Ubuntu 16.04) and then ATLAS itself.

Cross compiling the fortran compiler
====================================

System setup (Ubuntu 16.04)
---------------------------

	apt install build-essential python diffstat chrpath texinfo

Build fortran
-------------

	git clone https://github.com/ni/nilrt.git
	cd nilrt
	git checkout nilrt/17.0

	MACHINE=xilinx-zynq DISTRO=nilrt ./nibb.sh config

Modify `conf/local.conf', add the following to the end:

	FORTRAN_forcevariable = ",fortran"
	RUNTIMETARGET_append_pn-gcc-runtime = " libquadmath"
	
	PREFERRED_VERSION_libgfortran="5.3.0"
	PREFERRED_VERSION_nativesdk_libgfortran="5.3.0"

Now, you can build fortran:

	. env-nilrt-xilinx-zynq
	bitbake gcc
	bitbake libgfortran

Install fortran
---------------

There will be ipk files in `build/tmp_nilrt_3_0_xilinx-zynq-glibc/deploy/ipk/cortexa9-vfpv3/`.
Copy these to the roborio:

* libgfortran
* libgfortran-dev
* gfortran
* gfortran-symlinks

SSH as admin, install the packages:

	opkg install *.ipk

Now you're ready to build ATLAS.

Building ATLAS (RoboRIO)
========================

Warning: ATLAS needs a lot of memory, so grab an empty USB memory stick (4GB+)
and convert it to a linux swap partition, and then activate the swap.

Download ATLAS 3.10.3 and LAPACK 3.7.1. Copy to the RoboRIO, and unpack 
ATLAS. Execute:

	opkg install bzip2 gcc-symlinks binutils-symlinks make
	ulimit -s 8096
	tar -xf atlas3.10.3.tar.bz2

	cd ATLAS
	mkdir BUILD
	cd BUILD

	../configure --shared --with-netlib-lapack-tarfile=/home/admin/lapack-3.7.1.tgz -m 667
	make
	make check
	make install

Be warned that ATLAS will take the RoboRIO to the edge of it's disk space
capacity, so remove anything from the system that you don't need. I found that
I had to delete a lot of NI builtin stuff that wasn't needed for the system to
run. I also deactivated a lot of system services that were eating large
amounts of memory.

The ATLAS files will be installed to /usr/local/atlas, you can grab them
remotely into a tar.gz file via:

	ssh roborio "cd /usr/local/atlas; tar -c ." | gzip > atlas.tar.gz
