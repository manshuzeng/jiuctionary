# LexBridge · Build & Optimization Notebook

A living log for building, tuning, and extending the bilingual dictionary tool.
Keep this open alongside the tool. Add a dated entry every time you change something.

---

## 1. What the tool is

A single-file HTML dictionary (`translator.html`) that:

- Translates **English ⇄ Chinese** (expandable to any language)
- Shows **IPA phonetics** + click-to-hear pronunciation
- Lists **common phrases / preposition collocations** with short bilingual examples
- Has a **View / Edit mode** toggle so you can correct any field
- Lets you **save entries** to a personal dictionary (localStorage), which then load instantly with no API cost
- **Exports** your saved dictionary as JSON

The "dictionary" behind it is Claude (`claude-sonnet-4-20250514`) via the Anthropic API — generative, not a fixed database. See §5 for optional real-dictionary integration.

---

## 2. Architecture at a glance

```
translator.html  (one file — HTML + CSS + JS, no build step)
├── API key         → localStorage['lexbridge_api_key']
├── Saved dict      → localStorage['lexbridge_saved_dict']  { "word": {entry}, ... }
├── doSearch()      → checks saved dict first, else calls Anthropic API
├── renderResult()  → draws EN→CN or CN→EN layout; fields become editable in Edit mode
├── collectEdits()  → reads edited fields back into the entry object
└── saveEntry()     → writes (edited) entry to saved dict
```

**Key design choice:** saved words are checked *before* the API. This makes your
curated words instant, free, and offline-capable. Edit + save = your version wins.

---

## 3. How to change things (quick reference)

| I want to... | Where to edit |
|---|---|
| Change the output format/fields | `systemPrompt` string inside `doSearch()` |
| Add a new language | `systemPrompt` (add instructions) + `isChinese()` detection logic |
| Change the model | `model:` field in the fetch body |
| Adjust response length | `max_tokens` in the fetch body |
| Restyle the look | `:root` CSS variables at the top |
| Change phonetic behavior | `speak()` function + `.phonetic` rendering |
| Add a real dictionary source | See §5 |

---

## 4. Optimization backlog (ideas, unordered)

- [ ] **Caching by normalized query** — already saving manually; could auto-cache every lookup for a session to avoid re-charging on repeat searches.
- [ ] **Pinyin for Chinese input** — add a `pinyin` field to the CN schema and render it like IPA.
- [ ] **Audio via real dictionary** — dictionaryapi.dev returns real human-voice MP3s; browser TTS is robotic. Could fetch audio URL and play it.
- [ ] **Word-of-the-day** from saved list — spaced-repetition review mode.
- [ ] **Tag/category system** — mark words as "work", "travel", "finance" for filtering.
- [ ] **Import JSON** — counterpart to export, so you can restore or merge dictionaries across devices.
- [ ] **Keyboard shortcuts** — `E` to toggle edit, `S` to save.
- [ ] **Multi-word / sentence mode** — detect when input is a phrase vs a single word and adjust prompt.
- [ ] **Register/domain hints** — let you request "finance sense" or "medical sense" of a word (relevant to equity research + healthcare vocab).
- [ ] **Confidence/verification badge** — when hybrid mode (§5) confirms IPA against a real dictionary, show a ✓.

---

## 5. Optional: hybrid with a real dictionary API

Claude generates natural phrases but can occasionally miss on rare-word IPA. A hybrid
flow fetches *verified* data first, then asks Claude to enrich it.

**Recommended free source:** `dictionaryapi.dev` — no key, real audio, IPA, CORS-friendly.

Sketch to drop into `doSearch()` for English input, before the Claude call:

```js
let verified = null;
try {
  const r = await fetch(`https://api.dictionaryapi.dev/api/v2/entries/en/${encodeURIComponent(query)}`);
  if (r.ok) {
    const arr = await r.json();
    const first = arr[0];
    verified = {
      ipa: first.phonetic || (first.phonetics.find(p => p.text)?.text) || '',
      audio: first.phonetics.find(p => p.audio)?.audio || ''
    };
  }
} catch (e) { /* fall back to Claude-only */ }
```

Then pass `verified.ipa` into the prompt ("Use this verified IPA: ...") and use
`verified.audio` for real pronunciation instead of browser TTS.

See §6 for the full API comparison.

---

## 6. Free dictionary API comparison (researched Jul 2026)

| API | Key? | IPA | Audio | Languages | Notes |
|---|---|---|---|---|---|
| **dictionaryapi.dev** | No | Yes | Yes (MP3) | English (+some) | Community-run, CORS-friendly, no formal rate limit. **Best default.** |
| **freedictionaryapi.com** | No | Yes | — | Many (Wiktionary) | 1,000 req/hr/IP. CC BY-SA attribution required. Good for multi-language expansion. |
| **Merriam-Webster** | Yes | Yes | Yes | English | Free ≤1,000/day, non-commercial only. Authoritative. |
| **Cambridge Dictionary API** | Yes | Yes | Yes | EN + many bilingual (incl. EN–Chinese) | Developer Hub signup. Strong for EN⇄CN specifically. |
| **Wiktionary-based (ultimate-dictionary-api)** | No | Yes | — | Any→any | Translations from Wiktionary boxes; less structured. |

**Guidance:**
- Stay Claude-only for the richest phrases/examples.
- Add `dictionaryapi.dev` if you want verified IPA + real audio for English (free, no key).
- Consider **Cambridge** if EN⇄Chinese accuracy becomes critical (it has native bilingual data).

---

## 7. Deployment recap

- **Just use it:** open `translator.html` in a browser.
- **Share it:** drag the file onto [netlify.com](https://netlify.com) → get a public URL. Others bring their own API key.
- **Version it:** put it in a GitHub repo, enable Pages. Then this notebook lives next to it as the changelog.

---

## 8. Change log

> Add newest entries at the top. Template:
> `### YYYY-MM-DD — short title` then what changed + why.

### 2026-07-05 — Initial build + edit mode
- v1: search EN/CN, IPA, phrases, examples, Claude-backed.
- v2: added View/Edit toggle, editable fields, save-to-dictionary (localStorage),
  "My words" view, instant load for saved words, JSON export.
- Created this notebook and a reusable SKILL.md (see `lexbridge-lookup-SKILL.md`).
