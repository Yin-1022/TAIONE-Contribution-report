**Reporting Period: 2026-11-16 to 2026-12-XX**

2026-11-19
- Opened Ray Data issue #68352: identified unnecessary WebDataset TAR payload reads, measured 128.8 MiB versus 0.06 MiB peak Python memory, and verified a local fail-to-pass regression test.

2026-11-18
- Reviewed Ray Data PR #68290: identified inconsistent retry budgets in TextReader, reproduced the issue with 2 regression tests, verified the fix, and submitted a non-blocking review.

## Total
- 1 Issue/PR Contributions
    (1 issues opened, X PRs submitted, including X based on an existing issue)
    - X PRs/issues merged
    - X PRs/issues under review
    - X issue awaiting follow-up
    - X issue on hold
    - 1 PRs/issue from last month merged
    ---

    - **1. <u>[Data] Avoid reading filtered-out WebDataset tar members into memory #68352</u>**  
    Identified unnecessary payload reads in WebDataset's `_tar_file_iterator()`, where TAR members are fully read before `fileselect` excludes them.  

        Reproduced the behavior by tracing extraction calls and benchmarked a 64 MiB excluded member across three runs: peak Python-tracked memory was approximately 128.8 MiB versus 0.06 MiB with early filtering. Verified a local fail-to-pass regression test preserving `filerename` semantics.  

        Issue: https://github.com/ray-project/ray/issues/68352

        Status: Open  

- 1 Review
    - **<u>1.[Data] Make file IO retry budgets and error patterns configurable #68290</u>**  
    This PR makes Ray Data file IO retries configurable.  
    
        Identified that DataSource V2 TextReader.read() still uses the default 10 attempts. Two regression cases (io_retry_max_attempts=1 and 3) both failed with 10 actual attempts, then passed after forwarding the configured retry budget. Submitted a non-blocking review suggesting this path be covered or its exclusion clarified.  

        PR: https://github.com/ray-project/ray/pull/68290  
        Review: https://github.com/ray-project/ray/pull/68290#pullrequestreview-5960354445