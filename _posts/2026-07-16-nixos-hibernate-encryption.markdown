---
layout: post
title:  "setting up hibernation in NixOS with disk encryption"
tags: nixos encryption
---
i have installed NixOS on my computer using the graphical installer. i selected the options to enable hibernation and full disk encryption. after the setup there was an option to hibernate in the GUI, but it did not work properly. the system started afresh and did not resume. this is the solution that worked for me:
1. in the terminal type `sudo blkid`. you should see something like this:

        /dev/mapper/luks-a645xxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx: LABEL="root" UUID="1ca1xxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" BLOCK_SIZE="4096" TYPE="ext4"
        /dev/nvme0n1p3: UUID="9bb4xxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" TYPE="crypto_LUKS" PARTUUID="a61cxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
        /dev/nvme0n1p1: UUID="3F58-XXXX" BLOCK_SIZE="512" TYPE="vfat" PARTLABEL="EFI" PARTUUID="485fxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
        /dev/nvme0n1p2: UUID="a645xxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" TYPE="crypto_LUKS" PARTLABEL="root" PARTUUID="6df9xxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
        /dev/mapper/luks-9bb4xxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx: UUID="986fxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" TYPE="swap"

2. ensure that the following lines are in the `configuration.nix`:

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

3. type `sudo nixos-rebuild switch` in the terminal and reboot.
4. hibernation should be working properly now.

i was also dissatisfied with the size of swap, so i made it larger. first i commented out any swap-related lines in `configuration.nix`, rebuilt the system and rebooted to check if the system boots. then i booted from a live USB and shrank the main partition. the system booted fine. i again booted from the live USB, deleted the old swap partition and created a new one using the same password as for the main partition. the system again booted fine, but it had waited for two minutes for the old swap partition. i changed the UUID of the swap in `configuration.nix` and `hardware-configuration.nix` to the new value, rebuilt the system and rebooted. everything worked fine.
        
i've probably commited a war crime by doing this, but it currently works; if it ain't broke, don't fix it. providing a wrong answer to a question on the internet is the most likely way to get a correct one, so if there's something very wrong with this solution, please notify me.
