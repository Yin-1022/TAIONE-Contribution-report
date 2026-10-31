2026-10-31
- Attempted to find another issue but didn't get much meaningful result :(
- Reviewed Ray Data PR #67645 and identified a regression where S3 schema inference bypasses RetryingPyFileSystem without restoring Ray-level retry semantics. 

2026-10-30
- Updated Ray Data PR #67505 after reviewer feedback by rejecting duplicate join keys before execution and adding regression coverage for duplicate keys on both operands.
- Opened Ray Data issue #67604 for contradictory Dataset.repartition() behavior that warns shuffle=True will be ignored before rejecting the same argument combination.
- Reviewed Ray Data PR #67576 and verified Pandas metadata handling across Arrow-backed list, struct, map, and dictionary columns, control extension dtypes, map_batches(), and Parquet round-trips.

2026-10-29
- Reviewed Ray Data PR #67492 and verified Normalizer zero-row handling plus Vectorizer integer-count and missing-value behavior across multiple columns, append mode, and Pandas nullable values.
- Reviewed Ray Data PR #67465 and identified a remaining correctness issue where NaN stratification keys miss the precomputed largest-remainder allocation, causing train_test_split() to return fewer test rows than requested.
- Reviewed Ray Data PR #67524 and identified stale-cache correctness issues in OrdinalEncoder, OneHotEncoder, and LabelEncoder when encoder statistics are mutated in place without changing dictionary identity or size.


