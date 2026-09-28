# BioShock 2 (single-player) – Mana and Ammo

**Target:** Bioshock2HD.exe (32-bit), [Steam/GOG, version]
**Tools:** Cheat Engine 7.x (Auto Assembler, AOB scanning)
**Scope:** Single-player campaign only.

## Approach
Instead of freezing addresses (which change every launch), each cheat is a
code injection found by AOB signature, so it survives restarts.

## 1. Unlimited mana (EVE)
- Found the writer with [how: scan for changing float, "find what writes"].
- The writer is a `movss [edi+CBC], xmm0`. Value is a float.
- Naive freeze fails because [reason]. Instead the hook compares the new value
  to the current one and skips the write when it's lower (i.e. being spent).
- Positive IEEE-754 floats order the same as integers, so a plain integer
  `cmp` works.

## 2. Unlimited ammo
- Clip: hook on `sub [esi], eax`, skip when the decrement is 1.
- Reserve: hook on `sub [ebx+54], eax`, skipped so reloads don't drain the reserve.
- [Side effects found, e.g. whether other items become free]

## 3. What I learned
- Wildcarded AOBs vs static addresses
- Overwriting whole instructions with a 5-byte jump and padding with nop
- [your own notes]

## Files
- `Bioshock_2.CT` – Cheat Engine table

