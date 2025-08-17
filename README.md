Automated Pet Feeder — Logic & Simulation
Overview
This project designs and simulates the logic of a low‑cost automated pet feeder for a shelter. It:

Dispenses food at scheduled times

Monitors whether/what amount was eaten (via bowl weight)

Alerts on issues (bin empty/low, not eaten)

Deliverables (what’s included)

Step 1: Analysis (problem, assumptions, inputs/outputs)

Step 2: Data table (inputs, outputs, operational parameters)

Step 3: Flowchart (.drawio) of the control logic

Step 4: Word Code (plain-English implementation)

Step 5: Test scenarios and refinements

Repository structure

Step1_Analysis/ — problem_statement.md

Step2_Data/ — data_table.md

Step3_Flowchart/ — flowchart.drawio

Step4_Word_Code/ — word_code.md

Step5_Testing/ — test_scenarios.md, logs_examples.txt

Docs/ — README.md, ai_reflection.md

How to view

Flowchart: open Step3_Flowchart/flowchart.drawio in draw.io)

Word Code: Step4_Word_Code/word_code.md

Tests: Step5_Testing/test_scenarios.md

Key parameters 

dispenseAmount_g (e.g., 60)

waitMinutes (30)

decreaseThreshold_g (e.g., 10)

Time window(s): e.g., 08:00, 18:00

Future work

Hardware mapping (HX711 + load cell, SG90 servo, hopper level sensor)

Retry/under‑dispense detection and motor feedback

UI and secure remote alerts

License/Author

Author: Sijan Khadka

