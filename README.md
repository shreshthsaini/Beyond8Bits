<p align="center">
  <img src="static/images/logo-512.png" alt="Beyond8Bits logo" width="120"/>
</p>

# Beyond8Bits — Seeing Beyond8Bits

**Subjective and Objective Quality Assessment of HDR-UGC Videos**
CVPR 2026

Shreshth Saini¹, Bowen Chen¹, Neil Birkbeck², Yilin Wang², Balu Adsumilli², Alan C. Bovik¹,³
¹ Laboratory for Image and Video Engineering (LIVE), UT Austin &nbsp;·&nbsp; ² Google / YouTube &nbsp;·&nbsp; ³ University of Colorado Boulder

<p align="left">
  <a href="https://shreshthsaini.github.io/Beyond8Bits/"><img alt="Project Page" src="https://img.shields.io/badge/Project%20Page-Beyond8Bits-6b2bd9?style=flat-square"></a>
  <a href="https://arxiv.org/abs/2603.00938"><img alt="arXiv" src="https://img.shields.io/badge/arXiv-2603.00938-b31b1b?style=flat-square&logo=arxiv&logoColor=white"></a>
  <a href="static/pdfs/beyond8bits.pdf"><img alt="Paper PDF" src="https://img.shields.io/badge/Paper-PDF-4c1d95?style=flat-square"></a>
  <a href="static/pdfs/beyond8bits-supp.pdf"><img alt="Supplementary PDF" src="https://img.shields.io/badge/Supplementary-PDF-312e81?style=flat-square"></a>
  <a href="https://live.ece.utexas.edu/research/beyond8bits/index.html"><img alt="Dataset" src="https://img.shields.io/badge/Dataset-LIVE%20UT%20Austin-ff5a8a?style=flat-square"></a>
  <a href="https://utexas.box.com/s/pvz8zpmpogvpy62pqpar2e54c6ovyd5z"><img alt="UT-Box" src="https://img.shields.io/badge/UT--Box-Full%20Package-bf5700?style=flat-square"></a>
  <a href="https://github.com/shreshthsaini/Beyond8Bits"><img alt="GitHub" src="https://img.shields.io/badge/GitHub-Beyond8Bits-181717?style=flat-square&logo=github"></a>
</p>

---

## TL;DR

Beyond8Bits is — to our knowledge — the **largest crowdsourced HDR User-Generated Content (UGC) video quality dataset** to date, paired with **HDR-Q**, the first multimodal large language model for HDR-UGC VQA. HDR-Q is trained with **HAPO** (HDR-Aware Policy Optimization), an RL objective that extends GRPO with HDR-specific grounding, dual-entropy regularization, and entropy-weighted credit assignment.

| | Paper-reported (full) | Publish-ready release |
|---|---:|---:|
| **HDR source videos** | **6,861** (2,253 Crowd + 4,608 Vimeo) | **5,917** (2,153 Crowd + 3,764 Vimeo) |
| **Transcoded videos** | **~44,276** | **41,419** (12,918 Crowd + 22,584 Vimeo transcodes + 5,917 references) |
| **Crowd ratings** | **>1.5 M** on Amazon Mechanical Turk | ~1.46 M |
| **Avg. ratings per video** | ~35 | ~35 |
| **Resolutions** | 360p / 720p / 1080p (+ source) | same |
| **Bitrate ladder** | 0.2 – 5 Mbps | 0.2 / 0.5 / 1 / 2 / 3 Mbps |
| **Rating instrument** | Continuous 0–100 Likert, ITU-R BT.500-14 | same |
| **Split** | 70 / 20 / 10 (train / val / test), by source | 70 / 10 / 20 (28,987 / 4,151 / 8,281) |
| **MOS aggregation** | SUREAL MLE; median inter-subject SRCC 0.90 | same |

---

## Repository layout

```
Beyond8Bits/
├── index.html                         # Project page (hosted via GitHub Pages)
├── images_data.json                   # Carousel: frame paths + predicted / MOS scores
├── static/
│   ├── css/                           # bulma + project stylesheet
│   ├── js/                            # fontawesome
│   ├── images/                        # overview figure, distribution plots, AMT UI
│   └── pdfs/                          # paper & supplementary (drop-in once camera-ready)
├── images/
│   └── sample_frames/                 # 99 sample HDR frames used by the page carousel
└── data/
    ├── Beyond8Bits_publish.csv        # Full release: 41,419 transcoded clips × 12 columns
    ├── Beyond8Bits_publish.txt        # Matching list of 41,419 hashed video IDs
    ├── Beyond8Bits_publish_crowd.csv  # CHUG-compatible crowd-only subset (5,992 rows)
    └── Beyond8Bits_publish_crowd.txt  # Matching crowd-only ID list
```

Per-rating raw CSVs and the full archive live on UT-Box: https://utexas.box.com/s/pvz8zpmpogvpy62pqpar2e54c6ovyd5z

