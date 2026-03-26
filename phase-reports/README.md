# Phase Reports

This folder collects completion reports from each phase agent. The orchestrator session reviews these before authorizing the next phase.

## Expected Reports

| File | Source Project | Phase |
|------|---------------|-------|
| `PHASE_1_REPORT.md` | MBS/ | MBS Platform (SSO + billing + entitlements) |
| `PHASE_2_REPORT.md` | Innerlab/ | Inner Lab Middleware + Dashboard |
| `PHASE_3_REPORT.md` | CWG/ | CWG Migration + Refactor |
| `PHASE_4_REPORT.md` | YogaGhost/ | FlowState Migration + Refactor |
| `PHASE_5_REPORT_{product}.md` | Each Arcade/SW app | Standalone SSO Migration (11 reports) |

## Workflow

1. Phase agent finishes work and generates report in its project root
2. Agent also copies the report to this folder
3. User opens the orchestrator session (MBSPlatform) and asks it to review
4. Orchestrator reads the report, checks for deviations, updates downstream instructions if needed
5. User proceeds to next phase only after orchestrator gives green/yellow/red assessment
