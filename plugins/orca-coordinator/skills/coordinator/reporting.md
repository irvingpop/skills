# Reporting to the coordinator

Your spec gives you a report file path relative to the root of your worktree, under `.coordinator/`. Write your report and diff file there, never outside your worktree; the coordinator reads them from there. Pass the full path with `--report-path` on `worker_done`.

## Report file

Start with this section. The coordinator reads it first and reads further only when it needs to.

```
## Summary
Outcome: <succeeded, failed, or partial, in one line>
Evidence: <the commands and their results, files and lines, or sources that prove the outcome>
For the coordinator: <decisions, questions, and risks, or "none">
```

After the summary, use whatever structure suits the work.

## Diff file (implementers)

Write the diff of the files you own, including uncommitted changes and new files, to a file next to your report. The reviewer reviews that file.

## Review format (reviewers)

Your summary's outcome gives two verdicts: spec compliance (pass or fail) and quality (pass or fail). Quality also fails for needless additions and needless duplication. After the summary, list each finding with its severity:

- **Critical:** the change is wrong, unsafe, or breaks something.
- **Important:** the change does not meet the spec, or has a defect that must be fixed before it is done.
- **Minor:** an improvement that is not required.

For each finding, give the file and line, what is wrong, and why.
