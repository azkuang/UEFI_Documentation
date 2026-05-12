# UEFI / EDK II Learning Path

## Context for New Sessions

The user has completed a "Hello World" UEFI `.efi` application and is working through a
structured curriculum to master the TianoCore EDK II framework, UEFI architecture, and
the UEFI Capsule Update mechanism.

**Test platform:** OVMF running under QEMU (no vendor-specific firmware stacks)
**Spec library:** `/home/wsdb/Documents/UEFI_Documentation/Split/`
- `UEFI_Spec/` — UEFI Specification 2.11
- `UEFI_PI/` — UEFI Platform Initialization Specification 1.9
- `ACPI/` — ACPI Specification

**All code examples must provide:** `.c` source, `.inf` module file, and `.dsc` integration
instructions.

**All architectural claims must cite** the spec name, chapter, and section number. Any
information not from the provided spec files must be flagged with:
> ⚠️ **Outside spec docs** — based on general EDK II knowledge, not the provided spec files.

---

## Progress Tracker

| Module | Topic | Status |
|--------|-------|--------|
| 0 | Hello World `.efi` | ✅ Complete |
| 1 | UEFI Image Model & EFI System Table | ✅ Complete |
| 2 | Boot Services: Events, TPL, Memory | ⬜ Not started |
| 3 | Runtime Services & NVRAM Variables | ⬜ Not started |
| 4 | PI Architecture: SEC → PEI → DXE | ⬜ Not started |
| 5 | UEFI Driver Model | ⬜ Not started |
| 6 | Firmware Update: FMP & Capsule | ⬜ Not started |

Update this table as modules are completed.

---

## Module 1 — The UEFI Image Model
**Phase context: BDS / UEFI Application space**

### Read
| Spec | File | Sections |
|------|------|---------|
| UEFI Spec 2.11 | `UEFI_Spec/3_2 Overview...` | §2.1 Boot Manager, §2.1.1 UEFI Images, §2.1.2 UEFI Applications, §2.1.4 UEFI Drivers |
| UEFI Spec 2.11 | `UEFI_Spec/5_4 EFI System Table...` | §4.1 UEFI Image Entry Point, §4.3 EFI System Table |

### Understand
Three image subtypes exist, distinguished by PE32+ Subsystem value (UEFI Spec §2.1.1):
- `EFI_IMAGE_SUBSYSTEM_EFI_APPLICATION` (10) — unloaded when entry point returns
- `EFI_IMAGE_SUBSYSTEM_EFI_BOOT_SERVICE_DRIVER` (11) — stays resident until `ExitBootServices()`
- `EFI_IMAGE_SUBSYSTEM_EFI_RUNTIME_DRIVER` (12) — stays resident forever; gets virtual address fixup via `SetVirtualAddressMap()`

The `.inf` `MODULE_TYPE` field controls which subsystem value the EDK II build tools emit.

### Build
Extend hello world to walk and print all fields of `EFI_SYSTEM_TABLE`:
- `FirmwareVendor`, `FirmwareRevision`
- `ConIn` / `ConOut` handle addresses
- `BootServices` and `RuntimeServices` table pointers

---

## Module 2 — Boot Services: Events, TPL, and Memory
**Phase context: DXE / BDS (before ExitBootServices)**

### Read
| Spec | File | Sections |
|------|------|---------|
| UEFI Spec 2.11 | `UEFI_Spec/8_7 Services — Boot Services...` | §7 intro, §7.1 Event/Timer/Task Priority Services, Table 7.2 TPL Usage |
| UEFI Spec 2.11 | same file | §7.2 Memory Allocation Services |

### Understand
Boot services split into two categories (UEFI Spec §7 intro):
- **Global** — available on all platforms (`AllocatePool`, `CreateEvent`, `LocateProtocol`)
- **Handle-based** — device-specific (`OpenProtocol`, `HandleProtocol`)

