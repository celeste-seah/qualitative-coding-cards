---
name: qualitative-coding-cards
description: "First Cycle qualitative coding of mixed research evidence (transcripts, worksheets, observer notes) laid out in a Figma file, written back as small colour-coded cards placed next to the evidence via the Figma MCP. Claude reads the participant section, segments it into incidents, applies Saldaña coding lenses (default Descriptive + Values + Emotion + In Vivo) and creates one movable card per code, tagged with retrievable metadata. Use when: '/qualitative-coding-cards', 'code this participant section', 'add coding cards to this Figma page', or the user shares a Figma URL to research evidence and wants First Cycle codes on the canvas."
---

# Qualitative Coding Cards

First Cycle coding where Claude does the analytic thinking and Figma is the canvas. Same outcome as the Figma-native `qualitative-coding-cards-figma` skill: one card per code, collocated with its evidence, ready for manual Second Cycle work. Method terminology follows Saldaña, *The Coding Manual for Qualitative Researchers* (4th ed., 2021). See [REFERENCE.md](REFERENCE.md) for lens definitions, decision rules, Figma scripts and pitfalls; [EXAMPLES.md](EXAMPLES.md) for invocation and sample output.

## Quick start

```
/qualitative-coding-cards <figma-url-to-participant-section> [--lens descriptive,values,emotion,invivo]
```

## Prerequisites

1. Load the `figma-use` skill before any `use_figma` call. Non-negotiable.
2. Batch-load Figma tools in one `ToolSearch`: `select:use_figma,get_metadata,get_screenshot`.
3. Design files only (`figma.com/design/...`). FigJam boards need a different node set; stop and say so.

## Workflow

1. **Inspect structure.** Read-only `use_figma` script on the URL node (REFERENCE.md 4a). `get_metadata` fails on sections this large, so treat it as optional. Identify page title, containing activity section, participant section, and its children by source type: transcript, worksheet or artefact, observer notes. Derive `Session | Activity | Participant` from those names (shortened to identifiers). Check for existing cards from earlier passes (REFERENCE.md, "Detect existing cards").
2. **Read the evidence.** One read-only `use_figma` script per participant returns every TEXT and STICKY node with its characters, bounds and parent. Observer notes are usually stickies, which a TEXT-only search misses. Responses over roughly 20k characters fail in transport, so return transcripts in slices of about 2,500 characters and strip non-ASCII if a slice still fails. Screenshot image-only worksheets with `get_screenshot`. Read transcript, artefact and notes together as one evidence set.
3. **Align before coding.** If the URL is an activity section holding several participants, pilot on the first and run the rest in parallel afterwards, one write call per participant. Confirm in one message: lenses (default Descriptive, Values, Emotion, In Vivo), metadata mapping, participant colour (reuse if the participant already has cards). Then code one participant as a pilot and pause for review.
4. **Segment into incidents.** One complete codable thought, not a speaker turn or fixed length. Skip facilitator prompts and acknowledgements unless the exchange itself constructs meaning. Trivial passages get no card.
5. **Code as a lumper.** For each incident pick only the lenses that add distinct analytic value. Prefer one strong code over four overlapping ones. Reuse wording from the running codebook when meaning matches. Flag ambiguity in the review note rather than over-interpreting.
6. **Write codes in Saldaña form.** One to five words, sentence case to match cards already on the board. Each lens has its own grammar (see REFERENCE.md, "Code form by lens"): Values codes carry a `V:` `A:` or `B:` prefix and name the value, attitude or belief itself; In Vivo codes are the participant's words in quotes; Descriptive codes are topic nouns; Process codes are gerunds. Do not write sentence-length gerund phrases for Values, that is Process coding wearing a Values label.
7. **Write cards.** `use_figma`, about three cards per call, using the card script in REFERENCE.md. Each card: code as primary text, `[Session | Activity | Participant]` line, small lens label. The code text is underlined and hyperlinked to the card's own frame node, so a copy made during Second Cycle work links back to the card sitting beside its evidence. Layer name `[Session | Activity | Participant] Code`. Store lens, code, session, activity, participant, source type, locator, speaker and evidence as shared plugin data. Fill colour is per participant, never per lens.
8. **Place with the evidence.** Transcript cards overlap the transcript's right edge near the utterance. Artefact cards sit on the artefact's edge. Note cards sit beside the note. Same-incident cards stack tightly. Cards never cover other cards or the essential evidence. No detached coding lane.
9. **Review the pilot.** `get_screenshot` the section. Remove redundant cards, fix overlaps, check metadata is separate from code text and colours are consistent. Report codebook, counts, skipped passages and ambiguities, then wait for the go-ahead before the next participant.

## Lens rules (Saldaña)

- **Descriptive**: a topic noun ("Agency prefix + number"). Strongest on worksheets, artefacts and observer notes. Weak on interview talk, so do not lean on it there.
- **In Vivo**: participant's exact words in double quotes. Verify against the text node's characters. Rarely a full sentence.
- **Values**: `V:` what matters ("V: Ability to call back"), `A:` a stance ("A: Won't answer unfamiliar agencies"), `B:` a proposition held true ("B: Number alone doesn't identify the agency"). Cue phrases: "it's important", "I need", "I think", "I want". Keep the prefix so Second Cycle can check whether values, attitudes and beliefs agree.
- **Emotion**: only when the emotion is stated or clearly shown, including in observer notes. Do not infer from tone of text alone. Note anger and frustration usually have a triggering emotion worth looking for.

## Boundaries

First Cycle only. No categories, themes, hierarchies or reconciliation across coders unless asked. Never edit or move the user's evidence frames. Cards are the only nodes this skill creates.
