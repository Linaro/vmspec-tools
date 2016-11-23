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

The default test artifacts are system qemu, /usr/share/qemu-efi/QEMU\_EFI.fd
and Ubuntu 16.4 LTS arm64 image.
