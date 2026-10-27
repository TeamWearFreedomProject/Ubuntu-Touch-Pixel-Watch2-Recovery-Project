# Pixel Watch 2 postmarketOS Stage 1

This workflow **does not flash** a device. It checks the actively maintained upstream pmbootstrap command line and downloads the SHA256-verified official Alpine Linux **aarch64** minirootfs.

**This is an Alpine baseline, not a finished postmarketOS google-eos root filesystem.** Proper pmbootstrap installation images require a device package and suitable kernel packaging first.

## Run on GitHub
Merge this PR. Under Actions select **Pixel Watch 2 postmarketOS Stage 1 - toolchain and ARM64 baseline**, click **Run workflow**, then retrieve the short-lived artifact **eos-pmos-stage1-arm64-baseline**. The job includes pmbootstrap version/commit and the SHA256 file.

## Stage 2
Create and validate the device-google-eos package, choose armhf versus aarch64, examine the known working AsteroidOS kernel USB Gadget setup, and establish boot logging before touching boot partitions. The Ubuntu Touch TWD9 flash layout is not automatically valid for a postmarketOS image.
