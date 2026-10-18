# Development workflow: strict TDD

## Instructions stored in the personal fork only

Save this file on `local/vibe-coding` and push that branch to the personal fork,
`Fitosoft/home-assistant-niu-component`. It must never reach the upstream
repository, `marcelwestrahome/home-assistant-niu-component`.

On development and release branches, keep `AGENTS.md` as a locally ignored,
untracked file. Never stage or commit it on a development or release branch. Use
the local `.git/info/exclude`, not the shared `.gitignore`, for this exclusion.
Do not merge or cherry-pick commits from `local/vibe-coding` into other branches.
When these instructions change, update the instruction branch in the personal
fork and the local copies in active worktrees. Local push protection is a safeguard, not a
replacement for checking the outgoing commits and files before a push.

For every behavioral change or bug fix:
1. Write or update a test before modifying production code.
2. Run the test and verify it fails for the expected reason (RED).
3. Implement the minimum code required to pass (GREEN).
4. Refactor while keeping all tests green.
5. Run the relevant test suite and report results.

Never weaken or remove tests solely to make a change pass.

Follow Ponytail principles: reuse existing code, prefer simplicity, and avoid
unnecessary abstractions.

If a test cannot be executed, explain why.

## Ponytail principles

Source: <https://github.com/DietrichGebert/ponytail>, especially its
[agent rules](https://github.com/DietrichGebert/ponytail/blob/main/AGENTS.md).

- Read the affected code, callers, tests, fixtures, and configuration before
  editing. Trace the behavior and fix the root cause in shared code.
- Reuse existing project code and conventions first, then the standard library
  or platform features, then existing dependencies.
- Make the smallest complete, readable change. Avoid speculative features,
  dependencies, wrappers, and abstractions.
- Preserve validation, error handling, security, and accessibility.
- Report what changed, what was tested, anything not checked, and known risks.

The strict TDD workflow above applies to every behavioral change or bug fix.
Documentation-only changes do not require a manufactured RED test.

## Local Python environment and tests

Work from the repository root. Match the Python 3.12 version used by
`.github/workflows/unit-tests.yaml`.

```bash
python3.12 -m venv venv
source venv/bin/activate
python --version
python -m unittest discover -s tests/unit -p "test_*.py" -v
```

Alternatively, run tests without activating the environment:

```bash
venv/bin/python -m unittest discover -s tests/unit -p "test_*.py" -v
```

For a focused RED/GREEN cycle, run the affected test, for example:

```bash
venv/bin/python -m unittest tests.unit.test_api -v
```

The current unit suite uses Python's standard-library `unittest` runner and the
existing Home Assistant/dependency stubs in `tests/unit/ha_stubs.py`. It needs no
third-party packages, NIU credentials, or running Home Assistant instance.
`venv/` is already excluded by `.gitignore`.

These unit tests do not establish compatibility with a real Home Assistant
installation or the live NIU API. `tox.ini` currently contains lint configuration,
not a tox test environment. Do not replace the existing test setup unnecessarily.

## Git remotes

- `origin`: <https://github.com/Fitosoft/home-assistant-niu-component.git>
- `upstream`: <https://github.com/marcelwestrahome/home-assistant-niu-component.git>

Base contributions on the personal fork and keep changes focused for upstream
review. Do not commit the virtual environment or credentials.
Before synchronizing with upstream, verify the fork's parent repository using
GitHub metadata; do not infer the parent from a repository name alone.

## GitHub communication

- Write GitHub comments, issues, PR titles and descriptions, review text, and
  release notes in clear B1-level English. Use short sentences and familiar
  words. Explain technical terms when needed.
- Always show the exact proposed text to the user for correction and review
  before posting it on GitHub.
- Post approved text in the user's name only after the user explicitly approves
  that text and its destination. Approval of one draft does not approve other
  drafts or later changes to the wording. A request to draft is not permission
  to post.

## Upstream collaboration and PR handling

- The user reports that the repository owner made them a co-owner of the
  upstream project.
- The established workflow is independent review: another developer has
  reviewed, approved, and merged the user's PRs. That developer has also grouped
  several PRs into a release for Home Assistant users. GitHub identifies `syssi`
  as the publisher of release `2.1.10`.
- Keep asking for independent review. If no reviewer is available, follow the
  current pre-release workflow below and clearly record the missing review.
  Do not call an assistant review an independent developer approval. Co-owner
  access alone is not permission to merge or publish a release; publication
  still needs the user's explicit approval.

## Current pre-release workflow

The planned first beta is `2.2.0b1`, combining PRs #80, #81, and #86 on the
separate `release/2.2.0b1` branch. Follow this order:

1. The assistant performs a final code review of the combined release branch.
   Check the full affected behavior, tests, and migrations. Report findings and
   resolve blocking problems using strict TDD.
2. Test the combined version in a real Home Assistant instance connected to the
   local NIU simulator from PR #80. Include setup, two vehicles, existing-entry
   migration, camera identity, reload, and restart.
3. Make a Home Assistant backup before changing the user's real installation.
   Confirm that the backup is available. Then test with the user's real Home
   Assistant installation and NIU vehicle, including sensors and the route
   camera. Do not change that installation before the backup step is complete.
4. Record the exact tested commit, Python and Home Assistant versions, test
   cases, results, and limits. List reviews that happened and reviews that are
   still open or did not happen. Never treat silence as approval or simulated
   data as a real-vehicle test.
5. After the user's release approval, publish a GitHub pre-release and invite
   all recently active project users to test it. Prepare the recipient list and
   exact invitation texts for the user's review before posting. Keep the stable
   release separate from beta changes.
6. Create and pin a GitHub discussion with the user's approved text. Include
   HACS beta setup, the exact version to install, backup and restore guidance,
   test steps, known limits, and how to report issues. Link it from the release
   and test invitations. Prepare the discussion with the release so users have
   working instructions when testing starts.

An independent review remains welcome, but an unavailable reviewer need not
block this pre-release path after the documented checks and user approval.
Successful automated tests alone do not complete the Home Assistant test steps.
The earlier comment drafts for #86 and #82 remain on hold; revise them around
the agreed pre-release and show them to the user again before posting.

## Proposed stable release to announce beta testing

The user proposed a stable release with no changes to integration behavior,
adding a README notice and a link to the beta testing discussion. This remains
a proposal, not permission to publish.

- Build it separately from the last stable release (`2.1.10`), not from the beta
  branch. A possible next stable version is `2.1.11`; confirm the version is
  still available before use.
- Limit changes to the README, release notes, and required version metadata in
  `custom_components/niu/manifest.json`. Verify that the integration code is
  otherwise unchanged from the stable release.
- Put the notice and working discussion link in the release notes as well as
  the README, since users may only open the update information.
- Prefer publishing it once the discussion and beta are available. Explain
  clearly that the stable update does not enable beta versions.
- Review the texts with the user and obtain release approval before publishing.
  Do not describe an update as risk-free: installation can still need a Home
  Assistant restart, even when integration behavior has not changed.
