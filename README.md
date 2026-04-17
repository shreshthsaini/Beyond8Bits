# Beyond8Bits — Seeing Beyond 8&#8209;Bits

**Subjective and Objective Quality Assessment of HDR-UGC Videos** — CVPR 2026

Shreshth Saini¹, Bowen Chen¹, Neil Birkbeck², Yilin Wang², Balu Adsumilli², Alan C. Bovik¹
¹ The University of Texas at Austin &nbsp;·&nbsp; ² YouTube, Google Inc.

> Project page: **https://shreshthsaini.github.io/Beyond8Bits/**
> Predecessors this release subsumes/extends: [CHUG (ICIP 2025)](https://live.ece.utexas.edu/research/chug/index.html) · [BrightRate / BrightVQ (WACV 2026 Oral)](https://shreshthsaini.github.io/BrightVQ/)

---

## TL;DR

Beyond8Bits is — to our knowledge — the **largest crowdsourced HDR User-Generated Content (UGC) video quality dataset** to date, together with a **reasoning-aware MLLM baseline** trained with our HDR-Aware Policy Optimization (**HAPO**) RL objective.

| | |
|---|---|
| **HDR source videos** | **5,917** (2,153 Crowd + 3,764 Vimeo) |
| **Transcoded / distorted videos** | **41,419** (15,071 Crowd + 26,348 Vimeo) |
| **Subjective ratings collected** | **~1,457,736** on Amazon Mechanical Turk |
| **Average ratings per video** | ~35 |
| **Resolutions** | 360p → 1080p |
| **Rating instrument** | Continuous 0–100 single-stimulus |
| **Subsumes** | CHUG (ICIP 2025), BrightVQ (WACV 2026 Oral) |

---

## Repository layout

```
Beyond8Bits/
├── index.html                  # Project page (hosted via GitHub Pages)
├── images_data.json            # Carousel: frame paths + predicted / MOS scores
├── static/
│   ├── css/                    # bulma + project stylesheet
│   ├── js/                     # fontawesome
│   ├── images/                 # overview figure, distribution plots, AMT UI
│   └── pdfs/                   # paper & supplementary (drop-in once camera-ready)
├── images/
│   ├── chug-logo.png
│   └── sample_frames/          # 99 sample HDR frames used by the page carousel
└── data/
    ├── Beyond8Bits_publish_crowd.csv   # CHUG-compatible publish-ready subset (5,992 rows)
    └── Beyond8Bits_publish_crowd.txt   # Matching list of hashed video IDs
```

Placeholders you may want to drop in when the camera-ready lands:

- `static/pdfs/beyond8bits.pdf` — main paper
- `static/pdfs/beyond8bits-supp.pdf` — supplementary
- `data/Beyond8Bits_full.csv` / `.txt` — full-dataset metadata manifest (Crowd + Vimeo partitions)
- `data/Beyond8Bits_ratings_raw.csv` — per-rating raw CSV (worker-anonymized)
- `static/images/` — final Beyond8Bits SI/TI convex-hull, per-partition MOS histograms, scatter plots, etc.

All placeholder hooks are already wired up in `index.html`.

---

## What’s in the dataset

Beyond8Bits unifies two earlier sub-studies and adds a substantially larger Vimeo-sourced partition:

| Partition | Sources | Transcoded videos | Notes |
|---|---:|---:|---|
| **Crowd** (includes **CHUG**) | 2,153 | 15,071 | Smartphone-captured in-the-wild HDR-UGC |
| **Vimeo** | 3,764 | 26,348 | Permissively-licensed Vimeo HDR uploads |
| **Total** | **5,917** | **41,419** | |

Each source is transcoded across a common bit-ladder (resolution × bitrate) and scored with the same AMT instrument, so cross-partition analysis is apples-to-apples.

The per-video metadata CSV (`data/Beyond8Bits_publish_crowd.csv`) exposes the following columns:

| Column | Description |
|---|---|
| `Video` | Hashed video ID (primary key, used for S3 download) |
| `mos_j` | MOS after Sureal (bias-corrected) aggregation |
| `sos_j` | Standard-of-scores (dispersion) after Sureal |
| `ref` | Indicator for reference / source anchor videos |
| `name` | Original filename (`<resolution>_<bitrate>_#<content>.mp4`) |
| `bitladder` / `resolution` / `bitrate` | Transcoding parameters |
| `orientation` | Portrait / Landscape |
| `framerate`, `height`, `width` | Native playback attributes |
| `content_name` | Source-content anchor (used for content-aware splits) |

The full-dataset CSV additionally carries the original `type` (Crowd / Vimeo), `raw_scores` (list per video), `path`, and `split` columns.

---

## Downloading the videos

Videos are hosted on an S3 mirror (`s3://ugchdrmturk/videos/`). You can use either the AWS CLI or stream directly in a browser.

```bash
# clone
git clone https://github.com/shreshthsaini/Beyond8Bits.git
cd Beyond8Bits

# single video
aws s3 cp s3://ugchdrmturk/videos/VIDEO_ID.mp4 ./Beyond8Bits_Videos/

# bulk — all IDs in the published manifest
cat data/Beyond8Bits_publish_crowd.txt | while read video; do
  aws s3 cp s3://ugchdrmturk/videos/${video}.mp4 ./Beyond8Bits_Videos/
done
```

To stream a single video directly in the browser, replace `VIDEO_ID` in:

```
https://ugchdrmturk.s3.us-east-2.amazonaws.com/videos/VIDEO_ID.mp4
```

Example: https://ugchdrmturk.s3.us-east-2.amazonaws.com/videos/9ae245a27cc5ea9d2f3fae9692250281.mp4

---

## Loading the scores in Python

```python
import pandas as pd

df = pd.read_csv("data/Beyond8Bits_publish_crowd.csv")
print(df.columns.tolist())
# ['Video', 'mos_j', 'sos_j', 'ref', 'name', 'bitladder',
#  'resolution', 'bitrate', 'orientation', 'framerate',
#  'content_name', 'height', 'width']

# quick sanity check
print(df["mos_j"].describe())
print(df["resolution"].value_counts())
print(df["orientation"].value_counts())
```

---

## Method (summary)

We accompany the dataset with a reasoning-aware **Multimodal LLM baseline** trained with **HAPO — HDR-Aware Policy Optimization**, a reinforcement-learning objective that explicitly conditions the MLLM’s reasoning trace on HDR-specific perceptual cues (local luminance, gamut expansion, highlight / shadow fidelity, temporal stability). Training code is intentionally **not** released in this repo for now — this repository is the **dataset + project page** companion to the paper. Full method details are in the CVPR 2026 paper / supplementary.

---

## Running the project page locally

```bash
# from the repo root:
python -m http.server 8000
# then open http://localhost:8000/
```

The carousel loads `images_data.json` from the site root; the 99 sample frames live in `images/sample_frames/`.

---

## License

- **Metadata / CSVs / TXT manifests / website code:** CC BY-SA 4.0.
- **Video payloads:** retain their original licenses (CC-licensed Vimeo content; crowd-contributed videos released under a non-exclusive research redistribution agreement).
- **Permitted use:** non-commercial research only. For commercial use, please reach out.

See the project page for the full terms of use.

---

## Citation

```bibtex
@InProceedings{Saini_2026_CVPR_Beyond8Bits,
    author    = {Saini, Shreshth and Chen, Bowen and Birkbeck, Neil and Wang, Yilin and Adsumilli, Balu and Bovik, Alan C.},
    title     = {Seeing Beyond 8-Bits: Subjective and Objective Quality Assessment of HDR-UGC Videos},
    booktitle = {Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR)},
    month     = {June},
    year      = {2026}
}
```

Please also consider citing the predecessor sub-studies Beyond8Bits extends:

```bibtex
@INPROCEEDINGS{Saini_2025_ICIP_CHUG,
  author    = {Saini, Shreshth and Bovik, Alan C. and Birkbeck, Neil and Wang, Yilin and Adsumilli, Balu},
  booktitle = {2025 IEEE International Conference on Image Processing (ICIP)},
  title     = {CHUG: Crowdsourced User-Generated HDR Video Quality Dataset},
  year      = {2025},
  pages     = {2504-2509},
  doi       = {10.1109/ICIP55913.2025.11084488}
}

@InProceedings{Saini_2026_WACV_BrightRate,
    author    = {Saini, Shreshth and Chen, Bowen and Wang, Yilin and Birkbeck, Neil and Adsumilli, Balu and Bovik, Alan C.},
    title     = {BrightRate: Quality Assessment for User-Generated HDR Videos},
    booktitle = {Proceedings of the IEEE/CVF Winter Conference on Applications of Computer Vision (WACV)},
    month     = {March},
    year      = {2026},
    pages     = {1522-1532}
}
```

---

## Contact

Shreshth Saini — `saini.2 at utexas.edu` · [website](https://shreshthsaini.github.io) · [Google Scholar](https://scholar.google.co.in/citations?user=OpZ-5K4AAAAJ&hl=en)
