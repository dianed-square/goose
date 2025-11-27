# Recipe Documentation Automation - Development Session Notes

**Last Updated**: 2025-11-26  
**Branch**: `test-pr-creation` (commit `17ea77de5b4`)  
**Status**: Ready for Test Scenario 3 (Attempt 8)

## Quick Start for New Session

1. **Current state**: All code is committed and pushed to `test-pr-creation` branch
2. **Next step**: Run Test Scenario 3 (Attempt 8) - see instructions below
3. **Expected**: PR should correctly remove `context` field AND add back 2 validation rules

## Test Scenario 3 - Instructions

**Setup on test-pr-creation branch:**
- Added `context` field to Core Recipe Schema table (should be removed)
- Removed 2 validation rules (should be re-added):
  - `validate_json_schema` - "JSON schema must be valid"
  - `validate_parameters_in_template` - "All template variables must have corresponding parameter definitions"

**To run:**
1. Close/delete PR #3 if it exists
2. Go to: https://github.com/dianed-square/goose/actions/workflows/docs-update-recipe-ref.yml
3. Click "Run workflow"
4. Select branch: `test-pr-creation`
5. Set inputs:
   - `old_version`: `v1.9.0`
   - `new_version`: `v1.15.0`
   - `dry_run`: `false`
6. Click "Run workflow"

**Expected results:**
- Remove `context` field from Core Recipe Schema table
- Add back 2 validation rules to Validation Rules section
- No unintended changes (retry/settings should stay)
- PR created with proper title, body, labels

## All Issues Fixed (24 total)

