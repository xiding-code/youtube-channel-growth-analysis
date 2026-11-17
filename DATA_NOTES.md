# Data directory

Collected 2026-07-26. Sources are public and unauthenticated.

## Files

| File | Contents |
|---|---|
| `master_videos.csv` | **Main table.** Long-form videos and Shorts combined. Use this for analysis. |
| `channels.csv` | Channel-level metadata: subscribers, reported video count, uploads playlist ID |
| `api_ids_*.json` | Raw video ID + `publishedAt` pairs from `playlistItems.list` |
| `api_videos.json` | Raw video statistics from `videos.list` |
| `shortids_*.txt` | Video IDs YouTube classifies as Shorts, per channel |
| `lttstore_skus_*.csv` | LTT Store catalog exploded to variant level |
| `lttstore_raw.json` | Raw Shopify `products.json` response |
| `videos_clean.csv` | **Cleaned table** written by `02_clean_eda.ipynb` — adds `format` and `rbi` |
| `peers.json` | Metadata for the 12 comparison channels |
| `peer_*.json` | Comparison-channel videos, 2022-07 onward |
| `peer_comparison.csv` | Two-year reach ratios, LMG vs peers |

`master_videos.csv` columns: `id`, `title`, `channel`, `role`, `kind` (long/short),
`views`, `likes`, `comments`, `duration_sec`, `duration_min`, `date`, `published_at`.

## Provenance

Two collection passes were run. The current files come from the second.

**Pass 1 (superseded), `yt-dlp` flat-playlist mode.** Fast but two defects: publish dates were
approximate — for videos older than a year only the year survived, the month and day collapsing
to the scrape date — and view counts were rounded to three significant figures. The main channel
also hit rate limiting a few thousand items deep, capping it at 2,266 of 7,263 retrievable uploads (31% including Shorts; 2,000 of 6,997 long-form = 28.6%).

**Pass 2 (current), YouTube Data API v3.** Complete catalog with exact `publishedAt` timestamps
and unrounded view counts, at roughly 450 quota units against a 10,000/day free allowance.

Shorts classification still comes from `yt-dlp` reading the `/shorts` tab. The API returns
long-form videos and Shorts interleaved with no distinguishing field, and inferring from duration
is unreliable, so the `/shorts` ID set is used as YouTube's own ground truth.

## Limitations

**1. View counts are cumulative, not a time series.**

A video published in 2013 has been accumulating views for thirteen years; one published last week
has not. Comparing absolute view counts across periods is biased toward older videos.

Estimating the accrual curve from cross-sectional data was attempted and abandoned. Isotonic
regression with bootstrap confidence intervals, cross-checked by Mann-Whitney tests on 1,077
videos, produced a *flat* attainment profile (0.56–0.66 of the mature level from day 0 through
day 75) rather than the monotonic rise that accrual would produce. The reason is that age and
era are perfectly collinear — an older video is necessarily an earlier one — so a decline in
recent per-video performance cannot be separated from views that have not yet accrued.

The analysis therefore uses a **rolling baseline index**: each video divided by the median of the
nine videos preceding it on the same channel. Because the comparison group is adjacent in time,
era is held approximately constant by construction and no accrual estimate is needed.

**2. No sales data.**

`products.json` exposes price, variants and stock status but not units sold. Revenue cannot be
derived from it — only price architecture and assortment structure.

**3. Store catalog is a current snapshot.**

Delisted products are absent, so no product-launch time series is built. `published_at` also
resets when a product is delisted and relisted, making it a lower bound on first-sale date rather
than the date itself. It is used descriptively only.

**4. Floatplane scale is not observable.**

Pricing is public; subscriber counts are not.

**5. The rolling baseline index cannot carry inference.**

Diagnostics across k = 3…50 show residual time correlation is negligible everywhere (0.013–0.054)
and log-index dispersion converges by k≈15, so any window between 9 and 15 is defensible and the
exact value does not matter. But Ljung-Box rejects independence in 57–86% of channel/format series
at *every* k. This is structural: a video's own views enter the baseline of the next k videos, so
serial correlation is mechanically induced.

The index is a normalisation, not a residual with known sampling properties. It is used for
within-format ranking and outlier detection only. Interval estimates in this project are computed
on raw view counts over well-separated periods.

**6. Peer group is hand-selected.**

The 12 comparison channels were chosen by judgement before results were seen, not sampled at
random. This is a real selection risk and is disclosed rather than hidden.

**7. Coverage shortfall against reported counts.**

Private, deleted and members-only uploads are included in YouTube's `videoCount` statistic but
cannot be retrieved. The coverage check at the end of section 5 in `01_collect.ipynb` quantifies
the gap per channel.
