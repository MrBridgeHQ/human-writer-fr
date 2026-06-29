# Human Writer — French (fr) — a Claude skill

A Claude Code skill that produces **French** prose reading as human-authored, sanitizes French AI drafts to remove detector tells, and scores any French draft for AI-detection risk before publication.

This is the **French member** of the `human-writer` per-language family — one autonomous skill per language (`en`, `fr`, `es`, `pt`, `de`, `ar`, `hi`), all built on the same architecture. They install side by side and do not conflict: each activates on its own language triggers.

## Why this skill exists

`mr-bridge.com` ships localized marketing and editorial copy in French. Modern Sonnet / Opus / GPT-class French output is fluent enough to ship, but it carries fingerprints that commercial detectors (Copyleaks, GPTZero, Originality.ai) latch onto — and French ChatGPT output is *louder* than its English counterpart, with its own tells the bare statistical detectors never checked:

- **Statistical:** low burstiness, narrow type-token ratio (language-agnostic).
- **Stylistic:** the same forty or fifty inflated phrases repeated across drafts ("il convient de noter", "force est de constater", "dans un monde où", "tirer parti de", "robuste", "incontournable"), formulaic frames ("Il ne s'agit pas seulement de X, il s'agit de Y", "Plongeons au cœur de").
- **Typographic (French-specific):** the tiret cadratin "—" used where native French uses commas, colons, or parentheses (French prose rarely uses it, so it reads as an even stronger tell than in English), and **de-accented prose** (a body emitted without é è ê ë à â î ï ô ù ç œ — a frequent AI/plain-text failure mode that reads as broken French on sight).

A draft drops, a client runs it through a detector, the score comes back at 70%+, and it's rejected. The fix is a disciplined sweep of the specific French tells detectors weight. This skill encodes that doctrine plus a deterministic analyzer so any Claude Code session can produce, clean, or audit French prose to a target score before delivery.

## What it can do

**Three modes:**
- **Write (écrire):** produce new French content already engineered to score LOW_RISK.
- **Clean (nettoyer):** rewrite an existing French AI draft to strip tells, preserving meaning.
- **Audit (auditer):** score a French draft, surface the top offending tells, recommend fixes without rewriting it.

**One language, fully specialized:**
- **French (fr):** ~150-entry suspect vocabulary (Tier 1 + Tier 2), French AI-construction regex bank, French tricolon detector (`et` conjunction), and a dedicated **accent-stripping** typography detector (catches fully de-accented bodies and all-caps de-accented headings) — calibrated NOT to false-fire on clean native prose.

**Four content-types** (each with its own adapter): marketing long-form, short-comms, technical, editorial-SEO.

**Optional external integration:** live scoring via Copyleaks, GPTZero, or Originality.ai (`--external <provider>`). Lazy `httpx` import, so the analyzer works fully offline.

## What's inside

```
human-writer-fr/
├── SKILL.md                          # Orchestration hub: routing + master checklist + anti-patterns (fr)
├── README.md                         # This file
├── INSTALL.md                        # Installation instructions
├── requirements.txt                  # pyyaml (required) + httpx (optional)
├── references/
│   ├── tells-stylistic-fr.md         # ⭐ CORE: FR suspect vocab + AI constructions + typography + accent doctrine
│   ├── tells-statistical.md          # burstiness / TTR (language-agnostic)
│   ├── tells-structural.md           # bullets / headers / tricolons / emoji
│   ├── humanization-techniques.md    # the ten humanization moves (French worked examples)
│   ├── adapter-marketing.md          # marketing long-form adapter
│   ├── adapter-short-comms.md        # short-form comms adapter
│   ├── adapter-technical.md          # technical-docs adapter (prose-only contract)
│   ├── adapter-editorial-seo.md      # editorial-SEO adapter
│   ├── external-detectors.md         # Copyleaks / GPTZero / Originality.ai integration notes
│   ├── checklists.md                 # pre-publish checklists
│   └── _self-audit.md                # why doctrine files flag themselves; ignore-marker mechanism
└── scripts/
    ├── rules.yaml                    # ⭐ fr: vocab + constructions + thresholds (incl. min_accent_ratio)
    └── analyze.py                    # ⭐ fr-adapted: tricolon (et) + detect_accent_stripping
```

