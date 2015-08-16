### Maintainer: Michael Allen <mike.jallen at gmail dot com>
### Contributor: Michael Duell <mail@akurei.me> PGP-Key: 6EE23EBE

# Modified by JavaAtom <fausitan.merula@gmail.com> for 3.1.X kernel due to Intel IOMMU bug

###########################################################################################################
#                                          Patch and Build Options
###########################################################################################################
_menu="n"	     # menuconfig option [m = make menuconfig; x = make xconfig; n = none]
_config="pkg"	 # "local":  make localmod config - compile ONLY probed modules - see notes below!
               # "old":    make with old config (/proc/config.gz)
               # "pkg":    use this package's config
_compress_modules="y" #compress(gzip -9) modules to save about 100MB of space
###########################################################################################################
## LOCALMODCONFIG OPTION
 # As of mainline 2.6.32, running with this option will only build the modules that you currently have
 # probed in your system VASTLY reducing the number of modules build.
 #
 # WARNING - make CERTAIN that all modules are modprobed BEFORE you begin making the pkg!
 # Read, https://bbs.archlinux.org/viewtopic.php?pid=830221#p830221
 # To keep track of which modules are needed for your specific system/hardware, give graysky's module_db script
 # a try: http://aur.archlinux.org/packages.php?ID=41689
 #
 # Note that if you use graysky's script, this PKGBUILD will auto run the reload_data base for you to probe
 # all the modules you have logged!
 #

_basekernel=3.1
pkgver=${_basekernel}.9
pkgrel=1
_kernelname=-lqx
pkgname=linux${_kernelname}-${_basekernel}
_lqxpatchname="${pkgver}-1.patch"
arch=('i686' 'x86_64')
pkgdesc="Linux kernel and modules with Liquorix patches"
license=('GPL2')
groups=('base')
url="http://liquorix.net/"
backup=(etc/mkinitcpio.d/${pkgname}.preset)
depends=('coreutils' 'linux-firmware' 'module-init-tools>=3.16' 'mkinitcpio>=0.8')
optdepends=('crda: to set the correct wireless channels of your country')
if [ "$_menu" = "x" ]; then
   makedepends=('qt')
fi
options=(!strip)
# pwc, ieee80211 and hostap-driver26 modules are included in linux now
# nforce package support was abandoned by nvidia, kernel modules should cover everything now.
# kernel24 support is dropped since glibc24
replaces=('kernel24' 'kernel24-scsi' 'kernel26-scsi'
          'alsa-driver' 'ieee80211' 'hostap-driver26'
          'pwc' 'nforce' 'squashfs' 'unionfs' 'ivtv'
          'zd1211' 'kvm-modules' 'iwlwifi' 'rt2x00-cvs'
          'gspcav1' 'atl2' 'wlan-ng26' 'aufs' 'rt2500' 'kernel26-lqx')
install='linux-lqx.install'
source=(ftp://ftp.kernel.org/pub/linux/kernel/v3.x/linux-${_basekernel}.tar.bz2
http://liquorix.net/sources/${_lqxpatchname}.gz
http://liquorix.net/sources/${_basekernel}/config.i386
http://liquorix.net/sources/${_basekernel}/config.amd64
linux-lqx.preset)

provides=('linux-headers' "linux=$pkgver") # for when you have no other kernel

