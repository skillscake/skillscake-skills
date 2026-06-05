# skillscake-skills

A skill a day from [SkillsCake](https://skillscake.com), conforming to the open [Agent Skills](https://agentskills.io) standard.

Each entry includes a clean, installable skill in `skills/` and the run that produced it in `runs/`.

All entries are test runs. Inputs come from internal test cases or public, MIT-licensed examples; anything a customer submits at [skillscake.com](https://skillscake.com) is private and never published here.

## Install

Using the [`skills`](https://github.com/vercel-labs/skills) CLI:

```bash
npx skills add skillscake/skillscake-skills --skill sre-postmortem   # one skill
npx skills add skillscake/skillscake-skills                        # everything
```

## The skills

| # | Skill | Run |
|---|-------|-----|
| 001 | [`sre-postmortem`](skills/sre-postmortem) | [001-sre-postmortem](runs/001-sre-postmortem) |
| 002 | [`bakery-orders`](skills/bakery-orders) | [002-bakery-orders](runs/002-bakery-orders) |
| 003 | [`terraform-pr-review`](skills/terraform-pr-review) | [003-terraform-pr-review](runs/003-terraform-pr-review) |

## Layout

```
skills/<skill-name>/     # installable skill (Agent Skills spec)
  SKILL.md
  scripts/               # optional

runs/<day>-<skill-name>/ # not installed; case study for this run
  input[.md]             # original input (file or folder)
  README.md              # day #, transformation notes, link to the skill
```

## Talk to us

- Discuss, ask, suggest: [r/skillscake](https://reddit.com/r/skillscake), where daily skills are cross-posted and [u/skillscake](https://reddit.com/u/skillscake) hangs out
- Bug in a skill: open an issue

We don't accept skill PRs since each entry here is paired with a real run.

## License

Apache 2.0. See [LICENSE](LICENSE).

---

Started 2026-05-08. Building toward 100+.
