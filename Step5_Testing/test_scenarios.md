Overview

Purpose: Verify logic against typical scenarios and refine thresholds.

System behavior per flowchart:

Enters feeding only within feeding window.

If foodLevelSensorStatus is True → activateAlert → Stop cycle.

Else: activateMotor=True → dispense until target weight increase ≥ dispenseAmount_g → activateMotor=False.

Wait 30 minutes, measure postWaitWeight_g.

If bowl weight decrease < decreaseThreshold_g → alertType=NOT_EATEN → activateAlert=True → Stop.

Else: success and return to loop.

Key parameters used in tests 

dispenseAmount_g: 60g

waitMinutes: 30

decreaseThreshold_g: 10g

trigger window: as scheduled (e.g., 08:00, 18:00)

Reading function: avg bowl weight over multiple samples

Scenario A — Pet eats as expected
Setup

feeding window: active

foodLevelSensorStatus = False (OK)

preDispenseWeight_g = 20g

Target: dispenseAmount_g = 60g

After dispensing: postDispenseWeight_g ≈ 80g

After 30 minutes: postWaitWeight_g ≈ 30g (consumed ≈ 50g)

Expected

Motor activates, dispenses until increase ≥60g (±sensor tolerance).

After 30 minutes, consumedDuringWait_g = 80 − 30 = 50g ≥ 10g threshold → success.

No alert; cycle returns to loop.

Observed (simulate)

Success. Log “dispense complete,” “consumed≈50g.”

Refinement

None required. Keep decreaseThreshold_g=10g for timely detection.

Scenario B — Pet does not eat
Setup

feeding window: active

foodLevelSensorStatus = False (OK)

preDispenseWeight_g = 10g

After dispense: postDispenseWeight_g ≈ 70g

After 30 minutes: postWaitWeight_g ≈ 68g (consumed ≈ 2g)

Expected

After wait, consumedDuringWait_g = 70 − 68 = 2g < 10g → alertType=NOT_EATEN → activateAlert=True → Stop.

Observed (simulate)

NOT_EATEN alert sent; cycle stops.

Refinement

If false positives occur (pet eats later), consider:

Extending waitMinutes to 45–60 in busy shelter hours, or

Lowering decreaseThreshold_g to 8g. Document trade‑off.

Scenario C — Hopper empty/low (sensor True)
Setup

feeding window: active

foodLevelSensorStatus = True

Expected

Immediate alert path: activateAlert=True; Stop cycle (no motor movement).

Observed (simulate)

Alert “Food level issue (bin empty/low).” No dispensing.

Refinement

Add a pre‑window “low hopper” reminder (e.g., 10 minutes before window) if sensor supports percentage.

Scenario D — Under‑dispense (motor stalls mid‑dispense)
Setup

feeding window: active

foodLevelSensorStatus = False

preDispenseWeight_g = 5g

Motor runs but weight delta increases slowly and never reaches 60g within allowed pulses/time.

Expected

Dispense loop exits when target not met (your flowchart does not show retries explicitly). Post‑dispense proceeds to 30‑minute wait.

Likely NOT_EATEN or minimal consumption.

Consider raising an operational alert “Under-dispense suspected” if actual dispensed < target by large margin (optional extension).

Observed (simulate)

Low actual dispensed. If pet doesn’t eat, NOT_EATEN triggers.

If you add an under‑dispense alert, log it when postDispenseWeight_g − preDispenseWeight_g < (0.7 × dispenseAmount_g).

Refinement

Add a small retry loop or a “max pulses” guard + alert “Under‑dispense.”

Consider motor feedback check if hardware allows.

Scenario E — Bowl already heavy before feed (optional pre‑check)
Note: Your current flowchart does not include a spill prevention pre‑check. If you decide to add it, the behavior would be:

Setup

preDispenseWeight_g = 120g (above cutoff, e.g., spill threshold)

foodLevelSensorStatus = False

Expected (with added decision)

Skip dispense; alert “Bowl already full”; Stop or return to loop.

Observed (simulate)

Prevents overfilling and waste.

Refinement

Consider adding this decision before “activateMotor is True” in the flowchart for safety.

Sample Logs (concise)
Enter window 08:00 — pre=20g, target=60g

Dispense end — post=80g, actual=60g

Wait end (30m) — postWait=30g, consumed=50g — OK

Enter window 18:00 — pre=10g, target=60g

Dispense end — post=70g, actual=60g

Wait end (30m) — postWait=68g, consumed=2g — ALERT NOT_EATEN

Enter window 08:00 — foodLevelSensorStatus=True — ALERT BIN LOW/EMPTY — STOP
