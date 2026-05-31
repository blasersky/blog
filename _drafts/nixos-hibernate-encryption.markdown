---
layout: post
title:  "Setting up hibernation in NixOS with disk encryption"
tags: nixos encryption
---
I have installed NixOS on my computer using the graphical installer. I selected the options to enable hibernation and full disk encryption. After the setup there was an option to hibernate in the GUI, but it did not work properly. The system started afresh and did not resume. This is the solution that worked for me:
1. In the terminal type `sudo blkid`. You should see something like this:

        /dev/mapper/luks-a645xxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx: LABEL="root" UUID="1ca1xxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" BLOCK_SIZE="4096" TYPE="ext4"
        /dev/nvme0n1p3: UUID="9bb4xxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" TYPE="crypto_LUKS" PARTUUID="a61cxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
        /dev/nvme0n1p1: UUID="3F58-XXXX" BLOCK_SIZE="512" TYPE="vfat" PARTLABEL="EFI" PARTUUID="485fxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
        /dev/nvme0n1p2: UUID="a645xxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" TYPE="crypto_LUKS" PARTLABEL="root" PARTUUID="6df9xxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
        /dev/mapper/luks-9bb4xxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx: UUID="986fxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" TYPE="swap"

2. Ensure that the following lines are in the `configuration.nix`:

        boot.initrd.luks.devices."luks-a645xxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx".device =
        "/dev/disk/by-uuid/a645xxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx";
        boot.initrd.luks.devices."luks-9bb4xxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx".device =
        "/dev/disk/by-uuid/9bb4xxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx";
        swapDevices = [
          {
            device = "/dev/disk/by-uuid/986fxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx";
          }
        ];
        boot.resumeDevice = "/dev/mapper/luks-9bb4xxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx";

3. Type `sudo nixos-rebuild switch` in the terminal and reboot.
4. Hibernation should be working properly now.

I was also dissatisfied with the size of swap, so I made it larger. First I commented out any swap-related lines in `configuration.nix`, rebuilt the system and rebooted to check if the system boots. Then I booted from a live USB and shrank the main partition. The system booted fine. I again booted from the live USB, deleted the old swap partition and created a new one using the same password as for the main partition. The system again booted fine, but it had waited for two minutes for the old swap partition. I changed the UUID of the swap in `configuration.nix` and `hardware-configuration.nix` to the new value, rebuilt the system and rebooted. Everything worked fine.
        
I've probably commited a war crime by doing this, but it currently works; if it ain't broke, don't fix it. Providing a wrong answer to a question on the Internet is the most likely way to get a correct one, so if there's something very wrong with this solution, please notify me.
