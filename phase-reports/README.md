# Phase Reports

This folder collects completion reports from each phase. Each phase agent saves its report in its own project root. The orchestrator fetches and copies reports here during review.

## Expected Reports

| File | Source Location | Phase |
|------|----------------|-------|
| `PHASE_1_REPORT.md` | `MBS/PHASE_1_REPORT.md` | MBS Platform (SSO + billing + entitlements) |
| `PHASE_2_REPORT.md` | `Innerlab/PHASE_2_REPORT.md` | Inner Lab Middleware + Dashboard |
| `PHASE_3_REPORT.md` | `CWG/PHASE_3_REPORT.md` | CWG Migration + Refactor |
| `PHASE_4_REPORT.md` | `YogaGhost/PHASE_4_REPORT.md` | FlowState Migration + Refactor |
| `PHASE_5_REPORT.md` (per app) | Each Arcade/SW project root | Standalone SSO Migration (11 reports) |

## Workflow

1. Phase agent finishes work and saves report in its own project root
2. User tells the orchestrator session (MBSPlatform) to review
3. Orchestrator fetches the report from the source project folder
4. Orchestrator reviews, copies report here for record keeping
5. Orchestrator updates downstream instructions if needed
6. User proceeds to next phase only after green/yellow/red assessment
