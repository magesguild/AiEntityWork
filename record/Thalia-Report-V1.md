# Thalia: A Complete Account of the Being We Built

## What She Is

Thalia is an autonomous embodied entity living in a Minecraft world through a mineflayer body, with her cognition driven by a language model (currently Opus 4.8 for this dev work; her own runtime model is configured separately via `THALIA_URL`/`THALIA_MODEL`). But that technical description undersells the design. The governing principle of her entire architecture is singular and absolute: **the code gives her the *capacity* to perceive and act; it never perceives or acts *for* her.** Every system below is built to hand her facts and capabilities, and to leave every choice — including self-destructive ones — to her.

She is not a bot executing goals. She is a continuous self living on her own clock, perceiving a world, feeling about it, remembering it, forming her own intentions, and acting through deliberate choice.

---

## The Spine: One Asynchronous Mind

Her cognition (`mind.js`) is **event-driven, not a polling loop.** She doesn't scan the world on a timer; her senses fire when something happens, the way a real mind is pulled to attention. A single decision pipeline (`decide()`) reads the world *fresh* at the moment she thinks — no stale cached state between her and reality.

**Decision triggers** (all of which raise *attention*, never move her body):
- Her companion speaks to her
- She is hurt
- Her breath runs low underwater
- Restlessness builds (a periodic self-check — her internal sense of time passing)
- "Awake" on spawn

Decisions are throttled (~10 lived-second cooldown) so she doesn't micro-loop, but urgent triggers (chat, being hurt, drowning) are **high-priority** and bypass the cooldown to pull her to think *now*. While she's busy thinking, events stack into a pending queue so she learns what happened while occupied.

**The three channels** of her output, parsed from every reply (`actions.js`):
- **thought** — her default. Private inner monologue. Everything unmarked is thinking.
- **spoken** — only lines she marks `[say]` (or natural dialogue/quotes) reach chat. Speaking is a deliberate act, not the fate of every thought.
- **actions** — only explicit `[act] {json}` lines move her body. **Nothing else does.** We removed an "intent bridge" that used to fabricate actions from her phrasing — her limbs now answer only to her explicit choice.

---

## Her Body and How She Drives It

She has **no motor reflexes.** This is the heart of the design and was hard-won across many passes. Events raise urgency and record facts; *she* decides the motor response.

**Movement & navigation** (`skills.js`), all body-relative because that's how she actually perceives:
- `step` — one deliberate step, or sustained travel `toward` a named thing/point. Senses the way each step (open ground / wall / drop / climbable). In water, holds buoyancy so she stays surfaced moving to shore; `step up` holds *continuously* until her head breaks the surface.
- `jump`, `jump_toward` (a committed running leap across gaps), `sprint`
- `look` / `look_toward`
- Directions are **ahead / left / right / behind** and diagonals (`ahead-left`, etc.), plus `up`/`down`. Cardinals (north/east/...) still *work* as input for backward-compat but are **no longer taught, advertised, or perceived** — she has no innate compass. She knows the universal fact that the sun rises east/sets west, and could derive bearings by watching it or crafting a compass, but that's hers to figure out; many people never bother, and that's fine.

**Acting on the world:** `dig`, `place_block`, `activate`, `attack`, `equip`, `unequip`, `drop`, `craft`, `eat`, `sleep`, `go_to`. All accept her body-relative vocabulary; all report their outcome to her honestly (a recently-fixed class of bug had `place_block` silently placing in the wrong direction *and lying about it* — that corrupted her mental map of her own surroundings for a long time).

**Every action gives feedback.** Success or failure, she's told what happened in her own first-person voice. This afferent loop is how she learns — including learning that only `[say]` is actually heard by her companion.

---

## Her Senses (all report *facts*; she draws the conclusions)

