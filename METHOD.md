# Method

## What ran

- **Agent:** the Lusor Harness at the build in production on 9 November 2026, run on an isolated copy of the agent service with production's configuration. This is not the open-source release, which has not been benchmarked.
- **Model:** DeepSeek V4.1 Flash, low reasoning effort, chose every action. Lusor's small check models verified lists and saved changes; their cost is included below.
- **Skills:** the app-skill catalog that shipped with that build, nine skills. Six of them were written for AndroidWorld's test apps and are published in [`skills/`](skills). The agent opened a skill when its description matched the task; no skill was attached to a task.
- **Screen-mapping tool:** the agent also had a screen-mapping tool that is part of Lusor's hosted service, not the open-source release. It was called in 37 of the 116 tasks and appears as `map_app` in `steps.json`.
- **Tuning before the run:** the six skills, and changes to the agent's general guidelines and tools, came from reviewing earlier AndroidWorld runs. Those earlier runs are not part of the score.
- **Memory:** each task ran in a new session with a new agent. The agent's memory service was replaced by a stand-in that stores nothing, and it received no memory requests during the run. Skill and guideline files did not change during the run.

## Tasks and environment

- **Tasks:** all 116 AndroidWorld tasks, from Mobilerun's public copy of the benchmark ([droidrun/android-world](https://github.com/droidrun/android-world) `c5467e6`).
- **Task parameters:** AndroidWorld generated each task's parameters from seed 42, one instance per task. The goal in each `task.json` is exactly the text the agent was given.
- **Instructions added to every goal:** the same two sentences followed every goal: "If the task asks a question, return exactly one row whose column "answer" holds only the answer." and "No human is available during this task; if blocked, try alternatives, then finish with done() and report the blocker." The first tells the agent where to put an answer in Lusor's output; the second replaces asking a person, which the benchmark cannot answer.
- **Attempts:** one per task (pass@1). No task was rerun; that single run is the score.
- **Limits per session:** 30 minutes, and a model-cost cap of $0.60 set for this run (production's cap is higher). No session reached either: the longest took 354 s and the most expensive cost $0.158.
- **Step budget:** AndroidWorld sets a step budget for each task (10 × the task's complexity). This run did not enforce it: a session ended when the agent finished, after 30 minutes, or at a $0.60 cost cap. 22 of the 116 tasks used more device actions than their budget, and all 22 passed. [`results.csv`](results.csv) and [`comparison/step-budget.json`](comparison/step-budget.json) list the budget and the run's action count for every task. Mobilerun's published run stayed within the budget on every task.
- **State between tasks:** each emulator started once from a clean snapshot. Before every task its apps, files, contacts, messages, calendar, media and system settings were reset to that snapshot and checked against a fingerprint of the clean copy; all 116 checks matched. The reset also returned Markor's remembered new-file extension to its default (before 17 Markor tasks; it was already at the default each time) and cleared OsmAnd's favourites, tracks and search history (before 3 OsmAnd tasks; already empty each time).
- **One change, applied to every task:** Chrome started with `--disable-accelerated-2d-canvas`, because the emulator's GPU canvas did not render drawn strokes otherwise.
- **Emulators:** Android Emulator 37.2.12, Pixel 6 device profile, Android 13 (API 33) Google APIs x86_64 system image, 4 cores, 4 GB RAM and a 6 GB data partition each. Four emulators ran tasks in parallel on one Linux x86_64 server with KVM.
- **Network:** AndroidWorld's tasks need no internet. The emulators' HTTPS traffic was routed only to the recording uploader.
- **On-phone agent:** Lusor's on-phone agent, version 0.29.0, built from the same commit as the agent service.

## Grading

- **Live:** graded with Mobilerun's fork of the AndroidWorld grader ([droidrun/android-world](https://github.com/droidrun/android-world) `c5467e6`): 116 / 116. The fork fixes setup and grading bugs in the original; its changes are listed in [`docs/benchmark-fixes.md`](https://github.com/droidrun/android-world/blob/c5467e6/docs/benchmark-fixes.md). The fork also rewords three task goals (the Markor note-and-SMS goal and two date goals); the agent was given the fork's wording, which is the `goal` in each `task.json`.
- **Re-graded:** the same final phone states, graded afterwards with Google's original, unmodified grader ([google-research/android_world](https://github.com/google-research/android_world) `30583e2`): 114 / 116.
- **The two tasks the graders disagree on:**
  - `AudioRecorderRecordAudioWithFileName`: the app adds `.m4a` itself, so the agent typed the name without it and saved exactly the requested file. Google's grader looks for the file you get by typing the full name (`….m4a.m4a`).
  - `ExpenseAddMultipleFromMarkor`: Google's grader expects the expense notes without the "Reimbursable." marker the source file puts on them.

  The agent did both tasks; it read those two instructions differently from Google's grader. Mobilerun's fork accepts the agent's reading on both.
- **Disclosure:** on `BrowserMultiply` the agent read the page's code to get the answer.

## Time and cost

- **Time per task:** measured by the benchmark runner, from the request that started the agent's session to the first status check (every 3 s) that found it finished. It includes the session's start-up, every model call, phone action and check, and excludes the task's setup and reset, and grading. Median 48.2 s.
- **Cost:** $2.95 for all 116 tasks ($2.75 for the model choosing actions, $0.20 for Lusor's check models): the cost the model provider reported for each request. The provider's prompt caching applied, as it does in production; 97% of input tokens were read from cache. Grading uses no model and costs nothing; emulator and server time are not included.

## Data

For every task, [`data/<task>/`](data) holds:

- `task.json`: the goal, both grader results, time, cost, tool calls, the agent's final summary, the skills it opened and any note.
- `steps.json`: each step's tool, its start and end in seconds from the start of the video, whether it succeeded, and the agent's note for that step.

[`actions/<task>/actions.json`](actions) adds each step's arguments as the agent sent them and, for taps on an element, the element and screen point it reported. It is derived from the same session record as `steps.json`, with matching step numbers and times; tool output is not included. `actions/SHA256SUMS` lists a checksum for each file.

`data/index.json` lists every task and the run's setup. `data/SHA256SUMS` lists a checksum for `index.json` and for every task's `task.json`, `steps.json`, poster and video. The videos and posters are on [data.lusor.ai](https://data.lusor.ai/benchmarks/androidworld/2026-11-09/index.json) under the same paths.

`task.json` also carries Mobilerun's published time and action count for each task, and an estimate of its cost. They are kept so every file matches its published checksum; [`comparison/`](comparison) explains them.
