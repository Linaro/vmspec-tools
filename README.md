# vmspec-tools
Tools for verifying vmspec compliance
Two ways to run the tool:
## Image verification
Test offline if an image is formatted properly:

```bash
# sudo ./vmspec-verify -i ubuntu-16.04-server-cloudimg-arm64-uefi1.img
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

