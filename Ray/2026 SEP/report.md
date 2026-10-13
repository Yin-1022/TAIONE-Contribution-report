2026-10-11
- Continue fixing the suggestion from PR(#66878)
- Help review a Ray Data PR (#66879) about .union()

2026-10-10
- Continue fixing the suggestion from PR(#66878)
- Help review a Ray Data PR (#66907) about write_deltalake()
- Help review and identity the mistake in one gemini bot's comment on Ray Core PR (#66818)

2026-10-09
- Open an issue(#66877) about an improvement of JSON error parsing
- Open an PR(#66878) for the issue
- Fix the suggestion from Code Review Bots and reviewers

2026-10-07 ~ 2026-10-08
- Found an issue about an improvement of JSON error parsing, which I might be able to contributed to
- Ask if the issue worths contributing to and get permitted.

2026-10-05 ~ 2026-10-06
- Look into file reading sysyem in Ray Data and attempt some comparsion

2026-09-20 ~ 
- Try to find any suitable good-first-issue about Ray Data, but there isn't much.
- Try to read more files in the RAY repo

2026-09-19
- Forked and cloned ray-project/ray
- Configured upstream remote
- Read CONTRIBUTING.rst
- Read AGENTS.md
- Started exploring Ray Data / Serve / RLlib

====
## Total
- 1 Issue/PR
    - **Improve JSON read error context #66878**
    Improves error messages for malformed JSON and JSONL inputs in ray.data.read_json() by including the offending input file path, which wasn't identified before, while preserving the original parsing diagnostics and adding regression tests for the adjustment.  
    PR: https://github.com/ray-project/ray/pull/66878  
    Issue: https://github.com/ray-project/ray/issues/66877

- 3 Review
    - **Honor per-task retry policy during actor restart #66818**  
    Identified an incorrect code review suggestion regarding absl::flat_hash_map::erase(). Verified that the method returns void, confirming that the existing iterator handling was correct and preventing a potential compilation error.  
    PR: https://github.com/ray-project/ray/pull/66818  
    Review: https://github.com/ray-project/ray/pull/66818#discussion_r4237514814

    - **Write Delta timestamps as microseconds in write_delta #66907**  
    Tested timestamp conversion overflow handling and suggested regression tests for out-of-range timestamps and timestamp-based map keys. Both suggestions were subsequently addressed by the PR author.  
    PR: https://github.com/ray-project/ray/pull/66907  
    Review: https://github.com/ray-project/ray/pull/66907#issuecomment-6095431852

    - **Push column selections through Union #66879**  
    Verified that projection pushdown correctly prunes unused columns in both Parquet readers. Suggested an additional regression test with sample code to ensure column pruning reaches the underlying data sources rather than only modifying the logical plan.  
    PR: https://github.com/ray-project/ray/pull/66879  
    Review: https://github.com/ray-project/ray/pull/66879#issuecomment-6105377231