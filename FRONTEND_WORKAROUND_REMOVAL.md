# Frontend Workaround Removal Guide

## Overview

This document provides guidance on removing ORT workarounds from the frontend configuration based on fixes included in this ORT build.

## ORT Fixes Included

This ORT build (branch: `fix/10888-license-choice-order-independence`) includes the following fixes:

| Issue | Description | Fixed By Commit |
|-------|-------------|-----------------|
| #10673 | Version ranges in package configurations don't work | 46dfc5d6f |
| #10721 | Package curation distorts declared license expressions | f01d85912 |
| #10815 | ORT analyze fails with optional NPM dependencies | c1b17d124 |
| #10888 | License choices depend on statement order non-deterministically | 63242d9fb |

## Frontend Workarounds That Can Be Removed

### 1. Version Ranges in Package Configurations (HIGH Priority)

**Issue**: #10673
**Status**: ✅ FIXED

**Action Required**:
- Replace specific versions of all package configurations (frontend) with version ranges covering all versions
- Look for exceptions where the package configuration only makes sense for a particular version
- E.g., for package curations this is the case when specifying git revisions

**Files to Update**:
- All `package-configuration.yml` files in frontend configuration
- Search for hardcoded version numbers and replace with ranges like `[1.0.0,2.0.0)`

**Example**:
```yaml
# Before (workaround):
- id: "NPM:@babel/core:7.23.0"
  package_configurations:
    - ...

# After (fixed):
- id: "NPM:@babel/core:[7.0.0,8.0.0)"
  package_configurations:
    - ...
```

---

### 2. Package Curation License Combining (HIGH Priority)

**Issue**: #10721
**Status**: ✅ FIXED

**Action Required**:
- Remove `concluded_license` from package curations for schemacrawler and scim2-sdk-common
- These are marked with comments mentioning the workaround
- **Important**: Do NOT touch `concluded_license` attributes in package configurations - only in package curations

**Files to Update**:
- `curations.yml` or similar curation files
- Search for `concluded_license` with comments about "workaround" or "PFM-ISSUE-31174"

**Example**:
```yaml
# Remove entries like this:
- id: "Maven:com.github.schemacrawler:schemacrawler:16.19.11"
  curations:
    concluded_license: "EPL-1.0 OR GPL-3.0-only WITH Classpath-exception-2.0"  # WORKAROUND - Remove this!
```

---

### 3. Optional NPM Dependencies (MEDIUM Priority)

**Issue**: #10815
**Status**: ✅ FIXED

**Action Required**:
- Contact CI/CD team (Heiko Bensch) to remove the preprocessing step that currently removes optional NPM dependencies
- This workaround was likely implemented in the pipeline/build scripts, not in ORT configuration files

**Files to Check**:
- CI/CD pipeline scripts (e.g., Jenkinsfile, .gitlab-ci.yml, GitHub Actions)
- Build scripts that preprocess `package.json` before ORT analysis
- Look for scripts that remove `optionalDependencies` field

---

### 4. License Choice Order Dependency (HIGH Priority)

**Issue**: #10888
**Status**: ✅ FIXED

**Action Required**:
1. Remove all package-specific license-choices (frontend)
2. Remove all duplicated license choices from the global license choice configuration
   - Choices should be order-independent now
   - `MIT OR Apache-2.0` is a duplicate of `Apache-2.0 OR MIT`
3. Check if the 2 workarounds mentioned in the "Note" section of the curation documentation are still needed
4. If not needed, remove those entries from the global choices configurations and from the documentation

**Files to Update**:
- `.ort.yml` - `license_choices` section
- Look for `package_license_choices` and remove package-specific entries
- Look for duplicated choice expressions in `repository_license_choices`

**Example**:
```yaml
# Remove package-specific license choices like this:
license_choices:
  package_license_choices:
    - package_id: "NPM:some-package:1.0.0"  # REMOVE - No longer needed
      license_choices:
        - given: "MIT OR Apache-2.0"
          choice: "MIT"

# Remove duplicate choices like this:
repository_license_choices:
  - given: "MIT OR Apache-2.0"
    choice: "MIT"
  - given: "Apache-2.0 OR MIT"  # REMOVE - Duplicate of above
    choice: "MIT"
```

---

## Workarounds That Cannot Be Removed Yet

### 1. Evaluator Path Excludes + License Choice (NO Priority)

**Issue**: #10867
**Status**: ❌ OPEN (not fixed in this ORT build)

**Current Workaround**:
- Custom `isEffective()` matcher in evaluator rules
- Replace `-isExcluded()` with `+isEffective()`

**Do NOT remove this workaround yet.**

---

### 2. @babel Packages Nested Provenance (LOW Priority)

**Issue**: #10996
**Status**: ❌ OPEN (not fixed in this ORT build)

**Current Workaround**:
- Configure `source_code_origins: [ARTIFACT]` for affected @babel packages

**Do NOT remove this workaround yet.**

---

### 3. Declared License Mappings (LOW Priority)

**Issue**: PFM-ISSUE-31680
**Status**: ❌ OPEN (not fixed in this ORT build)

**Current Workaround**:
- Mapping SSPL-1.0 to NONE for org.codelibs.elasticsearch packages

**Do NOT remove this workaround yet.**

---

## Action Plan

### Immediate Actions (After Deploying This ORT Build)

1. **Update Package Configurations** (HIGH Priority)
   - Replace specific versions with version ranges
   - Test with frontend repository

2. **Remove concluded_license from Curations** (HIGH Priority)
   - Remove workaround for schemacrawler and scim2-sdk-common
   - Verify that license expressions are correctly preserved

3. **Simplify License Choices** (HIGH Priority)
   - Remove package-specific license choices
   - Remove duplicate license choice expressions
   - Test that license resolution still works correctly

4. **Remove Optional Dependencies Preprocessing** (MEDIUM Priority)
   - Coordinate with CI/CD team
   - Test frontend analysis without preprocessing

### Testing Recommendations

For each workaround removal:
1. Run ORT analyzer on frontend repository
2. Compare results with previous run (before workaround removal)
3. Verify that:
   - All packages are still detected correctly
   - License expressions are correct
   - No new violations are introduced
4. Check that the specific issue the workaround addressed is now properly handled

### Rollback Plan

If issues occur after removing workarounds:
1. The workarounds can be re-applied in the configuration files
2. This ORT build contains the fixes, so the root cause is addressed
3. Any problems are likely configuration issues, not ORT bugs

---

## Summary

With this ORT build, you can safely remove **4 out of 7** workarounds from the frontend configuration:

- ✅ Version ranges (HIGH priority)
- ✅ Package curation license combining (HIGH priority)
- ✅ Optional NPM dependencies (MEDIUM priority)
- ✅ License choice order dependency (HIGH priority)

The remaining 3 workarounds must stay in place until corresponding ORT issues are fixed.

---

## Questions or Issues?

If you encounter problems while removing these workarounds, please:
1. Check that you're using this ORT build (with fix branch merged)
2. Review the specific commit that fixed the issue
3. Test the workaround removal in isolation
4. Contact the ORT development team if issues persist
