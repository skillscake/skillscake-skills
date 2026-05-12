We publish ~12 shared TF modules (`terraform-aws-eks-stack`, `terraform-aws-rds-postgres`, etc.) consumed by ~20 service repos pinned to tags. We use semantic release and conventional commits. Devs pick `feat:` when it should be `feat!:` and downstream breaks.

Make a skill that'll check the GitHub PR using the right in pipeline variables (part of CI pipeline), diffs changed `.tf` files vs the target branch, decides if it's breaking, posts a review with the recommended Conventional Commits prefix. What are all the changes that would be breaking vs normal?

Handle PRs that touch non-tf files (ignore those), examples-only changes (not breaking), and sub-modules under `modules/<name>/`.

Make sure the agent is reliable and repeatable. Keep the skill tight and quick to run despite working well. 