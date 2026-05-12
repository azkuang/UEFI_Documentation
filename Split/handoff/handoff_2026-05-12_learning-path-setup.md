# Handoff — UEFI Learning Path Setup
**Date:** 2026-05-12
**Session summary:** Established the full learning curriculum, file conventions, and
session tooling. No code has been written yet beyond the user's existing hello world.

---

## Who the User Is

A systems programmer learning UEFI/EDK II firmware development from scratch. The end
goal is mastering the Capsule Update mechanism (FMP, CapsulePei, CapsuleDxe). They
want architectural understanding paired with hands-on EDK II exercises at each step —
not just tutorials.

---

## Working Environment

- **Project directory:** `/home/wsdb/Documents/UEFI_Documentation/Split/`
- **Test platform:** OVMF under QEMU (no vendor-specific firmware stacks)
- **Spec library:**
  - `UEFI_Spec/` — UEFI Specification 2.11
  - `UEFI_PI/` — UEFI Platform Initialization Specification 1.9
  - `ACPI/` — ACPI Specification

---

## Rules for This Project

1. **Always cite specs** — every technical claim needs spec name, chapter, and section
   number. Flag anything not from the provided docs with:
   > ⚠️ **Outside spec docs** — based on general EDK II knowledge, not the provided spec files.

2. **All code examples require three files:** `.c` source, `.inf` module file, and `.dsc`
   integration instructions.

3. **Always search spec files first** before answering any technical question.

4. **Do not load handoff documents automatically** — the user will reference specific
   ones explicitly.

---

## Current State

**Completed:**
- Hello World `.efi` application (user built this before this session)
- Full 6-module learning path written and saved to:
  `UEFI_LEARNING_PATH.md` (in the project directory)
- `handoff/` directory and `handoff/README.md` index created

**Not started:**
- Module 1 through Module 6 (all exercises are pending)

---

## The Learning Path (summary)

Full details with spec citations are in `UEFI_LEARNING_PATH.md`.

| Module | Topic | Key Spec |
|--------|-------|----------|
| 1 | UEFI Image Model & EFI System Table walker | UEFI Spec §2, §4 |
| 2 | Boot Services: Events, TPL, Memory | UEFI Spec §7 |
| 3 | Runtime Services & NVRAM variable enumeration | UEFI Spec §8 |
| 4 | PI Architecture: SEC → PEI → DXE, HOBs, FVs | PI Spec Part I, III |
| 5 | UEFI Driver Model: DriverBinding, Supported/Start/Stop | UEFI Spec §11 |
| 6 | FMP & Capsule Update end-to-end | UEFI Spec §23 |

---

## Next Action for the New Agent

Start **Module 1**. The user is ready to write code.

Provide a complete EDK II implementation of a UEFI application that walks and prints
all fields of `EFI_SYSTEM_TABLE` (FirmwareVendor, FirmwareRevision, ConIn/ConOut handle
addresses, BootServices and RuntimeServices table pointers).

Before writing code, read the relevant spec sections:
- `UEFI_Spec/3_2 Overview-2.1 Boot Manager.pdf` — §2.1.1 UEFI Images, §2.1.2 UEFI Applications
- `UEFI_Spec/5_4 EFI System Table-4.1 UEFI Image Entry Point.pdf` — §4.1, §4.3

Deliver all three files: `.c`, `.inf`, `.dsc` integration instructions.
