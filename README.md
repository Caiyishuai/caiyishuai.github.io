# Yishuai Cai's Academic Homepage

> [https://caiyishuai.github.io](https://caiyishuai.github.io)

Personal academic homepage of **Yishuai Cai (蔡怡帅)**, Ph.D. student at NUDT, currently doing research at the PKU–PsiBot Joint Laboratory.

Research interests: Embodied AI · Vision-Language-Action Models · LLM Agents · Behavior Tree Planning · Safety Alignment.

## Project Structure

This site is a **single-file static page**. There is no Jekyll, no build step, and no JS framework — just one HTML file that GitHub Pages serves directly.

```
.
├── index.html                  # the entire site (HTML + inline CSS + inline JS)
├── images/                     # paper figures and avatar
├── google_scholar_crawler/     # optional: pulls citations from Google Scholar
├── update_citations.py         # local helper to refresh citation counts
├── LICENSE
└── README.md
```

## Local Preview

No toolchain required:

```bash
python3 -m http.server 4000
# then open http://127.0.0.1:4000
```

## Update Google Scholar Citations

```bash
pip3 install scholarly==1.5.1
python3 update_citations.py
```

The script writes JSON into `google-scholar-stats/`. The GitHub Action defined for this repo also refreshes citation data automatically on the `google-scholar-stats` branch.

## Acknowledgements

Originally adapted from [RayeRen/acad-homepage.github.io](https://github.com/RayeRen/acad-homepage.github.io) and later rewritten as a self-contained single HTML page.
