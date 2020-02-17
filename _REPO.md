# Github Repository Documentation

## Workflow

Features and bugs are recorded as issues in this repository. Issues are grouped into milestones with a clear due date. After finishing a milestone, I should start planning for the next milestone.

### Managing issues and milestones

Issues and milestones are managed by Github. For better visualization of the milestones, I duplicate them in GitKraken Timeline and keep them in sync. However, issues will not be duplicated as it doesn't provide much value. Whenever a milestone is finished, I tag the current master with a version number.

### Branching strategy

- Each issue will have its own branch. The branch naming convention is `<issue number>-<issue description (linked with dash)>`.
- Each commit should start with a gitmoji.
- Once an issue is finished, raise an PR with the basic template.
- Merge PR and delete the issue branch.

### CI pipeline

Potential CI pipeline to guard the merging. Once I have that in place, I will restrict committing to master branch directly.

### CD pipeline

CD pipeline is managed by Github as it has a webhook on master branch. When there's a new commit in master, it will deploy Github Pages.