## Installation

See `INSTALL.md`. TL;DR for macOS/Linux:

```bash
mkdir -p ~/.claude/skills
unzip human-writer-fr.zip -d ~/.claude/skills/
pip install --user -r ~/.claude/skills/human-writer-fr/requirements.txt
```

## How to invoke

Once installed, the skill auto-activates on French prose requests. Example prompts:

- "Écris un article de 600 mots sur le pricing des vins de garde, qui sonne humain, pas IA"
- "Nettoie ce brouillon IA pour faire passer son score Copyleaks sous 25"
- "Audite cet email en français pour détecter les tells d'IA avant de l'envoyer"
- "Humanise ce texte, il sonne trop ChatGPT"
- "Make this French landing-page copy read human, not AI"

If auto-activation misses, force it: "Use the `human-writer-fr` skill to..."

## The analyzer

`scripts/analyze.py` is a deterministic scorer that runs offline. It loads `scripts/rules.yaml` (French vocab lists, regex patterns, thresholds) and emits a 0-100 score plus a list of flagged tells.

```bash
# Audit mode (default, JSON output for tooling)
python3 scripts/analyze.py --input draft.md --lang fr --type marketing

# Human-readable report
python3 scripts/analyze.py --input draft.md --lang fr --type editorial-seo --format human

# Pipe via stdin
cat draft.md | python3 scripts/analyze.py --lang fr --type technical --format human
```

Required flags: `--lang fr`, `--type` (`marketing` / `short-comms` / `technical` / `editorial-seo`). `--input` is optional; stdin otherwise.

**Score bands** (4-band YAML, canonical):
- `0-24` **LOW_RISK:** ship it.
- `25-49` **MEDIUM_RISK:** apply the top 3 recommendations, re-score.
- `50-74` **HIGH_RISK:** in WRITE mode restart; in CLEAN mode apply a stronger rewrite.
- `75-100` **CRITICAL:** major rewrite.

**Detectors implemented:** em-dash density, sentence-length stdev (burstiness), TTR (lexical diversity), French suspect-vocabulary, French AI-construction regex bank, French tricolon (`et`), bullet parallelism, header pyramid, and the French-only **accent-stripping** detector (low-weight, ≥40-word guard, calibrated below the native-prose accent floor of ~0.033).

**Prose-only scoring.** The analyzer strips fenced code blocks, markdown data tables, and opt-in ignore regions before scoring. Wrap any intentionally AI-flavored citation so it doesn't count against you:

```
<!-- human-writer:ignore-start (citation de mauvais exemple) -->
Dans un monde où il convient de noter des solutions robustes et incontournables.
<!-- human-writer:ignore-end -->
```

## What's NOT inside

- **English content:** use `human-writer-en`. Other locales: `human-writer-es` / `-pt` / `-de` / `-ar` / `-hi`.
- **Document structure** (which sections, which schemas, which headings): use a dedicated structure/content tool. This skill is the stylistic filter applied on top of whatever produces the structure.
- **Technical SEO audit of a web project:** use a dedicated SEO tool. This skill handles content style only.

This skill is a stylistic filter invoked on top of structure-producing tools, never as a replacement.

## Part of the mr-bridge.com toolkit

This skill is part of the [mr-bridge.com](https://mr-bridge.com) toolkit for scraping, data, and content automation. Related resources:

- [mr-bridge.com](https://mr-bridge.com) — home
- [Scrapers](https://mr-bridge.com/scrapers) — the Apify Actor portfolio
- [MCP servers](https://mr-bridge.com/mcp-servers) — Model Context Protocol servers
- [AI workflows](https://mr-bridge.com/ai-workflows) — agents and automation
- [Studies](https://mr-bridge.com/studies) — data studies and one-pagers
- [Articles](https://mr-bridge.com/articles) — write-ups and guides
- [Solutions](https://mr-bridge.com/solutions) — end-to-end solutions

## License

Personal use. Customize freely. No warranty. The external-detector endpoints in `analyze.py` are copied verbatim from the master skill (language-agnostic POSTs) and carry `# (verify)` markers; they were not re-verified for French specifically.
