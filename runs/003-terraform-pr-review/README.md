# Day 3: terraform-pr-review

**2026-05-11**

A platform team publishing Terraform modules wanted CI to label each PR as breaking or non-breaking, so semantic-release picks the right Conventional Commits prefix instead of relying on devs to remember `feat!` vs `feat`.

## What changed

The user got a strong skill and a Terraform breaking-change taxonomy:

- PR → diff → classify → review end-to-end via `gh`
- Sub-modules under `modules/<name>/` evaluated from their own interface, not root
- `moved` blocks credited as mitigations for resource renames
- Catches the unobvious ones: outputs flipped to `sensitive`, provider version constraints tightened, `force_new` attributes

## Links

- [Original input](input.md)
- [Skill: terraform-pr-review](../../skills/terraform-pr-review/)
