# Reference: Lenses, Decision Rules, Figma Scripts

## 1. Coding lenses (Saldaña, 4th ed.)

Default set for this skill is an Eclectic combination of the four below (Saldaña, Ch. 9: two or more compatible First Cycle methods chosen on purpose, not at random). Offer alternatives only when the research question calls for them.

| Lens | Chapter | Code looks like | Best evidence source | Watch out |
|---|---|---|---|---|
| Descriptive | 6, Elemental | Topic noun, e.g. "Agency prefix + number". Names what was talked about or shown, not what it means | Worksheets, artefacts, observer notes, field-note style material | Saldaña advises against relying on it for small-sample interview talk; noun codes say little about what participants mean |
| In Vivo | 6, Elemental | Participant's own words, always in double quotes | Transcripts, worksheet free text | Overuse limits conceptual lift; keep to words that stand out (impact nouns, action verbs, metaphors, repeated phrases) |
| Values | 7, Affective | `V:` value as a noun phrase ("V: Ability to call back"), `A:` attitude as a stance ("A: Won't answer unfamiliar agencies"), `B:` belief as a proposition ("B: Number alone doesn't identify the agency") | Transcripts, worksheets, observed actions in notes | Values coding is values-laden. Code from the participant's perspective, not the researcher's judgement. Stated values and observed behaviour can differ |
| Emotion | 7, Affective | An emotion word or the participant's own emotional phrase | Transcripts plus observer notes on body language and voice | Emotions read from text alone are less reliable. Require explicit statement or a corroborating observer note. Anger and frustration are consequential; look for the trigger |

Other lenses the user may ask for: Process (gerund actions), Versus (X vs Y tensions), Concept (abstract ideas), Provisional (start list from prior research), Causation (why). Definitions are in the `dovetail-code` skill's REFERENCE.md.

## 2. Decision rules Claude applies while coding

**What gets coded** (Ch. 2). Participant activities, perceptions, and the tangible things they make and handle. Facilitator prompts are functional, not substantive, so they get no card unless the exchange is genuinely dialogic. Trivial asides are N/A. Observer notes are substantive and do get coded.

**Unit is the incident.** Break the text where the topic or subtopic shifts, like Saldaña's stanzas. One speaker turn can hold several incidents. Several turns can be one incident.

**Lump, do not split** (Ch. 2). Splitting gives nuance but floods the canvas and overwhelms categorisation. Aim for the essence of each incident. Sanity check: if a participant section produces more than roughly 25 cards, reread as a lumper before writing.

**Reuse codes.** Coding is for finding patterns. If a code is never reused across incidents or participants you are abbreviating, not coding. Keep a running codebook in the conversation: code, lens, one-line description, one example. Check it before inventing new wording.

**Code form by lens.** One to five words. Sentence case on the board. Each lens has a grammar; mixing them is the most common error.

| Lens | Grammar | Example |
|---|---|---|
| Values | `V:` + noun phrase for what matters. `A:` + stance for how they feel about something. `B:` + proposition for what they hold true | V: Simple, recognisable caller ID. A: Won't answer unfamiliar agencies. B: Number alone doesn't identify the agency |
| In Vivo | Participant's exact words in double quotes, the phrase that carries their meaning, not filler emphasis | "I can always call it back" |
| Descriptive | Topic noun. What was shown or discussed, not its meaning | Agency prefix + number |
| Emotion | Feeling word, or the participant's own phrase for it in quotes | Suspicion. "Kan cheong" |
| Process | Gerund only, for what people do | Calling back to verify |

Wrong: a gerund sentence labelled Values ("Relying on calling back to learn who called and why"). That is Process coding. Wrong: a topic noun labelled Values ("Caller ID preferences"). That is Descriptive.

**Simultaneous coding is fine when justified** (Ch. 5). Two lenses on one incident are correct when the passage carries both manifest and latent meaning. Four lenses on every incident signals an unfocused analysis. If the Descriptive, Values, Emotion and In Vivo candidates all say the same thing, keep the most analytically useful one.

**Memo as you go** (Ch. 3). After each participant, write a short analytic note in chat: emerging patterns, codes that recur, tensions, doubts. This is not put on the canvas.

**Ambiguity.** When a passage could reasonably carry two readings, pick one, and list the alternative in the review note. Do not create both cards.

## 3. Figma tool map

```
ToolSearch  select:use_figma,get_metadata,get_screenshot   # one call
get_metadata     # structure of the URL node: names, types, ids, positions
get_screenshot   # image-only worksheets, and visual review after writing
use_figma        # read text (read-only script), then write cards (write script)
```

Always load the `figma-use` skill first and pass `skillNames: "figma-use"`. Rules that bite here: `return` is the only output channel, load fonts before any text edit, colours are 0 to 1, page context resets each call, one `setCurrentPageAsync` per call, stop on error and read it before retrying.