### Workflow & Script Issues (1-21)
1. **libxcb missing**: Added libxcb1-dev (later removed - not needed for pre-built binary)
2. **output/ directory missing**: Added `mkdir -p output`
3. **GOOSE_REPO not set**: Added `GOOSE_REPO: ${{ github.workspace }}`
4. **ripgrep missing**: Added `ripgrep` to apt-get install
5. **cargo bin not in PATH**: Added PATH update (later removed with cargo install)
6. **Error messages hidden**: Added `set -o pipefail`
7. **Date command portability**: Added fallback date formats
8. **Git tags not fetched**: Added `fetch-tags: true` to checkout
9. **Fork missing upstream tags**: Added conditional upstream fetch step
10. **Upstream remote exists**: Added `|| git remote set-url` fallback
11. **Conflicting canary tag**: Added `--force` to git fetch
12. **Goose not configured**: Created config file with provider/model settings
13. **Provider configuration**: Configured OpenAI provider for testing
14. **Env var approach**: Cleaned up to use config file (CI/CD best practice)
15. **Config file missing**: Created ~/.config/goose/config.yaml with keyring: false
16. **Missing old-schema.json**: Added Step 1b to extract old-schema.json
17. **Wrong working directory**: Changed working-directory to output/
18. **Recipe file paths (first recipe)**: Updated to ./validation-changes.json (not ./output/...)
19. **AI not writing output (attempt 1)**: Made prompt explicit (didn't work)
20. **AI not writing output (final fix)**: Capture goose stdout in shell script
21. **Missing second recipe call**: Added goose run for update-recipe-reference.yaml

### Recipe Issues (22-24)
22. **AI removing unintended fields - ROOT CAUSE**: AI used `diff` format which removed more lines than intended
    - **Fix**: Added Critical Rule #4 with explicit guidance:
      - Use old_str/new_str (NOT diff format)
      - Include COMPLETE row to remove
      - Include 1-2 rows before/after for context
      - Verify exact match
      - Double-check not removing adjacent rows
    - **Also**: Streamlined recipe with Update Strategy Table

23. **Wrong file paths in update-recipe-reference.yaml**: Changed ./output/validation-changes.md to ./validation-changes.md

24. **AI using wrong file path for recipe-reference.md**: AI created shortened path ~/w/g/g/d/d/g/... instead of using RECIPE_REF_PATH
    - **Fix**: Added explicit instruction in prompt to use EXACT path from environment variable

## Key Learnings

### Path Issues
- Recipes run from `output/` directory, so paths should be `./file.md` not `./output/file.md`
- AI may create shortened paths - need explicit instructions to use environment variables
- Both recipes needed path fixes (issues #18, #23, #24)

### AI Output Handling
- AI may respond with text instead of using tools
- Solution: Capture stdout and write to file in shell script (issue #20)
- Need explicit instructions about file paths (issue #24)

### Table Editing Bug
- AI used `diff` format for table row removal
- Diff span was larger than visible context
- Accidentally removed adjacent rows (retry, settings)
- Solution: Explicit instruction to use `str_replace` with context rows (issue #22)

### Workflow Optimizations
- **Pre-built binary**: 5-10 min → 30 sec (removed Rust toolchain)
- **Removed libxcb**: Not needed for pre-built binary
- **Total speedup**: ~7-12 min → ~2-3 min (4-5x faster!)

## Test Results History

### Attempt 5 (Commit 9ca447ccb3d)
- ✅ Workflow completed, both recipes ran, PR created
- ❌ Removed retry/settings fields (Issue #22 - diff format bug)

### Attempt 6 (Commit 4265155e6e0)
- ✅ Workflow completed
- ❌ No PR created - second recipe couldn't find files (Issue #23)

### Attempt 7 (Commit 17ea77de5b4 - first push)
- ✅ Workflow completed, both recipes ran, PR created
- ✅ Removed context field correctly
- ❌ Didn't add validation rules (Issue #24 - wrong file path)

### Attempt 8 (Current - Commit 17ea77de5b4)
- All 24 issues fixed
- Explicit instruction to use RECIPE_REF_PATH
- **Expected**: Full success! 🎉

## Files Modified

### Workflow
- `.github/workflows/docs-update-recipe-ref.yml`
  - Added goose CLI installation (pre-built binary)
  - Added goose config file creation
  - Fixed working directory to `output/`
  - Added both recipe calls

### Scripts
- `documentation/automation/recipe-schema-tracking/scripts/run-pipeline.sh`
  - Added old-schema.json extraction (Step 1b)
  - Added output capture for goose run
  - Added validation-changes.md writing

### Recipes
- `documentation/automation/recipe-schema-tracking/recipes/synthesize-validation-changes.yaml`
  - Fixed file paths (./validation-changes.json not ./output/...)

- `documentation/automation/recipe-schema-tracking/recipes/update-recipe-reference.yaml`
  - Fixed file paths (./validation-changes.md not ./output/...)
  - Added Critical Rules section with table update guidance
  - Added Update Strategy Table
  - Added explicit instruction to use RECIPE_REF_PATH environment variable
  - Streamlined structure

### Test Branch
- `documentation/docs/guides/recipes/recipe-reference.md` (on test-pr-creation)
  - Added `context` field (intentional - to be removed by automation)
  - Removed 2 validation rules (intentional - to be re-added by automation)

## Provider Configuration

**Current (for testing in fork):**
- Provider: OpenAI (via `vars.GOOSE_PROVIDER` or default)
- Model: gpt-4o (via `vars.GOOSE_MODEL` or default)
- API Key: `secrets.OPENAI_API_KEY` (set in fork)
- Config file: `~/.config/goose/config.yaml` with `keyring: false`

**For upstream PR:**
- Team can set repository variables for their preferred provider
- Or use the defaults (openai/gpt-4o)
- Remove diagnostic logging before PR

## Remaining Work

### Test Scenarios
- [ ] **Test Scenario 3**: Full run with PR creation (Attempt 8 - in progress)
- [ ] **Test Scenario 4**: Auto-detection test (no versions specified)

### Phase 2: Release Trigger Testing
- [ ] Uncomment `release: types: [published]` in fork
- [ ] Create test release in fork
- [ ] Verify automatic PR creation on release

### Before Team PR
- [ ] Complete all test scenarios in fork
- [ ] Document any additional issues found
- [ ] Confirm with team which provider to use in upstream
- [ ] Create clean branch from upstream `main` for PR
- [ ] Copy changes to clean branch
- [ ] **Ensure `release` trigger is COMMENTED OUT in PR**
- [ ] Final review of TESTING.md
- [ ] Update workflow defaults to match upstream's provider config
- [ ] Remove diagnostic logging before team PR
- [ ] Delete test branch after testing
- [ ] Close/delete test PRs

## Repository Structure

```
documentation/automation/recipe-schema-tracking/
├── README.md                    # Project overview and quick start
├── TESTING.md                   # Comprehensive testing guide
├── SESSION-NOTES.md            # This file - development session notes
├── .gitignore                  # Excludes output/ directory
├── config/
│   └── serde-attributes.json   # Field defaults and custom deserializers
├── output/                     # Generated files (gitignored)
│   ├── *.json                  # Extracted schemas and changes
│   ├── *.md                    # Generated documentation
│   └── *.log                   # Pipeline logs
├── recipes/
│   ├── synthesize-validation-changes.yaml  # Step 1: Generate change docs
│   └── update-recipe-reference.yaml        # Step 2: Apply changes to docs
└── scripts/
    ├── run-pipeline.sh                     # Main orchestration script
    ├── extract-schema.sh                   # Extract OpenAPI schema
    ├── extract-validation-structure.sh     # Extract struct fields & functions
    └── diff-validation-structures.sh       # Compare and generate diff

.github/workflows/
└── docs-update-recipe-ref.yml              # GitHub Actions workflow
```

## Useful Commands

**Check current branch and status:**
```bash
cd /Users/dianed/Development/forked-goose
git status
git log --oneline -5
```

**View recent workflow runs:**
```bash
gh run list --repo dianed-square/goose --workflow=docs-update-recipe-ref.yml --limit 5
```

**Download artifacts from a run:**
```bash
gh run download <run-id> --repo dianed-square/goose --dir /tmp/artifacts
```

**View PR:**
```bash
gh pr list --repo dianed-square/goose
gh pr view <pr-number> --repo dianed-square/goose
gh pr diff <pr-number> --repo dianed-square/goose
```

**Test locally:**
```bash
cd documentation/automation/recipe-schema-tracking
./scripts/run-pipeline.sh v1.9.0 v1.15.0
```

## Contact & Context

- **Fork**: https://github.com/dianed-square/goose
- **Upstream**: https://github.com/block/goose
- **Workflow**: `.github/workflows/docs-update-recipe-ref.yml`
- **Documentation**: `documentation/docs/guides/recipes/recipe-reference.md`
- **Test branch**: `test-pr-creation`
- **Main branch**: `main` (needs fixes merged after testing complete)
