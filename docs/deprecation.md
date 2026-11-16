# Deprecation Policy

Releases here are dated, not numbered by impact — see the [versioning policy](versioning.md). A dated version cannot tell a user that something they depend on is about to disappear, so this document does that job instead.

> **Reviewers:** the 90-day figure below is a proposal introduced with this policy, not a decision the team made previously. Please push on it if you think it should be longer or shorter.

## What this policy protects

**Names.** Skill names, command names, agent names, and plugin names. These are what a user types or installs, so a rename is indistinguishable from a removal from where they sit.

**Skill intent.** The job a skill does and the prompts it fires on. A skill can keep its name, keep its description, and still stop doing what someone relied on it for — and nothing in the diff looks alarming.

## The intent test

Before merging a change to a skill, answer one question:

> Does a user's existing prompt still reach the same behavior?

**Yes — intent preserved.** Ship it normally.

- Typo and grammar fixes
- Tightened or clarified wording
- Added references or examples
- Corrected links

**No — intent broken.** Follow the deprecation path below.

- Narrowing the trigger language so the skill stops firing on prompts it used to catch
- Changing what the skill produces
- Dropping a capability it used to have
- Repurposing it toward a different task

## The deprecation path

Two routes. **Route 2 is the default.**

### Route 2 — ship it as a new skill (preferred)

Add the changed behavior under a new name and deprecate the old skill on the clock below. During the overlap a user gets both behaviors, and nothing breaks underneath them. Prefer this whenever intent has genuinely changed.

### Route 1 — deprecate in place (fallback)

Keep the name and change the behavior, having announced it first and served the full notice period. Use this only where a parallel skill would confuse more than it helps — two near-identical skills competing for the same prompts, for instance.

## The clock

> A deprecated thing is removed no earlier than **both** the next release **and** 90 days after the deprecation was announced. Whichever comes later governs.

Both conditions, not either. Releases here have no fixed cadence, so each clause covers a different failure:

- The **release clause** means a fast run of releases cannot retire something in a week just because two versions shipped.
- The **90-day clause** means a slow stretch still gives users real calendar time to notice.

**A quiet period does not unlock removal.** If 90 days pass and no release has shipped, the thing stays. Elapsed time alone is never sufficient — a release must also have happened. The clock starts at the announcement, not at the pull request that prompted it.

## What a deprecation requires

Every deprecation, on either route, ships with all three:

1. **A note in the affected `SKILL.md`**, saying what is deprecated, what to use instead, and the earliest release it may be removed in.
2. **A `### Deprecated` entry in `CHANGELOG.md`**, under that plugin's section.
3. **A line in the release announcement**, so it reaches users who read release notes and never read the repository.

### Removing a whole plugin

A plugin's entry in `.claude-plugin/marketplace.json` **stays until the clock expires**, even after the plugin is announced as deprecated. Removing the catalog entry early breaks resolution for everyone who already installed it, which is the outcome the notice period exists to prevent.

## The honest limit

A rename is mechanically checkable: the identifier is right there in the diff, and a script could catch it.

Intent is not. Whether a reworded skill still does the same job is a judgment made by the person writing the change and the person reviewing it, and reasonable people will sometimes disagree. This policy does not pretend otherwise, and no CI check validates it today. What it does is make the question explicit and put it in front of a reviewer, which is the part that was missing.

The author's assertion about intent belongs in the pull request template tracked by [#139](https://github.com/uw-ssec/savvy/issues/139).

## See also

- [Versioning policy](versioning.md)
- [Contributing Guidelines](../CONTRIBUTING.md)
