# NEXUS: AI CHAOS — GROWTH PLAYBOOK (ALL 4 AXES)

This playbook provides production-ready implementations for scaling NEXUS: AI CHAOS.
Choose ONE axis, or combine them sequentially.

---

## AXIS 1: VOLUME SCALE (2–5 Shorts/Day)

**Goal:** Maximize content output
**Effort:** 2-3 days
**ROI:** Immediate (more content = more views = faster monetization)
**Risk:** API costs scale, quality may suffer if not monitored

### Architecture

```
Topic Queue → Batch Scheduler → Parallel Execution → Multi-output
                    ↓
         [6 AM, 12 PM, 6 PM UTC]
```

### Implementation

**Step 1: Topic Queue System**
- Create `topics_queue.json` (manually curated or auto-fed from trends)
- Format: `{"topic": "...", "priority": 1-5, "created_at": "timestamp"}`
- Rotate through queue, mark as processed

**Step 2: Batch Scheduler Workflow**
- Add new GitHub Actions workflow: `batch_generator.yml`
- Schedule: `cron: '0 6,12,18 * * *'` (3 runs/day)
- Pull next 1-2 topics from queue
- Spawn 2-5 parallel NEXUS_AI_CHAOS_PIPELINE executions
- Wait for all to complete, aggregate results

**Step 3: Rate Limiting (CRITICAL)**

Cost per video:
- ChatGPT: ~$0.01 (gpt-4-turbo)
- Grok Imagine: ~$0.05 (estimate, needs verification)
- FFmpeg: $0 (local)
- TTS: ~$0.01 (optional)
- **Total per video: ~$0.07**

Budget safety:
- 5 shorts/day × $0.07 = $0.35/day
- ~$10/month baseline
- Set GitHub Actions concurrent job limit: 2 (prevents runaway)
- Add spend alerts via AWS/GCP billing

**Step 4: Failure Handling**
- If ChatGPT fails: Re-queue topic with backoff (exponential, max 3 retries)
- If Grok fails: Skip visual, use static cyberpunk backdrop, flag for manual review
- If FFmpeg fails: Fallback to no-audio version
- Never skip topic: All failures are logged and retried

### Monitoring

```
Metrics:
- Execution latency (per topic, per step)
- API error rates (429, 400, 500, timeout)
- Queue depth (how many topics waiting)
- Cost per video (trending up/down)
- Success rate (% of topics → final video)
```

Alerts:
- Email if success rate < 85% in a day
- Slack notification if cost/video > $0.10
- Warn if queue grows > 10 items (backlog risk)

---

## AXIS 2: QUALITY SCALE (Hook A/B Testing)

**Goal:** Maximize watch time and engagement
**Effort:** 3-5 days
**ROI:** Long-term (better CTR → sustainable growth → higher CPM)
**Risk:** Complexity (analytics, ML classification)

### Architecture

```
Topic → 3 Hook Variants → YouTube Publish (staggered) → Collect Metrics → Winner → Scale
           (ChatGPT)
```

### Implementation

**Step 1: Dual-Hook Generation (ChatGPT)**
- Modify ChatGPT prompt: "Generate 3 different hooks for this topic"
- Hook A: "Shocking stat" (retention via surprise)
- Hook B: "Question format" (retention via curiosity)
- Hook C: "Visual threat" (retention via fear/dark aesthetic)
- All generate scripts with same duration/scene structure

