Word Code 

Variables and constants

currentTime: string HH:MM (from RTC)

feedingWindow: struct {start, end} for the next scheduled feed

foodLevelSensorStatus: boolean (True means problem/empty per your diagram logic; see note below)

activateMotor: boolean

activateAlert: boolean

dispenseAmount_g: number (target grams to dispense)

bowlWeight_g: number (current bowl weight)

preDispenseWeight_g: number

postDispenseWeight_g: number

postWaitWeight_g: number

waitMinutes: number = 30

decreaseThreshold_g: number (minimum decrease considered “eaten enough”)

alertType: enum {NOT_EATEN, GENERAL}

loop: continuous

Note on foodLevelSensorStatus:

Your diagram uses “foodLevelSensorStatus is True?” followed by Yes → alert and Stop, No → continue. That implies True = “bin empty/low” (i.e., an error). The code below follows that exactly. If you intended True=OK, flip the condition accordingly.

Initialization

activateAlert = False

activateMotor = False

Read bowlWeight_g

Load feedingWindow, dispenseAmount_g, decreaseThreshold_g

Log “System start, ready to enter time loop”

Time loop (runs continuously)
Loop:

Update currentTime

If currentTime not within feedingWindow:

Continue Loop (do nothing; wait until within feeding window)

Else (within feeding window):

Go to Food Level Check

Food level check

Read foodLevelSensorStatus

If foodLevelSensorStatus is True:

activateAlert = True

alertType = GENERAL // bin empty/low or hopper issue

Send/Log alert: “Food level issue detected (bin empty/low). Feeding stopped.”

Stop current cycle (end this feed attempt); go back to Loop for next window

Else (foodLevelSensorStatus is False, i.e., OK):

Proceed to Dispense

Dispense sequence

preDispenseWeight_g = Read bowlWeight_g (stable average)

activateMotor = True

While (Read bowlWeight_g - preDispenseWeight_g) < dispenseAmount_g:

Run motor pulse (duration suitable for small portion step)

Wait short settle time (e.g., 1–2s)

Re-read bowlWeight_g (stable average)

If motor fault detected (optional): break and go to alert (GENERAL)

activateMotor = False

postDispenseWeight_g = Read bowlWeight_g

Wait window

Wait for waitMinutes (30 minutes total), sampling or sleeping in chunks (e.g., 60s) as needed

After wait completes:

postWaitWeight_g = Read bowlWeight_g

Consumption evaluation

consumedDuringWait_g = max(0, postDispenseWeight_g - postWaitWeight_g)

If consumedDuringWait_g >= decreaseThreshold_g:

Consider feeding successful

Log: “Feeding OK: dispensed ≈ (postDispenseWeight_g - preDispenseWeight_g)g, consumed ≈ consumedDuringWait_g g”

End cycle and return to Loop for the next time window

Else (not enough weight decrease):

alertType = NOT_EATEN

activateAlert = True

Send/Log alert: “NOT_EATEN: Food consumption below threshold after 30 minutes.”

End cycle and return to Loop for the next time window

Failsafe and stop branches (as in your diagram)

If at any point a Stop is reached (e.g., immediate alert path):

Ensure activateMotor = False

If activateAlert = True, send alert with current alertType

End current cycle; return to Loop

Logging (suggested, lightweight)

On entering feeding window: “Feeding window entered at currentTime”

On dispense start: “Dispense start; target=dispenseAmount_g; pre=preDispenseWeight_g”

On dispense end: “Dispense end; post=postDispenseWeight_g; actual_dispensed≈(postDispenseWeight_g - preDispenseWeight_g)”

On wait end: “Wait 30 min end; postWait=postWaitWeight_g; consumedDuringWait=consumedDuringWait_g”

On alerts: include alertType and key values

Example parameters (aligning with your chart)

dispenseAmount_g = 60

waitMinutes = 30

decreaseThreshold_g = 10 // example: 10g decrease considered “eaten enough”

Notes aligning to diagram semantics

There is no explicit “bowl full pre-check” in your current flow; if you want to avoid overfilling, insert an extra decision before Dispense:

If preDispenseWeight_g > spillThreshold_g: alert and stop.

Your flow directly stops on foodLevelSensorStatus = True; if you prefer a “retry later” behavior, replace Stop with “Return to time loop” and optionally queue an alert.

Ultra-compact pseudocode (same logic)

loop forever:
currentTime = RTC.read()
if not within(feedingWindow, currentTime): continue

if foodLevelSensorStatus == True:
activateAlert = True
alertType = GENERAL
alert("Food level issue; stopping")
stop cycle; continue

pre = avgBowlWeight()
activateMotor = True
while (avgBowlWeight() - pre) < dispenseAmount_g:
motor.pulse()
sleep 1-2s
activateMotor = False
post = avgBowlWeight()

sleep waitMinutes (30min)
postWait = avgBowlWeight()
consumed = max(0, post - postWait)

if consumed < decreaseThreshold_g:
activateAlert = True
alertType = NOT_EATEN
alert("Not eaten within 30min")
// else success

// return to loop for the next feeding window