## 4. Script scaffolds

### 4a. Read evidence (read-only)

```js
// SECTION_ID from get_metadata. Returns every text node under the participant section.
figma.skipInvisibleInstanceChildren = true
const section = await figma.getNodeByIdAsync("SECTION_ID")
const page = section.type === "PAGE" ? section : (() => { let n = section; while (n.type !== "PAGE") n = n.parent; return n })()
await figma.setCurrentPageAsync(page)
const texts = section.findAllWithCriteria({ types: ["TEXT"] })
const stickies = section.findAllWithCriteria({ types: ["STICKY", "SHAPE_WITH_TEXT"] })   // observer notes live here
return {
  section: { id: section.id, name: section.name, box: section.absoluteBoundingBox },
  page: page.name,
  texts: texts.map(t => ({
    id: t.id, name: t.name, parent: t.parent && t.parent.name,
    box: t.absoluteBoundingBox, characters: t.characters
  })),
  stickies: stickies.map(n => ({ id: n.id, box: n.absoluteBoundingBox, text: n.text.characters }))
}
```

Transport fails above roughly 20k characters of response. For a long transcript, return `characters.slice(a, b)` in chunks of about 2,500 and issue the chunk calls in parallel. If a chunk still fails, strip non-ASCII with `.replace(/[^\x20-\x7E\n]/g, "?")` for reading; anchors in the write script still match on the original text as long as the anchor phrase is plain ASCII.

Place transcript cards at `stickyLeft - 4 - W` rather than a fixed overlap: the transcript column and the observer sticky sit close together on these boards, and this is the furthest right a card can go without covering the sticky.

### 4b. Detect existing cards (read-only)

```js
const section = await figma.getNodeByIdAsync("SECTION_ID")
const cards = section.findAllWithCriteria({ sharedPluginData: { namespace: "qualcoding", keys: ["code"] } })
return cards.map(c => ({
  id: c.id, name: c.name, box: c.absoluteBoundingBox,
  participant: c.getSharedPluginData("qualcoding", "participant"),
  code: c.getSharedPluginData("qualcoding", "code"),
  lens: c.getSharedPluginData("qualcoding", "lens"),
  fill: c.fills[0] && c.fills[0].color
}))
```

Reuse the participant's existing fill colour. Feed the boxes into the placement step so new cards avoid old ones.

### 4c. Write cards

About three cards per call. Anchors come from 4a: for a transcript utterance, estimate its y as `box.y + (charIndex / characters.length) * box.height` (the Plugin API has no per-line geometry, so this is proportional). Pass card specs as literal JSON.

```js
await figma.loadFontAsync({ family: "Inter", style: "Semi Bold" })   // "Semi Bold" with a space
await figma.loadFontAsync({ family: "Inter", style: "Regular" })

const parent = await figma.getNodeByIdAsync("SECTION_ID")   // section or page the evidence lives in
const parentBox = parent.absoluteBoundingBox || { x: 0, y: 0 }
const occupied = OCCUPIED_BOXES        // [{x,y,width,height}] from 4b plus earlier calls
const specs = CARD_SPECS               // [{ code, lens, session, activity, participant, sourceType, locator, speaker, evidence, anchorX, anchorY, color:{r,g,b} }]
const W = 360, GAP = 6
const created = []

const overlaps = (a, b) => a.x < b.x + b.width && a.x + a.width > b.x && a.y < b.y + b.height && a.y + a.height > b.y

const text = (chars, size, style, opacity) => {
  const t = figma.createText()
  t.fontName = { family: "Inter", style }
  t.characters = chars
  t.fontSize = size
  t.fills = [{ type: "SOLID", color: { r: 0.1, g: 0.1, b: 0.1 }, opacity }]
  t.textAutoResize = "HEIGHT"
  return t
}

for (const s of specs) {
  const meta = `[${s.session} | ${s.activity} | ${s.participant}]`
  const card = figma.createAutoLayout("VERTICAL", {
    name: `${meta} ${s.code}`, itemSpacing: 6, cornerRadius: 4,
    paddingTop: 16, paddingBottom: 16, paddingLeft: 16, paddingRight: 16
  })
  card.fills = [{ type: "SOLID", color: s.color }]
  parent.appendChild(card)
  card.resize(W, card.height)          // resize resets both axes to FIXED...
  card.layoutSizingVertical = "HUG"    // ...so restore HUG on height after it

  const rows = [[s.code, 16, "Semi Bold", 1], [meta, 11, "Regular", 0.75], [s.lens.toUpperCase(), 10, "Semi Bold", 0.6]]
  const nodes = rows.map(([chars, size, style, op]) => {
    const t = text(chars, size, style, op)
    card.appendChild(t)
    t.layoutSizingHorizontal = "FILL"
    return t
  })

  // Title links to the card's own frame. A copy of the card made during Second Cycle
  // work then points back to the original sitting beside its evidence.
  const title = nodes[0]
  title.hyperlink = { type: "NODE", value: card.id }
  title.textDecoration = "UNDERLINE"

  // Place: overlap the evidence's right edge, then nudge down until clear of other cards.
  let abs = { x: s.anchorX - W * 0.4, y: s.anchorY, width: W, height: card.height }
  while (occupied.some(o => overlaps(abs, o))) abs.y += card.height + GAP
  card.x = abs.x - parentBox.x
  card.y = abs.y - parentBox.y
  occupied.push(abs)

  const data = { lens: s.lens, code: s.code, session: s.session, activity: s.activity, participant: s.participant,
    sourceType: s.sourceType, locator: s.locator, speaker: s.speaker, evidence: s.evidence }
  for (const k in data) card.setSharedPluginData("qualcoding", k, String(data[k] ?? ""))
  created.push(card.id)
}
return { createdNodeIds: created, occupied }
```

