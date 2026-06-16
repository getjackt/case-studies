# Toronto Foosball Club: Real-Time Event Platform

> A full-stack platform that runs a foosball club's league nights, DYP events, and tournaments, syncing a venue projector, a coordinator portal, and players' phones over WebSockets in real time. Laravel 12 + Vue 3, running as a PWA.

**Status:** Live, used by the Toronto Foosball Club
**Role:** Solo builder
**Stack:** Laravel 12, Vue 3 (Composition API), Inertia.js v2, Tailwind 4, shadcn-vue, Laravel Reverb (WebSockets), Filament v4, MySQL, Redis, Pest
**Live:** events.torontofoosballclub.ca (member login)

## Why it exists

I play competitive foosball at the Toronto Foosball Club, which runs weekly league nights and tournaments for 20+ players a night. The scoring and standings lived in spreadsheets and someone's memory, which falls apart the moment a bracket gets complicated or a score needs fixing. I built this as a hobby project to give the club one system for players, stats, and events, with a live scoreboard everyone in the room could watch.

It has grown into a platform that runs three different event formats, each with its own rules, all kept in sync across the room in real time.

## What I built

The system handles three event types, each with different rules.

**League nights.** Multi-week seasons with fixed teams across three game modes (Singles, Doubles, Roto), each with configurable point values, target goals, and special rules like deficit multipliers. Roster penalties apply automatically when a team is short-handed, standings update live, and finals week auto-generates from the season standings.

**DYP and Monster DYP.** Draw-your-partner events with individual signup and random or balanced pairing each round. Monster DYP pairs the top-ranked player with the bottom-ranked one to keep games competitive. The system tracks per-player win rate, goals, and goal differential.

**Tournaments.** The most involved format: round-robin ladder matches plus single and double elimination brackets. Double elimination carries full winners and losers bracket routing, a grand final with bracket reset (King Seat versus Challenger), and best-of-N series (bo1, bo3, bo5) where the coordinator enters per-game goals and the backend derives the series score. Edit a result and every downstream match recalculates.

Everything runs on a config-driven scoring engine. Point values, game modes, and tiebreakers live in the database rather than in code, because the club changes its rules between seasons and I did not want a code change every time. Scoring is idempotent: updating any match re-computes all dependent standings, streaks, and bracket positions, so a corrected score never needs manual cleanup.

## Key technical challenges

**Keeping three views in sync in a room full of people.** A league night runs across three surfaces at once: a full-screen projector on the venue TV, a coordinator portal for running the event, and players entering scores from their phones. All three stay in sync over WebSockets (Laravel Reverb). When a coordinator assigns a table or a player submits a score, it broadcasts to the projector and every other client instantly. The projector itself is fully remote-controlled from the coordinator portal: the coordinator can pin a match full-screen, toggle sidebar sections, set a ticker announcement, or change the rotation speed without ever touching the projector machine.

**The double-elimination cascade.** Bracket integrity is the hardest correctness problem in the system. In double elimination, a single edited result can move a player between the winners and losers brackets, change who they face next, and ripple all the way to the grand final reset. I modeled this so that editing any match deterministically recomputes the entire downstream bracket rather than patching individual matches, which is the only way to keep the bracket honest when a score gets entered wrong under pressure.

**A celebration layer with real stakes detection.** Match completions trigger projector overlays tiered by context, not just a generic animation. A regular win gets a 3.5-second emerald flash, a 7-plus win streak gets a pulsing "on fire" treatment, and a bracket final won with an undefeated record gets a rainbow shimmer. High-stakes bracket matches open with a "Tale of the Tape" intro built from each player's record, with rivalry detection between frequent opponents. The layer reads game state to decide how big a moment is.

**Theming without a design tool.** Each tournament can carry its own theme. An admin picks a single brand color and the backend derives the rest of the palette from it using OKLCH, producing a readable primary, a secondary, and matching foreground text with lightness targeted at WCAG AA contrast. One input in, an accessible palette out, no manual color-picking per event.

## What I'd do differently

The real-time layer leans on live broadcasts more than on state re-hydration. If a client drops and reconnects mid-event, it catches up on the next broadcast rather than pulling a fresh snapshot of the current state, which leaves room for a reconnecting view to drift until the next update lands. Building this again, I would add a clean resync path so any view that reconnects rebuilds itself from a single source of truth before it starts listening for changes.

It is also single-tenant by design. There is no organization or club boundary in the schema: users, leagues, and events all share one namespace that implicitly belongs to my club. For a hobby project serving a single room that was the right call, and it kept the data model simple. To let other clubs run their own events on it, I would introduce a tenant boundary and scope every entity under it, which touches most of the schema. Knowing that now, I would at least design that boundary up front, even without enforcing it from day one.

## Screenshots / Demos

Live at events.torontofoosballclub.ca (member login). A walkthrough of the projector display, coordinator portal, and live scoring is available on request.

## Closing

This is the project I built for fun, for a room I am actually in every week. Under the hood it is a real-time system synchronizing three clients over WebSockets, a deterministic double-elimination bracket engine, a config-driven and idempotent scoring model, and an accessible theming system derived from a single color. It runs the club's nights today, which is the only test that really matters for it.
