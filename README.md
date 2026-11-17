# GChat Agent Skill

Набор инструкций для AI-агентов, работающих с GChat через опубликованные MCP-инструменты.

## Установка

Установить навык с помощью [Vercel Skills CLI](https://github.com/vercel-labs/skills):

```bash
npx skills add gteam-bit/gchat-skills --skill gchat
```

CLI поддерживает несколько AI-агентов. Чтобы выбрать целевые агенты явно, добавьте, например, `--agent codex --agent claude-code`.

## Содержимое

- `skills/gchat/SKILL.md` — основные инструкции и маршрутизация по справочникам.
- `skills/gchat/references/` — рабочие сценарии, каталог MCP-инструментов и подробности builder-схем.
