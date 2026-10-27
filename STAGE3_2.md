# Stage 3.2 — Recover SSH on Pixel Watch 2 (eos)

**Experimental / unverified on the watch.** Stage 3.1 created a correct ext4 image but still yielded TCP port 22 `Connection refused` on the device. This revision changes **only** the 200 MiB `userdata.img` image build; do not reflash boot, init_boot or the Ubuntu rootfs just to test it.

## What changed

- Matches the [official google-eos README](https://gitlab.com/ubports/porting/community-ports/android13/google-eos/google-eos) by enabling `/usr/lib/systemd/system/ssh.service`.
- Also enables `ssh.socket` (see [UBports SSH documentation](https://docs.ubports.com/en/latest/userguide/advanceduse/ssh.html)) in `sockets.target.wants`.
- Generates device-unique SSH host keys at boot. Both SSH units depend on the host-key generator so they cannot start before it.
- Keeps the Stage 3.1 **passwordless** `phablet` profile and no client `authorized_keys`. A server host key and encrypted SSH transport **are still required by SSH**; the ext4 userdata image itself is **not encrypted**.
- Requires explicit `acknowledge_open_ssh: true` before GitHub Actions builds an image.

## Build using GitHub Actions

1. Merge the reviewed Stage 3.2 PR to `main`.
2. Open **Actions** → **Ubuntu Touch recovery - Stage 3.2 socket-backed SSH repair userdata** → **Run workflow**.
3. Leave Stage 2 tag `stage2-40430296102`, unless you intentionally built a newer matching rootfs.
4. Check the dangerous `acknowledge_open_ssh` box and run.
5. Download the successful artifact `eos-stage3-2-SOCKET-UNSAFE-userdata` and unzip it. Verify the extracted `userdata.img` is **209,715,200 bytes**; the ZIP is much smaller due to compression.

## Flash only the new userdata image

**WARNING: Flashing userdata erases the watch's current Ubuntu Touch user data, Wi-Fi configuration, and application data.** Check the device is the unlocked Pixel Watch 2 `eos` and that the bootloader connection is working. Stop if uncertain.

In PowerShell, from the platform-tools directory with the extracted image:

```powershell
.\fastboot.exe getvar product
.\fastboot.exe getvar is-userspace
.\fastboot.exe flash userdata .\userdata.img
.\fastboot.exe reboot
```

Use fastbootd (is-userspace: yes) for this port if your device requires it. Check that `flash userdata` reports `OKAY`; don't try to format/resize any other partitions.

When Ubuntu Touch boots, reconnect Wi-Fi and read the **new** watch IP address. From Windows:

```powershell
Test-NetConnection -ComputerName 192.168.8.188 -Port 22
ssh phablet@192.168.8.188
```

Replace `192.168.8.188` with the actual current watch IP. **Open port 22** is a prerequisite; if it is still refused, do not change credentials again: the SSH listener still failed to start. The Terminal app issue is independent and **not fixed by this build**.

## Security and recovery

**UNSAFE ON ANY SHARED OR UNTRUSTED NETWORK.** Anyone who can reach the watch on TCP 22 can attempt to obtain a `phablet` shell **without a password or authorized public key**. Use only an isolated, trusted test Wi-Fi; never expose port 22 to the Internet, never enable router port forwarding, and remove this rescue configuration after debugging.

There is no guarantee Stage 3.2 fixes the real cause of the device-level SSH startup failure. A green GitHub Actions run proves image assembly, **not** a running SSH daemon. If TCP 22 is still refused, the next step requires device boot/service logs (for example serial or another legitimate console route), rather than repeated blind flashes.
