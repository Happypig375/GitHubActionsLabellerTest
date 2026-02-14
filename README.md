# GitHubActionsLabellerTest

This repository demonstrates the Label workflow from [CSharpMath](https://github.com/hflexgrig/CSharpMath) that automatically labels pull requests based on public API changes.

## Workflow Overview

The `Label.yml` workflow runs on all pull requests and automatically applies labels based on the type of changes:

### How It Works

1. **Checks for changes in `PublicApi.Shipped.txt`**: 
   - If changes are detected in regular PRs (not from `action/ship-publicapi` branch), the workflow fails
   - This ensures public API changes are properly shipped through the release process

2. **Labels based on `PublicApi.Unshipped.txt` changes**:
   - **`Impact/Breaking change`**: If the changes include `*REMOVED*` entries (breaking changes)
   - **`Type/Enhancement`**: If the changes include only API additions (no breaking changes)
   - **`Type/Bug`**: If there are no public API changes
   - **`Type/Housekeeping`**: For PRs from the `action/ship-publicapi` branch

## Testing the Workflow

To test the workflow, create a pull request with one of the following scenarios:

### Scenario 1: Bug Fix (No API Changes)
- Make changes that don't modify `PublicApi.Unshipped.txt`
- Expected label: `Type/Bug`

### Scenario 2: New Feature (API Addition)
- Add new API entries to `PublicApi.Unshipped.txt`
- Expected label: `Type/Enhancement`

Example addition:
```
MyNamespace.MyClass.NewMethod() -> void
```

### Scenario 3: Breaking Change
- Add entries with `*REMOVED*` to `PublicApi.Unshipped.txt`
- Expected label: `Impact/Breaking change`

Example breaking change:
```
*REMOVED*MyNamespace.MyClass.OldMethod() -> void
```

### Scenario 4: Invalid Change (Should Fail)
- Modify `PublicApi.Shipped.txt` in a regular PR
- Expected result: Workflow fails with an error

## Files

- `.github/workflows/Label.yml`: The workflow that performs automatic labeling
- `PublicApi.Shipped.txt`: Contains the public API surface that has been released
- `PublicApi.Unshipped.txt`: Contains upcoming public API changes that haven't been released yet

## Security Considerations

This workflow uses `pull_request_target` which runs with elevated privileges (write access to pull requests). It checks out the PR code to analyze changes using git diff. While CodeQL may flag this pattern as potentially risky, the security impact is mitigated because:

- The workflow only runs read-only git commands (`git diff`) on the checked-out code
- No code from the PR is executed
- All scripts are inline PowerShell that don't execute files from the checkout
- The workflow only adds labels using the GitHub API with a restricted token

This is the standard pattern used in the original [CSharpMath repository](https://github.com/hflexgrig/CSharpMath) for automated API change detection and labeling.