# Shaka Player Maintenance Process

These are project-specific processes in addition to the
[general release process documentation](./).


<details>
<summary>
<h2>Release Branches</h2>
</summary>

Feature release are made from the `main` branch.  Any time that happens, our
release automation will automatically create a release branch.  So, for
example, when `v4.1.0` was released from `main`, a branch named `v4.1.x` was
automatically created.

Future bug fixes would be cherry-picked from `main` to `v4.1.x` to create the
releases `v4.1.1`, `v4.1.2`, etc., and the branch would become more stable over
time.
</details>


<details>
<summary>
<h2>Cherry-Picking</h2>
</summary>

Features are never cherry-picked to a release branch.  This would go against
[semantic versioning](https://semver.org/), which says that a patch-level
release (third digit) can only contain backward-compatible bug fixes.
Features scoped to test infrastructure or the demo app are an exception, since
those are not part of the library itself.

### Interactive Cherry-Pick Tool

We use a [custom tool](https://github.com/shaka-project/shaka-github-tools/tree/main/interactive-cherry-pick)
to perform an interactive cherry-pick.  This is similar to the interactive
rebase of `git`.  When you run the tool, your default editor opens with a list
of commits you may want to cherry-pick.  Any commits left in the file when you
quit the editor are cherry-picked using `git`.

If a commit fails to cherry-pick, the maintainer is given an error message from
git and must either fix the conflict or instruct git to skip the commit.

Cherry-picks can be done 1 or more times between bug-fix releases.  The release
is finally made by merging the release PR for the branch.

For example, after installing the tool:

```sh
# Go to the Shaka Player repo.
cd /path/to/shaka-player

# Check out a release branch tracking the _upstream_ repo:
git checkout -b v3.3.x upstream/v3.3.x
# Or if you already have such a branch locally:
git checkout v3.3.x

# Inspect all commits since a particular main-branch tag.
# If the most recent v3.3.x release was v3.3.1, the tag
# v3.3.1-main is the same point in history for the main branch.
interactive-cherry-pick v3.3.1-main

# If you don't use the recommended "upstream" remote name,
# you can specify the name of your remote.  For example,
# if you named the shaka-project/shaka-player remote "google",
# you could run this to cherry-pick from that specific remote:
interactive-cherry-pick v3.3.1-main google
```

Pre-work:
 1. Install the interactive cherry-pick tool:
    https://github.com/shaka-project/shaka-github-tools/tree/main/interactive-cherry-pick#installation
 1. Create a local branch in Shaka Player tracking an upstream release branch

Cherry-picking:
 1. Cherry-pick the necessary changes **skipping all feature commits** (see also
    ["What to Pick"](#what-to-pick) below)

    > :information_source: **NOTE**: If you aren't sure about a commit, you can
    > select it now and change your mind later if there are merge conflicts
    > and/or test failures

 1. Fix any merge conflicts that come up, or choose to skip the current commit
    with `git cherry-pick --skip`
 1. Run the compiler with `python build/all.py`
 1. Run the tests with `python build/test.py`
 1. If any tests fail, make the necessary changes to fix the cherry-picked
    commit that caused the failure (either to amend the earlier commits, or as
    a new commit on top)
 1. When the build and all tests pass, push the cherry-picked commits upstream
    with `git push`


### What to Pick

 - Most bug fixes are cherry-picked to active release branches.
 - Some bug fixes won't apply to a branch because they deal with features not
   present in that branch.
 - Some bug fixes won't apply to a branch because the code in the branch is too
   different, in which case [a decision must be made](https://quotecatalog.com/quote/mitch-hedberg-i-write-jokes-f-jpXNBB1)
   about whether porting the fix to the branch is worth the effort.  The
   maintainer will have to consider both the severity of the issue and the
   difficulty of fixing conflicts or reimplementation.
   - If a conflict is trivial to resolve, or if there are no conflicts in a
     commit, the commit may be pushed directly to the release branch.
   - If reimplementation of a commit is required, please submit a PR to the
     branch instead of pushing directly.
 - Non-code changes (docs, tests, demo, etc) are sometimes cherry-picked to
   branches, depending on what the change is and whether it makes sense for
   that branch.
 - **Features are never cherry-picked** to release branches, as this would
   break semantic versioning.
</details>


<details>
<summary>
<h2>Release Policy</h2>
</summary>

Maintainers can make a feature release at almost any time.  There is no need to
wait to batch up any particular number of changes or any particular significant
features.


### Chromecast app IDs for feature releases

As long as we have our own Cast communication in Shaka Player (see
https://github.com/shaka-project/shaka-player/issues/4214 for plans to change
this), each feature release branch needs its own Cast receiver app registered.
The newly registered ID must be put in `demo/index.html` and
`docs/tutorials/ui.md` (search `data-shaka-player-cast-receiver-id` and
`castReceiverAppId`).

The registration must currently be done by Google.  It is acceptable for a
non-Google maintainer to make a feature release, then ask the team at Google to
follow up with an updated receiver ID in a `.1` release afterward.  The impact
will only be to the demo app and documentation.
</details>


<details>
<summary>
<h2>Branch Maintenance Policy</h2>
</summary>

We will cherry-pick bug fixes to LTS branches, plus the most recent 2 release
branches.

There are two kinds of branches we will dub LTS:
 - The last feature release (minor release number) before a breaking release
   (major release number) will receive fixes for 1 year
   - _For example, v3.3 (right before v4.0) will be fixed for 1 year past the
     release of v4.0.0 (until April 30, 2023)._
 - The release currently used by the Cast Application Framework will continue
   to receive fixes until CAF upgrades to a newer Shaka Player release by
   default
   - _As of June, 2022, this is currently v3.2._
   - _Cherry-picking for any CAF-specific branch is the responsibility of the
     team at Google, rather than non-Google maintainers._

In addition to LTS branches, the most recent two branches will always receive
fixes.  If the most recent release branch is v4.1, the most recent two branches
would be v4.1 and v4.0.  After v4.2 is released, the most recent two branches
would become v4.2 and v4.1.  Unless v4.0 was the current CAF branch, we would
stop fixing v4.0 when v4.2 is released.
</details>


<details>
<summary>
<h2>When to Make Breaking Changes</h2>
</summary>

We should make breaking changes infrequently if we can.  The goal should be one
breaking release per year at most.  If changes can be made in a
backward-compatible way (with a shim or behind a non-default configuration),
they should be.  Then they can be released in a normal feature release.

Deprecated features, shims, and temporary configurations can be removed when a
major (breaking) release is finally made.

For example, in v4.0, we were forced to make a change that disabled HLS support
for older smart TVs.  In that same release, we removed deprecated methods,
configurations that were added to maintain compatibility with older releases,
etc.  Many of them had been marked deprecated for 6-12 months already.

Breaking changes need to be listed in
[`docs/tutorials/upgrade.md`](https://github.com/shaka-project/shaka-player/blob/main/docs/tutorials/upgrade.md),
with details on what application developers will need to do to cope with each
change.
</details>


<details>
<summary>
<h2>Internal Release Processes</h2>
</summary>

After making public releases, the following internal steps are taken at Google
to make the release available in our internal systems.  Detailed documents on
these processes are internal-only (go/shaka-player).

 1. Update the Chromecast receiver app URLs for the Shaka Player Demo

    > :pencil: **TODO**: Create subdomains in appspot that point to the latest
    > release for a branch, to eliminate the need to update receiver app URLs.

 1. Update Google3 (internal source repo)

    > :pencil: **TODO**: Automate this?

 1. Update [Google Hosted Libraries](https://developers.google.com/speed/libraries)

    > :pencil: **TODO**: Automate this?
</details>


<details>
<summary>
<h2>History</h2>
</summary>

Before v3.0, we didn't _strictly_ follow semantic versioning.

We would sometimes deprecate functionality (strictly speaking, a breaking
change) without bumping the major version.  App developers would get one
feature release with backward compatibility and a warning log before we removed
or changed part of the API in a breaking way.

We would also sometimes cherry-pick minor, backward compatible features to
release branches to get those out more quickly.

Since v3.0, we have stopped these practices and followed semantic versioning
strictly.
</details>
