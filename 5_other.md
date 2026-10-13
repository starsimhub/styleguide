# Other guidance

This is the dumping ground for other important information that didn't have an obvious home somewhere else.

## Data use considerations

Be very careful about including sensitive data in a repository. Highly sensitive data, such as personally identifiable information, **should not be included under any circumstances**. Other non-public data should not be included in repositories that are destined to be made public. If in doubt, leave it out.

Any non-public data that gets merged into the main branch of the repository must be excised from the repository's history using e.g. `git-filter-branch`, BFG, or other similar methods. If non-public data is accidentally published, it should be considered compromised by default, even after it has been retroactively removed from the repository. This is because automated scrapers monitor GitHub pushes for any potentially sensitive data and archive it before it can be deleted.

## AI usage policy

Obviously everyone uses AI to write code these days, and that's fine. If you used AI "lightly" (e.g. proofreading, writing tests, updating docs), you don't need to declare its use. If an agent decided what to write and wrote it with no input or review from you, then you do need to declare its use (but also, please don't do this, it creates more work rather than less). Between those extremes, use your judgement about what to declare, but err on the side of being specific about who did what. For example, in the pull request (or commit message) you could say something like "I used Claude (Opus 5.5) to review the issue and propose a solution; I proposed a different solution, which Claude implemented, and I reviewed the changes."
