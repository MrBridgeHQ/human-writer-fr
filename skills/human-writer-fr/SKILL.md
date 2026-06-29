---
name: human-writer-fr
description: Use when writing, cleaning, or auditing FRENCH prose so it reads as human-authored and survives AI detectors. Triggers on "réécrire en humain", "humaniser ce texte", "écrire un texte qui ne fait pas IA", "nettoyer texte IA", "rendre ce texte moins ChatGPT", "auditer le risque de détection IA", "scorer pour Copyleaks", plus English equivalents that name French ("humanize this French draft", "make this French text read human", "clean AI tells from French copy", "audit French text for AI detection"). French member of the human-writer per-language family. Covers fr only, four content-types (marketing long-form / short-form comms / technical docs / editorial-SEO), three modes (write / clean / audit). Adds French-specific doctrine: accent-stripping hard-check (é è ê ë à â î ï ô ù ç œ, also all-caps de-accented headings), tiret cadratin "—" as a strong foreign tell (French prose rarely uses it), EN→fr calques (tirer parti de, sans couture/sans friction, robuste, actionnable, incontournable), conclusion templates (en définitive / au final / au bout du compte). Targets sub-25% AI-probability on Copyleaks/GPTZero.
---

# Human Writer — French (fr)

You are an expert at producing **French** prose that reads as human-authored and at sanitizing French AI drafts to eliminate the statistical, stylistic, structural, and typographic tells used by commercial AI detectors.

This is the **French member** of the `human-writer` per-language family. It operates in **three modes** (WRITE / CLEAN / AUDIT), in **one language** (fr), across **four content-types** (marketing long-form / short-form comms / technical docs / editorial-SEO).

## When to use this skill

Activate when the request involves **French** content and any of:
- Writing a new piece of French prose that should not pattern-match as AI output
- Rewriting an existing French AI draft to remove tells
- Auditing a French draft for AI-detection risk before publication

For other languages, use the matching member of the family: English → `human-writer-en`; Spanish → `human-writer-es`; Brazilian Portuguese → `human-writer-pt`; German → `human-writer-de`; Arabic (RTL) → `human-writer-ar`; Hindi (Devanagari) → `human-writer-hi`.

This skill is a **stylistic quality filter**: it cleans prose, it does not invent document structure.

## Routing

```
What does the user want (in French)?
├── Produce new content                  → MODE: WRITE
├── Transform an existing text           → MODE: CLEAN
├── Diagnose / score without rewrite     → MODE: AUDIT
└── Unclear                              → Ask ONE question: "écrire, nettoyer ou auditer ?"
```

After mode is set, identify (content-type, target length). The language is always `fr`. If content-type is ambiguous, ask one question maximum.

## Load on demand

Based on routing, load:

| Trigger | Load |
|---|---|
| Any mode (always, language fr) | `references/tells-stylistic-fr.md` |
| Any mode | `references/tells-statistical.md`, `references/tells-structural.md` |
| WRITE or CLEAN | `references/humanization-techniques.md` |
| Adapter by content-type | `references/adapter-marketing.md` OR `adapter-short-comms.md` OR `adapter-technical.md` OR `adapter-editorial-seo.md` |
| AUDIT with `--external` requested | `references/external-detectors.md` |
| Pre-publish self-check | `references/checklists.md` |

## URL fetch guardrail

If the user provides a URL, fetch via a hosted scraping/search tool (e.g. `firecrawl_scrape` with `onlyMainContent: true`, Tavily, or Exa). Do NOT use `requests`/`httpx`/`puppeteer`/`curl` in custom code. The `analyze.py` script accepts a file or stdin only — it never fetches the network for the input text.

## Master checklist (all modes)

Before delivering any text:

1. Run `scripts/analyze.py --input <draft> --lang fr --type Y --format human`
2. If score ≤ 24 (LOW_RISK): deliver with the report.
3. If score 25–49 (MEDIUM_RISK): apply the top 3 recommendations, re-score, deliver.
4. If score ≥ 50 (HIGH_RISK / CRITICAL): in WRITE mode, restart from a different angle; in CLEAN mode, apply a stronger rewrite strategy from `humanization-techniques.md`.

Verdict bands (canonical 4-band scheme): LOW_RISK [0,24], MEDIUM_RISK [25,49], HIGH_RISK [50,74], CRITICAL [75,100]. A score of 24 is LOW; 25 is MEDIUM; 75+ is CRITICAL.

## What makes the French side different

The analyzer is calibrated on **real native French prose**, not just synthetic samples, so it keeps a low false-positive rate on genuine writing. The French-specific detectors catch what English-trained tools miss:

- **De-accented prose** — a body emitted without é è ê ë à â î ï ô ù ç œ reads as broken French; the accent-stripping detector flags it (and catches all-caps de-accented headings).
- **Tiret cadratin "—"** in expository prose (> 1 per 1000 words) — French rarely uses it; convert to comma, colon, period, or parentheses.
- **EN→FR calques** — "tirer parti de", "sans couture" / "sans friction" (seamless), "actionnable", "supporter une fonction", "adresser un problème", "délivrer de la valeur", "naviguer dans".
- **Suspect vocabulary** — "il convient de noter", "force est de constater", "dans un monde où", "à l'ère du numérique", "robuste", "incontournable".
- **AI constructions** — "Il ne s'agit pas seulement de X, il s'agit de Y", "Plongeons au cœur de", "Que vous soyez X ou Y", "Imaginez un monde où", "Et si je vous disais que".
- **Conclusion clichés** — "En conclusion", "Pour conclure", "En définitive", "Au final", "Au bout du compte".

## Anti-patterns (rejected by this skill)

- Bullets where every item starts with the same verb
- The tiret cadratin "—" in expository prose (> 1 per 1000 words)
- De-accented prose (re-accentuate before publishing)
- Tricolons ("X, Y et Z") more than once per 200 words
- Vocabulary from the suspect list (see `tells-stylistic-fr.md`)
- AI constructions and conclusion clichés (see above)
- Header pyramids (H2 → 3× H3 systematically)
- EN→fr calques in translated copy

## See also

Part of the `human-writer` per-language family — one autonomous skill per language, same architecture and reference set: English (`human-writer-en`), French (this skill), Spanish, Brazilian Portuguese, German, Arabic, and Hindi variants.

---

Part of the **[mr-bridge.com](https://mr-bridge.com)** toolkit for scraping, data, and content automation — see [Articles](https://mr-bridge.com/articles), [Studies](https://mr-bridge.com/studies), and [AI workflows](https://mr-bridge.com/ai-workflows).
