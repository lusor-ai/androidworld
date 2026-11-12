# Comparison with Mobilerun

[Mobilerun](https://mobilerun.ai/benchmark/) also reports 116 / 116 on AndroidWorld, graded with its own fork of the grader. [`mobilerun.csv`](mobilerun.csv) puts each task's published figures next to Lusor's.

| Column | Source |
| --- | --- |
| `lusor_time_s`, `lusor_cost_usd` | This run. Time is from the agent's start to its finish, excluding setup and grading. Cost is the actual model cost. |
| `mobilerun_published_time_s`, `mobilerun_published_actions` | Mobilerun's published per-task data. Mobilerun does not publish what its time covers. |
| `speed_ratio` | `lusor_time_s / mobilerun_published_time_s`. Across tasks the median ratio is 0.19 (about 5.2× faster); Lusor was slower on 5 tasks. This is a median of per-task ratios, not a ratio of medians. |
| `mobilerun_est_cost_usd` | **An estimate, not Mobilerun's bill:** Mobilerun's published action count × $0.0766, the mean cost per call measured for gpt-6-astra at low effort in the Lusor Harness. Mobilerun does not publish its cost. |

What each side publishes:

| | Mobilerun | Lusor |
| --- | --- | --- |
| Graders | its fork of the grader | its fork of the grader, and Google's original |
| Per task | goal, answer, score, time, actions with arguments, settings | goal, both grader results, time, cost, steps with arguments (tool, arguments, timing, note) |
| Screens | a screenshot per action and a replay GIF | a video of the whole session |
| Time definition | not published | agent start to finish |
| Cost | not published | measured |
| Step budget | AndroidWorld's budget kept on every task | not enforced; 22 tasks went over it ([`step-budget.json`](step-budget.json)) |
| Checksums | none | `SHA256SUMS` |
| App guidance | method page and harness; the app cards and memory used for AndroidWorld are not published | the six AndroidWorld skills, in [`../skills`](../skills) |

Every figure on the AndroidWorld leaderboard is reported by the team that ran it, including this one.
