# action-ghcr-prune

GitHub Action to prune/delete container versions from GitHub Container Registry (ghcr.io).

## ⚠️ Word of caution

By default, both `prune-untagged` and `prune-tags-regexes` inputs are disabled and as result no versions will be matched for pruning. Either or both inputs must be explicitly configured for versions to be pruned. This behaviour helps to avoid pruning versions by mistake when first configuring this action.

As this action is destructive, it's recommended to test any changes to the configuration of the action with a dry-run to ensure the expected versions are matched for pruning. For more details about dry-runs, see the [dry-run input](#dry-run).

This is especially true when the [`prune-tags-regexes` input](#prune-tags-regexes) is used as regular expressions can easily match all versions of a container and result in complete deletion of all available versions.

## Quick start

Pruning all untagged versions older than 7 days, except the 2 most recent:
```yml
steps:
  - name: Prune
    uses: vlaurin/action-ghcr-prune@v0.5.0
    with:
      token: ${{ secrets.YOUR_TOKEN }}
      organization: your-org
      container: your-container
      dry-run: true # Dry-run first, then change to `false`
      keep-younger-than: 7 # days
      keep-last: 2
      prune-untagged: true
```

For more pruning strategies, [see filters](#Filters).

## Permissions

This action uses the Github Rest API [deletePackageVersionForOrg()](https://octokit.github.io/rest.js/v18#packages-delete-package-version-for-org) resource which states:
> To use this endpoint, you must have admin permissions in the organization and authenticate using an access token with the `packages:read` and `packages:delete` scopes. In addition:
> [...]
> If `package_type` is container, you must also have admin permissions to the container you want to delete.

As a result, for this action to work, the token must be associated to a user who has admin permissions for both the organization and the package. If this is not the case, then dry-runs will work as expected but actual runs will fail with a `Package not found` error when attempting to delete versions.

## Inputs

### token

**Required** Secret access token with scopes `packages:read` and `packages:delete` and write permissions on the targeted container. See [Creating a personal access token
](https://docs.github.com/en/github/authenticating-to-github/keeping-your-account-and-data-secure/creating-a-personal-access-token) for more details about GitHub access tokens.

### organization

Name of the organization owning the container package.

:warning: This input is mutually exclusive with input `user`.
Only one of the 2 can be used at any time.
If neither are provided, then the packages of the authenticated user (cf. `token`) are considered.

### user

Name of the user owning the container package.

:warning: This input is mutually exclusive with input `organization`.
Only one of the 2 can be used at any time.
If neither are provided, then the packages of the authenticated user (cf. `token`) are considered.

### container

**Required** Name of the container package for which versions should be pruned.

### dry-run

**Optional** Boolean controlling whether to execute the action as a dry-run. When `true` the action will print out details of the version that will be pruned without actually deleting them. Defaults to `false`.

As this action is destructive, it's recommended to test any changes to the configuration of the action with a dry-run to ensure the expected versions are matched for pruning.

### :mag: Filters

This action supports 2 types of filters:

- Exclusion filters, prefixed with `keep-`, exclude versions from pruning, preventing them from being deleted.
- Inclusion filters, prefixed with `prune-`, select the versions to prune.

**Exclusion filters always take precedence over inclusion filters.**
This means that if a version of a container is matched by both an exclusion and an inclusion filter, the exclusion will take priority and the version will not be pruned.

Versions that are not matched by any filter are preserved.

#### keep-last

**Optional** Count of most recent, matching containers to exclude from pruning. Defaults to `0` which means that all matching containers are pruned.

#### keep-tags

**Optional** List of tags to exclude from pruning, one per line.
Any version with at least one matching tag will be excluded.
Matching is exact and case-sensitive.

#### keep-tags-regexes

**Optional** List of regular expressions for tags to exclude from pruning, one per line.
Each expression will be evaluated against all tags of a version. Any version with at least one tag matching the expression will be excluded from pruning.

For example, pruning all versions with tags starting with either `pr-` or `test-`, except the ones ending with numbers 42 or 1337:

```yml
steps:
  - name: Prune
    uses: vlaurin/action-ghcr-prune@v0.5.0
    with:
      token: ${{ secrets.YOUR_TOKEN }}
      organization: your-org
      container: your-container
      dry-run: true # Dry-run first, then change to `false`
      keep-tags-regexes: |
        42$
        1337$
      prune-tags-regexes: |
        ^pr-
        ^test-
```

#### keep-younger-than

**Optional** Minimum age in days a version must have to qualify for pruning. All versions below that age at time of execution are excluded from pruning. Defaults to `0` which means no versions will be excluded from pruning.

#### prune-tags-regexes

**Optional** List of regular expressions for tags to prune, one per line.
Each expression will be evaluated against all tags of a version.
Any version with at least one tag matching the expression will be pruned.
Disabled by default (ie. no versions pruned based on tags).

:warning: **Please note:** Extra care should be taken when using `prune-tags-regexes`, please make sure you've read the [Word of caution](#word-of-caution)


For example, pruning all versions with tags starting with either `pr-` or `test-`:
```yml
steps:
  - name: Prune
    uses: vlaurin/action-ghcr-prune@v0.5.0
    with:
      token: ${{ secrets.YOUR_TOKEN }}
      organization: your-org
      container: your-container
      dry-run: true # Dry-run first, then change to `false`
      prune-tags-regexes: |
        ^pr-
        ^test-
```

#### prune-untagged

**Optional** Boolean controlling whether untagged versions should be pruned (`true`) or not (`false`). Defaults to `false`.

## Outputs

### count

The count of container versions which were successfully pruned by the action.

### prunedVersionIds

An array containing all the version IDs successfully pruned as part of the run.

### dryRun

Boolean flag indicating whether the execution was a dry-run, as per input `dry-run`. This output can be used to determine if other outputs relates to a dry-run or actual pruning of versions.

## Contribute

### Build

This action is compiled into a single JS file using [@vercel/ncc](https://github.com/vercel/ncc). This produces a `dist/` folder which must be checked in with the code.

Compilation can be done using:

```bash
npm run build
```

## License

The scripts and documentation in this project are released under the [MIT License](LICENSE).
