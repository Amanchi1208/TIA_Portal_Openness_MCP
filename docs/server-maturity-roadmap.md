# Server Maturity Roadmap — closing the gap vs T-IA Connect / Openness Manager

Status date: 2026-06-16. This is the C#/server work the **skill cannot deliver**
(those are capabilities, not documentation). Prioritized by *value × feasibility*.
Each item: what, why it matters, Openness feasibility, rough effort, first step.

Competitor reference points: **T-IA Connect** (REST+MCP, full F-safe, VCI/Git, UMAC,
SiVArc) and **TIA Openness Manager** (PLCSIM unit tests, fingerprint diff, OPC UA
browser, encrypted vault, Git UI). See SKILL.md §18 for the full matrix.

---

## P0 — Finish what's started

### 0. Fix the V21 download cast bug — DONE (2026-06-17)
- **What:** `DownloadToPlc` failed on V21 with the `ConnectionConfiguration` → `IConfiguration`
  cast (SKILL.md §13). **Fixed:** navigate to a `ConfigurationTargetInterface` (the actual
  `IConfiguration`) via `Modes → PcInterfaces → TargetInterfaces` and pass it to `Download()`;
  also fixed the `StopModules` selection (`StopAll`, not the non-existent "StopModule").
- **Verified:** end-to-end on a real S7-1200 (江夏 安全PLC) → `state=Success, 0 errors`.
- **Follow-up (optional):** the route auto-selects the FIRST PG/PC interface — add a
  `targetInterfaceName` parameter if multi-NIC selection becomes a problem.

---

## P1 — Highest parity value

### 1. Safety (F-block) support
- **What:** list / read / export / import F-blocks; read **F-signature**; safety
  **login/logoff** (safety password); optionally trigger F-program compile.
- **Why:** both competitors lead with this; SKILL.md §4 currently says "must be done
  in TIA UI manually". Biggest single credibility gap for safety projects (江夏 安全PLC
  is already in scope).
- **Feasibility:** **Good** for read/export/import/signature — Openness `Siemens.
  Engineering.Safety` is already imported in `Portal.cs`; T-IA Connect proves the
  surface exists V16–V21. **Login/logoff** feasible (safety password handler).
  **Authoring brand-new F-FCs** with safety instructions is the hard tail — defer.
- **Effort:** M (read/signature/export) → L (login + compile).
- **First step:** add `[L2] GetSafetyProgramInfo` (signature, F-collective sig,
  safety mode) read-only tool over `SafetyAdministration` / failsafe data; ship it
  draft-Release and verify on 安全PLC before adding write paths.

### 2. PLCSIM Advanced simulation + unit testing
- **What:** start a virtual S7-1500, download to it, drive inputs, assert outputs;
  AI-generated test suites are the Openness-Manager headline.
- **Why:** lets the whole "author → compile → **prove it works**" loop run without
  hardware — turns the MCP from a generator into a verifier.
- **Feasibility:** **Separate dependency.** Not Openness — needs the **PLCSIM
  Advanced API** (`Siemens.Simatic.Simulation.Runtime`, separate install/license).
  Clean to add as an optional module that no-ops when PLCSIM Advanced is absent.
- **Effort:** L (new runtime integration + test harness format).
- **First step:** spike `Siemens.Simatic.Simulation.Runtime` instance create/start
  behind a `[L2] PlcSimStart/Stop/SetIO/ReadIO` set; gate on availability like the
  OPC UA reader does.

---

## P2 — Rounding out

### 3. OPC UA write + method calls + subscriptions
- **What:** today only **read** (`ReadPlcLiveValuesOpcUa`). Add write, method
  invoke, and an address-space browse/subscribe (Openness Manager's "AI Canvas").
- **Why:** read-only is half a client; commissioning needs supervised writes.
- **Feasibility:** Good — `Workstation.UaClient` is already a dependency.
- **Effort:** M. Guard writes behind an explicit safety gate (these touch a live CPU).

### 4. Native diff / Git helper (over §16 text export)
- **What:** wrap `ExportBlocksAsDocuments` into a one-call "snapshot to git dir +
  diff vs last snapshot"; optional fingerprint compare like the competitors.
- **Why:** code review / change history without the licensed VCI feature.
- **Feasibility:** Good — pure orchestration of existing export tools + git CLI.
- **Effort:** S–M. Lowest risk; mostly glue. The text-export workaround (§16)
  already covers the 80% case, so this is convenience, not a blocker.

### 5. Block protection / know-how protect
- **What:** `Protect` / `Unprotect` over `PlcBlockProtectionProvider` (the reference
  `TiaHelper.cs` already shows the call shape).
- **Why:** IP protection on delivered blocks; Openness Manager has an encrypted vault.
- **Feasibility:** Good (proven call shape in-repo). **Effort:** S.

---

## Explicitly NOT planned (low value / high cost here)
- UMAC user/rights management, SiVArc rule-based screen generation, full Git UI,
  encrypted credential vault. These are product-suite features; out of scope for an
  MCP whose job is engineering automation. Revisit only on concrete user demand.

---

## Sequencing suggestion
P0.0 (verify download) → P1.1 read-only safety (draft Release, verify on 安全PLC) →
P2.5 block protection (cheap win) → P2.3 OPC UA write → P1.2 PLCSIM (largest) →
P2.4 git helper. Follow the repo rule: **draft Release first, promote only after
real-machine verification.**
