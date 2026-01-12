# vibe-plugins

Claude Code skills and plugins marketplace.

## Installation

### Via Claude Code Marketplace

Add this marketplace to Claude Code:

```bash
/plugin marketplace add eastlondoner/vibe-plugins
```

Then install plugins:

```bash
/plugin install vibe-skills@vibe-plugins
```

### Via npm

```bash
npm install vibe-plugins
```

## Available Plugins

| Plugin | Description |
|--------|-------------|
| `vibe-skills` | Example skills plugin with code review skill |

## Development

This repo is set up as a submodule in the main claude repo for local development:

```bash
# In the main claude repo
cd submodules/plugins/vibe-plugins
# Make changes, commit, push
```

## Structure

```
.claude-plugin/
  marketplace.json     # Marketplace definition
plugins/
  vibe-skills/      # Plugin directory
    .claude-plugin/
      plugin.json      # Plugin manifest
    skills/
      code-review/
        SKILL.md       # Skill definition
    commands/          # Slash commands (optional)
package.json           # npm package config
README.md
```

## Adding New Plugins

1. Create a new directory under `plugins/`
2. Add `.claude-plugin/plugin.json` manifest
3. Add skills in `skills/<name>/SKILL.md`
4. Add entry to `.claude-plugin/marketplace.json`
5. Bump version in `package.json`

## License

MIT