**Step 2: YouTube Publishing (Staggered)**
- Publish all 3 variants to YouTube Shorts
- Space them 12 hours apart
- Tag each with `hook_variant=A|B|C` in description
- Randomize user exposure (don't let same user see all 3)

**Step 3: Metrics Collection**
- After 24 hours, pull YouTube Analytics
- Track: Click-Through Rate (CTR), Watch Time, Drop-off Rate, Completion Rate
- Score: `(CTR × 0.4) + (Completion% × 0.6)` = Hook Score

**Step 4: Promotion Rules**
- Hook with highest score → Promote to main channel (pin, cross-post)
- Other hooks → Archive or delete
- Learning: Update ChatGPT prompt weights based on winning hook type

### Monitoring

```
Metrics:
- Hook A effectiveness (avg CTR, completion)
- Hook B effectiveness
- Hook C effectiveness
- Winner rate distribution (which hooks win most often)
- Engagement trend over 30 days
```

Actionable Insights:
- If "Shocking stat" wins 70% → Double down on data-driven hooks
- If "Question format" wins → Add more open-ended curiosity hooks
- If CTR is declining → Refresh hook style entirely

---

## AXIS 3: DISTRIBUTION SCALE (Auto-Publish)

**Goal:** Maximize platform coverage (YouTube, Instagram, X)
**Effort:** 2-3 days (YouTube implementation, then extend)
**ROI:** Immediate (same content, multiple audiences)
**Risk:** Platform-specific quirks, quota limits

### Architecture

```
Video Master → YouTube Shorts → Instagram Reels → X Video
                   (auth)           (auth)        (auth)
                   ↓                 ↓              ↓
                [queue]          [queue]        [queue]
```

### Implementation

**Step 1: YouTube Shorts OAuth Setup**
- Create YouTube Data API credentials (OAuth 2.0)
- Store refresh token in GitHub Secret: `YOUTUBE_REFRESH_TOKEN`
- Implement token refresh logic (24hr expiry)
- Create separate workflow: `publish_youtube.yml`

**Step 2: Upload Logic**
```
1. Retrieve video_youtube_shorts.mp4
2. Prepare metadata (title, description, tags)
3. Upload via YouTube API with:
   - Privacy: UNLISTED (review first)
   - Notify: No email
   - Auto-caption: Enabled
4. Return video_id
5. Update status log
```

**Step 3: Quotas & Rate Limiting**
- YouTube Shorts: 100 uploads/day limit (safe at 5/day)
- Instagram Reels: 25 uploads/day limit (safe at 5/day)
- X Video: No hard limit, but respect 15/15min rate limit
- Stagger uploads: 5min between each platform

**Step 4: Rollback Strategy**
- If YouTube upload fails: Keep video in queue, retry in 1hr
- If Instagram fails: Try again in 2hrs (longer backoff)
- If X fails: Queue for next batch
- Never delete published content unless explicitly flagged

### Monitoring

```
Metrics:
- Upload success rate per platform
- Time-to-publish (queue wait + upload time)
- Quota usage per platform
- Engagement per platform (views, likes, shares)
```

Alerts:
- If platform quota > 75% → Slow down uploads
- If upload fails 3x → Escalate to manual review

---

## AXIS 4: INTELLIGENCE SCALE (Auto-Ingest Trends)

**Goal:** Always be on-trend
**Effort:** 4-5 days
**ROI:** Medium-term (better topic selection → higher CTR)
**Risk:** Signal noise (not all trends = good content)

### Architecture

```
CVE Feed     Twitter Trends    Reddit Trending
   ↓              ↓                   ↓
  [Parse]     [Extract]          [Classify]
     ↓              ↓                   ↓
  [Score & Rank]
     ↓
 [Topic Queue]
     ↓
 [NEXUS Pipeline]
```

### Implementation

**Step 1: CVE Feed Ingestion**
- Source: NVD API (`https://services.nvd.nist.gov/rest/json/cves`)
- Filter: CVSS score > 7 (critical only)
- Daily schedule: `cron: '0 8 * * *'` (8 AM UTC)
- Convert to topic: "Zero-Day in {product} — CVSS {score} — Patch Now"

**Step 2: Twitter/X Trends**
- Source: Twitter API v2 (requires Academic Research access)
- Monitor: `#cybersecurity`, `#AI`, `#infosec` trending keywords
- Filter: Only retweets > 1000 (signal strength)
- Convert: "Breaking: {trend} — What You Need to Know"

**Step 3: Reddit Trending**
- Source: Reddit API (r/cybersecurity, r/ArtificialIntelligence, r/security)
- Criteria: Upvotes > 500, comments > 100
- Parse title: Extract key terms
- Convert: "{reddit_title} — Explained"

**Step 4: Ranking Logic**

Score each candidate topic:
```
Score = (Freshness × 0.3) + (Signal_Strength × 0.4) + (Category_Match × 0.3)

Freshness: Hours since signal (newer = higher)
Signal_Strength: Engagement (views/shares/upvotes)
Category_Match: Is it cyber+AI focused? (binary: 0 or 1)
```

Keep top 10 by score → Feed to topic queue

### Monitoring

```
Metrics:
- # signals processed per day
- Top trending topics (auto-generated)
- Topic-to-video latency (hours from trend to published)
- Video performance by source (CVE vs Twitter vs Reddit)
```

Insights:
- If CVE-sourced videos underperform → Reduce CVE weight
- If Twitter trends = higher engagement → Increase Twitter weight
- If latency > 24hrs → Topic is stale (refresh sources)

---

## COMBINATION STRATEGY

**Phase 1 (Weeks 1-2):** VOLUME only
- Get to 5 shorts/day consistently
- Monitor costs
- Establish baseline metrics

**Phase 2 (Weeks 3-4):** Add QUALITY
- Start A/B testing hooks
- Identify winning patterns
- Refine ChatGPT prompts based on winners

**Phase 3 (Weeks 5-6):** Add DISTRIBUTION
- Publish to Instagram & X
- Track cross-platform engagement
- Optimize metadata per platform

**Phase 4 (Week 7+):** Add INTELLIGENCE
- Auto-feed from trends
- Validate signal quality
- Continuously optimize ranking

**By Week 8:** You'll have a fully autonomous, trending-aware, multi-platform content machine.

---

## ESTIMATED TIMELINE

- VOLUME: 2-3 days → 5 videos/day
- QUALITY: +3 days → Hook A/B testing
- DISTRIBUTION: +2 days → Multi-platform
- INTELLIGENCE: +4 days → Trend-driven
- **Total: ~2 weeks to full autonomous operation**

---

## FINAL NOTE

Pick ONE axis to implement first. Don't try all 4 at once—that's chaos.
Each axis stabilizes in 5-7 days before adding the next.