build() {
  KARCH=x86
  sed -i -e "s/+EXTRAVERSION = -lqx${_lqxrel}/+EXTRAVERSION =/" ${_lqxpatchname}
  cd ${srcdir}/linux-${_basekernel}

  # Add Liquorix patches
  patch -Np1 -i ${srcdir}/$_lqxpatchname
   
  # Trying oldcfg if possible and if selected
  if [ "$_config" = "old" ]; then
    if [ -e /proc/config.gz ]; then
      zcat /proc/config.gz > ./.config
    else
      echo "WARNING: There's no /proc/config.gz... You cannot use the old config. Aborting..."
      exit 1
    fi         
  else
    if [ "$CARCH" = "x86_64" ]; then
      cat ../config.amd64 >./.config
    else
      cat ../config.i386 >./.config
    fi
  fi

  if [ "${_kernelname}" != "" ]; then
    sed -i "s|CONFIG_LOCALVERSION=.*|CONFIG_LOCALVERSION=\"${_kernelname}\"|g" ./.config
  fi

  # get kernel version
  make prepare
  _kernver="$(make kernelrelease)"

  ### Optionally load needed modules for the make localmodconfig
  # See http://aur.archlinux.org/packages.php?ID=41689
  if [ $_config = "local" ]; then
    msg "If you have modprobe_db installed, running reload_database now"
    if [ -e /usr/bin/reload_database ]; then
      /usr/bin/reload_database
    fi
    msg "Running Steven Rostedt's make localmodconfig now"
    make localmodconfig
  else
    yes "" | make config
  fi

  if [ $_menu = "m" -o $_menu = "y" ]; then
    msg "Running make menuconfig"
    make menuconfig
  fi
  if [ $_menu = "x" ]; then
    msg "Running make xconfig"
    make xconfig
  fi

  # build!
  make ${MAKEFLAGS} bzImage modules

  ### package_linux

  mkdir -p ${pkgdir}/{lib/modules,lib/firmware,boot}
  make INSTALL_MOD_PATH=${pkgdir} modules_install

  cp System.map ${pkgdir}/boot/System.map26${_kernelname}
  cp arch/$KARCH/boot/bzImage ${pkgdir}/boot/vmlinuz-linux${_kernelname}

  # add vmlinux
  #cp vmlinux ${pkgdir}/usr/src/linux-${_kernver}
  install -m644 -D vmlinux ${pkgdir}/usr/src/linux-${_kernver}/vmlinux

  # install fallback mkinitcpio.conf file and preset file for kernel
  install -m644 -D ${srcdir}/linux-lqx.preset ${pkgdir}/etc/mkinitcpio.d/${pkgname}.preset

  # set correct depmod command for install
  sed \
    -e  "s/KERNEL_NAME=.*/KERNEL_NAME=${_kernelname}/g" \
    -e  "s/KERNEL_VERSION=.*/KERNEL_VERSION=${_kernver}/g" \
    -i ${startdir}/linux-lqx.install
  sed \
    -e "s|source .*|source /etc/mkinitcpio.d/linux${_kernelname}.kver|g" \
    -e "s|default_image=.*|default_image=\"/boot/initramfs-${pkgname}.img\"|g" \
    -e "s|fallback_image=.*|fallback_image=\"/boot/initramfs-${pkgname}-fallback.img\"|g" \
    -i ${pkgdir}/etc/mkinitcpio.d/${pkgname}.preset

  echo -e "# DO NOT EDIT THIS FILE\nALL_kver='${_kernver}'" > ${pkgdir}/etc/mkinitcpio.d/${pkgname}.kver

  ### package_linux-headers

  install -D -m644 Makefile \
    ${pkgdir}/usr/src/linux-${_kernver}/Makefile
  install -D -m644 kernel/Makefile \
    ${pkgdir}/usr/src/linux-${_kernver}/kernel/Makefile
  install -D -m644 .config \
    ${pkgdir}/usr/src/linux-${_kernver}/.config

  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/include
  for i in acpi asm-generic config generated linux math-emu media net pcmcia scsi sound trace video xen; do
    cp -a include/$i ${pkgdir}/usr/src/linux-${_kernver}/include/
  done

  # copy arch includes for external modules
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/arch/x86
  cp -a arch/x86/include ${pkgdir}/usr/src/linux-${_kernver}/arch/x86/

  # copy files necessary for later builds, like nvidia and vmware
  cp Module.symvers ${pkgdir}/usr/src/linux-${_kernver}
  cp -a scripts ${pkgdir}/usr/src/linux-${_kernver}

  # fix permissions on scripts dir
  chmod og-w -R ${pkgdir}/usr/src/linux-${_kernver}/scripts

  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/.tmp_versions
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/arch/$KARCH/kernel

  cp arch/$KARCH/Makefile ${pkgdir}/usr/src/linux-${_kernver}/arch/$KARCH/
  if [ "$CARCH" = "i686" ]; then
    cp arch/$KARCH/Makefile_32.cpu ${pkgdir}/usr/src/linux-${_kernver}/arch/$KARCH/
  fi
  cp arch/$KARCH/kernel/asm-offsets.s ${pkgdir}/usr/src/linux-${_kernver}/arch/$KARCH/kernel/

  # add headers for lirc package
  # note: zc0301 removed due to compiling errors
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/video
  cp drivers/media/video/*.h ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/video/
  for i in bt8xx cpia2 cx25840 cx88 em28xx et61x251 pwc saa7134 sn9c102; do
    mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/video/$i
    cp -a drivers/media/video/$i/*.h ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/video/$i
  done

  # add docbook makefile
  install -D -m644 Documentation/DocBook/Makefile \
    ${pkgdir}/usr/src/linux-${_kernver}/Documentation/DocBook/Makefile

  # add dm headers
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/drivers/md
  cp drivers/md/*.h  ${pkgdir}/usr/src/linux-${_kernver}/drivers/md

  # add inotify.h
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/include/linux
  cp include/linux/inotify.h ${pkgdir}/usr/src/linux-${_kernver}/include/linux/

  # add wireless headers
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/net/mac80211/
  cp net/mac80211/*.h ${pkgdir}/usr/src/linux-${_kernver}/net/mac80211/

  # add dvb headers for external modules
  # in reference to:
  # http://bugs.archlinux.org/task/9912
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb/dvb-core
  cp drivers/media/dvb/dvb-core/*.h ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb/dvb-core/

  # add dvb headers for external modules
  # in reference to:
  # http://bugs.archlinux.org/task/11194
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/include/config/dvb/
  [[ -e include/config/dvb/ ]] && cp include/config/dvb/*.h ${pkgdir}/usr/src/linux-${_kernver}/include/config/dvb/

  # add dvb headers for http://mcentral.de/hg/~mrec/em28xx-new
  # in reference to:
  # http://bugs.archlinux.org/task/13146
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb/frontends/
  cp drivers/media/dvb/frontends/lgdt330x.h ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb/frontends/
  cp drivers/media/video/msp3400-driver.h ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb/frontends/

  # add xfs and shmem for aufs building
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/fs/xfs
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/mm
  cp fs/xfs/xfs_sb.h ${pkgdir}/usr/src/linux-${_kernver}/fs/xfs/xfs_sb.h

  # add headers for virtualbox
  # in reference to:
  # http://bugs.archlinux.org/task/14568
  cp -a include/drm ${pkgdir}/usr/src/linux-${_kernver}/include/

  # add headers for broadcom wl
  # in reference to:
  # http://bugs.archlinux.org/task/14568
  cp -a include/trace ${pkgdir}/usr/src/linux-${_kernver}/include/

  # copy in Kconfig files
  for i in `find . -name "Kconfig*"`; do
    mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/`echo $i | sed 's|/Kconfig.*||'`
    cp $i ${pkgdir}/usr/src/linux-${_kernver}/$i
  done

  chown -R root.root ${pkgdir}/usr/src/linux-${_kernver}
  find ${pkgdir}/usr/src/linux-${_kernver} -type d -exec chmod 755 {} \;
  cd ${pkgdir}/lib/modules/${_kernver} && \
    (rm -f source build; ln -sf ../../../usr/src/linux-${_kernver} build)

  msg "Removing unneeded architectures..."
  # remove unneeded architectures
  rm -rf ${pkgdir}/usr/src/linux-${_kernver}/arch/{alpha,arm,arm26,avr32,blackfin,cris,frv,h8300,ia64,m32r,m68k,m68knommu,mips,microblaze,mn10300,parisc,powerpc,ppc,s390,sh,sh64,sparc,sparc64,um,v850,xtensa}
  msg "Removing the firmware..."
  # remove the firmware
  rm -rf ${pkgdir}/lib/firmware
  if [ $_compress_modules = "y" ]; then
    # gzip -9 all modules to save 100MB of space
    msg "Compressing modules..."
    find "$pkgdir" -name '*.ko' -exec gzip -9 {} \;
  fi
}

# vim:ts=2:sw=2:expandtab
# oh hai thar: checksums
sha512sums=('131b814325d888bb11ba552bc5bdd91ee2d8e139b06ac7ba94e716bcbbcc99ba30867eeea4aab580907916203bc526ae0bccda688151b250ba7943e193e50252'
            'f7be2717333dd5df0eeee850c859348dd13e03e546ea6a58674a2aa7ee826b3db8b2315234a04c7acc2db7722e4ba6661faf7be4e7ab9f7a565b393a9df0ecee'
            '2cf4b4e1ba55c1610fe61f654503b4aaa965a3aa2aac7270440d0c1213c5b5ad3ad7bff85bf01bb648fa816ff3a89737904cf5252b93e2f14dc2b34216973a7f'
            '9698effef8635d1b58d741ff5bfe9df3a426e338ebf4b0cace4dc0ac68e4a0c1312ee21a049b399dc0318ac9e6a4a01c6e400be2ad73197d89fab831b4dc8db0'
            '65fd484647b52ed8b75ab5d5f296ab1b79917ee83625bee9e74a8b3c1f9b155674001934c0a81e630997050816fdfd2b597eaf076838f250d61b51e4ec2aac16')