Notes: card sizing matches the reference card in the Caller ID Research file (360 wide, 16px title, 11px metadata, 10px uppercase lens). `anchorX` is the evidence node's right edge. If the parent is an auto-layout frame rather than a section or page, append to the nearest SECTION or the page instead, otherwise the card will reflow the user's layout. Worksheets and notes use the same script with their own anchors.

### 4d. Participant palette (0 to 1 RGB, light enough for dark text)

| # | Colour | r | g | b |
|---|---|---|---|---|
| 1 | Sand | 0.98 | 0.89 | 0.66 |
| 2 | Mint | 0.75 | 0.93 | 0.80 |
| 3 | Sky | 0.74 | 0.87 | 0.98 |
| 4 | Lilac | 0.87 | 0.80 | 0.97 |
| 5 | Rose | 0.98 | 0.78 | 0.82 |
| 6 | Peach | 0.99 | 0.83 | 0.70 |
| 7 | Sage | 0.82 | 0.89 | 0.74 |
| 8 | Slate | 0.82 | 0.86 | 0.90 |

Assign in participant order. Check existing cards first so a returning participant keeps their colour.

## 5. Common errors

| Error | Why | Avoid by |
|---|---|---|
| Card per speaker turn | Mechanical segmentation | Segment by incident; reread for topic shifts |
| Four cards saying one thing | Forcing every lens on every incident | Pick the most analytically useful lens; Simultaneous coding only when meanings differ |
| Descriptive codes dominating a transcript | Descriptive is the easy default | Save Descriptive for artefacts and notes; use In Vivo, Values, Emotion on talk |
| Values codes written as gerund sentences or bare topics | Mixing lens grammars | Use `V:` `A:` `B:` plus a one to five word noun, stance or proposition. Gerunds belong to Process, topic nouns to Descriptive |
| Emotion inferred from flat text | Reading tone into a transcript | Require stated emotion or observer-note corroboration |
| In Vivo code paraphrased | Working from memory | Copy from the returned `characters`; keep the quotes |
| Codebook forks per participant | Not checking earlier wording | Keep the codebook in chat and check before new wording |
| Cards stacked at (0,0) or over each other | Skipped placement step | Always compute anchors and collision-check against `occupied` |
| Cards reflow the user's frame | Appended into an auto-layout parent | Append to the SECTION or page |
| Evidence moved or edited | Script touched non-card nodes | Only ever create card nodes; never set properties on evidence |
| Font error on text | Skipped `loadFontAsync` | Load Inter Semi Bold and Regular at the top of every write script |
| Title link missing or pointing elsewhere | Hyperlink set before `card.id` exists, or set to the evidence node | Append the card first, then set `title.hyperlink = { type: "NODE", value: card.id }` and underline it |

## 6. Glossary

| Term | Plain English | Saldaña |
|---|---|---|
| Incident | One complete codable thought; the unit a card represents | Ch. 2 stanzas and units |
| Lens | The First Cycle method that produced the code | Ch. 4 |
| Lumper / splitter | Coding the essence of a passage vs coding every phrase | Ch. 2 |
| Eclectic coding | Purposeful mix of compatible First Cycle methods | Ch. 9 |
| Simultaneous coding | Two or more codes on one datum, when both meanings are real | Ch. 5 |
| Codebook | Running list of codes with descriptions and examples | Ch. 2 |
| Analytic memo | Reflective note on what the codes are starting to say | Ch. 3 |
| Second Cycle | Grouping codes into categories and themes; manual, out of scope | Ch. 12 to 14 |

## References

Saldaña, J. (2021). *The Coding Manual for Qualitative Researchers* (4th ed.). SAGE.
