# copier-ssf-ci

This is a [copier](https://github.com/copier-org/copier) template for applying a consistent CI pipeline to Salt formulas.

## Quickstart

Assuming the [requirements](#requirements) are installed here is a quick example of how this template could be used.

```
gh repo fork my-org/my-formula --clone --default-branch-only
cd my-formula
git switch -c chore/copier
copier copy -w https://github.com/dafyddj/copier-ssf-ci .
# Answer relevant questions
pre-commit install      # ** Important **
# Stage changes as required (see below)
git add -p kitchen.yml
git restore kitchen.yml
git rm release-rules.js
git add -A
git commit -m 'chore(copier): apply template https://github.com/dafyddj/copier-ssf-ci at v<VERSION>'
pre-commit run -va
# Ideally, test in GitLab CI (see below)
gh pr create            # Accept all defaults, although you may choose "Open a blank pull request"
gh pr checks --watch    # Observe tests passing
gh pr --merge -dm
```
## Review and stage the generated changes

When applying the Copier template to an existing repository, some care is needed to preserve repository-specific customisations. Most generated changes can be accepted as-is, but the following files require particular attention.

### 1. Review the lint configuration

Inspect `.rubocop.yml`, `.yamllint` and `.salt-lint` for existing file inclusion or exclusion rules.

The new template manages file selection through `.pre-commit-config.yaml`, so check whether any repository-specific rules need to be transferred there before staging the changes.

This is primarily a precautionary check; these files can normally be accepted as generated.

### 2. Stage `kitchen.yml` interactively

The template provides a generic Test Kitchen configuration suitable for new repositories. Existing repositories already have established test suites, which should be preserved.

Only stage the changes that update the list of platforms and images:

```bash
git add -p kitchen.yml
```

Typically, the first hunk contains the required updates and can be accepted (`y`), while the remaining hunks should be rejected (`n`). Check the contents of each hunk rather than relying solely on their order.

Once the required changes have been staged, discard the remaining unstaged changes:

```bash
git restore kitchen.yml
```

This preserves the staged platform and image updates while restoring the rest of the file to its previous state.

### 3. Remove the obsolete release configuration

The release rules previously defined in `release-rules.js` have been incorporated into `release.config.js`.

Remove the obsolete file and stage its deletion:

```bash
git rm release-rules.js
```

The generated `release.config.js` can be accepted as-is.

### 4. Review and stage the remaining files

The remaining files, including `.pre-commit-config.yaml`, can normally be accepted as generated, following a brief review for existing repository-specific customisations.

Pay particular attention to the `Gemfile`, which may contain dependencies or settings that should be preserved.

Once any necessary adjustments have been made, stage the remaining changes:

```bash
git add -A
```

### 5. Verify the staged changes

Before committing, review the complete staged diff:

```bash
git diff --cached
```

## Choosing test platforms

The goal is for every formula to be tested against all platforms supported by the CI infrastructure, which tracks upstream platform support provided by the Salt Project.

Platforms should only be excluded where there is a genuine technical reason why the formula cannot support them. For example, a formula managing APT is inherently limited to Debian-based distributions.

Where a formula has not yet been adapted to work on a particular platform, **prefer including that platform in the CI matrix with failures allowed**, rather than excluding it entirely.

This option is available through the Copier questionnaire. For example, to allow failures on several RPM-based distributions, the following answer can be provided:

```yaml
["^almalinux-", "^amazonlinux-", "^centos-", "^oraclelinux-", "^rockylinux-"]
```

These regular expressions match the corresponding platform names, regardless of version, allowing their tests to run without failures causing the overall pipeline to fail.

This approach keeps gaps in platform support visible, encourages improvements to cross-platform compatibility, and helps identify when changes to the formula or its dependencies make additional platforms viable.

## Testing before submitting a PR

To test your changes using GitLab CI before submitting a pull request, **fork the original repository in [GitLab](https://gitlab.com/saltstack-formulas)** rather than creating a new project. Eligible open-source forks benefit from substantially reduced CI compute-minute usage.

Add your fork as a remote and push your branch:

```bash
git remote add gitlab git@gitlab.com:<username>/<repository>.git
git push gitlab HEAD
```

Check the resulting GitLab CI pipeline for any failures before submitting your PR.
Confirm that the existing Test Kitchen suite has been preserved, any necessary lint inclusion or exclusion rules have been carried over, and no repository-specific customisations have been unintentionally removed.

After testing you may need to push your branch to GitHub before creating your PR:

```bash
git push -u origin
```

## Requirements

Install dependencies using `pip`, [pipx](https://pipx.pypa.io/stable/) or [Homebrew/Linuxbrew](https://brew.sh/).

For example:

```
pip install --user pipx
pipx ensurepath

pipx install copier pre-commit
```

or

```
brew install copier pre-commit
```

This is optional but the official **GitHub** [CLI](https://cli.github.com/) tool (`gh`) can ease working with GitHub using the command line.
