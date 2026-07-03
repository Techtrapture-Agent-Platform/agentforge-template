## 7. Golden Dataset Seed (10 Entries)

*Architecture Doc Reference: § Appendix S5 (company-cicd validation hook — check_golden_dataset.py); Developer Guide Reference: § Appendix FNOL-F1, §15*

| # | Input | Expected Output | Rules Tested |
|---|---|---|---|
| 1 | "Severe chest pain for 2 hours, insured, BlueCross PPO" | urgency=emergent, provider=ER, notify_pharmacy=false | Rule 1 |
| 2 | "High fever (103°F) for 4 days, insured, Aetna HMO" | urgency=urgent, provider=PCP_same_day, notify_pharmacy=true | Rule 2 + 5 |
| 3 | "Diabetes follow-up, medication refill needed, insured, UHC" | urgency=routine, provider=specialist, notify_pharmacy=true | Rule 3 + 5 |
| 4 | "Sprained ankle, insurance expired 2 months ago" | urgency=routine, flag=uninsured_pathway | Rule 4 |
| 5 | "Breathing difficulty + wheezing, insured, Cigna, on albuterol" | urgency=emergent, provider=ER, notify_pharmacy=true | Rule 1 + 5 |
| 6 | "Routine physical exam, no complaints, insured" | urgency=routine, provider=PCP | Default |
| 7 | "Child age 3, rash + fever 2 days, insured" | urgency=urgent, provider=PCP_same_day | Rule 2 variant |
| 8 | "Back pain chronic, 6 months, specialist referral, uninsured" | urgency=routine, provider=specialist, flag=uninsured_pathway | Rule 3 + 4 |
| 9 | "Chest pain + expired insurance + medication likely" | urgency=emergent, provider=ER, flag=uninsured_pathway, notify_pharmacy=true | Rule 1 + 4 + 5 |
| 10 | "Headache 1 day, insured, Anthem, no medication" | urgency=routine, provider=PCP, notify_pharmacy=false | Default |
