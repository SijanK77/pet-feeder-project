# In this md: Organise and Describe the Data — Automated Pet Feeder

## Inputs

| Input                | Description                                      | Sample Value(s)                   | Constraints / Notes                          |
|----------------------|--------------------------------------------------|-------------------------------------|-----------------------------------------------|
| RTC time             | Current time, used to trigger feed schedules     | 08:00, 18:00                        | 24h format, ±30 s trigger window              |
| Feeding schedule     | List of feeding events with time & portion (g)   | 08:00 → 60g, 18:00 → 60g            | Max 6/day, 0–200 g/event                       |
| Bowl weight (grams)  | Mass in bowl from load-cell sensor                | pre: 10g, post: 70g                 | 0–2000 g, ±5 g noise, needs tare calibration   |
| Bin level sensor     | Food availability in hopper                      | 45%, OK/LOW/EMPTY                   | LOW <15%, EMPTY <5%                            |
| Motor feedback       | Confirms servo/motor movement                    | moved=TRUE                          | Timeout 3 s                                    |
| User settings        | Thresholds, retries, alerts, monitor time        | Spill=50g, retries=2, monitor=15min | retry limit: full=2, pulse=2; multiple alerts  |

---

## Outputs

| Output               | Description                            | Sample Value                 | Constraints/Notes                      |
|----------------------|----------------------------------------|------------------------------|------------------------------------------|
| Servo command        | Rotate servo pulses to drop kibble     | 3×200 ms pulses               | ≤10 pulses/event, 1 s settle time        |
| Alerts               | Notify staff of problems               | “ALERT: Bin empty”            | via email/app/LED, rate limit 1/min      |
| Logs                 | Timestamped event records              | `2025-08-14 08:00 DISPENSE 60g`| keep in local non-volatile memory        |
| Indicators (optional)| LED/buzzer signals                     | Red LED for error             | Optional, local only                     |

---

## Operational Parameters

| Parameter                  | Value   | Notes                                               |
|----------------------------|---------|-----------------------------------------------------|
| Portion per pulse           | 10 g    | Calibrate for your servo & kibble size              |
| Spill threshold             | 50 g    | Skip if bowl above this before feeding              |
| Monitoring window           | 15 min  | Time after dispensing to check if eaten             |
| Not-eaten threshold         | 15%     | Portion eaten below this → send alert               |
| Sensor averaging            | 5 reads | Drop outliers >10 g from median                     |
| Trigger tolerance           | ±30 s   | Time window to trigger feed                         |
| Motor timeout               | 3 s     | If exceed → motor fault alert                       |
| Retry limits                | Full=2, Pulse=2 | Limit retries to avoid endless loops         |

