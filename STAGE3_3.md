# Stage 3.3 — minimal official-layout userdata (NO AUTH; experimental)

Fresh 200 MiB ext4 userdata image for the **Google Pixel Watch 2 (eos/aurora)** running the existing Stage 2 Ubuntu Touch 20.04 ARMHF rootfs. It follows the upstream [google-eos README](https://gitlab.com/ubports/porting/community-ports/android13/google-eos/google-eos/-/raw/main/README.md) for the on-disk layout, wizard marker, ext4 switches, and single SSH service symlink. **Passwordless SSH is a non-official, dangerous customization.**

## Why Stage 3.3?

- **Stage 3.2 may hang at the G boot logo.** Its custom ssh.socket activation, host-key systemd service, and ordering drop-ins are removed. The exact cause of the hang is **not yet proven**.
- The official structure has user-data/phablet/.config/lomiri/wizard-has-run, a symlink to /usr/lib/systemd/system/ssh.service in system-data/etc/systemd/system/multi-user.target.wants/, and a 200M ext4 userdata image.
- To meet the request for **no login password or client public key**, the workflow also seeds Ubuntu Touch extrausers, clears the phablet password, permits PAM null passwords, and configures SSH to accept blank-password logins.
- SSH still **requires a server host key**, separate from client authorized_keys. We generate one during the Action rather than adding boot-time services.
- **This artifact is not tested on the real watch.** A successful GitHub Action proves file generation only.

## Building on a phone

1. Merge the reviewed Stage 3.3 pull request.
2. GitHub **Actions** → **Ubuntu Touch recovery - Stage 3.3 minimal official userdata (UNSAFE NO AUTH)** → **Run workflow**.
3. Keep Stage 2 tag set to **stage2-40430296102** for the previous rootfs.
4. Select **acknowledge_open_ssh** only if you accept the security risk.
5. Download the successful artifact **eos-stage3-3-MINIMAL-NOAUTH-userdata**. Extract userdata.img and check it is **209,715,200 bytes**.

## Flashing from Pixel 8 / Termux (after charging the watch)

**Stop until the watch battery is sufficiently charged. Flashing replaces existing userdata and Wi-Fi configuration.**

In the Watch 2 bootloader, run these read-only checks:

~~~sh
fastboot devices
fastboot getvar product
fastboot getvar current-slot
fastboot getvar is-userspace
~~~

Check that the device is your **Pixel Watch 2 (product: aurora, expected slot: a)**. If not already in fastbootd, use the existing fastboot reboot fastboot mode-switch and recheck is-userspace; USB permission prompts may reappear.

Then flash **only userdata** (adjust path if your phone extracted the ZIP elsewhere):

~~~sh
fastboot flash userdata /storage/emulated/0/Download/eos-stage3-3-MINIMAL-NOAUTH-userdata/userdata.img
fastboot reboot
~~~

Do **not** flash or resize the system, product, system_ext, or super partitions again. Do not run fastboot oem unlock; it is already unlocked.

When Ubuntu Touch boots fully, connect its Wi-Fi and use the newly shown IP:

~~~sh
ssh phablet@WATCH_IP
~~~

No client key or password is required if the service is running and PAM accepts the configuration. **Connection refused** is a problem with the listening service or network, not a password problem. Do not repeatedly flash.

## High-risk limitations

- **Everybody who can reach port 22 can attempt a shell with no credentials.** Never use public, school, or shared Wi-Fi; no router port forwarding.
- **The Action artifact contains a private SSH *server* host key.** Keep the artifact private; do not publish userdata.img in GitHub Releases. Remove copies when done.
- SSH traffic remains encrypted as required by the protocol. Omitting the *client* public key does not eliminate the server host key.
- If the G logo still hangs on a charged device, stop and collect boot logs or return to an image confirmed to boot. The new image is not guaranteed to fix it.
