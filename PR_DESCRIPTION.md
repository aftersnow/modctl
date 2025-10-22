# Fix: Correct spelling from StoargeDir to StorageDir

## 📋 Summary

This PR fixes a consistent spelling error throughout the codebase where `StoargeDir` was used instead of the correct `StorageDir`.

## 🐛 Issue

During a code review of the `modctl logout` functionality, a systematic spelling error was discovered across the entire codebase. The field name `StoargeDir` in the configuration struct and all its references were misspelled.

## 🔧 Changes Made

### Files Modified:
- **`pkg/config/root.go`** - Fixed struct field definition (2 occurrences)
- **`cmd/root.go`** - Fixed command-line flag binding (1 occurrence)
- **All command files in `cmd/`** - Fixed backend initialization calls (16 files):
  - `attach.go`, `build.go`, `extract.go`, `fetch.go`, `inspect.go`
  - `list.go`, `login.go`, `logout.go`, `prune.go`, `pull.go`
  - `push.go`, `rm.go`, `tag.go`, `upload.go`

### Before:
```go
type Root struct {
    StoargeDir      string  // ❌ Misspelled
    // ...
}

b, err := backend.New(rootConfig.StoargeDir)  // ❌ Misspelled
```

### After:
```go
type Root struct {
    StorageDir      string  // ✅ Correct spelling
    // ...
}

b, err := backend.New(rootConfig.StorageDir)  // ✅ Correct spelling
```

## ✅ Validation

- **17 files changed, 17 insertions(+), 17 deletions(-)**
- **No functional changes** - the error was consistent across the codebase
- **Command-line interface unchanged** - the flag name `--storage-dir` was already correct
- All references now use the correct spelling `StorageDir`

## 📊 Impact

- ✅ **Code Quality**: Improves code readability and professionalism
- ✅ **Maintainability**: Eliminates confusion for future developers
- ✅ **Consistency**: Ensures consistent naming throughout the codebase
- ⚠️  **Breaking Change**: This changes internal field names but maintains API compatibility

## 🚀 Testing

The changes are purely cosmetic (spelling correction) and maintain the same functionality. The command-line interface remains unchanged as the flag name was already correct.

---

## How to Create the PR

1. **Navigate to GitHub**: Go to https://github.com/aftersnow/modctl
2. **Create Pull Request**: GitHub should show a banner suggesting to create a PR from branch `cursor/check-modctl-logout-for-bugs-6487`
3. **Fill in details**:
   - **Title**: `Fix: Correct spelling from StoargeDir to StorageDir`
   - **Description**: Copy the content above
4. **Review and Submit**: Review the changes and create the PR

**Branch**: `cursor/check-modctl-logout-for-bugs-6487`  
**Target**: `main`  
**Commit**: `32ac60f`