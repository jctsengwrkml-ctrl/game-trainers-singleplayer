# BioShock 2 (single-player): Mana and Ammo

**Target:** `Bioshock2HD.exe` (32-bit) · single-player campaign only
**Tools:** Cheat Engine (Auto Assembler, AOB scanning)
**Game version:** [Steam/GOG + build, check the exe properties]

## Approach
The table uses code injection located by AOB signature, so it survives game
restarts. An early static-address entry was dropped for exactly this reason:
pointers into game memory change every launch. Each script backs up the
original bytes and restores them on disable.

## 1. Unlimited mana (EVE)
- **Hook:** `Bioshock2HD.exe+85D104`, `movss [edi+CBC], xmm0`. The mana value
  is a 4-byte float stored at offset `0xCBC` of the player object.
- **Context:** the instructions just before it clamp the new value against
  bounds and pick the result into `xmm0`, so this is the final store of a
  set-mana routine.
- **Behavior:** instead of freezing the value, the hook compares the new value
  with the current one and skips the write if it is lower (mana being spent).
  Increases still go through.
- **Detail:** for positive IEEE-754 floats, the bit patterns sort the same as
  integers, so a plain integer `cmp` works.
- **Patch size:** the original instruction is 8 bytes. A 5-byte `jmp` plus
  `nop 3` overwrites it exactly.

## 2. Unlimited ammo (clip)
- **Hook:** `Bioshock2HD.exe+BA9140`, `sub [esi], eax`, the ammo decrement.
- **Behavior:** when the amount subtracted is 1 (one shot), the subtraction is
  skipped. A check that `esi` is above `0x01000000` makes the hook ignore
  stack and low-memory addresses.
- **Patch size:** the 5-byte `jmp` overwrites two instructions (2 + 3 bytes),
  so the hook re-executes the second one (`mov eax,[ebp+0C]`) itself.
- **Not yet tested:** because this looks like a generic decrement, other
  items that drop by 1 may also become free.

## 3. Unlimited ammo v2 (reserve)
- **Hook:** `sub [ebx+54], eax`, the reserve decrement on reload. It follows
  an `add [esi+54], ecx`, which is the ammo moving into the clip.
- **Behavior:** the `sub` is skipped, so reloading no longer drains the reserve.
- **Patch size:** the hook is placed at `INJECT+3` so a 5-byte `jmp` plus one
  `nop` covers the 6 bytes exactly.

## What I learned
- AOB signatures with wildcards vs static addresses
- Overwriting whole instructions with a 5-byte `jmp` and padding with `nop`
- Replicating displaced instructions inside the injected code
- Why comparing to the current value works better than freezing it
- Restoring the exact original bytes on `[DISABLE]`

## Known limitations
- Side effects on other entities or items haven't been tested.
- AOB signatures may break on other game builds.

## Files
- `Bioshock_2.CT`: Cheat Engine table