The TPL model (UEFI Spec §7.1, Table 7.2):
- `TPL_APPLICATION` — normal execution, blocking I/O allowed
- `TPL_CALLBACK` — interrupt-like, file system and disk I/O okay
- `TPL_NOTIFY` — no blocking; signal events and return immediately

`RaiseTPL()` / `RestoreTPL()` mask lower-priority notifications.

### Build
Write a UEFI application that:
1. Creates a periodic timer event with `CreateEvent()` + `SetTimer()`
2. Fires a callback at `TPL_CALLBACK` every 500ms that increments a counter
3. After 3 seconds, cancels the timer and prints the count

---

## Module 3 — Runtime Services & NVRAM Variables
**Phase context: RT (available before and after ExitBootServices)**

### Read
| Spec | File | Sections |
|------|------|---------|
| UEFI Spec 2.11 | `UEFI_Spec/9_8 Services — Runtime Services...` | §8 intro, §8.1 Runtime Services Rules, §8.2 Variable Services |
| UEFI Spec 2.11 | same file | `GetVariable`, `SetVariable`, `GetNextVariableName` function definitions |

### Understand
Variables are addressed by `{GUID namespace, CHAR16 name}` and carry attribute flags:
- `EFI_VARIABLE_NON_VOLATILE` — survives power loss (stored in flash)
- `EFI_VARIABLE_BOOTSERVICE_ACCESS` — visible during boot only
- `EFI_VARIABLE_RUNTIME_ACCESS` — visible to the OS

Capsule update state is persisted via NVRAM variables across the reboot required to apply
an update — this is revisited in Module 6.

### Build
Write a UEFI application that calls `GetNextVariableName()` in a loop to enumerate every
NVRAM variable, printing `GUID`, `Name`, and `DataSize`.

---

## Module 4 — PI Architecture: SEC → PEI → DXE
**Phase context: SEC, PEI, DXE — before UEFI services exist**

### Read
| Spec | File | Sections |
|------|------|---------|
| PI Spec 1.9 | `UEFI_PI/UEFI_PI_Spec_Pre-EFI-Initialization_Core_Interface.pdf` | Part I intro — PEI phase, PEI Services table, PPI services |
| PI Spec 1.9 | `UEFI_PI/UEFI_PI_Spec_Shared_Architectural_Elements.pdf` | Part III — Firmware Volume (FV) format, HOB list |
| PI Spec 1.9 | `UEFI_PI/UEFI_PI_Spec_Driver_Execution_Environment_Core_Interface.pdf` | DXE Core overview, DXE Services table |

### Understand
The PI Spec governs everything before UEFI services are available:

```
Power-on → SEC (no memory, cache-as-RAM)
         → PEI (PEIMs communicate via PPIs, produce HOBs)
         → DXE (protocols, bulk of firmware)
         → BDS → UEFI Shell / OS loader
```

Key concepts:
- **PPI** — PEI's equivalent of a UEFI protocol; pointer published into the PPI database
- **HOB** (Hand-Off Block) — data structures PEI produces and DXE consumes; the only cross-phase data channel
- **FV** (Firmware Volume) — on-flash container format holding PEIMs, DXE drivers, and raw data

### Build
Write a DXE driver (not an application):
- `MODULE_TYPE = DXE_DRIVER` in `.inf`
- Install a custom protocol GUID using `InstallMultipleProtocolInterfaces()`
- Add a `[Depex]` section depending on `gEfiVariableArchProtocolGuid`
- Observe the DXE dispatcher hold the driver until the dependency is satisfied

---

## Module 5 — UEFI Driver Model
**Phase context: DXE**

### Read
| Spec | File | Sections |
|------|------|---------|
| UEFI Spec 2.11 | `UEFI_Spec/12_11 Protocols — UEFI Driver Model...` | §11.1 EFI Driver Binding Protocol, §11.4 Supported(), Start(), Stop() |
| UEFI Spec 2.11 | `UEFI_Spec/10_9 Protocols - EFI Loaded Image...` | §9.1 EFI Loaded Image Protocol |
| UEFI Spec 2.11 | `UEFI_Spec/11_10 Protocols – Device Path Protocol...` | §10.1 Device Path Overview |

