---
name: mac-maintenance
description: "Mac upkeep for Chase's machine: Homebrew update/upgrade, pull clean repos under ~/Projects, and empty Trash. Use when asked for Mac cleanup, maintenance, package refresh, repo refresh, or trash cleanup."
---

# Mac Maintenance

Run terse machine upkeep. Skip dirty repos unless Chase explicitly asks to handle them.
Do not broaden repo roots beyond `~/Projects` unless Chase names them.

## Run

1. Homebrew:

```bash
brew update && brew upgrade
```

2. Repos under `~/Projects`:

```zsh
for repo in ~/Projects/*/.git(N); do
  dir=${repo:h}
  git -C "$dir" status --short --branch
  git -C "$dir" pull --ff-only
done
```

Skip dirty repos. Report skipped paths.

3. Empty Trash:

```bash
osascript -e 'tell application "Finder" to empty trash'
```

4. Finish with terse counts:

- brew: upgraded / already current
- repos: pulled / skipped / failed
- trash: emptied / failed
