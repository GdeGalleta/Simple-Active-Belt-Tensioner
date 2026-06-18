# Free-Cord Protection — Hardware Bring-up & Testing

Task brief for running on the **Windows PC with the belts/motors connected**, using
Claude Code. Branch: `feature/free-cord-protection`.

## Goal

Validate the new **free-cord protection** on real hardware and, critically,
**verify the per-motor velocity sign**. Then tune the limits.

## How it works (what you're testing)

- Each motor integrates how much **cord it has reeled in** (mm) since the motors
  were enabled. It converts the motor's reported velocity (RPM, bytes 2-3 of the
  drive response) to mm using the roller diameter: `mm_per_rev = π · diameter`.
- The counter **floors at 0** — paying cord out (pulling the belt) follows the
  most-extended point, so only net reel-in beyond it counts.
- If a motor reels in **more than its limit** (shoulder / waist), a belt is
  assumed released → **all motors are switched off**. Re-enable with the
  **Enable Motors** toggle (this resets the counters via `Check()`).
- Settings live in the **Tension** tab, under **Enable Free-Cord Protection**
  (off by default): Roller Diameter (32 mm), Shoulder Max Reel-in (100 mm),
  Waist Max Reel-in (100 mm, 4WD only).

## ⚠️ The thing to verify: velocity sign

`Sources/Plugin/MotorController.cs`, in `Motor.SetTorque()`:

```csharp
double windingSign = _isRightSide ? -1.0 : 1.0; // VERIFY sign on hardware
double deltaMm = windingSign * (velocityRpm / 60.0) * dt * mmPerRev;
_reeledMm = Math.Max(0, _reeledMm + deltaMm);
```

`_reeledMm` must **increase when the motor reels cord IN** (winding). The reported
velocity sign for the right-side motors (`Right`, `RightWaist`) is unconfirmed. If
it's inverted, winding would make `_reeledMm` stay at 0 (floored) and the
protection would never trip for that motor.

- **Symptom of wrong sign:** releasing that belt (motor spins reeling freely) does
  **not** trip; instead, *pulling the belt out* might.
- **Fix:** flip the sign for the affected side. If only the right side is wrong,
  the current `_isRightSide ? -1.0 : 1.0` already inverts right vs left — so if
  BOTH sides are wrong, swap to `_isRightSide ? 1.0 : -1.0`; if only one side is
  wrong, the mapping needs to be per-motor (tell Claude which motors misbehave).

## Build & deploy (from Sources/Plugin/AGENTS.md)

1. Ensure the `SIMHUB_INSTALL_PATH` environment variable points to the SimHub
   install dir (all references resolve from there).
2. Open `Sources/Plugin/User.ActiveBeltTensioner.sln` in Visual Studio (2015+),
   build **Debug**.
3. The post-build step `XCOPY`s the DLL/PDB and the `.resx` files into
   `%SIMHUB_INSTALL_PATH%`. **Restart SimHub** to load the new build.

## Reading logs

SimHub writes logs (log4net) to a `Logs\` folder inside `%SIMHUB_INSTALL_PATH%`
(date-stamped `.log` files). Open the newest and filter for our prefix:

```powershell
# adjust path; tail the newest SimHub log and show only plugin lines
Get-Content -Wait (Get-ChildItem "$env:SIMHUB_INSTALL_PATH\Logs\*.log" |
  Sort-Object LastWriteTime | Select-Object -Last 1).FullName |
  Select-String "SABT:"
```

The trip is logged as:
`SABT: <Label> reeled <X>mm > <Y>mm, disabling motors`

### Optional: see `_reeledMm` live (temporary instrumentation)

To watch the counter move (very helpful for the sign check), temporarily add a
throttled log inside `SetTorque()` right after `_reeledMm = Math.Max(...)`:

```csharp
// TEMP debug — remove before committing
Logging.Current.Info($"SABT: {Label} reeled={_reeledMm:F0}mm v={velocityRpm}");
```

This fires per command (~15-30 Hz/motor) so it's noisy — use only during bring-up
and **remove it before the final commit**.

## Test procedure

1. **Enable protection** (Tension tab) and set the limits (start with defaults:
   32 mm diameter, 100 mm shoulder/waist).
2. **Sign check (per motor):** buckle up, enable motors. While the tensioner reels
   in slack, confirm in the log that `_reeledMm` for each motor **goes up** (with
   the temp log). If a motor's value stays at 0 while it's clearly reeling →
   inverted sign for that motor → fix and rebuild.
3. **Trip check:** with belts taut and driving/idle tension applied, **release one
   buckle**. The motor reels freely; within ~its limit (e.g. 100 mm ≈ 1 turn) it
   should trip and **all motors switch off**. Confirm the trip log line.
4. **Recovery:** re-enable with **Enable Motors**; confirm counters reset and
   normal tensioning resumes.
5. **Tune limits:** if it trips during normal buckle-up slack take-up, raise the
   Max Reel-in; if it lets too much cord wind before stopping, lower it. Find a
   value above your slack take-up and safely below the clamp-to-pulley distance.
6. Repeat for **waist motors** in a 4WD setup.

## When done

- Remove any temporary debug logging.
- If the sign needed changing, keep that fix.
- Commit on `feature/free-cord-protection` and push. Keep history clean
  (amend/squash the bring-up fixes into the feature commit if you prefer a single
  clean commit, then `git push --force-with-lease`).