### Understand
`EFI_DRIVER_BINDING_PROTOCOL` has three functions:
- `Supported()` — "can I drive this handle?" (must not modify state)
- `Start()` — bind to the handle, install child protocols
- `Stop()` — detach cleanly

The separation of `Supported()` from `Start()` allows BDS connect-all-drivers to work
generically across unknown hardware.

### Build
Write a driver binding driver that:
1. In `Supported()`, checks whether the handle has `EFI_PCI_IO_PROTOCOL`
2. In `Start()`, installs a stub `EFI_DISK_INFO_PROTOCOL` on matching handles
3. Verify with the UEFI Shell `dh -d` command

---

## Module 6 — Firmware Update: FMP & Capsule
**Phase context: DXE (FMP installation) → RT (UpdateCapsule call) → PEI (CapsulePei on next boot)**

### Read
| Spec | File | Sections |
|------|------|---------|
| UEFI Spec 2.11 | `UEFI_Spec/24_23 Firmware Update and Reporting...` | §23.1 `EFI_FIRMWARE_MANAGEMENT_PROTOCOL`, §23.1.1 GUID, §23.1.2 `GetImageInfo()`, §23.1.3 `SetImage()` |
| UEFI Spec 2.11 | same file | `EFI_FIRMWARE_IMAGE_DESCRIPTOR` — `LastAttemptStatus`, `LastAttemptVersion` fields |
| UEFI Spec 2.11 | `UEFI_Spec/9_8 Services — Runtime Services...` | `UpdateCapsule()`, `QueryCapsuleCapabilities()` |

### Understand
`EFI_FIRMWARE_MANAGEMENT_PROTOCOL` (GUID `86C77A67-...`, UEFI Spec §23.1.1):
- `GetImageInfo()` — returns array of `EFI_FIRMWARE_IMAGE_DESCRIPTOR` per updatable image; includes `Version`, `LastAttemptVersion`, `LastAttemptStatus`
- `SetImage()` — accepts a new image buffer, validates via `CheckImage()`, programs the device
- `GetImage()` — retrieves current image (enables rollback)

Capsule update flow:
```
OS calls UpdateCapsule() [RT phase]
  → firmware sets OsIndications NVRAM variable
  → system reboots
  → CapsulePei [PEI phase] finds capsule in memory
  → CapsuleDxe [DXE phase] walks FMP instances, calls SetImage()
  → boot resumes, OS reads LastAttemptStatus from NVRAM
```

### Build
Write an FMP stub DXE driver:
1. `GetImageInfo()` returns one `EFI_FIRMWARE_IMAGE_DESCRIPTOR` with hardcoded version `1`
2. `SetImage()` prints the new image size and sets `LastAttemptStatus = LAST_ATTEMPT_STATUS_SUCCESS`
3. Run `CapsuleApp` in OVMF against your driver and observe the handoff

---

## Reference Map — Specs to Modules

| Module | Primary Spec | Supporting Spec |
|--------|-------------|-----------------|
| 1 — Image Model | UEFI Spec §2, §4 | — |
| 2 — Boot Services | UEFI Spec §7 | — |
| 3 — Runtime/NVRAM | UEFI Spec §8 | — |
| 4 — PI / PEI / DXE | PI Spec Part I, III | PI Spec Part II (DXE CIS) |
| 5 — Driver Model | UEFI Spec §11, §9, §10 | — |
| 6 — FMP / Capsule | UEFI Spec §23 | UEFI Spec §8 (UpdateCapsule) |

**Future Module 7 (optional):** ACPI table construction — how firmware builds the tables
the OS consumes. Spec: `ACPI/5_5 ACPI Software Programming Model...` §5 overview.
