Situation:
During a billing outage, customers were directly affected — bad bills and payment data started going out due to a combination of a recent deployment and an issue in the underlying processing/data logic.
Task:
Every minute bill sending continued, more customers would receive incorrect bills or face payment issues. I needed to stop the damage immediately, without the luxury of waiting for a full root-cause investigation first.
Action:
Within minutes of noticing the issue, I flagged it and got a quick verbal go-ahead, then paused bill sending immediately — before we had confirmed the actual root cause. I acted on my working hypothesis that the processing/data logic was producing the bad output, rather than waiting to prove it conclusively, because the cost of being wrong (a short pause) was far lower than the cost of continuing to send bad bills while we investigated properly.
That hunch was only partially right. Once we dug in over the next few hours, we found the real root cause was the recent deployment interacting badly with the existing data logic — not the data logic alone, as I'd first assumed. But pausing immediately meant we had the room to investigate properly without more customers being affected in the meantime.
Result:
By acting immediately rather than waiting for full certainty, we contained customer impact to the window before the pause, instead of letting it run for the several hours it took to find and fix the actual root cause. Once the full cause was identified — the deployment/data logic interaction — we fixed it and resumed bill sending safely.