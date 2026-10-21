# EDA findings — Twitch Top Games and Clips

**Pair:** Nathan Rambo Prediger, Alexandre Cholat · **Dataset:** Top 20 Twitch games and their top 100 most viewed clips from September 2026 · **Date:** October 21, 2026
**Rows × columns:** Clips (1992 × 17), Channels (877 × 11), Games (20 × 4) · **File:** `../data/clips.parquet`, `../data/channels.parquet`, `../data/games.parquet`

---

## 1. Missingness

**Which columns, and how much?**

| Column | Missing | % | Sentinel |
| --- | ---: | ---: | --- |
| `game_id` (channels) | 5 | 0.57% | `NaN` |
| `video_id` (clips) | 477 | 23.95% | `NaN` |
| `vod_offset` (clips) | 477 | 23.95% | `NaN` |
| `igdb_id` (games) | 6 | 30.00% | `NaN` |

**Do they go missing together?** Yes, `video_id` and `vod_offset` are missing on the exact same 477 rows in the clips dataset.

**What is the mechanism?** If a broadcaster does not archive past broadcasts (VODs) or if the 14–60 day VOD retention window expires, Twitch drops both the source `video_id` and the `vod_offset` simultaneously, while keeping the clip itself. Missing `igdb_id`s occur because the categories don't have a verified mapping in the Internet Game Database.

**MCAR, MAR or MNAR, and how do you know?** MAR or MNAR depending on the broadcaster's account settings (whether they enable VODs) or Twitch's retention policies for partners vs. affiliates. 

**Decision.** Dropped `video_id`, `vod_offset`, and `igdb_id` entirely since they are irrelevant identifiers outside our scope. Row counts and medians are unchanged because we dropped the columns, not the rows. We also dropped `delay` from channels because all values were exactly 0.

---

## 2. Distributions and outliers

**Per numeric column of interest:**

| Column | min | median | max | Impossible values | Ambiguous values |
| --- | ---: | ---: | ---: | ---: | ---: |
| `view_count` | 80 | 3672.5 | 215,980 | 0 | 0 |
| `duration` | 4.9 | 24.0 | 60.0 | 0 | 50 |
| `delay` | 0.0 | 0.0 | 0.0 | 0 | 0 |

**Impossible** — violates the constraints of the data (a negative duration).
None found

**Extreme but plausible**. 
215,980 views clips. This is expected in top clip data.

**Ambiguous** — cannot be resolved from the file alone (a zero-distance trip). 
There are 50 clips with a duration of 4.9 seconds. Since the minimum Twitch clip length is exactly 5.0 seconds, these are likely small errors. We will keep them.

**Shape.** 
`view_count` is right-skewedbecause mean (8266.01) is much higher than the median (3672.5) because a few viral clips pull the average.

---

## 3. Temporal patterns

**Range covered:** 2026-09-01T00:00:00Z to 2026-09-30T23:59:59Z. This makes sense, its what we asked for.

**Gaps:** No gaps, we visualized this.

**Cycles:** Tuesday, then wedsnday, are when most clips get made. By hour, big uptick starting at 10 untill 18, then bell curve trends downwards until midnight. High activity at start of month.

**Timezone:** Timestamps are in UTC because parsing the `created_at` column with `pd.to_datetime(..., utc=True)`.

---

## 4. Representativeness

**Who or what is in this dataset?** The population is the top twitch content: the top 20 most popular games, and only the top 100 most viewed clips for each of those games during a single month.

**Who or what is missing?** Small streamers, niche games, low-view clips, youtube or other platform streamer clips

**One conclusion this data cannot support.** The average clip gets 3000++ views

**Does more data fix it?** No, the issue is sampling bias. Our query explicitly was for top categories and highest views so that average or bottom-tier Twitch performance is not represented.

---

## 5. What you need to do next

1. **Missing values in game_id** - 5 channels are missing a `game_id` - We will filter these out or impute them based on the channel's most played game - Cost: loss of 5 channel records if dropped.
2. **Temporal analysis** - We haven't explored temporal trends in clip creation - We will plot clip frequencies by day of the week / hour - Consequence: unlocks insights on when viral clips actually occur.
3. **Analyze tags** - The `tags` column is currently a parsed list but not aggregated - We will explode the column to see tag frequency - Consequence: allows us to categorize streamers beyond just their active game.
   