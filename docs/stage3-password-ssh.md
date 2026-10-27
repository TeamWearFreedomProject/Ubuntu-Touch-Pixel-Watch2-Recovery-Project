# Stage 3: password-only SSH userdata for Pixel Watch 2 (eos)

This step creates **only** a 200 MiB `userdata.img` compatible with the [official eos README](https://gitlab.com/ubports/porting/community-ports/android13/google-eos/google-eos/-/raw/main/README.md). It uses the Stage 2 Ubuntu Touch 20.04 ARMHF image to seed the matching account database. **It never flashes or wipes the watch.**

## Set up the password

1. Open repository **Settings → Secrets and variables → Actions → New repository secret**.
2. Set the secret **Name** to `WATCH_SSH_PASSWORD`.
3. Set **Secret** to a **unique, long password** (at least 12 characters, preferably 16+). Do not use your GitHub password or a reused PIN. Never commit it or type it into workflow inputs, issues, or PR comments.
4. After merging the Stage 3 workflow PR, open **Actions → Ubuntu Touch recovery - prepare password-only SSH userdata → Run workflow**.
5. Leave `stage2_tag` as `stage2-40430296102` unless you intentionally use another Stage 2 image.
6. Download **eos-stage3-password-userdata** from the completed workflow run's **Artifacts** section. It contains `userdata.img`, `SHA256SUMS`, and `README-USERDATA.txt`. The artifact expires after **3 days**, but you may rerun the workflow while the Stage 2 draft release is available.

**Do not make this artifact public or publish it in a GitHub Release.** It contains the salted hash of your SSH password. Keep the image on your trusted devices only.

## Result

The generated ext4 image:
- Marks the Lomiri initial setup wizard as completed.
- Enables `ssh.service` at boot.
- Sets `phablet` (UID 32011)'s password in UBports' `/var/lib/extrausers/shadow` and preserves its other account files.
- Enables `PasswordAuthentication` and PAM; disables user public-key authentication and remote root login.
- Contains **no** `authorized_keys` file.

From any computer or phone with an SSH client, after boot and Wi-Fi setup:

```sh
ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no phablet@WATCH_IP
```

The SSH server still requires its **host keys** for encrypted connections; this removes only the need to manage a separate **client public/private key pair** on each connecting device.

**Security tradeoff:** Password logins can be attacked over untrusted networks. Use a long unique password and trusted Wi-Fi; never forward TCP port 22 from your router to the public internet. Wi-Fi roaming can change the watch's IP address, so discover `WATCH_IP` on the current network.

## Important flash warning

Creating `userdata.img` and checking its contents are non-destructive. **Flashing this image to the `userdata` partition will erase the device's current user data.** Verify firmware TWD9, current slot A, partition sizes and the separate Stage 2 boot/system images before considering any flash commands. This workflow deliberately does **not** contain any fastboot flashing commands.

## Upstream references

- [Pixel Watch 2 eos README](https://gitlab.com/ubports/porting/community-ports/android13/google-eos/google-eos/-/raw/main/README.md)
- [Ubuntu Touch rootfs builder (extrausers account setup)](https://ci.ubports.com/job/ubuntu-touch-rootfs/job/ubports%252Ffocal/357/console)
- [UBports SSH docs](https://docs.ubports.com/en/latest/userguide/advanceduse/ssh.html)
