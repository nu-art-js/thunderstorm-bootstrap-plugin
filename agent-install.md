# Install `bootstrap-thunderstorm-project` on this machine

You are an agent. Do what this file says, then stop. Install the skill on the machine you are running on. Stay in the current workspace. Do not commit.

The skill has to be machine-wide, at `~/.cursor/skills/bootstrap-thunderstorm-project`. A copy under a project's `.cursor/skills/` does not count. Never write under `~/.cursor/skills-cursor/`.

## 1. Find the skill source

The skill directory is `skills/bootstrap-thunderstorm-project/` and it must contain `SKILL.md`.

- If this file is inside a checkout of `thunderstorm-bootstrap-plugin`, that checkout is `REPO_DIR`.
- Otherwise clone it, or update the clone if it is already there:

```bash
mkdir -p "${HOME}/.cursor/skill-sources"
DEST="${HOME}/.cursor/skill-sources/thunderstorm-bootstrap-plugin"
if [[ -d "$DEST/.git" ]]; then
  git -C "$DEST" pull --ff-only
else
  git clone git@github.com:nu-art-js/thunderstorm-bootstrap-plugin.git "$DEST"
fi
```

`REPO_DIR` is that checkout. Confirm the skill file is there:

```bash
test -f "$REPO_DIR/skills/bootstrap-thunderstorm-project/SKILL.md"
```

## 2. Symlink it into personal skills

```bash
mkdir -p "${HOME}/.cursor/skills"
SRC="$REPO_DIR/skills/bootstrap-thunderstorm-project"
DST="${HOME}/.cursor/skills/bootstrap-thunderstorm-project"
rm -rf "$DST"
ln -s "$SRC" "$DST"
```

Replacing whatever is already at `DST` is the install.

## 3. Verify

```bash
readlink "${HOME}/.cursor/skills/bootstrap-thunderstorm-project"
```

The link must resolve to `$SRC`. The `name` in `SKILL.md` frontmatter must be `bootstrap-thunderstorm-project`.

Tell the user the destination and the source path. Do not run the skill.
