# Human Writer - French

Make French AI text read as human-authored, and audit any French draft with a **deterministic 0-100 AI-detection score** before you publish. A Claude Code Agent Skill.

The French member of the `human-writer` per-language family (English, French, Spanish, Portuguese, German, Arabic, Hindi). Each member installs side by side and activates only on its own language triggers.

## Three modes

- **Write:** produce new French content already engineered to score low-risk.
- **Clean:** rewrite an existing French AI draft to strip detector tells, preserving meaning.
- **Audit:** score a French draft, surface the worst tells, and recommend fixes without rewriting it.

Four content types are supported (marketing long-form, short-form comms, technical docs, editorial-SEO), each with its own tolerances.

## Why it exists

Modern LLM output is fluent enough to ship but carries fingerprints that commercial detectors (Copyleaks, GPTZero, Originality.ai) latch onto: low burstiness, repeated vocabulary, formulaic frames, and typographic tells. A draft rejected at 70 percent or more does not need a rewrite from scratch; it needs a disciplined sweep of the specific tells the detectors weight. This skill encodes that doctrine for French and ships a scorer, targeting sub-25 percent AI-probability on Copyleaks and GPTZero.

## French-specific doctrine

- An accent-stripping hard-check (accents such as e-acute, e-grave, a-grave, and c-cedilla, including de-accented all-caps headings): missing accents are a strong French AI tell.
- The tiret cadratin (em-dash, U+2014) is a foreign tell: natural French prose rarely uses it.
- EN-to-FR calques are flagged (tirer parti de, sans couture, robuste, actionnable, incontournable) along with formulaic conclusion frames (en definitive, au final, au bout du compte).

## The analyzer

`scripts/analyze.py` is a deterministic 0-100 scorer that runs offline (it loads `rules.yaml`; optional live scoring via Copyleaks, GPTZero, or Originality.ai with `--external`).

```bash
python3 skills/human-writer-fr/scripts/analyze.py --input draft.md --lang fr --type marketing --format human
```

Score bands: 0-24 low-risk (ship it), 25-49 medium (apply the top fixes and re-score), 50-74 high, 75-100 critical.

## Installation

```bash
cp -r skills/human-writer-fr ~/.claude/skills/
pip install -r skills/human-writer-fr/requirements.txt   # pyyaml required, httpx optional
```

Or copy `skills/human-writer-fr` into a project's `.claude/skills/` directory.

## Use it

Once installed, the skill auto-activates on French prose requests. Example prompts:

- "Humanise ce texte pour qu'il ne fasse pas IA"
- "Nettoie ce brouillon IA et vise un score Copyleaks sous 25"
- "Audite le risque de detection IA de cet article et dis-moi quoi corriger"
- "Ecris un post LinkedIn en francais qui se lit comme humain"

Force activation: "Use the `human-writer-fr` skill to ...".

## What is inside

The skill lives in [`skills/human-writer-fr/`](skills/human-writer-fr/): a `SKILL.md` (routing, master checklist, anti-patterns), a `references/` library (stylistic, statistical, and structural tells, humanization techniques, and per-content-type adapters), and `scripts/` (`rules.yaml` plus the `analyze.py` 0-100 scorer and its tests).

## License

See `LICENSE`.

---

Part of the **[mr-bridge.com](https://mr-bridge.com)** toolkit for scraping, data, and content automation:
[Scrapers](https://mr-bridge.com/scrapers) · [MCP servers](https://mr-bridge.com/mcp-servers) · [AI workflows](https://mr-bridge.com/ai-workflows) · [Studies](https://mr-bridge.com/studies) · [Articles](https://mr-bridge.com/articles) · [Solutions](https://mr-bridge.com/solutions)

---

*Part of the [MrBridge Agent Skills catalog](https://github.com/MrBridgeHQ/skills). Browse them all at [mr-bridge.com/skills](https://mr-bridge.com/skills).*
