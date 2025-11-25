# GitHub Actions Workflows

This directory contains automated workflows for the goose project.

## Recipe Documentation Automation

**File**: `update-recipe-docs.yml`

Automatically updates the Recipe Reference Guide when recipe schema or validation rules change between releases.

### How It Works

```
┌─────────────────────────────────────────────────────────────────┐
│                    1. TRIGGER (Release or Manual)                │
└─────────────────────────────────────────────────────────────────┘
                                  ↓
┌─────────────────────────────────────────────────────────────────┐
│              2. DETECT VERSIONS (Previous → Current)             │
│  - Auto-detect from releases (production)                        │
│  - Manual input (testing)                                        │
└─────────────────────────────────────────────────────────────────┘
                                  ↓
┌─────────────────────────────────────────────────────────────────┐
│           3. EXTRACT & COMPARE (Deterministic Scripts)           │
│  - Extract schema from both versions                             │
│  - Extract validation rules from both versions                   │
│  - Diff to detect changes                                        │
└─────────────────────────────────────────────────────────────────┘
                                  ↓
┌─────────────────────────────────────────────────────────────────┐
│              4. SYNTHESIZE CHANGES (AI Recipe)                   │
│  - Generate human-readable change documentation                  │
│  - Explain breaking changes with migration guidance              │
└─────────────────────────────────────────────────────────────────┘
                                  ↓
┌─────────────────────────────────────────────────────────────────┐
│            5. UPDATE DOCUMENTATION (AI Recipe)                   │
│  - Update Core Recipe Schema table                               │
│  - Update Field Specifications sections                          │
│  - Update Validation Rules section                               │
│  - Update examples                                               │
└─────────────────────────────────────────────────────────────────┘
                                  ↓
┌─────────────────────────────────────────────────────────────────┐
│                 6. CREATE PR (or Upload Artifacts)               │
│  - Dry-run: Upload artifacts for review                          │
│  - Production: Create PR with changes                            │
└─────────────────────────────────────────────────────────────────┘
```

### Testing the Workflow

#### Prerequisites

1. **Fork the repository** (you already have this at `dianed-square/goose`)
2. **Push the workflow** to your fork:
   ```bash
   cd /Users/dianed/Development/forked-goose
   git add .github/workflows/
   git commit -m "Add recipe documentation automation workflow"
   git push origin automation-testing
   ```

3. **Enable GitHub Actions** in your fork (if not already enabled):
   - Go to your fork on GitHub
   - Click "Actions" tab
   - Click "I understand my workflows, go ahead and enable them"

#### Test 1: Dry-Run with Known Versions (Recommended First Test)

This test runs the full pipeline but doesn't create a PR - just uploads artifacts for review.

1. Go to your fork on GitHub: `https://github.com/dianed-square/goose`
2. Click "Actions" tab
3. Click "Update Recipe Documentation" workflow
4. Click "Run workflow" button
5. Fill in the form:
   - **Branch**: `automation-testing`
   - **Previous version**: `v1.14.0`
   - **New version**: `v1.15.0`
   - **Dry run mode**: `true` ✅
6. Click "Run workflow"

**Expected Result**:
- Workflow runs successfully
- No changes detected (v1.14.0 → v1.15.0 had no recipe changes)
- Artifacts uploaded with extraction results
- No PR created

#### Test 2: Dry-Run with Changes (Validate Change Detection)

Test with versions that have actual changes.

1. Run workflow again with:
   - **Previous version**: `v1.9.0`
   - **New version**: `v1.15.0`
   - **Dry run mode**: `true` ✅

**Expected Result**:
- Workflow detects changes (4 validation rules added, 1 field removed)
- Generates `validation-changes.md` with human-readable documentation
- Updates `recipe-reference.md` (visible in artifacts)
- Uploads artifacts with all generated files
- No PR created (dry-run mode)

**Review the artifacts**:
- Download the artifact zip file
- Check `validation-changes.md` - should document all changes
- Check `update-summary.md` - should show what was updated
- Compare the updated `recipe-reference.md` with the original

#### Test 3: Full Run with PR Creation (Final Validation)

Once you're confident the dry-runs work correctly, test PR creation.

