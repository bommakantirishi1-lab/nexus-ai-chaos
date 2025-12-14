EVOLUTION_ARCHITECTURE.md# NEXUS AI CHAOS - Evolution Architecture (v0.1.0 → v0.2.0)

## Overview

This document describes the scaling architecture implemented for NEXUS AI CHAOS to safely evolve from single-video generation (v0.1.0) to multi-platform distribution at volume (v0.2.0+).

## Design Principles

1. **Generation Purity**: Core generation workflow unchanged. No platform risk embedded.
2. **Failure Isolation**: Each platform has independent failure domain. YouTube down ≠ Instagram down.
3. **Sequential Before Parallel**: Volume testing runs sequentially before parallelism. Metrics first, scaling second.
4. **Trivial Rollback**: Disable one workflow = one-click recovery. No cascading failures.
5. **Platform-Specific Logic**: Each distribution workflow handles platform specs independently.

## Architecture Layers

### Layer 1: Generation (NEXUS_AI_CHAOS_PIPELINE.yml)
- **Purpose**: Produces video artifacts
- **Trigger**: Manual or scheduled
- **Status**: Unchanged from v0.1.0
- **Output**: MP4 artifact + metadata JSON
- **Failure Impact**: Low (only affects generation, not distribution)

### Layer 2: Volume Testing (NEXUS_VOLUME_DRY_RUN.yml)
- **Purpose**: Validate infrastructure at scale before distribution
- **Trigger**: Manual (dev/ops only)
- **Execution**: Sequential (no parallelism yet)
- **Input**: Run count (5, 10, 25)
- **Output**: Metrics JSON (latency, cost, failure rate)
- **Success Criteria**:
  - 10 sequential generations, zero failures
  - <100MB artifact storage
  - Clean metrics JSON
  - No platform exposure

**Why Sequential?**
- Latency is measurable and clean (no concurrency noise)
- API rate-limit behavior is predictable
- Cost per video is accurate
- Failure modes are isolated to single invocation

### Layer 3: Platform Distribution (Isolated)

Each platform has its own workflow. All follow the same pattern:

#### NEXUS_DISTRIBUTE_YOUTUBE.yml
- **Spec**: 9:16 aspect ratio, <60s, H.264/AAC
- **Auth**: YouTube API v3 (YOUTUBE_API_KEY, YOUTUBE_CHANNEL_ID)
- **Retry**: 3x with exponential backoff
- **Failure**: Does NOT affect Instagram or X
- **Rollback**: Disable workflow, artifacts remain for manual retry

#### NEXUS_DISTRIBUTE_INSTAGRAM.yml
- **Spec**: 9:16 aspect ratio, <90s, caption+hashtag handling
- **Auth**: Instagram Graph API (INSTAGRAM_ACCESS_TOKEN, INSTAGRAM_BUSINESS_ACCOUNT_ID)
- **Retry**: 3x with exponential backoff
- **Failure**: Does NOT affect YouTube or X
- **Rollback**: Disable workflow, artifacts remain for manual retry

#### NEXUS_DISTRIBUTE_XVIDEO.yml
- **Spec**: 16:9 or 9:16 adaptive, tweet composition
- **Auth**: X API v2 (X_API_KEY, X_API_SECRET, X_BEARER_TOKEN)
- **Retry**: 3x with exponential backoff
- **Failure**: Does NOT affect YouTube or Instagram
- **Rollback**: Disable workflow, artifacts remain for manual retry

## Execution Flow

```
Generation (1 job) → Volume Test (10 jobs sequential) → Distribution (3 independent jobs)
```

### Phase 1: Generate
```
Manual Trigger → NEXUS_AI_CHAOS_PIPELINE → MP4 Artifact
```
Time: ~2-5 minutes

### Phase 2: Validate at Scale (Optional but Recommended)
```
Manual Trigger → NEXUS_VOLUME_DRY_RUN (10 sequential runs) → Metrics JSON
```
Time: ~20-30 minutes for 10 runs

### Phase 3: Distribute to Platforms
```
Manual Trigger → NEXUS_DISTRIBUTE_YOUTUBE → YouTube Success
Manual Trigger → NEXUS_DISTRIBUTE_INSTAGRAM → Instagram Success
Manual Trigger → NEXUS_DISTRIBUTE_XVIDEO → X Success
```
Time: ~2-3 minutes each (parallel possible, but manual for now)

## Data Flow

```
Generation Output
  ├── video_master.mp4 (artifact)
  ├── metadata.json
  └── artifact_id (GitHub Actions artifact)

Volume Test Input
  └── artifact_id OR S3 URL

Volume Test Output
  └── metrics-{test_id}.json (latency, cost, failure_rate)

Distribution Input (per platform)
  ├── artifact_id OR S3 URL
  ├── title/caption
  ├── hashtags/tags
  └── platform-specific metadata

Distribution Output (per platform)
  ├── platform_post_id
  ├── platform_url
  └── platform_status (success/failure)
```

## Risk Mitigation

### What Could Go Wrong?

| Risk | Isolation | Mitigation |
|------|-----------|------------|
| YouTube API rate limit | ✅ Isolated to YouTube workflow | Independent retry (3x). Instagram/X unaffected |
| Instagram auth failure | ✅ Isolated to Instagram workflow | Independent retry (3x). YouTube/X unaffected |
| X API outage | ✅ Isolated to X workflow | Independent retry (3x). YouTube/Instagram unaffected |
| Volume test failure | ✅ No platform exposure | Abort, don't proceed to distribution |
| Cost explosion | ✅ Metrics visible before scaling | Volume test shows cost baseline. Abort if > threshold |

### One-Click Rollback

1. Disable problematic workflow in GitHub Actions UI
2. Artifacts remain available (7-day retention)
3. No data loss, no cascade failures
4. Manual retry available anytime

## Next Steps (Beyond v0.2.0)

### v0.3.0 - Automation
- Parallelize volume dry runs (10 concurrent)
- Trigger distribution automatically after volume test passes
- Scheduled daily generation + distribution

### v0.4.0 - Observability
- Webhook notifications to Discord (success/failure per platform)
- Metrics aggregation (daily reports)
- Performance dashboard

### v0.5.0 - Intelligence
- A/B testing (different scripts, measure engagement)
- Platform-specific content modulation
- Engagement feedback loop

## Success Metrics (v0.2.0)

✓ Generation workflow stable and unchanged
✓ Volume dry run validates 10 sequential generations
✓ YouTube distribution isolated (manual, working)
✓ Instagram distribution isolated (manual, working)
✓ X distribution isolated (manual, working)
✓ Each platform failure is independent
✓ One-click rollback available
✓ Zero cascade failures
✓ Artifacts retained for manual retry
