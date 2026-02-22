# Toronto Foosball Club - Real-time event platform

Full-stack app that runs weekly foosball league nights, DYP (draw-your-partner) events, and tournaments with bracket play. Laravel 12 + Vue 3, shipped as a PWA. Replaces spreadsheets and manual scorekeeping for a club with 20+ players per night.

## What it actually does

Three event types, each with different rules:

**League nights.** Multi-week seasons with fixed teams. Four game modes (Singles, Doubles, two Roto variants), each with configurable point values, target goals, and special rules like deficit multipliers. Roster penalties auto-apply when teams are short-handed. Standings update live. Finals week auto-generates from season standings.

**DYP / Monster DYP.** Individual signup, random or balanced partner pairing each round. Monster DYP pairs top-ranked with bottom-ranked to keep it competitive. Tracks per-player win rate, goals, differential.

**Tournaments.** This is the big one. Ladder matches for round-robin play, plus single and double elimination brackets. Double elimination has full winners/losers bracket routing, Grand Finals with bracket reset (King Seat vs Challenger), and score edit cascading. Change a result and all downstream matches reset. Best-of-N scoring (bo1, bo3, bo5) where the coordinator enters per-game goals and the backend derives series scores.

## The real-time system

Everything runs through WebSockets (Laravel Reverb). Three views stay in sync:

**Projector display.** Runs full-screen on the venue TV. Shows a bracket carousel that rotates between game modes, live standings with auto-scroll, a sidebar with active tables / upcoming matches / recent results. The projector is fully remote-controlled by the coordinator. They can pin a specific match full-screen, toggle sidebar sections, set a ticker announcement, or adjust rotation speed. All without touching the projector machine.

**Coordinator portal.** Match management, table assignments, bracket operations, player enrollment. Every action broadcasts to the projector and other coordinators instantly.

**Mobile score entry.** Players submit scores from their phones.

The projector also has a celebration system. Match completions trigger overlays with tiers based on context. A regular win gets a 3.5s emerald flash. A 7+ win streak gets a pulsing red "on fire" treatment. Bracket finals with an undefeated record get a rainbow shimmer. High-stakes bracket matches trigger a "Tale of the Tape" intro with player records, rivalry detection, and catchphrases.

## Scoring

All config-driven. Point values, game modes, and tiebreakers are stored in the database, not hardcoded. Scoring is idempotent: updating a match re-computes all dependent totals. The club changes rules between seasons, so hardcoding would mean code changes every time.

Tournament themes use OKLCH color derivation. The admin picks one hex color and the backend generates four accessible variants that meet WCAG contrast ratios.

## Decisions

- **WebSockets over polling.** Latency matters when everyone in the room just watched the goal happen and they're staring at the projector. Even a few seconds of delay feels broken.
- **Config-driven scoring.** Rules change between seasons. Point values, game modes, tiebreaker order, all in the database.
- **Separate projector from coordinator.** The projector is a public view (no auth), the coordinator portal is private. Different data needs, different update frequencies.
- **Idempotent scoring.** Wrong score entered? Fix it, and all standings, streaks, and downstream bracket matches recalculate. No manual cleanup.

## Tech

Laravel 12, Vue 3 (Composition API), Inertia.js v2, Tailwind 4, shadcn-vue, Laravel Reverb (WebSockets), Filament v4 (admin), MySQL, Redis, Pest

## Access

Live system, free member login required. Demo available on request.
