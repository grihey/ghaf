# Ghaf specific Microvm hypervisor options

Microvm is the component defining the virtual machines launch services generated for systemd. 
It inputs a set of options which are mapped to hypervisor command line call. Nevertheless is may 
happen that some hypervisor options are not supported by microvm  - e.g. adding specific devices.
This document considers such cases.


### Options definitions

A virtual machine is defined under Ghaf’s subdirectory ``microvmConfigurations/VM_NAME/default.nix``, e.g.:

```
microvmConfigurations/memshare/default.nix
https://github.com/jkuro-tii/ghaf/blob/main/microvmConfigurations/memshare/default.nix
```

This file contains hypervisor’s options for running the VM. For each hypervisor there is a bunch of microvm’s defined options:
[https://astro.github.io/microvm.nix/options.html]

The way they are processed can be found in corresponding ``.nix`` files (runners) in the microvm repository:

[https://github.com/astro/microvm.nix/blob/main/lib/runners/crosvm.nix]

[https://github.com/astro/microvm.nix/blob/main/lib/runners/qemu.nix]


Setting hypervisor option has form of: ``microvm.option = value;`` , e.g.:

```
microvm.mem = 512;
microvm.vcpu = 2;
```

### Generated hypervisor start commands

In result of building of the Ghaf tree there are generated command lines for starting the virtual machines. They reflect all parameters specified above -
both those specified explicitly and defaults. They are located under the Ghaf’s ``/var/lib/microvms/ directory``, e.g.:

```
ls /var/lib/microvms/memsharevm-vm-debug/current/bin
```

```
microvm-balloon
microvm-console
microvm-run
microvm-shutdown
```

The command which starts the hypervisor is the microvm-run bash script. Here is a sample generated:

```
 #! /nix/store/96ky1zdkpq871h2dlk198fz0zvklr1dr-bash-5.1-p16/bin/bash -e
exec '/nix/store/zsf59dn5sak8pbq4l3g5kqp7adyv3fph-qemu-host-cpu-only-7.1.0/bin/qemu-system-x86_64' '-
name' 'memshare' '-M' 'microvm,accel=kvm:tcg,x-option-roms=off,isa-serial=off,pit=off,pic=off,rtc=off,
mem-merge=on' '-m' '2512' '-cpu' 'host' '-smp' '17' '-machine' 'virt,accel=kvm' '-nodefaults' '-no-
user-config' '-nographic' '-no-reboot' '-serial' 'null' '-device' 'virtio-serial-device' '-chardev'
'pty,id=con0' '-device' 'virtconsole,chardev=con0' '-chardev' 'stdio,mux=on,id=con1,signal=off' '-
device' 'virtconsole,chardev=con1' '-device' 'virtio-rng-device' '-drive' 'id=root,format=raw,
media=cdrom,file=/nix/store/xnnqb3sb1l4kbx7s0ijazph5r0c0xhx5-rootfs.squashfs,if=none,aio=io_uring' '-
device' 'virtio-blk-device,drive=root' '-kernel' '/nix/store/ds5cmyby0p4ikw91afmrzihkz351kls7-linux-
6.2/bzImage' '-append' 'console=hvc1 console=hvc0 reboot=t panic=-1 root=/dev/vda ro init=/init
devtmpfs.mount=0 stage2init=/nix/store/0mbhpna8hplbsaz1il3n99f0zincr4vs-nixos-system-memshare-
22.11.20230310.824f886/init boot.panic_on_fail loglevel=4 regInfo=/nix/store
/j8id92qsd58qjnzq4xz6v5l38rlpq6is-closure-info/registration' '-sandbox' 'on' '-qmp' 'unix:memshare.
sock,server,nowait' '-device' 'virtio-balloon' '--option 1 --option 2'
```

for the input parameters:
```
microvm.hypervisor = "qemu";
```
Note that microvm sets several others.

```
microvm.mem = 2000;
microvm.balloonMem = 512;
microvm.vcpu = 17;
microvm.qemu.extraArgs = [ "--option 1 --option 2" ];
```
### Adding an option to the hypervisor command line

Microvm may not supply parameters for all possible options - e.g. adding specific devices. Processing of all microvm configuration options is
done in the mentioned above hypervisor’s runner .nix file

The runners support the ``extraArgs`` parameter. It allows setting any option in qemu command line invocation. Its value is a list of strings. In this
example the following ``extraArgs`` definition:

```
microvm.qemu.extraArgs = [
"-object memory-backend-file,id=mem1,mem-path=/dev/shm/virtio_pmem.img"
"-device virtio-pmem-pci,memdev=mem1,id=nv1"
];
```

results in the generated command line parameters:

```
'-object memory-backend-file,id=mem1,mem-path=/dev/shm/virtio_pmem.img' '-device v
irtio-pmem-pci,memdev=mem1,id=nv1'
```

## Note:

Support for the crosvm’s ``extraArgs`` parameter was added on April 7 2023. Verify that your ``flakes.lock`` file refers to the proper version.
