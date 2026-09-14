<div align="center">

<!-- PLACEHOLDER: plugin logo, 128px wide -->
<img src="docs/images/Icon128.png" alt="Multiplayer Replication Lint" width="128"/>

# Multiplayer Replication Lint

**Find the replication mistakes in your Unreal project before your players do.**
*Blueprints and C++. Plain language explanations. One click fixes where it is safe.*

[![Unreal Engine](https://img.shields.io/badge/Unreal%20Engine-5.1%20to%205.8-313131?style=for-the-badge&logo=unrealengine&logoColor=white)]()
[![Editor only](https://img.shields.io/badge/Runtime%20cost-None-2EA043?style=for-the-badge)]()
[![Fab](https://img.shields.io/badge/Get%20it%20on-Fab-5865F2?style=for-the-badge)](https://www.fab.com/sellers/Kybrien)
[![Support](https://img.shields.io/badge/Support-Discord-5865F2?style=for-the-badge&logo=discord&logoColor=white)](https://discord.gg/BwhyxQAAUn)

**English** · [Documentation en français](README_fr.md)

[Setup](#-setup) · [Quick start](#-quick-start) · [The tool](#-the-tool) · [Fixes](#-safe-fix-and-suggested-fix) · [Rules](#-the-27-rules) · [CI](#-continuous-integration) · [Troubleshooting](#-troubleshooting)

</div>

---

<!-- PLACEHOLDER: hero screenshot, the Issues view after a scan with a finding selected and its explanation visible -->
<img src="docs/images/Hero.png" alt="Multiplayer Replication Lint Issues view" width="100%"/>

## Why this plugin

Most replication bugs are not crashes. A variable that never reaches clients, an RPC dropped because
the actor does not replicate, a reliable call fired every frame that ends up kicking a player. The game
runs fine on your machine, and breaks in the first real session.

Unreal already gives you great tools to **measure** a running session. None of them reads your project
and tells you what is **wrong** before you press Play.

|  | Multiplayer Replication Lint | Network Profiler / Networking Insights |
|---|:---:|:---:|
| **Finds mistakes without running the game** | Yes | No, needs a session |
| **Reads Blueprint graphs** (Tick chains, spawns, RepNotify) | Yes | No |
| **Explains why it breaks and how to fix it** | Yes | No |
| **One click fix for configuration mistakes** | Yes | No |
| **Fails a CI build on a new critical issue** | Yes | No |
| **Measures real bandwidth in a live session** | No | Yes |

> The two are complementary. Multiplayer Replication Lint catches the setup mistakes, the profiler tells you what your
> game actually costs on the wire. Use both.

---

## 📦 Setup

### 1. Install

Copy the `MultiplayerReplicationLint` folder into your project's `Plugins/` folder, then relaunch the
editor and accept the compile prompt.

Check **Edit → Plugins** that *Multiplayer Replication Lint* is enabled.

### 2. Open the tool

**Tools → Multiplayer Replication Lint**. The window is a normal editor tab, dock it wherever you like.

<!-- PLACEHOLDER: screenshot of the Tools menu with the entry highlighted -->
<img src="docs/images/ToolsMenu.png" alt="Tools menu entry" width="50%"/>

### 3. Tell it about your C++ (optional)

Blueprints under `/Game` are always scanned. C++ classes are scanned only for the modules you list.

**Edit → Project Settings → Plugins → Multiplayer Replication Lint → Native Modules To Scan**, add your game module
name (for example `MyGame`).

> [!NOTE]
> C++ scanning reads reflection data: `UPROPERTY` and `UFUNCTION` flags, replication settings, class
> defaults. It does **not** parse function bodies. A C++ `Tick()` that calls an RPC every frame is
> invisible to it.

---

## 🚀 Quick start

1. Open **Tools → Multiplayer Replication Lint**
2. Click **Run Full Scan**. A progress dialog runs through your project. You can cancel it at any
   time and keep what was found so far.
3. Read your grade, from **A** to **E**, on the Dashboard
4. Open **Issues** and start with the red ones
5. Select a row. The panel at the bottom tells you what was detected, why it breaks in a real game, and
   how to fix it
6. Click **Open** to jump into the Blueprint, or **Fix** when the button is there
7. Save, scan again, watch the grade climb

**Your first scan will not be empty, even in a blank project.** The plugin ships four C++ demo actors,
three of them broken on purpose, so you can see real findings straight away.

<!-- PLACEHOLDER: short GIF, Run Full Scan then the grade appearing on the Dashboard -->
<img src="docs/images/FirstScan.gif" alt="First scan" width="90%"/>

---

## 🧭 The tool

Eight views, grouped in the left navigation. Every entry has a one line description under it.

### Dashboard

Project health at a glance.

- **The grade**, A to E, in colour
- **Stat tiles**: classes scanned, replicated actors, variables and RPCs, findings per severity
- **Grade trend** over your last 12 full scans
- **Since last scan**: what is new and what got fixed
- **Top 5 issues**, each with an Open button

Before the first scan it shows a three step guide instead of an empty page.

<!-- PLACEHOLDER: Dashboard screenshot with a few scans of history -->
<img src="docs/images/Dashboard.png" alt="Dashboard" width="90%"/>

### Issues

The heart of the workflow. Every finding in one sortable table.

- **Filter** by text, severity and category. Tick *Show ignored* to see what you dismissed
- **Select a row** to read the explanation: *what was detected*, *the risk in a real game*,
  *the recommended fix*, and the rule ID
- **Open** jumps into the Blueprint, **Browse** selects it in the Content Browser
- **Safe Fix** or **Suggested Fix** appears on findings the tool can correct, see [Fixes](#-safe-fix-and-suggested-fix)
- **Ignore** hides a finding you decided to keep. It leaves the grade and the reports
- **Ignore everything in a category or on a class** from the detail panel, reversible through
  *Show ignored*

Double click a row to open the asset.

<!-- PLACEHOLDER: Issues view with a Critical finding selected and the explanation panel visible -->
<img src="docs/images/Issues.png" alt="Issues view" width="90%"/>

### Diagnose

*"Why isn't this replicating?"* answered straight.

Pick a Blueprint. You get a diagnosis for the actor, for each of its replicated variables and for each
of its RPCs: the conditions that have to be true for it to reach clients, which ones are met, and a
verdict that names the blocking cause in plain words.

| It checks | For example |
|---|---|
| The actor replicates | *Replicates is off in Class Defaults: nothing will ever be sent* |
| Dormancy | *Net Dormancy is Initial: replicates once at spawn, then stops* |
| Relevancy | *Distance based: far away players will not receive it* |
| The variable is marked Replicated | and its owning component replicates too |
| The RepNotify handler exists | |
| The RPC direction and reliability | *Unreliable: may be dropped under packet loss* |
| Ownership | *Pawns, PlayerControllers and PlayerStates are owned by construction* |

**Shortcut:** right click any Blueprint in the Content Browser → **Diagnose Replication**.

<!-- PLACEHOLDER: Diagnose view showing a BLOCKED verdict on a variable -->
<img src="docs/images/Diagnose.png" alt="Diagnose view" width="90%"/>

### Classes, RPCs, Variables

Three inventory views, each with its own search and an Open button.

| View | One row per | Shows |
|---|---|---|
| **Classes** | class | Replicates, movement, update frequency, number of variables, RPCs and issues |
| **RPCs** | RPC | Server / Client / Multicast, reliable, validation, estimated payload, called from Tick |
| **Variables** | replicated variable | Type, RepNotify, estimated size, sorted biggest first |

### Lag Lab

Test your game under real network conditions in one click.

1. Pick a preset and click **Apply**
2. Launch Play In Editor with at least 2 players, Net Mode **Play As Client**
3. Play. Type `stat net` in a client window to see the ping

| Preset | Ping | Loss |
|---|:---:|:---:|
| Clean LAN | 0 | 0 % |
| Good connection | 40 to 60 ms | 0 % |
| Average internet | 80 to 120 ms | 0 % |
| Poor connection | 130 to 180 ms | 0 % |
| Very poor | 220 to 300 ms | 0 % |
| Slight packet loss | 40 to 60 ms | 1 % |
| Heavy packet loss | 60 to 100 ms | 5 % |
| Bad Wi-Fi | 50 to 200 ms | 2 % |
| Mobile network | 100 to 350 ms | 3 % |
| Overseas (EU to US East) | 80 to 110 ms | 0 % |
| Very long distance (EU to Asia) | 220 to 300 ms | 1 % |
| Asymmetric home connection | 57 to 102 ms | 2 % upload only |

The ping is what `stat net` should show, not a per packet delay. The emulation runs on the server, so
every client gets the same conditions.

> [!TIP]
> **The asymmetric preset is the interesting one.** Downloads are fast, uploads are slow and lossy,
> like a saturated home connection. Your character looks fine to everyone else, but your own inputs
> arrive late. Symmetric testing never shows that.

**Disable emulation** puts everything back to clean conditions. The settings are written to your
editor play settings, the same place as **Editor Preferences → Play → Network Emulation**.

<!-- PLACEHOLDER: Lag Lab view with a preset applied and the status line visible -->
<img src="docs/images/LagLab.png" alt="Lag Lab" width="90%"/>

### Rules

Every rule the tool checks, with three short paragraphs each: what it detects, why it matters, and when
it is legitimately wrong. Untick a rule here to stop it reporting.

<!-- PLACEHOLDER: Rules view with one rule selected -->
<img src="docs/images/Rules.png" alt="Rules view" width="90%"/>

---

## 🩹 Safe Fix and Suggested Fix

Eight rules offer a fix button in the Issues view. Every fix is equally safe mechanically: one Class
Defaults value, one transaction, undoable, no graph touched. The label says what the tool is claiming
about your intent.

**Safe Fix** (RL_VAR_001, RL_RPC_001, RL_RPC_002, RL_COMP_001): the setup cannot work and there is one
correct value.

**Suggested Fix** (RL_ACT_001, RL_ACT_002, RL_ACT_003, RL_ACT_005): a heuristic flagged it, and the
current value may well have been deliberate. The confirmation dialog says so.

| Rule | Fix |
|---|---|
| `RL_VAR_001` Replicated variable on a non-replicated class | Turns **Replicates** on |
| `RL_RPC_001` RPC on a non-replicated actor | Turns **Replicates** on |
| `RL_RPC_002` NetMulticast on a non-replicated actor | Turns **Replicates** on |
| `RL_COMP_001` Replicated component on a non-replicated actor | Turns **Replicates** on |
| `RL_ACT_001` Replicate Movement on a likely static actor | Turns **Replicate Movement** off |
| `RL_ACT_005` Always Relevant on a gameplay actor | Turns **Always Relevant** off |
| `RL_ACT_002` NetUpdateFrequency very high | Sets it to your configured maximum |
| `RL_ACT_003` Low NetUpdateFrequency on a Pawn | Sets it to your configured Pawn minimum |

**What a fix will never do:**

- **Touch a graph.** It changes one Class Defaults value, exactly like a click in the Details panel. No
  node is added, moved or edited.
- **Touch C++.** Findings on C++ classes have no Fix button.
- **Save for you.** The Blueprint is marked modified. You review it and save it yourself.
- **Surprise you.** A dialog describes the exact change before anything happens.

Actors already placed in your open levels follow the new value too, as long as they were still using
the default. An actor where you deliberately changed the value is left alone.

> [!IMPORTANT]
> **Undo before you compile.** `Ctrl+Z` reverts a fix, on the Blueprint and on the placed actors. Once
> the Blueprint has been compiled, undo no longer brings the old value back. Set it back by hand in
> Class Defaults if you change your mind after a compile.

Everything else is deliberately not auto fixable. Moving an RPC out of Tick or adding an authority
check is a design decision, and a tool guessing at design is how you get a second bug.

---

## 📋 The 27 rules

**Severity** says how bad it is. **Confidence** says how sure the tool is.

| Severity | Meaning |
|---|---|
| **Critical** | Replication is broken. Something will not reach clients. |
| **Major** | Works on your machine, fails in a real session. Disconnects, desyncs. |
| **Warning** | Costs bandwidth or CPU for nothing, or is fragile. |
| **Info** | Worth knowing. Nothing is broken. |

| Confidence | Meaning |
|---|---|
| **Definite** | Provable from the class setup. If it fires, it is real. |
| **High confidence** | Almost always right. A throttle the tool cannot see is the usual exception. |
| **Potential** | A heuristic. Read the explanation before acting. |
| **Informational** | A heads up rather than a defect. |

Rules marked *experimental* are heuristics that can produce false positives by design. Turn them all
off at once with **Enable Experimental Diagnostics** in Project Settings.

<details>
<summary><b>▸ RPCs, 7 rules</b></summary>

<br/>

| ID | Rule | Severity | Confidence | Fix |
|---|---|:---:|:---:|:---:|
| `RL_RPC_001` | RPC on a non-replicated actor | Critical | Definite | ✅ |
| `RL_RPC_002` | NetMulticast on a non-replicated actor | Critical | Definite | ✅ |
| `RL_RPC_003` | Reliable RPC called from Event Tick | Major | High | |
| `RL_RPC_004` | RPC called from Event Tick | Warning | High | |
| `RL_RPC_005` | Large RPC payload | Warning | High | |
| `RL_RPC_006` | Reliable NetMulticast *(experimental)* | Warning | Potential | |
| `RL_RPC_007` | RPC with many parameters | Warning | Informational | |

</details>

<details>
<summary><b>▸ Variables, 7 rules</b></summary>

<br/>

| ID | Rule | Severity | Confidence | Fix |
|---|---|:---:|:---:|:---:|
| `RL_VAR_001` | Replicated variable on a non-replicated class | Critical | Definite | ✅ |
| `RL_VAR_002` | Many replicated variables on one class | Warning | Informational | |
| `RL_VAR_003` | Large replicated property | Warning | High | |
| `RL_VAR_004` | RepNotify function missing or empty | Warning | High | |
| `RL_VAR_005` | Replicated dynamic array | Warning | Informational | |
| `RL_VAR_006` | Replicated reference to a plain UObject *(experimental)* | Warning | Potential | |
| `RL_VAR_007` | GameState or PlayerState variable without RepNotify *(experimental)* | Info | Informational | |

</details>

<details>
<summary><b>▸ Blueprint logic, 2 rules</b></summary>

<br/>

| ID | Rule | Severity | Confidence | Fix |
|---|---|:---:|:---:|:---:|
| `RL_BP_001` | Replicated actor spawned without authority check *(experimental)* | Major | Potential | |
| `RL_BP_002` | Replicated variable written in Event Tick | Warning | High | |

</details>

<details>
<summary><b>▸ Components, 2 rules</b></summary>

<br/>

| ID | Rule | Severity | Confidence | Fix |
|---|---|:---:|:---:|:---:|
| `RL_COMP_001` | Replicated component on a non-replicated actor | Critical | Definite | ✅ |
| `RL_COMP_002` | Replicated components use the legacy replication path *(experimental)* | Info | Informational | |

</details>

<details>
<summary><b>▸ Actor settings, 6 rules</b></summary>

<br/>

| ID | Rule | Severity | Confidence | Fix |
|---|---|:---:|:---:|:---:|
| `RL_ACT_001` | Replicate Movement on a likely static actor *(experimental)* | Warning | Potential | ✅ |
| `RL_ACT_002` | NetUpdateFrequency very high | Warning | High | ✅ |
| `RL_ACT_003` | Low NetUpdateFrequency on a Pawn | Warning | High | ✅ |
| `RL_ACT_004` | Replicated actor without network data *(experimental)* | Info | Informational | |
| `RL_ACT_005` | Always Relevant on a gameplay actor *(experimental)* | Warning | Potential | ✅ |
| `RL_ACT_006` | Initially dormant actor carries replicated data | Major | High | |

</details>

<details>
<summary><b>▸ Ownership and security, 3 rules</b></summary>

<br/>

| ID | Rule | Severity | Confidence | Fix |
|---|---|:---:|:---:|:---:|
| `RL_OWN_001` | Client RPC with unclear ownership *(experimental)* | Major | Potential | |
| `RL_SEC_001` | Server RPC without WithValidation *(experimental)* | Info | Informational | |
| `RL_SEC_002` | Server RPC accepts an object reference from the client *(experimental)* | Warning | Potential | |

</details>

The full explanation of each rule, including when it is legitimately wrong, lives in the **Rules** view
inside the editor.

> [!NOTE]
> **Five rules only exist for Blueprints.** `RL_RPC_003`, `RL_RPC_004`, `RL_BP_001` and `RL_BP_002`
> follow the Event Tick execution chain, which only exists in a Blueprint graph. `RL_VAR_004` needs to
> see the body of the RepNotify function, and C++ bodies are not parsed.

---

## 📊 Grade, baseline and history

### How the grade is computed

Only **confirmed** findings count, meaning confidence Definite or High. Heuristics are listed under
**Needs review** and never move the grade, unless you tick *Potential Findings Affect Grade*.

| Grade | When |
|:---:|---|
| **A** | No Critical, no Major, no Warning |
| **B** | Warnings only |
| **C** | 1 or 2 Major, no Critical |
| **D** | 1 or 2 Critical, or 3 Major and more |
| **E** | 3 Critical and more |

Info findings and ignored findings never count.

### Baseline

On a real project the first scan can list a hundred findings. You are not going to fix them all today,
and a list that never gets shorter is a list nobody reads.

**Set Baseline** freezes the current findings as your accepted backlog. From then on the Dashboard
tells you what is **new** since that moment and what got **fixed**. The day to day question becomes
*"did I just add a problem?"*, which is one you will actually act on.

- **Update Baseline** replaces it with the current findings
- **Clear Baseline** deletes it, and every finding is reported again
- Tick **Hide Baselined Findings** in Project Settings to keep only new ones in the Issues view

The baseline is saved under `Config/ReplicationLint/`, so you can commit it and share it with your
team.

> [!NOTE]
> The baseline is an editor feature. The CI commandlet gates on severity, grade and count, it does not
> read the baseline.

### History

Every full scan is saved in `Saved/ReplicationLint/History/`. That is what feeds the grade trend and
the *since last scan* diff. A folder scan is partial, so it is kept out of the history on purpose:
otherwise everything outside the folder would show up as *fixed*.

---

## 🙈 Ignoring findings

Some findings are intentional. A GameState with 40 replicated variables is a data hub, not a mistake.

- **Ignore** on a row hides that one finding
- **Ignore everything in a category** or **on a class** from the detail panel
- **Show ignored** brings them back, with **Restore** on each

Ignored findings leave the grade, the reports and the CI gates. The list is stored in your project
config, so the whole team and the build machine agree on what was accepted.

To skip whole areas instead, use **Ignored Content Folders** and **Ignored Class Names** in Project
Settings.

---

## 📄 Reports

**Export Report** in the top bar, after a scan.

| Format | Good for |
|---|---|
| **HTML** | Sending to the team, attaching to a milestone |
| **Markdown** | GitHub, wikis, pull requests |
| **JSON** | CI and your own tooling |
| **CSV** | Excel, Google Sheets, bug trackers |

Reports go to `Saved/ReplicationLint/Reports/` and the folder opens on its own. Change it with
**Custom Report Directory** in Project Settings.

<!-- PLACEHOLDER: screenshot of an HTML report open in a browser -->
<img src="docs/images/ReportHTML.png" alt="HTML report" width="80%"/>

**Scan one folder only:** right click a folder in the Content Browser → **Scan with Multiplayer Replication Lint**.

---

## 🤖 Continuous integration

A commandlet runs the same scan headless, so a pull request that adds a critical replication bug can
fail like a failing unit test.

```
UnrealEditor-Cmd.exe MyProject.uproject -run=ReplicationLintScan ^
  -FailOn=critical -Format=json+html -unattended -nopause -nosplash
```

| Option | Default | What it does |
|---|---|---|
| `-Paths=/Game/A+/Game/B` | `/Game` | Content paths to scan, separated by `+` |
| `-Format=json+html` | `json` | Report formats: `html`, `md`, `json`, `csv` |
| `-FailOn=critical` | `critical` | Fail if a finding at or above this severity exists. `none` turns it off |
| `-MinGrade=C` | *off* | Fail if the grade is worse than this letter |
| `-MaxIssues=20` | *off* | Fail above this number of findings |
| `-MinConfidence=definite` | *off* | Only findings at or above this confidence can fail the build |
| `-NoCache` | *off* | Clear the incremental cache first, for a cold reproducible scan |

| Exit code | Meaning |
|:---:|---|
| `0` | Every gate passed |
| `1` | A gate failed. This is what your pipeline acts on. |
| `2` | The scan could not run: bad argument, no result |

<details>
<summary><b>▸ Start permissive, then tighten</b></summary>

<br/>

Failing the build on day one against a legacy project mostly teaches people to ignore the check.

```
Week 1   -FailOn=none                                    report only
Week 2   -FailOn=critical -MinConfidence=definite        block what is provable
Later    -FailOn=major -MaxIssues=15 -MinGrade=B         ratchet down
```

</details>

<details>
<summary><b>▸ GitHub Actions</b></summary>

<br/>

```yaml
name: Replication audit

on: [pull_request]

jobs:
  replication-audit:
    runs-on: [self-hosted, unreal]   # needs an engine installation
    steps:
      - uses: actions/checkout@v4

      - name: Run Multiplayer Replication Lint
        shell: pwsh
        run: |
          & "$env:UE_ROOT\Engine\Binaries\Win64\UnrealEditor-Cmd.exe" `
            "${{ github.workspace }}\MyProject.uproject" `
            -run=ReplicationLintScan `
            -Format=json+html `
            -FailOn=critical `
            -unattended -nopause -nosplash

      - name: Upload report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: replication-report
          path: Saved/ReplicationLint/Reports/
```

`if: always()` matters: you want the report precisely when the gate failed.

</details>

---

## 🍳 Recipes

### Your first pass on an existing project

The first scan on a project that has never been audited is long. Do not try to fix it in one sitting.

1. **Run Full Scan**, then sort Issues by severity
2. **Fix the Criticals.** They are provable: something you wrote will never reach clients. Most have
   a Safe Fix button
3. **Read the Majors**, do not batch them. Each one is a real bug in a real session, and most need a
   design decision
4. **Set Baseline.** Everything left becomes accepted backlog
5. From then on, the Dashboard answers the only question that matters day to day: *did I just add a
   problem?*

### Wiring it into a pull request

Do this in three steps, weeks apart, not all at once.

| When | Command | Effect |
|---|---|---|
| Week 1 | `-FailOn=none` | Nothing blocks. Everyone sees the alerts and gets used to them |
| Week 2 | `-FailOn=critical -MinConfidence=definite -NewOnly` | Only provable, newly added problems block |
| Later | `-FailOn=major -MinGrade=B` | Tighten as the backlog shrinks |

Failing every build on day one against a legacy project teaches the team to switch the check off.

### Tuning the rules to your game instead of ignoring them

If a rule fires constantly and you keep clicking Ignore, the threshold is wrong for your game, not the
finding.

- A competitive shooter legitimately runs player pawns above 100 Hz. Raise **Max Net Update
  Frequency** rather than ignoring `RL_ACT_002` forty times
- A GameState is a data hub and will hold many replicated variables. Raise **Max Replicated Variables
  Per Class**
- A whole third-party folder you do not own goes in **Ignored Content Folders**, not in the ignore
  list one finding at a time

Ignore is for the exception. Settings are for the pattern.

### Finding out why one actor will not replicate

Faster than reading the whole Issues list when you already know which Blueprint misbehaves.

1. Right-click the Blueprint in the Content Browser → **Diagnose Replication**
2. Read the ACTOR block first. If it says BLOCKED, nothing below it matters
3. Then read the block for the variable or RPC you care about
4. Lines marked **CHECK** are conditions the tool cannot verify statically, for example ownership.
   They are the usual answer when everything says PASS and it still does not work

### Handing a report to someone who does not use Unreal

Export **HTML**. It opens in any browser, carries every explanation, and needs no plugin, no engine
and no context. It is the right artifact for a producer, a client or a milestone review.

Export **CSV** instead if they are going to turn findings into tickets.

---

## ⚙️ Settings

**Edit → Project Settings → Plugins → Multiplayer Replication Lint**. Saved in `Config/DefaultReplicationLint.ini`, so the
team shares them through source control.

<details>
<summary><b>▸ Thresholds</b></summary>

<br/>

| Setting | Default | Used by |
|---|:---:|---|
| **Max Replicated Variables Per Class** | 25 | `RL_VAR_002` |
| **Max Net Update Frequency** | 100 | `RL_ACT_002` and its Fix |
| **Min Net Update Frequency For Pawns** | 10 | `RL_ACT_003` and its Fix |
| **Large Replicated Property Bytes** | 64 | `RL_VAR_003` |
| **Large RPC Payload Bytes** | 256 | `RL_RPC_005` |
| **Max RPC Parameters** | 6 | `RL_RPC_007` |

A competitive shooter legitimately pushes player pawns above 100 Hz. Tune these to your game rather
than ignoring findings one by one.

</details>

<details>
<summary><b>▸ Rules</b></summary>

<br/>

| Setting | Default | Purpose |
|---|:---:|---|
| **Disabled Rules** | *empty* | Rule IDs that never report. Same as unticking them in the Rules view. |
| **Enable Experimental Diagnostics** | ✅ | Master switch for every experimental rule. |
| **Severity Overrides** | *empty* | Report a rule at the severity your team agreed on. |
| **Ignored Diagnostic Ids** | *empty* | Filled by the Ignore button. Safe to edit by hand. |
| **Ignore Reasons** | *empty* | Optional note per ignored finding. Six months later, *why is this suppressed?* is a real question. |

</details>

<details>
<summary><b>▸ Scan scope</b></summary>

<br/>

| Setting | Default | Purpose |
|---|:---:|---|
| **Native Modules To Scan** | `ReplicationLintDemo` | C++ modules to scan on top of Blueprints. |
| **Ignored Content Folders** | *empty* | Folders to skip entirely, for example `/Game/ThirdParty`. |
| **Ignored Class Names** | *empty* | Classes to skip, name without prefix. |

</details>

<details>
<summary><b>▸ History, cache, baseline, reports</b></summary>

<br/>

| Setting | Default | Purpose |
|---|:---:|---|
| **Save Scan History** | ✅ | Keeps a snapshot of each full scan for the trend and the diff. |
| **Max History Entries** | 30 | Oldest snapshots are deleted beyond this. |
| **Use Incremental Cache** | ✅ | Reuses results for Blueprints that did not change since the last scan. |
| **Hide Baselined Findings** | ❌ | Shows only new findings in the Issues view. |
| **Default Export Format** | HTML | Format picked by default in Export Report. |
| **Custom Report Directory** | *empty* | Where reports are written. Empty means `Saved/ReplicationLint/Reports/`. |

</details>

<!-- PLACEHOLDER: screenshot of the Project Settings page -->
<img src="docs/images/ProjectSettings.png" alt="Project Settings" width="70%"/>

---

## 🧠 Behaviour you should know

**Zero cost in your shipped game.** All three modules are Editor modules, so nothing at all is
compiled into a packaged build.

**The demo actors.** The `ReplicationLintDemo` module holds four C++ actors: three broken on purpose and
one clean reference. They are never scanned by default: sample content that is broken on purpose has
no business putting Critical findings into your project. Click **Run Demo Audit** to scan them once.

**It reads what is in the editor, not only what is saved.** A Blueprint you changed and did not save
yet is scanned with your changes. That is why a finding disappears right after a fix, before you
save.

**Repeated scans are fast.** Blueprints whose file did not change are not reloaded, their results come
from a cache in `Saved/ReplicationLint/Cache/`. The cache throws itself away when you change a
threshold, a rule toggle, the editor language or the plugin version, so it never serves stale results.

**Blueprint analysis prefers missing a case to inventing one.** The Tick walker follows execution pins
one graph at a time. It does not expand your macros and does not follow delegates, interfaces or
timelines. A throttled call it cannot see through will be flagged, and the explanation says so.

**Sizes are estimates.** Property and payload sizes come from in memory reflection sizes, not from what
actually goes on the wire after delta compression and quantization. Good for spotting the big ones,
not for a bandwidth budget. Use Networking Insights for that.

**The first scan of a big project takes a while**, because it has to load every Blueprint once. The
dialog can be cancelled at any time and partial results are kept.

**Language.** The interface is in English. The plugin is built on Unreal's localization system and
French, Spanish, Russian and Simplified Chinese are planned, but nothing is claimed until a language
is complete and verified.

---

## 🔧 Troubleshooting

<details open>
<summary><b>▸ Most common questions</b></summary>

<br/>

| Symptom | Cause |
|---|---|
| **My C++ classes are not in the results** | Their module is not in **Native Modules To Scan**. |
| **A Tick rule never fires on my C++ class** | Tick rules read Blueprint graphs only. C++ function bodies are not parsed. |
| **A rule never reports anything** | It is unticked in the Rules view, or it is experimental and **Enable Experimental Diagnostics** is off. |
| **No Fix button** | The finding is on a C++ class, or its rule has no safe automatic fix. |
| **Ctrl+Z does not revert a fix** | The Blueprint was compiled after the fix. Undo before compiling, or set the value back by hand. |
| **An ignored finding came back** | You renamed the class, the variable or the function. An ignore is tied to the exact names. |
| **The grade did not move after a fix** | Scan again. The grade is computed at scan time. |

</details>

<details>
<summary><b>▸ Lag Lab</b></summary>

<br/>

| Symptom | Cause |
|---|---|
| **`stat net` shows 0 everywhere** | Play Net Mode is **Standalone**, so there is no network to emulate. The window title says *NetMode: Standalone*. Switch to **Play As Client**, the Lag Lab has a button for it. |
| **Ping is higher than the preset** | PIE adds its own frame time on top. Apply **Clean LAN**, note the ping, and compare the preset against that. The preset should add its range on top of it. |
| **The Max column shows one big spike** | `stat net` keeps the worst moment, including connection. Watch the live value for a few seconds instead. |
| **The host has no lag** | You are playing as Listen Server. The host is the server, so it has no connection to delay. Use Play As Client. |

</details>

<details>
<summary><b>▸ CI</b></summary>

<br/>

| Symptom | Cause |
|---|---|
| **Exit code 2** | An unknown value in an option. The log names it. |
| **A finding the team ignored fails the build** | The ignore list was not committed. It lives in the project config. |
| **Every run is slow on the build machine** | The agent starts with an empty `Saved/` each time, so the cache is cold. Expected on ephemeral agents. |

</details>

### Logs

Filter the **Output Log** on `LogReplicationLint`. Each fix writes one line with the number of
placed actors it updated. The CI commandlet prints its summary, then one `GATE FAILED` line for each
gate that did not pass.

---

## 🎯 Scope

**Multiplayer Replication Lint does static analysis. It is not a runtime profiler.**

| Multiplayer Replication Lint does | Multiplayer Replication Lint does not |
|---|---|
| Read your classes and Blueprint graphs | Watch a live session |
| Flag setups that cannot work or will cost too much | Measure real bandwidth |
| Explain every finding and how to fix it | Rewrite your game logic |
| Fix configuration mistakes in one click | Touch graphs or C++ |
| Fail a build on new critical issues | Replace testing with real players |

**Also out of scope:**

- **C++ function bodies.** Only reflection data is read.
- **Bandwidth numbers from a real session.** That is Networking Insights' job.

---

## 💬 Support

<div align="center">

**Found a bug? Need a feature? Have a question?**

[![Discord](https://img.shields.io/badge/Ask%20a%20question-Discord-5865F2?style=for-the-badge&logo=discord&logoColor=white)](https://discord.gg/BwhyxQAAUn)
[![Fab](https://img.shields.io/badge/Fab-Listing-FF6B00?style=for-the-badge)](https://www.fab.com/sellers/Kybrien)

*Built by Kybrien, the developer of Twitch StreamSync and Even Richer Discord Presence.*

</div>

---

<div align="center">
<sub>

**MULTIPLAYER REPLICATION LINT IS AN INDEPENDENT UNREAL ENGINE PLUGIN AND IS NOT AFFILIATED WITH,**

**ENDORSED BY, OR SPONSORED BY EPIC GAMES, INC. UNREAL AND UNREAL ENGINE ARE TRADEMARKS OR**

**REGISTERED TRADEMARKS OF EPIC GAMES, INC. IN THE UNITED STATES OF AMERICA AND ELSEWHERE.**

Copyright 2026 Kybrien. All Rights Reserved.

</sub>
</div>
