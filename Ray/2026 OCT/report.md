2026-10-24
- Opened a PR (#67374) to fix issue #67312 by ensuring output backpressure is only counted when unread output exists, with regression tests covering terminal-only waits and pending output scenarios.

2026-10-23
- Identified and opened an issue (#67312) while reviewing PR #67180, reporting that terminal-only waits were incorrectly counted as task output backpressure despite having no unread output.

2026-10-21
- Fix the comment of PR (#67164)

2026-10-20
- Help review Ray Data PR (#67154), locally verify a redundant double-copy path for NumPy batches, and raise the performance concern for follow-up discussion\
- Help review Ray Data PR (#67161) and raise a concern about whether forcing preserve_order=True in the test may hide the default ordering semantics of train_test_split(shuffle=False)

2026-10-19
- Open a PR (#67164) for issue (#67002) to reject duplicate synthesized column names and prevent generated IDs from silently overwriting synthesized path or row_hash columns

2026-10-14
- Help review a Ray Serve PR (#66910) and identify a recovery edge case where redeploying the same configuration after terminal deployment actor failure does not reset retries or recover the deployment

## Total
- 1 Issue/PR
    - **Reject duplicate synthesized column names #67164**  
    Prevents synthesized columns in Parquet DataSourceV2 from silently overwriting one another when they share the same name, such as a generated checkpoint ID colliding with the synthesized `path` or `row_hash` column. Adds early validation and regression tests for conflicting and non-conflicting synthesized column configurations.  
    PR: https://github.com/ray-project/ray/pull/67164  
    Issue: https://github.com/ray-project/ray/issues/67002

- 3 Review
    - **<u>[Serve] Surface terminal deployment actor failures outside UPDATING #66910</u>**  
    This PR improves Ray Serve deployment status handling by surfacing terminal deployment actor failures across more deployment states instead of leaving deployments indefinitely marked as retrying.  

        I identified a recovery edge case where, after recreation retries are exhausted, redeploying the exact same deployment configuration returns no change and does not reset the exhausted retry state, leaving the deployment unable to recover unless its configuration or version changes.  
        PR: https://github.com/ray-project/ray/pull/66910  
        Review: https://github.com/ray-project/ray/pull/66910#discussion_r4267141271

    ---

    - **<u>[Data] Make zero_copy_batch=False copy default batches #67154</u>**  
    This PR fixes writable batch behavior when `zero_copy_batch=False` and the default `batch_size=None` is used.  
        
        I traced and locally verified that the NumPy path performs two full-batch copies: one at the Arrow block level and another after NumPy conversion. 
        
        I confirmed that the second copy alone is sufficient for NumPy batches, raised the resulting performance concern, and helped establish that the broader optimization is better handled as a format-aware follow-up issue rather than blocking this PR.  
        PR: https://github.com/ray-project/ray/pull/67154  
        Review: https://github.com/ray-project/ray/pull/67154#discussion_r4320981519

    ---

    - **<u>[Data] Deflake test_train_test_split by preserving block order #67161</u>**  
    This PR stabilizes `train_test_split` tests by enabling deterministic block ordering with `preserve_order=True`.  

        I raised a concern that forcing ordered execution may cause the test to exercise behavior different from the default `preserve_order=False` setting, and questioned whether the flake instead exposes ambiguity in the ordering contract of `train_test_split(shuffle=False)`. 
        
        The discussion concluded that the test-only fix is reasonable for this PR, while the default ordering semantics may deserve a separate follow-up issue.  
        PR: https://github.com/ray-project/ray/pull/67161  
        Review: https://github.com/ray-project/ray/pull/67161#discussion_r4322072170