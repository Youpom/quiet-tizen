# Recovery safety net — Samsung QA65QE1DAUXZN

**Your TV:** `QA65QE1DAUXZN` · firmware `T-NKLDUABC-0090-2123.9` · Tizen **9.0** · chassis **NKLD**
**Your Mac's LAN IP:** `192.168.1.172` · **TV IP:** `192.168.1.125`
**SDB:** `~/tizen-studio/tools/sdb`

> This TV is an **undocumented chassis** for rooting — no published success *or*
> failure reports, and no validated recovery recipe specific to it. So this net
> is built around **not needing drastic recovery in the first place**, then the
> ladder from gentlest to most drastic if you do. Read it fully *before* any
> change. Keep a copy on your phone (it's on your GitHub fork too).

---

## 0. The golden rules (these keep "mistake" from becoming "brick")

1. **Protect the SDB lifeline.** As long as `sdb connect` works, almost any
   *software* change is reversible. The things that kill the lifeline — and are
   therefore **forbidden**: editing `/etc/passwd`, replacing Samsung's SDB
   verification key, touching secure boot / the bootloader, or broad-deleting
   system directories. Never do these, for any reason.
2. **Snapshot before you write. Every time.** No database/system edit without a
   fresh backup pulled to your Mac *and* left on the TV first (see §2).
3. **One change → reboot → verify SDB still connects and the TV still works →
   next change.** Never batch. The moment something looks wrong, **stop** and
   recover; do not "try one more thing."
4. **Never re-run a failed step by deleting the original backup** to make it
   succeed. Inspect first.
5. **The temporary-root route changes nothing permanent just to *get* root** —
   so the real brick risk lives in the *cleanup DB edits* and the
   *home/startup reassignment*, not in the foothold. Treat those two as the
   danger zone.

---

## 1. Before you touch anything — pre-flight

- [ ] **Confirm SDB works:** `~/tizen-studio/tools/sdb connect 192.168.1.125:26101`
      then `sdb devices` shows `QA65QE1DAUXZN  device`.
- [ ] **Record the originals you'll need to restore** (write them down off-TV):
      the current Home/startup app id, and anything a cleanup step will change.
- [ ] **Full snapshots pulled to the Mac** — during the root session, the repo's
      `recipes/snapshot.sh` captures the system DBs with SQLite `.backup`; pull
      them under `.local/` and confirm the files are non-empty before any write.
- [ ] **Know your firmware recovery availability** (see §5) *before* you need it.
- [ ] **Keep Developer Mode ON and Host PC IP = 192.168.1.172** the whole time.

---

## 2. Snapshot & rollback (do this inside the root session, before edits)

The project is built for exactly this: **field-specific rollback, not whole-DB
replacement.** For each change you plan:

1. Capture the current rows/values (`recipes/snapshot.sh`), pull to `.local/`.
2. Write the **inverse** operation (the exact SQL/vconf to put it back).
3. Apply the single change.
4. Reboot, verify.
5. If wrong → apply your saved inverse (Tier 0 below).

Keep both a copy **on the TV** (`/opt/usr/share/quiet-tizen-backups/…`) and
**on the Mac** (`.local/`).

---

## 3. Recovery ladder — gentlest first

### Tier 0 — Roll back from your own snapshot *(if SDB still works)*
Your first and best option. Reconnect, re-acquire temporary root, apply the
inverse operations you saved in §2, reboot, verify. This is why §2 is
non-negotiable. If SDB connects, you are almost certainly fine.

If SDB got wedged (connects on the port but handshake fails):
```
~/tizen-studio/tools/sdb kill-server
~/tizen-studio/tools/sdb start-server
~/tizen-studio/tools/sdb connect 192.168.1.125:26101   # retry a few times
```
On the TV: toggle **Developer Mode off/on**, or restart the TV, to un-wedge the
dev service.

### Tier 1 — Factory reset from the menu *(if you can reach Settings)*
Undoes settings-level and most Smart-Hub-level breakage.
- **Settings → General & Privacy → Reset** → enter PIN (**default `0000`**).
- The TV wipes settings, reboots to the initial setup screen.
- This does **not** reinstall the OS, but it restores stock Home/startup and
  clears bad configuration.

### Tier 2 — Reset when Smart Hub is broken but the picture works
If the launcher/home is broken but you can still get into the settings menu
(via the remote's Settings/Menu button, or an HDMI input + the menu), do the
Tier 1 reset from there. The native Samsung menu stays available underneath
Quiet Home by design — use it.

### Tier 3 — USB firmware re-flash *(heavy recovery)*
Reinstalls the stock OS. Use if the TV boots but is badly broken, or shows a
firmware error. **Requires that Samsung publishes downloadable firmware for your
exact model** (not guaranteed for every 2024 model/region — check §5 now, not
later).
- Download the firmware for `QA65QE1DAUXZN` from your model's Samsung support
  page → **Downloads / Manuals & Software → Firmware**.
- Format a **≥4 GB USB stick as FAT32**, extract the firmware to its **root**.
- TV: **Settings → Support → Software Update → Update Now** (it reads the USB).
- **Do not remove the USB or cut power mid-update** — that is the one action
  that turns a soft problem into a mainboard failure.

### Tier 4 — Samsung service
If the screen is fully black with no response and no USB recovery is possible,
this is the backstop. Rooting voids the warranty; be aware a service center may
charge. A mainboard swap restores the TV.

---

## 4. Do **NOT** use the service / "secret" menu to recover
The hidden service menu can **worsen** a brick (there are documented cases of
TVs going dead after entering service mode). It is **not** a recovery tool.
Stay out of it.

---

## 5. Firmware for USB re-flash — DOWNLOADED & VERIFIED ✅
Stock firmware for this exact chassis is on the Mac at
`~/QA65QE1D-recovery-firmware/`, extracted and **MD5-verified** against
Samsung's own checksums:

| Folder | Image | Version | MD5 (verified) |
|---|---|---|---|
| `T-NKLDUABC-0090/` | standard reinstall (`upgrade.msd`) | **2123.9 — your exact firmware** | `607df6…b657b18` ✅ |
| `T-NKLDUABC/` | OSU deep OS recovery (`upgrade_OSU.msd`) | 6500 | `e5d5ea…bdc02734` ✅ |

**Put it on a USB stick (do this before any change):**
1. USB stick **≥8 GB, formatted FAT32** (not exFAT/NTFS — the TV won't read them).
2. Copy **both** folders (`T-NKLDUABC-0090` and `T-NKLDUABC`) to the **root** of
   the stick.
3. Keep the stick with the TV.

**To re-flash a soft-bricked TV that still powers on:**
- TV → **Settings → Support → Software Update → Update Now** → it detects the USB
  and reinstalls. **Never remove the USB or cut power mid-update** — that is the
  one action that turns a soft brick into a mainboard failure.
- `T-NKLDUABC-0090` restores your exact stock firmware; the OSU image is the
  deeper fallback.

Your Tier 3 net is real and ready: a soft-brick that still powers on can be
returned to stock. (It still cannot fix a hardware/backlight failure.)

---

## 6. Quick reference card

| Thing | Value |
|---|---|
| Model / firmware | `QA65QE1DAUXZN` / `T-NKLDUABC-0090-2123.9` |
| Platform | Tizen 9.0, ARMv7 |
| TV IP / Mac IP | `192.168.1.125` / `192.168.1.172` |
| Reconnect SDB | `sdb kill-server && sdb start-server && sdb connect 192.168.1.125:26101` |
| Factory reset | Settings → General & Privacy → Reset · PIN `0000` |
| Stock firmware (USB re-flash) | `T-NKLDUABC_OS_0090.zip` (~2.9 GB) from downloadcenter.samsung.com → extract → FAT32 USB |
| Un-wedge dev svc | Toggle Developer Mode off/on, or restart TV |
| Forbidden | passwd, SDB key, secure boot, bootloader, broad deletes, service menu |

---

*Prepared as a safety reference. It does not make the root step safe — it makes a
soft failure more likely to be recoverable. The strongest protection remains:
snapshot first, change one thing, verify, and keep the SDB lifeline intact.*
