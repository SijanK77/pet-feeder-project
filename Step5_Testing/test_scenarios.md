Overview

Purpose: Verify the flowchart and word code with realistic situations and tune thresholds.

Key behaviors under test:

Feeding only when within the feeding window.

foodLevelSensorStatus True → activateAlert → Stop (no motor).

Dispense until weight increase ≥ dispenseAmount_g.

After 30 minutes, if bowl weight decrease < decreaseThreshold_g → alertType = NOT_EATEN → activateAlert → Stop.

Otherwise, success and return to loop.

Parameters used for tests (adjust to your design)

dispenseAmount_g = 60

waitMinutes = 30

decreaseThreshold_g = 10

Example windows: 08:00, 18:00

Scenario A — Pet eats as expected

Setup:

feeding window: active

foodLevelSensorStatus = False

preDispenseWeight_g = 20

postDispenseWeight_g ≈ 80 (≈60g dispensed)

postWaitWeight_g ≈ 30 (consumed ≈50g)

Expected:

No alert. Cycle returns to loop.

Observed (sim):

Success; logs indicate consumed≈50g.

Refinement:

None; thresholds seem fine.

Scenario B — Pet does not eat

Setup:

feeding window: active

foodLevelSensorStatus = False

preDispenseWeight_g = 10

postDispenseWeight_g ≈ 70

postWaitWeight_g ≈ 68 (consumed ≈2g)

Expected:

NOT_EATEN alert after 30 min; Stop.

Observed (sim):

NOT_EATEN alert triggered.

Refinement:

If false alarms occur, consider waitMinutes=45 or decreaseThreshold_g=8.

Scenario C — Hopper empty/low (sensor True)

Setup:

feeding window: active

foodLevelSensorStatus = True

Expected:

Immediate alert; Stop. No motor.

Observed (sim):

Alert sent; cycle stopped.

Refinement:

Add pre-window reminder if the sensor can report LOW before EMPTY.

Scenario D — Under‑dispense (stall or jam while dispensing)

Setup:

feeding window: active; sensor False

preDispenseWeight_g = 5

Motor runs but actual increase < 60g (e.g., only +20g)

Expected:

Your chart proceeds to wait and then evaluates eating; if pet doesn’t compensate, likely NOT_EATEN alert.

Optional improvement: flag “Under‑dispense suspected” when actual dispensed <70% of target.

Observed (sim):

Minimal consumption; NOT_EATEN fired.

Refinement:

Add a retry limit or “max pulses” guard and alert “Under‑dispense.”

Scenario E — Outside feeding window (sanity check)

Setup:

currentTime not within window

Expected:

Time loop keeps checking; no dispense, no alerts.

Observed (sim):

Idle loop only.

Refinement:

None.

Sample log excerpts (add to Step5_Testing/logs_examples.txt if you like)

08:00: Enter window; pre=20g; target=60g

08:01: Dispense end; post=80g; actual=60g

08:31: Wait end; postWait=30g; consumed=50g; status=OK

18:00: Enter window; pre=10g; target=60g

18:01: Dispense end; post=70g; actual=60g

18:31: Wait end; postWait=68g; consumed=2g; ALERT=NOT_EATEN

08:00: foodLevelSensorStatus=True; ALERT=BIN LOW/EMPTY; STOP
