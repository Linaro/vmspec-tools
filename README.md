# vmspec-tools
Tools for verifying vmspec compliance
Two ways to run the tool:
## Image verification
Test offline if an image is formatted properly:

```bash
$ sudo apt install qemu-system cloud-utils
$ sudo ./vmspec-verify -i distro-version-cloud.img
VMSPEC-IMAGE-GPT: PASS
VMSPEC-IMAGE-ESP: PASS
VMSPEC-IMAGE-BOOTAA64: PASS
skipping runtime tests
```

## Runtime verification
Boot the image with an hypervisor, and run vmspec-verify inside

```bash
$ qemu-system-aarch64 -m 1024 -cpu cortex-a57 -M virt \
  -device virtio-blk-device,drive=image -drive if=none,id=image,file=centos7-cloud-image.qcow2 \
  -netdev bridge,id=user0 -device virtio-net-device,netdev=user0 -nographic
...
localhost login:
...
# ./vmspec-verify 
VMSPEC-IMAGE-GPT: PASS
VMSPEC-IMAGE-ESP: PASS
VMSPEC-IMAGE-BOOTAA64: PASS
VMSPEC-EFI: PASS
VMSPEC-EFI-DTB: PASS
VMSPEC-EFI-ACPI: PASS
VMSPEC-OS-EFI: PASS
VMSPEC-OS-EFI-RTC: PASS
VMSPEC-OS-ACPI: PASS
```

## Full verification

```bash
Usage: vmspec-boot [-h] [-i IMAGE] [-f FIRMWARE] [-q QEMU] [-s] [-p]

VMSPEC boot testing

optional arguments:
  -h, --help            show this help message and exit
  -i IMAGE, --image IMAGE
                        Hard drive Image
  -f FIRMWARE, --firmware FIRMWARE
                        Firmware to test
  -q QEMU, --qemu QEMU  Qemu to test
  -s, --silent          Hide QEMU output
  -p, --persistent      Test persistent UEFI variables
```

To boot default test artifacts with UEFI persistent variable tests:

```bash
# ./vmspec-boot -p
...
```
This will boot the image twice, first with vgic-v2 and writing an
UEFI variable, and second time with vgic-v3 and reading the variable
back. The tests will use kvm if run natively on aarch64, and tcg else.

## Test artefacts

By default the test will use the following:

* system installed qemu-system-aarch64 (from the qemu-system-arm package)
* /usr/share/qemu-efi/QEMU\_EFI.fd (from qemu-efi package)
* Ubuntu 16.10 LTS arm64 image (downloaded from https://cloud-images.ubuntu.com)

## Distros

For convenience we provide a list of distro images which can be used for
testing with this tool:
* http://dl.fedoraproject.org/pub/fedora-secondary/releases/25/CloudImages/aarch64/images/
* http://builds.96boards.org/snapshots/developer-cloud/debian/cloud-image/15/debian-cloud-image.qcow2
* http://builds.96boards.org/snapshots/developer-cloud/centos/cloud-image/26/centos7-cloud-image.qcow2
* https://cloud-images.ubuntu.com/yakkety/current/yakkety-server-cloudimg-arm64.img
* https://cloud-images.ubuntu.com/xenial/current/xenial-server-cloudimg-arm64-uefi1.img
* http://download.opensuse.org/ports/aarch64/distribution/leap/42.2/appliances/


Note that not all all of these work with the vmspec-boot tool as it relies on
the image using cloud-init.  Other images can be verified externally using
vmspec-verify -i and booted manually (see vmspec-boot for an example QEMU
command line) and cloning this tool inside the guest and running vmspec-verify
from within the guest as well.

## Limitations

* If this system is run on hardware with KVM that doesn't support GICv3 or
  GICv2, the corresponding test will be reporte as FAIL.
* We have successfully tested this tool with QEMU v2.6.2, Linux host (KVM) v4.7,
  and the Ubuntu 16.10 cloud image.
