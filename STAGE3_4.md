# Stage 3.4 — experimental passwordless Swipe lock-screen fix (Pixel Watch 2)

Stage 3.3 booted Ubuntu Touch on Pixel Watch 2 but showed an `Ubuntu → Passphrase` lock screen that would not unlock using an empty string. Stage 3.4 **keeps the same official google-eos userdata layout** and changes only the phablet account's password-aging metadata in addition to the existing Stage 3.3 no-auth configuration.

## Why this may fix the lock screen

Stage 3.3 emptied field 2 of `/var/lib/extrausers/shadow` for `phablet`, but copied field 3 (last password change, `sp_lstchg`) unchanged. AccountsService interprets a zero `sp_lstchg` as **PasswordMode = 1 (SET_AT_LOGIN)** even when the password hash is blank. Lomiri only selects **Swipe (no security)** when AccountsService exposes **PasswordMode = 2 (NONE)**.

Stage 3.4 sets `phablet` shadow field 2 to empty and field 3 to a positive UTC day number (the Actions build day). It verifies both fields and logs only the numeric original/new field 3 value, never the password hash. It does **not** patch boot, init_boot, system, super, product, system_ext or SSH services.

**This is a source-based hypothesis, not yet proven on your physical watch.** The build log will show `Stage 2 original last-change day:`; if it was already nonzero, look for a different cause before reflashing.

References:

- [Lomiri securityprivacy.cpp: getSecurityType](https://sources.debian.org/src/lomiri-system-settings/1.3.1-6/plugins/security-privacy/securityprivacy.cpp/) detects `PasswordMode=NONE` to select `Swipe`.
- [AccountsService user.c: user_local_update_from_pwent](https://sources.debian.org/src/accountsservice/0.6.21-8/src/user.c/) gives `sp_lstchg == 0` priority over empty passwords.
- [Official google-eos installation README](https://gitlab.com/ubports/porting/community-ports/android13/google-eos/google-eos/-/raw/main/README.md) defines the base userdata layout.

## Run on GitHub Actions (on Pixel 8)

1. Merge this Stage 3.4 pull request.
2. GitHub → Actions → **Ubuntu Touch recovery - Stage 3.4 passwordless Swipe unlock (UNSAFE NO AUTH)** → Run workflow.
3. Use the default Stage 2 release tag (`stage2-40430296102`) if it still exists as a draft; tick the explicit open-SSH danger acknowledgment.
4. After success, open the **Create official userdata tree** and **Clear UNIX password** step logs; verify `phablet: blank password, nonzero last-change day ...`.
5. Download `eos-stage3-4-SWIPE-NOAUTH-userdata`, extract `userdata.img` (209,715,200 bytes), and verify checksum before considering a flash.

## Optional non-destructive diagnosis before flashing

If Ubuntu Touch is connected to Wi-Fi even while locked, try `ssh phablet@WATCH_IP` from Termux. `Connection refused` indicates the SSH daemon or listening address is unavailable, **not** a password mismatch. If accessible, gather `id`, `getent group nopasswdlogin`, and DBus `PasswordMode` without changing anything.

## Only if you decide to reflash

Make sure the watch battery is sufficiently charged. **Flashing userdata destroys current user data, apps, and saved Wi-Fi**. Check `fastboot devices`, `fastboot getvar product` (expect aurora), `fastboot getvar current-slot` (previously a), and `fastboot getvar is-userspace` first.

After confirming exact target and extracted file location, in fastbootd:

```sh
fastboot flash userdata /storage/emulated/0/Download/eos-stage3-4-SWIPE-NOAUTH-userdata/userdata.img
fastboot reboot
```

Do **not** run `fastboot erase userdata` first; flashing a complete image already replaces the partition. No need to run `fastboot oem unlock`, and do not touch any other partition.

## Severe security warning

This image has no `phablet` password and intentionally allows SSH on TCP 22 without a client key or password. Anyone on a network that can reach the watch may take control of it. **Only use on an isolated private test network**; never on public/school Wi-Fi and never port-forward TCP 22. SSH transport encryption and a server host key are still mandatory. GitHub Action artifact includes a server private host key, so do not publish it publicly.
