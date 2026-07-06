---
name: lexbridge-lookup
description: Bilingual English–Chinese dictionary lookup. Use whenever the user searches a single word or short phrase and wants translation, IPA pronunciation, common collocations (especially preposition combinations), and short bilingual example sentences. Handles both directions — English input returns Chinese meaning + phrases; Chinese input returns multiple English options with nuance. Expandable to other languages.
---

# LexBridge lookup skill

Encodes the canonical format for a bilingual dictionary entry. Use this both as the
system-prompt spec for the LexBridge tool and as a standalone lookup format.

## When to use
- User searches a **single word** or **short phrase** (not a full sentence).
- User wants pronunciation, meaning, and *how the word is actually used* — collocations and examples, not just a gloss.

## Detect direction
- Input contains any CJK character (`[\u4e00-\u9fff]`) → **Chinese → English** format.
- Otherwise → **English → Chinese** format.

## Output format — English input

Return this structure (as JSON for the tool, or as formatted text for chat):

- **Word** + part of speech
- **IPA** — standard phonetic notation, e.g. `/rɪˈzɪl.i.ənt/`. Must be accurate; prefer verified IPA if a real dictionary source is available.
- **Chinese translation** — primary sense first, then secondary if the word is polysemous.
- **English definition** — 1–2 concise sentences.
- **Phrases (4–6)** — prioritize:
  - preposition collocations (`depend **on**`, `consist **of**`)
  - common verb/noun/adjective combinations
  - each with: the phrase, its Chinese meaning, one **short** example (< 12 words), and the example's Chinese translation.

## Output format — Chinese input

Return **multiple English translation options** (this is the key value — showing differences):

For each option:
- **English word** + part of speech + IPA
- **Register tag**: `common` / `formal` / `casual`
- **Nuance note (in Chinese)** — one line on when to use this option vs the others.
- **Phrases (2–4 per option)** — same phrase/meaning/example/example-translation shape as above.

## Style rules
- Example sentences: **short and concise** (< 12 words). Everyday, natural usage.
- Phrases favor **real collocations**, not dictionary filler.
- IPA is **standard** notation, never ad-hoc respelling.
- Chinese uses appropriate characters for the user's variety (default Simplified 简体).
- Keep the whole entry scannable — this is a quick-reference tool, not an essay.

## JSON schemas (for tool integration)

English:
```json
{
  "type": "en",
  "word": "resilient",
  "pos": "adjective",
  "ipa": "/rɪˈzɪliənt/",
  "chinese_translations": ["有韧性的", "有弹性的"],
  "en_definition": "Able to recover quickly from difficulties; tough.",
  "phrases": [
    { "phrase": "resilient to", "meaning_cn": "对……有抵抗力", "example": "The system is resilient to failure.", "example_cn": "系统对故障有很强的抵抗力。" }
  ]
}
```

Chinese:
```json
{
  "type": "cn",
  "input": "坚持",
  "translations": [
    { "en": "persist", "pos": "v.", "ipa": "/pərˈsɪst/", "register": "common",
      "nuance_cn": "强调在困难中继续做某事。",
      "phrases": [ { "phrase": "persist in", "meaning_cn": "坚持做", "example": "She persisted in her efforts.", "example_cn": "她坚持不懈地努力。" } ] }
  ]
}
```

## Extending to other languages
Change only the direction-detection and the target-language instructions. The schema
is language-agnostic: swap `chinese_translations` for the target language's field, or
generalize to `target_translations`. Add a `pinyin` (or romanization) field alongside
`ipa` when the target script needs it.
