# YouTube Channel Growth Analysis: ShortCircuit

What separates breakout uploads on ShortCircuit (2.5M subscribers), the unboxing channel of the Linus Media Group network, and what would a change be worth?

Video walkthrough: https://youtu.be/W1ufq1cNFHs

## Data

- 11,187 uploads from the four network channels (Linus Tech Tips, ShortCircuit, TechLinked, Techquickie), collected on 2026-07-26 through the YouTube Data API v3, with exact publish timestamps and unrounded view counts. Shorts are identified from each channel's `/shorts` tab.
- 3,765 long-form transcripts, 2021-01-01 to 2026-06-29.
- Raw data is not included. To rebuild it, put a YouTube Data API key in `.env` as `YOUTUBE_API_KEY` and run notebooks 01, 04 and 06.
- `DATA_NOTES.md` documents provenance (a first `yt-dlp` pass was discarded because it rounded view counts and approximated publish dates) and seven known limitations.

## Method and results

- **Baseline.** View counts are cumulative, so older uploads look stronger. Each upload is indexed to the median of the nine preceding uploads of the same format on the same channel, which keeps the comparison group adjacent in time.
- **Positive control.** From transcript features alone, a classifier names the source channel with 94.2% accuracy against a 38.6% majority baseline (1,202 test uploads), so the text pipeline carries real signal.
- **Breakout model.** TF-IDF and SVD text features with logistic regression; a breakout is the top 20% of the index. On ShortCircuit the test AUC is 0.745 (95% CI 0.682 to 0.805), and the model's top 76 picks held 32 breakouts against 15 expected by chance (2.1x). On Linus Tech Tips the same model reaches 0.553 (CI includes 0.5), so the finding is specific to ShortCircuit.
- **Pacing.** Among 841 ShortCircuit uploads at least one year old, the slowest-talking quartile has 1.71x the median views of the fastest (600,593 vs 350,826; Mann-Whitney p = 7.1e-18) at the same median runtime (12.6 minutes). This is observational; an A/B test is the next step.
- **Sizing.** At an assumed $25 CPM, 2024 long-form views (91.9M) correspond to about $2.30M in sponsor value. The combined uplift is +29.2% on a median basis; on total views the pace component falls from +26.6% to +16.0%, so +29% is the optimistic end of the range.

`10_deck_claims.ipynb` recomputes every number used in the video and flags the two that carry caveats.

## Notebooks

| Notebook | Content |
|---|---|
| `01_collect` | Channel catalogs through the YouTube Data API |
| `02_clean_eda` | Cleaning, format labels, rolling baseline index |
| `03_analysis` | Channel-level analysis |
| `04_content_fetch` | Transcript and metadata fetch |
| `05_content_analysis` | Topic clusters (TF-IDF, SVD, k-means) |
| `06_full_collection` | Full transcript collection and what full coverage changes |
| `07_content_model` | Positive control and breakout model |
| `08_link_audit` | Commercial links in video descriptions |
| `09_dark_theme` | Figures for the presentation |
| `10_deck_claims` | Every claim in the video, recomputed |

Author: Xincheng (George) Ding
