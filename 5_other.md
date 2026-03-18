# Other guidance

This is the dumping ground for other important information that didn't have an obvious home somewhere else.

## Data use considerations

Be very careful about including sensitive data in a repository. Highly sensitive data, such as personally identifiable information, **should not be included under any circumstances**. Other non-public data should not be included in repositories that are destined to be made public. If in doubt, leave it out.

Any non-public data that gets merged into the main branch of the repository must be excised from the repository's history using e.g. `git-filter-branch`, BFG, or other similar methods. If non-public data is accidentally published, it should be considered compromised by default, even after it has been retroactively removed from the repository. This is because automated scrapers monitor GitHub pushes for any potentially sensitive data and archive it before it can be deleted.