2026-10-28
- Opened PR #67505 for issue #67415 to fix asymmetric and multi-key join schema validation by checking each side against its configured keys and preserving positional key order.
- Help review a Ray Data Parquet PR (#67469) and verify that lazy row-group fragment construction preserves row-group and partition semantics while deferring footer access until scan time.

2026-10-27
- Updated PR #67374 based on maintainer review by consolidating repetitive output-backpressure regression tests into a cleaner parametrized test while preserving the same coverage.
- Help review a Ray Data LLM PR (#67460) and verify that disabled request limiting works correctly and that the original prompt is preserved when request preparation mutates the input before an error.
- Help review a Ray Core PR (#67461) and confirm that the previous log-based readiness check could fire before the entrypoint process started, while the new atomic PID-file signal correctly represents process readiness.
- Help review a Ray Serve PR (#67434) and identify a shutdown-timing edge case where the new 10ms polling floor can consume very small graceful-shutdown timeout budgets before completion is observed.


2026-10-26
- Opened issue #67415 identifying two schema-validation defects in Dataset.join(validate_schemas=True): asymmetric on / right_on keys can bypass validation because each side is checked against the wrong schema, while valid multi-key joins can be rejected because key types are collected in schema order rather than configured key order.
- Reviewed PR #67406 and independently verified its NULL-sharding logic with multi-key shard_keys, including rows where either or both shard keys are NULL, and suggested adding regression coverage for the newly introduced multi-key OR / AND predicate construction.
- Reviewed PR #67373 and validated the broader is_extension_array_dtype() behavior across nullable numeric, categorical, timezone-aware datetime, and period dtypes, as well as mixed all-null/non-null blocks, confirming that concrete Arrow types and cross-block schema consistency are preserved.

2026-10-25
- Opened issue #67387 identifying inconsistent handling of non-finite Dataset.mix() weights, where NaN and +inf are accepted while -inf is misclassified as a positivity error, and proposed consistent finite-and-positive validation across logical and physical mix operators.
- Reviewed PR #67382 and raised a non-blocking semantic question about whether HashShuffle GPU usage should remain reported after aggregator shutdown, noting the differing zero-after-shutdown behavior in the existing GPU Shuffle implementation.

2026-10-24
- Opened a PR (#67374) to fix issue #67312 by ensuring output backpressure is only counted when unread output exists, with regression tests covering terminal-only waits and pending output scenarios.
- Reworked PR #67164 after #67157 superseded its original constructor-time fix, adapting it to the new plan-time BindGeneratedIdColumn flow to distinguish physical Parquet-column conflicts from synthesized path / row_hash collisions, with regression tests preserving the intended behavior.

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
- 5 Issue/PR
    - **[Data] Reject duplicate synthesized column names #67164**  
    Prevents synthesized columns in Parquet DataSourceV2 from silently overwriting one another when they share the same name, such as a generated checkpoint ID colliding with the synthesized `path` or `row_hash` column. Adds early validation and regression tests for conflicting and non-conflicting synthesized column configurations.  
      - Update: PR #67157 later moved generated-ID binding from ParquetDatasourceV2 construction time to the plan-time BindGeneratedIdColumn rule, superseding the original implementation and incidentally preventing the silent overwrite. 
        
        I reworked #67164 for the new architecture so BindGeneratedIdColumn now explicitly distinguishes collisions with physical Parquet columns from collisions with synthesized columns such as path and row_hash. 
        This avoids misreporting synthesized-column conflicts as input-data schema conflicts, provides more accurate guidance to users, and adds regression coverage to preserve the intended collision semantics across future generated-ID or synthesized-column changes.  

      PR: https://github.com/ray-project/ray/pull/67164  
      Issue: https://github.com/ray-project/ray/issues/67002

    ---

    - **[Data] Avoid counting terminal waits as output backpressure #67374**
    Identified and reported a pre-existing metrics accounting issue (#67312) while reviewing PR #67180, where exhausted tasks waiting only for terminal confirmation were incorrectly counted as experiencing output backpressure. Submitted a PR to distinguish genuine unread-output backpressure from terminal-only waits, with regression tests covering empty tasks, unread output, and pending output after stream exhaustion.  
    PR: https://github.com/ray-project/ray/pull/67374  
    Issue: https://github.com/ray-project/ray/issues/67312

    ---

    - **[Data] Dataset.mix does not consistently reject non-finite weights #67387**
    Identified inconsistent validation of non-finite weights in Dataset.mix(). The existing positivity-only check allows NaN and positive infinity to pass, while negative infinity is rejected as a positivity error rather than as a non-finite value.  
        
        I reproduced the behavior through the public Dataset.mix() API and traced the same validation assumption through both the logical Mix operator and physical MixOperator. Proposed requiring weights to be both finite and positive before normalization or execution, so NaN and ±inf are rejected consistently.  

        Issue: https://github.com/ray-project/ray/issues/67387

    ---

    - **[Data] Dataset.join(validate_schemas=True) mishandles asymmetric and multi-key joins #67415**  
        Identified two correctness issues in join schema validation: asymmetric on / right_on keys are checked against the wrong schemas, while multi-key types are collected in schema order instead of configured key order, causing valid joins to be rejected.  

        I reproduced both behaviors on master and opened PR #67505 to validate each side against its own configured keys, preserve positional key order, and add regression tests covering asymmetric and reordered multi-key joins. 

        Issue: https://github.com/ray-project/ray/issues/67415
        PR: https://github.com/ray-project/ray/pull/67505

    ---

    - **[Data] repartition() warns that shuffle is ignored before rejecting the same argument #67604**
        Identified a public API contract inconsistency in Dataset.repartition(): when target_num_rows_per_block is combined with shuffle=True, Ray first warns that shuffle will be ignored and then immediately raises a ValueError saying the combination is invalid.

        I reproduced the behavior on current master, checked related issues and historical API intent, and proposed keeping the existing ValueError while removing the contradictory warning, with regression coverage to ensure the invalid combination produces one consistent response.

        Issue: https://github.com/ray-project/ray/issues/67604

- 16 Review
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

    ---

    - **<u>[Data] Finalize exhausted tasks without output-read budget #67180</u>**  
    This PR fixes a Ray Data execution issue where exhausted tasks could remain active when the output-read budget was zero, delaying task finalization and resource release.

        I independently tested the terminal-completion retry mechanism through the real executor and verified that terminal errors do not overtake deferred final outputs when using the threaded metadata fetcher. I confirmed the expected ACTIVE → DRAINED → FINISHED lifecycle and found no blocking correctness issues.

        During the review, I also discovered a separate, pre-existing output-backpressure accounting issue, which led to issue #67312 and my follow-up PR #67374.  
        PR: https://github.com/ray-project/ray/pull/67180  
        Review: https://github.com/ray-project/ray/pull/67180#issuecomment-6355775805

    ---

    - **<u>[Data] Account for HashShuffle aggregator GPU usage in logical resource accounting #67382</u>**  
    This PR updates HashShuffle resource accounting so aggregator GPUs configured through aggregator_ray_remote_args are reflected in operator and global logical GPU usage instead of being hardcoded to zero.  

        I raised a non-blocking semantic question around the new shutdown lifecycle behavior: after aggregator actors are released, the test still expects current_logical_usage() and running_logical_usage() to report the configured GPU footprint.

        So I compared this with the existing GPU Shuffle implementation, which explicitly drops logical GPU usage to zero once its actor pool is shut down, and asked whether both shuffle implementations should follow the same post-shutdown semantics.  

        I also noted that the proposed behavior is consistent with HashShuffle's existing CPU/memory accounting, so the review focused on clarifying which resource-usage semantic should be intentionally preserved rather than treating it as a correctness blocker.  

        PR: https://github.com/ray-project/ray/pull/67382  
        Review: https://github.com/ray-project/ray/pull/67382#issuecomment-6394644010
    
    
    ---

    - **<u>[Data] Fix SQL sharding with null keys #67406</u>**  
    This PR fixes silent row loss in ray.data.read_sql() when shard-key values are NULL, routing rows containing NULL shard keys to task 0 while preserving hash-based sharding for non-null rows.

        I independently tested the new multi-key path with NULL values in either shard key and with both keys NULL, confirming that every input row was preserved exactly once.

        I then pointed out that the new per-key IS NULL / IS NOT NULL predicate construction supports multiple shard keys, while the added regression test only covers a single key, and suggested adding direct multi-key coverage.

        PR: https://github.com/ray-project/ray/pull/67406  
        Review: https://github.com/ray-project/ray/pull/67406#issuecomment-6416646511
    
    ---

    - **<u>[Data] Preserve Pandas extension dtypes for all-null columns #67373</u>**  
    This PR changes PandasBlockAccessor.to_arrow() so explicitly typed Pandas extension columns retain their concrete Arrow type when every value is null, instead of being coerced to Arrow null.

        I reviewed whether broadening the condition to all Pandas ExtensionDtypes could introduce unintended behavior, and locally tested nullable numeric, categorical, timezone-aware datetime, and period dtypes.

        I also verified a mixed-block map_batches() case with one all-null Int64 block and one non-null block, confirming that the resulting Dataset kept a consistent int64 schema, and concluded that the broader extension-dtype handling is reasonable.

        PR: https://github.com/ray-project/ray/pull/67373  
        Review: https://github.com/ray-project/ray/pull/67373#issuecomment-6416938166

    ---

    - **<u>[Data][LLM] Fix max_pending_requests<=0 crash and empty prompt in vLLM error rows #67460</u>**  
    This PR fixes Ray Data LLM inference failures when request limiting is disabled and prevents the original prompt from being lost in error rows after request preparation mutates the input.  

        I verified the async nullcontext() path and locally exercised an error occurring after the prompt had already been removed from the input row, confirming that the original prompt is still preserved correctly.  

        PR: https://github.com/ray-project/ray/pull/67460  
        Review: https://github.com/ray-project/ray/pull/67460#issuecomment-6446564315

    ---

    - **<u>[Core] Deflake _run_hanging_command tests #67461</u>**  
    This PR replaces a log-based subprocess readiness check in Ray Jobs tests with an atomically published PID file to remove a startup race.

        I traced the entrypoint launch sequence and confirmed that the old "Waiting..." check could succeed before the shell actually started, while the new PID-file signal exposes a complete PID belonging to a live process and remains compatible with the affected callers.

        PR: https://github.com/ray-project/ray/pull/67461  
        Review: https://github.com/ray-project/ray/pull/67461#issuecomment-6447258630

    ---

    - **<u>[Serve] Don't busy-loop the replica drain when graceful_shutdown_wait_loop_s=0 #67434</u>**  
    This PR prevents Ray Serve replica shutdown from busy-looping at graceful_shutdown_wait_loop_s=0 by introducing a 10ms minimum polling interval and rate-limiting drain logs.

        I verified that immediate HAProxy checks and minimum draining-period semantics remain intact, and identified a non-blocking edge case where very small graceful_shutdown_timeout_s values can expire before the next 10ms polling check observes request completion.  

        PR: https://github.com/ray-project/ray/pull/67434  
        Review: https://github.com/ray-project/ray/pull/67434#issuecomment-6449026658

    ---

    - **<u>[Data] build row-group sub-fragments lazily so footer reads run in the scan threads #67469</u>**  
    This PR replaces eager fragment.subset() construction with make_fragment() for grouped Parquet row groups, allowing footer reads to be deferred until the parallel scan stage.

        I verified that non-contiguous row-group selection, partition expressions, and scan results remain equivalent, and confirmed that fragment construction no longer accesses the file until scanning begins.  

        PR: https://github.com/ray-project/ray/pull/67469  
        Review: https://github.com/ray-project/ray/pull/67469#issuecomment-6476275533

    ---

    - **<u>[Data] Avoid per-feature Series concat and float upcasting in Vectorizers and zero-norm NaNs in Normalizer #67492</u>**  
    This PR improves HashingVectorizer and CountVectorizer by avoiding intermediate Pandas concatenations and fixes Normalizer so all-zero rows no longer produce NaNs.

        I verified the behavior through the public transform_batch() path, covering all three normalization modes, append mode, multiple vectorized columns, integer count preservation, and missing values represented by None, np.nan, and pd.NA. The updated behavior matched the intended semantics.  

        PR: https://github.com/ray-project/ray/pull/67492  
        Review: https://github.com/ray-project/ray/pull/67492#issuecomment-6485554264

    ---

    - **<u>[Data] Fix train_test_split sizes lost to float truncation and stratified counts #67465</u>**  
    This PR fixes floating-point truncation and uses largest-remainder allocation to preserve the requested size of stratified train/test splits.

        I identified and reproduced a remaining correctness issue with NaN stratification keys. Because normalized NaN values still do not compare equal, the precomputed allocation can be missed and fall back to a floored per-group count, causing train_test_split() to return fewer test rows than requested.  

        PR: https://github.com/ray-project/ray/pull/67465  
        Review: https://github.com/ray-project/ray/pull/67465#issuecomment-6486452102

    ---

    - **<u>[Data] Cache PyArrow vocabulary arrays and inverse label mapping in encoders #67524</u>**  
    This PR caches encoder lookup structures to avoid rebuilding PyArrow vocabulary arrays and inverse label mappings on every batch.

        I identified and reproduced a stale-cache correctness issue in OrdinalEncoder, OneHotEncoder, and LabelEncoder. The cache only checks dictionary identity and length, so same-size in-place changes to stats_ are not detected, causing stale ordinal mappings, zeroed one-hot outputs for updated categories, and outdated inverse labels.  

        PR: https://github.com/ray-project/ray/pull/67524  
        Review: https://github.com/ray-project/ray/pull/67524#issuecomment-6486585464

    ---

    - **<u>[Data] Keep pandas metadata on Arrow blocks parsable by pandas #67576</u>**  
    This PR rewrites Pandas metadata dtype names that Pandas cannot parse, allowing Arrow-backed columns to convert back to Pandas without failing while preserving already valid metadata.

        I verified the behavior beyond the original list-column case using Arrow-backed struct, map, and dictionary columns, all of which round-tripped successfully with their values preserved. I also checked nullable integers, booleans, strings, categoricals, and timezone-aware datetimes as controls, and validated both the public map_batches(...).to_pandas() path and write_parquet() → pd.read_parquet() interoperability.

        PR: https://github.com/ray-project/ray/pull/67576  
        Review: https://github.com/ray-project/ray/pull/67576#issuecomment-6525885326

    ---

    - **<u>[Data] Skip the S3 HEAD request on each Parquet file open by passing listed file sizes #67645</u>**  
    This PR reduces unnecessary S3 requests in the DataSourceV2 Parquet path by reusing file sizes already obtained during directory listing. It passes known sizes to PyArrow when opening Parquet files and bypasses RetryingPyFileSystem for sized S3 opens so PyArrow can avoid issuing an extra HEAD request before reading the footer.

        I identified a retry regression in the schema-inference path. Unlike the footer-reader path, schema inference unwraps RetryingPyFileSystem for S3 without adding an explicit call_with_retry(), so errors configured through DataContext.retried_io_errors are no longer retried at the Ray layer.  
        
        I then reproduced the behavior with a fail-once filesystem: master retried the configured transient error and successfully inferred the schema on the second attempt, while the PR propagated the first failure immediately after the sized-open path was forced to unwrap the filesystem.  

        PR: https://github.com/ray-project/ray/pull/67645  
        Review: https://github.com/ray-project/ray/pull/67645#issuecomment-6544259452