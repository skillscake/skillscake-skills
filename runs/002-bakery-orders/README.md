# Day 2: bakery-orders

**2026-05-09**

A bakery owner taking custom cake orders through Instagram DMs wanted Claude Cowork to draft replies in their voice, quote from a menu doc, and book pickups on Apple Calendar.

## What changed

SkillsCake built a reliable skill with explicit gating:

- IG DM → menu lookup → Apple Calendar end-to-end
- Voice cloned from last 50 sent DMs; lowercase with a bad/good example
- Bake added guardrails not in the ask: never auto-send, book only on confirmed deposit, never promise an exact replica of another baker's work

## Links

- [Original input](input.md)
- [Skill: bakery-orders](../../skills/bakery-orders/)