---

## Per-video metadata schema

`data/Beyond8Bits_publish.csv` (41,419 rows × 12 columns):

| Column | Description |
|---|---|
| `video_id` | Hashed video ID (primary key, used for S3 download) |
| `mos` | MOS after SUREAL bias-corrected aggregation |
| `sos` | Standard-of-scores (SUREAL dispersion) |
| `type` | Source partition — `Crowd` or `Vimeo` |
| `ref` | `1` if source / reference video; `0` if transcoded |
| `resolution` | Target resolution — `360p` / `720p` / `1080p` / `ref` (source) |
| `bitrate` | Target bitrate — `0.2M` / `0.5M` / `1M` / `2M` / `3M` (or `ref`) |
| `orientation` | `Portrait` or `Landscape` |
| `framerate` | Native playback framerate (fps) |
| `split` | `train` / `validation` / `test` (70 / 10 / 20 by source identity) |
| `height` | Native frame height (px) |
| `width` | Native frame width (px) |

Per-rating raw CSVs are available via the UT-Box mirror linked above — see the paper Appendix E for the rating protocol.

---

## Downloading the videos

Videos are hosted on an S3 mirror (`s3://ugchdrmturk/videos/`). You can use either the AWS CLI or stream directly in a browser.

```bash
# clone
git clone https://github.com/shreshthsaini/Beyond8Bits.git
cd Beyond8Bits

# single video
aws s3 cp s3://ugchdrmturk/videos/VIDEO_ID.mp4 ./Beyond8Bits_Videos/

# bulk — all 41,419 IDs in the published manifest
cat data/Beyond8Bits_publish.txt | while read video; do
  aws s3 cp s3://ugchdrmturk/videos/${video}.mp4 ./Beyond8Bits_Videos/
done
```

To stream a single video directly in the browser, replace `VIDEO_ID` in:

```
https://ugchdrmturk.s3.us-east-2.amazonaws.com/videos/VIDEO_ID.mp4
```

Example: https://ugchdrmturk.s3.us-east-2.amazonaws.com/videos/b882a56f87be722f24206e007c31ee4c.mp4

---

## Loading the scores in Python

```python
import pandas as pd

df = pd.read_csv("data/Beyond8Bits_publish.csv")
print(df.columns.tolist())
# ['video_id', 'mos', 'sos', 'type', 'ref', 'resolution',
#  'bitrate', 'orientation', 'framerate', 'split',
#  'height', 'width']

print(df["mos"].describe())          # MOS distribution
print(df["type"].value_counts())     # Crowd (15,071) vs Vimeo (26,348)
print(df["split"].value_counts())    # train / validation / test
print(df["resolution"].value_counts())  # ref / 360p / 720p / 1080p
```

---

## Method summary: HDR-Q + HAPO

Paired with the dataset, we release **HDR-Q**, the first MLLM designed for HDR-UGC VQA.

- **HDR-aware vision encoder** — a SigLIP-2 encoder adapted with HDR–SDR dual-domain contrastive supervision (captions generated by Qwen2.5-VL-72B). Frames stay at native 10-bit PQ / BT.2020; no linear down-scaling.
- **HAPO (HDR-Aware Policy Optimization)** — extends GRPO with three HDR-specific terms:
  - **HDR–SDR contrastive KL:** prevents modality neglect by contrasting rollouts with and without HDR tokens.
  - **Dual-entropy regularization:** per-pathway entropy penalties to avoid reward-hacking via entropy inflation.
  - **High-Entropy Weighting (HEW):** rescales group-normalized advantage with per-token entropy, focusing gradients on informative reasoning tokens.
- **Rewards:** Gaussian-weighted MOS regression reward `R_sc` + format reward `R_fmt` + group-level self-reward `R_self`.
- **Backbone + compute:** Ovis2.5 base with rank-4 LoRA adapters; 8 uniformly-sampled frames per clip; trained on 4× NVIDIA H200 GPUs; two-stage RL (modality alignment → full RFT).

---

## License

- **Metadata / CSVs / TXT manifests / website code:** CC BY 4.0 (as stated in the paper).
- **Video payloads:** retain their original licenses (CC-licensed Vimeo content; crowd-contributed videos released under a non-exclusive research redistribution agreement).
- **Permitted use:** non-commercial research only. For commercial use, please reach out.

See the project page for the full terms of use.

---

## Citation

```bibtex
@article{Saini_2026_Beyond8Bits,
    author  = {Saini, Shreshth and Chen, Bowen and Birkbeck, Neil and Wang, Yilin and Adsumilli, Balu and Bovik, Alan C.},
    title   = {Seeing Beyond8Bits: Subjective and Objective Quality Assessment of HDR-UGC Videos},
    journal = {arXiv preprint arXiv:2603.00938},
    year    = {2026}
}
```

_(CVPR 2026 proceedings BibTeX will replace this entry once the official version is published.)_

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
