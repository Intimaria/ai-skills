# ai-skills

A personal/team [Claude Code](https://docs.claude.com/en/docs/claude-code) plugin marketplace —
a collection of skills shared via this repo.

## Install 

Add the marketplace once, then install whichever plugins you want:

```
/plugin marketplace add Intimaria/ai-skills
/plugin install redmine-ticket-writer@ai-skills
```

`Intimaria/ai-skills` is the GitHub shorthand; a full Git URL
(`https://github.com/Intimaria/ai-skills.git`) also works. After installing, the skills become
available automatically in every project — Claude consults them when relevant.

To get updates later:

```
/plugin marketplace update ai-skills
```

## Plugins

| Plugin | What it does |
|--------|--------------|
| `redmine-ticket-writer` | Write/format Redmine tickets, issues and wiki pages in Textile (the markup Redmine uses), ready to copy-paste. |

## Layout

```
ai-skills/
├── .claude-plugin/
│   └── marketplace.json          # the marketplace manifest (lists the plugins)
└── plugins/
    └── redmine-ticket-writer/
        ├── .claude-plugin/
        │   └── plugin.json        # the plugin manifest
        └── skills/
            └── redmine-ticket-writer/
                ├── SKILL.md
                └── references/
```

## Adding a new skill

1. Create `plugins/<plugin-name>/.claude-plugin/plugin.json`.
2. Put the skill under `plugins/<plugin-name>/skills/<skill-name>/SKILL.md`.
3. Add an entry to `.claude-plugin/marketplace.json` `plugins[]` pointing at `./plugins/<plugin-name>`.
4. Commit and push.
