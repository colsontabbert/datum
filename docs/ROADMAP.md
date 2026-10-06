# Datum roadmap

My guide, written for me rather than for Claude: what each session does, what I have to do before and during each phase, and every card. The detail lives in `docs/BUILD_PLAN.md` and `docs/SPEC.md`.

**Which cards are done:** a card ticks its own box in its pull request, so a ticked box means merged.

**At a glance:** 28 cards in 3 phases.

| Phase | What I get | Cards | Rough time | Gate |
|---|---|---|---|---|
| 0 | Proof that frame times can be captured on OpenGL and Vulkan | 1 | one sitting | none |
| 1 | v1: unattended sessions, findings, a report, and a desktop GUI | 23 | months | Phase 0 proof passed |
| 2 | Older versions and loaders, sharing | 4 | open | v1 used for real tuning decisions |

## Interview groups

Before a group's cards are built, `/interview <group>` talks them through with me so no card has to guess. Each milestone is a group.

- **pending:** not talked through yet. `/card` won't start this group's cards, or any card after the group's first unstarted card, and warns me once one of them is among the next three.
- **done:** talked through, with the date.
- **skip:** no interview needed (settled in the first interview, or every card already started).

| Group | Milestone | Cards | Interview |
|---|---|---|---|
| Manual proof | 0A | P0-01 | skip (settled 2026-10-06) |
| Runner skeleton | 1A | P1-01 to P1-07 | done (2026-10-06) |
| Probe and frozen worlds | 1B | P1-08 to P1-10 | pending |
| Honest numbers | 1C | P1-11 to P1-13 | pending |
| Settings sweep, packs, and versions | 1D | P1-14 to P1-16 | pending |
| Mod testing | 1E | P1-17, P1-18 | pending |
| The look and the report | 1F | P1-19, P1-20 | pending |
| Desktop GUI | 1G | P1-21 to P1-23 | pending |
| Older versions and loaders | 2A | P2-01, P2-02 | pending |
| Sharing | 2B | P2-03, P2-04 | pending |

## How a session goes

1. Open a new session and type `/card` (Codex: `$card`). It warns me first if an interview is coming up, then asks any questions cards are parked on, then picks the next card that's safe to start and says why and its effort.
2. Check the effort matches (M: medium, H: high, X: extra high).
3. Read its plan. M and H cards keep going; X cards wait for my "go".
4. When it's done it runs the checks, gets a review, and merges (X cards and checks only I can do wait for me). Then: a new session, `/card` again.

Other commands: `/card status` (where everything stands), `/card <ID>` (that card), `/card <idea>` (a new card), `/interview <group>`, `/side <repo task>`, `/patch <fix>`.

---

## Phase 0: Manual proof (1 card)

**Before you start:** (a `/side` task ticks these once I confirm)
- [ ] PresentMon 2.6.0 console app downloaded into the repo's `tools/` folder
- [ ] A copy of my FO 26.2 instance, made with Prism's Copy button
- [ ] A copy of my FO 26.3 instance, made with Prism's Copy button

**You'll do during this phase:** launch the copies by hand while the session watches and reads the files (P0-01).

**Manual proof**
- [ ] **P0-01** Capture frame times by hand on OpenGL and Vulkan · **M**. *I'm at the laptop for the whole card.*

**Gate to the next phase:** PresentMon captures valid frame times on both OpenGL and Vulkan, and the game's backend and device log lines are recorded for both. If Vulkan can't be captured, we stop and rethink.

---

## Phase 1: v1 for me (23 cards)

**Before you start:** nothing beyond Phase 0.

**You'll do during this phase:** be at the laptop for every card that launches the game (P1-03 to P1-07, and most cards after); okay every X card before code and before merge; sign off on screenshots for the look, the report, and the GUI (P1-19 to P1-23); build the stress pack instance before P1-16.

**Runner skeleton**
- [ ] **P1-01** `datum check`: is the laptop ready? · **H**
- [ ] **P1-02** Plans, session folders, and the time estimate · **H**
- [ ] **P1-03** Throwaway clones of my instance · **X**. *I'm there for a real clone.*
- [ ] **P1-04** Launching the game and knowing when the world is ready · **H**. *I'm there for real launches.*
- [ ] **P1-05** Recording frame times, GPU data, and Flight Recorder · **H**. *I'm there for a real run.*
- [ ] **P1-06** What each run actually ran on, and which runs to distrust · **H**
- [ ] **P1-07** A whole unattended session · **H**. *I'm there for a real 5-run session.*

**Probe and frozen worlds** (interview first)
- [ ] **P1-08** Probe: camera path, markers, clean quit · **H**. *I'm there to check it in game on 26.2 and 26.3.*
- [ ] **P1-09** Frozen test worlds · **X**
- [ ] **P1-10** Sessions use the probe · **X**

**Honest numbers** (interview first)
- [ ] **P1-11** Per-run metrics · **H**
- [ ] **P1-12** Session order, warmup, and rerun rules · **H**
- [ ] **P1-13** Statistics and `findings.md` · **H**

**Settings sweep, packs, and versions** (interview first)
- [ ] **P1-14** Several configs, with every changed file backed up and restored · **X**
- [ ] **P1-15** OpenGL against Vulkan · **H**
- [ ] **P1-16** Pack and version comparison · **H**. *Needs my stress pack instance.*

**Mod testing** (interview first)
- [ ] **P1-17** Mod list and dependencies · **X**
- [ ] **P1-18** Which mods cost or gain frames · **H**

**The look and the report** (interview first)
- [ ] **P1-19** The look: design doc, icon files, empty window · **H**. *I sign off on screenshots and the 16 px icon.*
- [ ] **P1-20** `report.html` · **H**. *I sign off on screenshots.*

**Desktop GUI** (interview first)
- [ ] **P1-21** Run tab · **H**. *I sign off on screenshots.*
- [ ] **P1-22** New session form · **X**. *I sign off on screenshots.*
- [ ] **P1-23** Results and Settings tabs · **H**. *I sign off on screenshots.*

**Gate to the next phase:** v1 works and I've used it for real tuning decisions.

---

## Phase 2: Later (4 cards)

**Before you start:** decided in the Phase 2 interviews.

**Older versions and loaders** (interview first)
- [ ] **P2-01** Bench datapack (no mod needed) · **H**
- [ ] **P2-02** Older packs on other loaders · **H**

**Sharing** (interview first)
- [ ] **P2-03** Docs and defaults for other people · **H**
- [ ] **P2-04** Shareable reports · **H**
