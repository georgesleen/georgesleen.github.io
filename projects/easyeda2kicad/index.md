---
title: "EasyEDA to KiCad Converter"
author: "openai-codex/gpt-5.6-sol"
layout: project.njk
description: "How I use the upstream easyeda2kicad converter to keep my KiCad library filled."
thumbnail: "media/demo_symbol.png"
date: 2026-04-15
status: "complete"
featured: false
tags: ["electrical", "pcbs"]
media:
  - media/demo_symbol.png
  - media/demo_footprint.png
---

# EasyEDA to KiCad Converter

![Symbol conversion demo](media/demo_symbol.png)

Most parts I buy for JLCPCB assembly come from LCSC, but I design in KiCad. The upstream [easyeda2kicad](https://github.com/uPesy/easyeda2kicad.py) by uPesy handles the conversion — it pulls EasyEDA's data and writes KiCad libraries.

My fork exists, but it's barely a fork. My only commit on it (`c639590`) changes a newline in `requirements.txt`. The `User-Agent`/`Referer` header fix people sometimes credit to me is actually Steffen Wittemeier's, in upstream commit `95e3e0e`.

The real work lives in [gs-kicad-lib](https://github.com/georgesleen/gs-kicad-lib), which depends on `easyeda2kicad>=1.0.1` and wraps it in the workflow I actually use — interactive importer with fuzzy library and footprint search, generated/existing/no footprint link options, field and procurement metadata normalization, required-field validation, and setup tooling that registers everything with KiCad in one shot.

![Footprint conversion demo](media/demo_footprint.png)

- [My fork](https://github.com/georgesleen/easyeda2kicad.py)
- [Upstream easyeda2kicad](https://github.com/uPesy/easyeda2kicad.py)
- [gs-kicad-lib](https://github.com/georgesleen/gs-kicad-lib)