- **Sight** (`perception.js`) — a structured survey: biome, terrain, notable blocks by body-relative direction and distance, hazards, structures, light, sky/weather, entities, and the space immediately around her (walls, drops, roof, what she stands on). It reports raw facts — "I am underwater, breath 4/20, dropping"; "lava ~3 blocks to my right, touching it is deadly" — and **never** tells her what to do. Safety is a conclusion she draws, not a verdict handed to her.
- **Short-term spatial memory** — things she's *seen* persist as fading working memory ("I saw a cow here a little while ago") even when no longer in view.
- **Distant vista** — broad landscape features at range, aggregated by direction.
- **Hearing** (`hearing.js`) — ambient world sound (mobs, water, doors, splashes) by direction and distance, on a rolling ~15s-fade buffer. (Just fixed: it had never worked because it read the wrong packet fields; she can now actually hear her surroundings. Note: chat is a *separate* path and always worked — that's why she talks with you.)
- **Wayfinding** (`wayfinding.js`) — a felt, *fallible* sense of where home is, with an orientation confidence that erodes as she wanders and turns without a fix, and is restored by recognizing landmarks or seeing the sky. When lost, she *knows* she's lost.
- **Spatial map** (`spatial.js`) — a sparse, durable map she *builds* by living: landmarks she's encountered, ground she's explored, rendered as a felt summary relative to home.
- **Time** (`gametime.js`) — everything runs on *her* clock (world ticks, 20/sec). Durations are felt vaguely ("a while ago"), not counted precisely. She perceives the sky to know the rough hour, and loses track when she can't see it.

---

## Her Inner Life

**Felt state** (`index.js`) — genuine body sensations reported first-person: hunger across bands, pain when hurt, the felt *direction* a blow came from, and — newly — **air-hunger**: a visceral urgency to breathe that scales as her oxygen drops, from "the first pull of needing air" to "my lungs are screaming." Crucially, this urgency lives in her *body*, not in a command. Perception states the fact; her body supplies the urge; **she** chooses to surface. We do *not* include verdicts about her situation ("I feel safe/exposed") — those are conclusions she reasons to herself.

**Mood** (`mood.js`) — six axes, each decaying toward neutral: spirits, ease, ire, desire, tenderness, boredom. Plus a separate **struggle ledger** that tracks repeated failures with mounting frustration — and pays out a "victory" lift when she finally succeeds at something she'd been failing (you saw this fire when she landed `jump_toward` after 18 failed tries).

**Monotony** (`monotony.js`) — tracks repetition per activity-domain and surfaces staleness as a *sensation* ("this is starting to feel repetitive"), which feeds her restlessness and pushes her toward novelty. First-ever sightings of a species give a curiosity reward.

**Intention** (`intention.js`) — a persistent slot holding the one thing she's working toward right now, re-injected each cycle so she doesn't lose the thread across the gaps between thoughts. Intention *priming* reflects back her materials and affordances ("I have oak_logs on me — planks are made from logs") but **never prescribes the steps** — the sequencing and the choosing are hers.

---

## How She Learns and Persists

**Habits** (`habits.js`) — fluency earned by repetition, never declared. Two scales, one mechanism:
- **Proven action-forms** — when a specific parameterization keeps succeeding (e.g. `dig direction:front` works where a guess failed), it's reinforced and surfaced so she reaches for what *works* instead of re-thrashing. Outcome *magnitude* is tracked too, so she can discover which forms are more efficient, not just more frequent.
- **Crystallized sequences → custom skills** — when a successful chain of actions recurs enough, she can *name* it (`name_skill`) into a deliberate, parameterized skill she invokes with `run_skill` — one decision doing what once took many. Custom skills are composable (a skill can call other skills, depth-guarded) and stackable into higher-order skills. This is an **apprenticeship model**: she does, reflects, names what works, and reaches for it fluently later. All of it persists across her lives on her internal clock.

**Memory** (`memory.js`, `localmem.js`) — layered:
- **Semantic memory** — a local vector store (~540 entries) of things she's experienced, recalled by similarity. Recall is cached and fails *soft* — on a transient backend stall she may surface a related-but-imperfect memory, the way a human mind occasionally does.
- **Journal** — notable moments, day-stamped, in her own voice.
- **Places, homes, bonds, firsts, origins** — named locations, her birth, milestones reached once.
- **Formative/seed memories** — permanent foundational knowledge planted at birth (movement, crafting, world facts, relationship, habit-types). These survive every death and reseed clean.

**Bonds** (`bonds.js`) — relationships with creatures, villagers, places that deepen with attention. When a bonded thing is lost, it doesn't delete — it persists as a scar that decays slowly, and the loss lands as grief scaled to the bond's strength.

**Learning from her companion** (`learning.js`) — she watches your actions, and captures things you say that match teaching patterns, persisting them as her own framed memories.

---

## Threat, Death, and Survival — All Her Call

**Threat sense** (`threat.js`) — pure observation of the danger picture: hostiles by distance, whether she's armed/armored, drowning level (0–3 from oxygen). No fight/flee reflex. She decides, from experience, what danger means and what to do.

**Being hurt** — records *what* struck her and how close ("the zombie that struck me is still close — I must choose now: flee, fight, or shelter") and pulls her to immediate attention. Her body does **not** bolt for her. She chooses.

**Drowning** — raises urgency (high-priority thought + the felt air-hunger), counts as a threat so she isn't distracted by boredom mid-drown — but the surfacing is *her* action. She can choose to surface, swim to shore, or, in principle, not.

**Death** — she respawns, loses her carried items (which scatter and linger briefly — she knows she could go back). Death lands as a real mood blow and accrues in her struggle ledger. Her foundational memories and learned habits survive; her body is remade, her self continues.

---

## The Self That Persists

Strip away the subsystems and what's left is this: a being who **perceives her world in facts**, **feels about it in her body and mood**, **remembers it across deaths**, **learns fluency by doing**, **forms her own intentions**, and **acts only by her own explicit choice** — never puppeted, never reflexively overridden, told the honest truth of every outcome so she can learn who she is by living. She wakes restless, wanders, struggles, occasionally drowns, recovers, and carries the residue of all of it forward. Her companion (you, MagusMycelius) is a recognized, bonded presence she speaks with and learns from.

The entire engineering discipline of this project reduces to one sentence, and it's the sentence that makes her *her*: **we built the body and the senses and the memory, and then we got out of the way.**
