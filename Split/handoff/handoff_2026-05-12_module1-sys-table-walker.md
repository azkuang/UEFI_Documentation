# Handoff — Module 1: EFI System Table Walker

**Date:** 2026-05-12
**Project:** UEFI/EDK II Learning Path
**Curriculum:** `/home/wsdb/Documents/UEFI_Documentation/Split/UEFI_LEARNING_PATH.md`

---

## Current State

Module 1 is **in progress**. The user has the boilerplate stub in place and has been
working through the spec explanations. They have not yet written any implementation code.

---

## What Has Been Covered This Session

### Spec sections explained
- **UEFI Spec §4.1.1** — `EFI_IMAGE_ENTRY_POINT`: the two parameters (`ImageHandle`,
  `SystemTable`), what each one is architecturally, and when to use each
- **UEFI Spec §4.2.1** — `EFI_TABLE_HEADER`: the `Signature` UINT64 ASCII encoding,
  the `Revision` decimal-split encoding (upper 16 bits = major, lower 16 bits = minor
  where `minor/10` and `minor%10` give the display digits)
- **UEFI Spec §4.3.1** — `EFI_SYSTEM_TABLE`: all fields — firmware identity,
  console handle/protocol pairs, `BootServices`, `RuntimeServices`,
  `ConfigurationTable` array

### Key concepts the user understands
- Why handle and protocol pointer are always separate fields (opaque device identity
  vs. callable interface)
- The Revision encoding: `EFI_2_110_SYSTEM_TABLE_REVISION = ((2<<16) | 110)` → `"2.11"`
- `BootServices` is invalid after `ExitBootServices()`; `RuntimeServices` persists
- `ConfigurationTable` entries are `{EFI_GUID, VOID*}` pairs pointing to ACPI RSDP,
  SMBIOS, etc.

### Mentor style rules established this session
- Do NOT write exercise implementations — user writes all logic themselves
- Explain spec sections with architectural focus: why things exist, relationships,
  and constraints — not just definitions
- Provide boilerplate scaffolding + spec walkthroughs, then answer questions

---

## Current File State

**Stub (user will fill in):**
`/home/wsdb/src/edk2/MyPkg/SysTableWalker/SysTableWalker.c`

```c
#include <Uefi.h>
#include <Library/UefiLib.h>

EFI_STATUS
EFIAPI
UefiMain (
  IN EFI_HANDLE        ImageHandle,
  IN EFI_SYSTEM_TABLE  *SystemTable,
)
{
  return EFI_SUCCESS;
}
```

**INF:** `/home/wsdb/src/edk2/MyPkg/SysTableWalker/SysTableWalker.inf` — already written and builds cleanly

**DSC:** `MyPkg/SysTableWalker/SysTableWalker.inf` already added to `[Components]` in `/home/wsdb/src/edk2/MyPkg/MyPkg.dsc`

**Build command (verified working):**
```
cd /home/wsdb/src/edk2
source edksetup.sh
export PATH="$PATH:/home/wsdb/src/edk2/BaseTools/BinWrappers/PosixLike"
build -a X64 -t GCC5 -p MyPkg/MyPkg.dsc -m MyPkg/SysTableWalker/SysTableWalker.inf
```

Output lands at: `Build/MyPkg/DEBUG_GCC5/X64/SysTableWalker.efi`

---

## Next Action for New Agent

The user is ready to write their implementation. Your job is to:

1. Answer questions about spec concepts as they arise
2. Help debug build errors or unexpected output in OVMF
3. Do NOT write the implementation — only provide hints, explain concepts, and unblock

When the user finishes and the output in OVMF looks correct (all fields printed,
Revision decoded to `"2.11"`, ConfigurationTable GUIDs formatted correctly), mark
Module 1 complete in `UEFI_LEARNING_PATH.md` and move on to Module 2.
