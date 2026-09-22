# Examples

## Basic usage

```
/qualitative-coding-cards https://www.figma.com/design/abc123/Landline-UT?node-id=45-678
```

Claude inspects the node, proposes the `Session | Activity | Participant` mapping and default lenses, and waits for a yes before coding the pilot participant.

## Choosing lenses

```
/qualitative-coding-cards <url> --lens invivo,values
```

## Alignment message Claude sends before coding

```
Structure found
  Page      UT Round 2, Session 3          → S3
  Activity  Act 2: Verify the caller       → A2
  Participant P07 (section, 3 children)
    transcript     1 text node, 2,140 words
    worksheet      1 image frame (will screenshot)
    observer notes 1 text node, 310 words

Lenses   Descriptive, Values, Emotion, In Vivo (default)
Colour   Mint (P07 has no existing cards)
Pilot    P07 only, then pause

OK to proceed?
```

## Sample card

```
"just hang up and call back"
[S3 | A2 | P07]
IN VIVO
```

Layer name: `[S3 | A2 | P07] "just hang up and call back"`

The title line is underlined and links to the card's own frame node (`hyperlink: { type: "NODE", value: <card id> }`). Metadata and lens label carry no link.

Shared plugin data (namespace `qualcoding`): lens, code, session, activity, participant, sourceType, locator, speaker, evidence.

## Sample review output after the pilot

```
Pilot complete: P07

Incidents        9
Cards written    14   (Descriptive 3, Values 4, Emotion 2, In Vivo 5)
Codes reused     0    (first participant)
Skipped          facilitator prompts, two acknowledgements, warm-up chat

Codebook so far
  Values       V: Simple, recognisable caller ID
               A: Won't answer unfamiliar agencies
               B: Government never calls first
  Descriptive  Agency prefix + number
  Emotion      Anxiety, relief
  In Vivo      "just hang up and call back", "how would I know it's real"

Ambiguities
  "I'd probably ignore it" could be A: Ignores unknown callers or Process: Ignoring. Coded as Values.

Memo
  P07 frames safety as ending the call, not verifying it. Relief follows the
  decision to hang up, not confirmation of legitimacy. Watch for this in P08.

Next: continue with P08 using the same codebook and lens set?
```