1. Run workflow with:
   - **Previous version**: `v1.9.0`
   - **New version**: `v1.15.0`
   - **Dry run mode**: `false` ❌

**Expected Result**:
- Workflow runs successfully
- Creates a PR in your fork: `docs/recipe-reference-v1.15.0`
- PR contains updated `recipe-reference.md`
- PR description includes change summary and review checklist

**Review the PR**:
- Check that only `recipe-reference.md` was modified
- Verify changes match what you saw in the dry-run artifacts
- Confirm no unintended modifications
- Test that the documentation renders correctly

#### Test 4: Auto-Detection (Simulate Production)

Test the automatic version detection logic.

1. Run workflow with:
   - **Previous version**: *(leave empty)*
   - **New version**: *(leave empty)*
   - **Dry run mode**: `true` ✅

**Expected Result**:
- Workflow auto-detects the two most recent releases
- Compares them automatically
- Uploads artifacts

### Enabling Production Mode

Once all tests pass, enable automatic triggering on releases:

1. Edit `.github/workflows/update-recipe-docs.yml`
2. Uncomment the `release` trigger:
   ```yaml
   release:
     types: [published]
   ```
3. Commit and push to main branch (after PR review)

**What happens next**:
- When a new release is published, the workflow triggers automatically
- Compares the new release with the previous release
- If changes detected, creates a PR with documentation updates
- Team reviews and merges the PR

### Troubleshooting

#### Workflow doesn't appear in Actions tab

- Check that the workflow file is in `.github/workflows/` directory
- Ensure the file has `.yml` or `.yaml` extension
- Verify GitHub Actions is enabled in your fork

#### "No changes detected" when you expect changes

- Check the artifact `validation-changes.json` to see what was compared
- Verify the versions exist as git tags: `git tag | grep v1.15.0`
- Check the extraction scripts ran successfully in the logs

#### goose CLI installation fails

- The workflow installs goose from the current repository
- Ensure `crates/goose-cli` exists and builds successfully
- Check Rust toolchain is installed correctly

#### PR creation fails

- Verify the workflow has `contents: write` and `pull-requests: write` permissions
- Check that the branch name doesn't already exist
- Review the workflow logs for specific error messages

### Workflow Inputs Reference

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `old_version` | No | Auto-detect | Previous version tag (e.g., `v1.14.0`) |
| `new_version` | No | `HEAD` | New version tag (e.g., `v1.15.0`) |
| `dry_run` | No | `true` | If true, uploads artifacts but doesn't create PR |

### Reusing This Pattern

This workflow is designed as a template for other documentation automations. Key patterns to reuse:

1. **Version Detection Logic** (Step 2):
   - Auto-detect from releases
   - Manual override for testing
   - Fallback to sensible defaults

2. **Artifact Handling** (Step 5):
   - Always upload artifacts for review
   - Include all intermediate files
   - Use descriptive artifact names

3. **Dry-Run Mode** (Step 6):
   - Test without side effects
   - Review outputs before enabling PR creation
   - Useful for fork testing

4. **PR Creation** (Step 6):
   - Standardized format
   - Include change summary
   - Add review checklist
   - Link to related resources

### Future Enhancements

Potential improvements for future iterations:

- **Regression Testing**: Validate generated docs against source code
- **Change Notifications**: Post to Slack/Discord when changes detected
- **Automated Merging**: Auto-merge if all checks pass (with caution)
- **Multiple Documentation Targets**: Update other docs based on changes
- **Scheduled Runs**: Periodic checks for drift between docs and code

### Related Documentation

- [Recipe Documentation Automation](../../documentation/automation/README.md) - Details on the scripts and recipes
- [Recipe Reference Guide](../../documentation/docs/guides/recipes/recipe-reference.md) - The documentation being updated
- [GitHub Actions Documentation](https://docs.github.com/en/actions) - Official GitHub Actions docs

## Questions?

If you have questions about this workflow:
1. Check the extensive comments in `update-recipe-docs.yml`
2. Review the test results in your fork
3. Check the workflow logs for detailed execution information
4. Consult the [Recipe Documentation Automation README](../../documentation/automation/README.md)
