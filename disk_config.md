
# NixOS Disk Configuration 💿

This section documents the declarative disk configuration used for this NixOS setup. We use **Disko**, a Nix community utility that provides a declarative way to manage disk partitioning, formatting, and mounting. This eliminates the need for manual steps during installation, making system re-installation and setup significantly easier and more reproducible.

Disko configurations are defined in a Nix file, which describes the entire disk layout, including partitions, filesystems (like ext4 or ZFS), and mount points. The tool is flexible and supports various partitioning schemes like GPT and MBR, and multiple filesystems.

## Disko Usage

To apply the disk configuration, you run `disko` directly from the NixOS installer environment. The command partitions, formats, and mounts the specified device based on the configuration file.

The `disko-vm.nix` file in this repository defines the disk layout for my virtual machine. Below is the command to apply this configuration. It specifies `/dev/vda` as the target disk and uses the `destroy`, `format`, and `mount` modes to prepare the disk from scratch.

```bash
sudo nix --experimental-features "nix-command flakes" run github:nix-community/disko/latest -- --mode destroy,format,mount ./disko-vm.nix
````

## Current VM Disk Layout 💻

The following Nix code from the `disko-vm.nix` file outlines the partition scheme for a virtual machine. It creates a **GPT** partition table on the device `/dev/vda`, with separate partitions for the **EFI System Partition (ESP)**, the **root filesystem (`/`)**, and two **swap partitions**.

```nix
{
  disko.devices = {
    disk = {
      main = {
        device = "/dev/vda";
        type = "disk";
        content = {
          type = "gpt";
          partitions = {
            ESP = {
              size = "500M";
              type = "EF00";
              content = {
                type = "filesystem";
                format = "vfat";
                mountpoint = "/boot";
                mountOptions = [ "umask=0077" ];
              };
            };
            root = {
              end = "-1G";
              content = {
                type = "filesystem";
                format = "ext4";
                mountpoint = "/";
              };
            };
            encryptedSwap = {
              size = "10G";
              content = {
                type = "swap";
                randomEncryption = true;
                priority = 100;
              };
            };
            plainSwap = {
              size = "100%";
              content = {
                type = "swap";
                discardPolicy = "both";
                resumeDevice = true;
              };
            };
          };
        };
      };
    };
  };
}
```
