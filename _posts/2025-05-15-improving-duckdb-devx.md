# Improving DuckDB DevX

[DuckDB](https://duckdb.org/), an in-process SQL database, has been gaining traction since its development began in 2018, with a significant surge in interest starting in 2022 [Google Trends](https://trends.google.com/trends/explore?date=today%205-y&geo=US&q=duckdb&hl=en). Despite 57,000 commits from 427 contributors, only 90 have made more than 15 commits, reflecting a tight-knit [core team of contributors](https://github.com/duckdb/duckdb/graphs/contributors).  

As DuckDB grows and adopts a more open stance toward community contributions compared to its row-oriented counterparts, now is the perfect time to enhance the developer experience and make contributing easier.


## Challenges for Contributors

* DuckDB has an extensive CI due to emphasis on correctness.
	* Takes 3+ hours and runs ~30 checks.
	* It's optimized for releasing new versions of duckdb and ensuring they're high quality releases.
* DuckDB authors made many choices around the time it came of age and haven't really kept up to date with all the tooling.
	* This is not necessarily a problem. Sometimes choosing what works is a good direction.
	* Examples: ubuntu 22.04 in CI, `clang-format` released in 2020.
* You'll see commits that say "forgot to run formatter"
	* `git log --oneline | grep format$ | wc -l` => 506
* Workarounds are needed to keep the old tools working
	* This involves overriding decisions made by old tools
	* Having to deal with the complexity of workarounds (such as writing temp files)


## A Balanced Solution: The `duckdb_devx` Branch

Rather than overhauling DuckDB’s established practices, which prioritize stability and correctness, I propose a complementary approach to streamline the contributor experience. My experimental [`duckdb_devx` branch](https://github.com/adsharma/duckdb/tree/duckdb_devx) introduces tools and workflows for faster iteration without sacrificing quality. Key features include:

- Faster CI: A [simplified CI pipeline](https://github.com/adsharma/duckdb/blob/duckdb_devx/.github/workflows/Fast.yml) delivers feedback in ~20 minutes, covering major platforms (Linux, macOS, Windows).
- Pre-Commit Hooks: Automated formatting checks prevent “forgot to run formatter” commits, keeping your commit history clean.
- Modernized Tooling: Upgraded to Ubuntu 24.04, macOS 14, and Windows Server 2019. (Feedback on preferred CI images is welcome!)
- Improved Formatting: Uses the latest stable clang-format with in-place formatting, eliminating issues with temporary files.
- Nightly Test Binary: Moves expensive unit tests to a separate nightly build, speeding up the development cycle.  

Try the duckdb_devx branch and share your feedback in the comments below or tag me in a [DuckDB discussion](https://github.com/duckdb/duckdb/discussions). Your input will shape this effort!

## Conclusion

The duckdb_devx branch isn’t meant to replace DuckDB’s rigorous CI pipeline, which ensures top-notch releases. Instead, it offers a faster, more contributor-friendly workflow for iterating on changes before submitting a pull request or while discussing ideas upstream. If you’re a DuckDB contributor, give it a spin and let me know how it works for you!
