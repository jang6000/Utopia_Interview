# How I work with an AI coding assistant

This repo is the setup I use every day with Claude Code. It was put together for my application to Utopia, in answer to the question about rules files.

- [CLAUDE.md](CLAUDE.md): the rules file the assistant reads at the start of every session. It covers my mission (reduce human error), how I like to be talked to, how I write requests, the hard rules, and the working loop.
- [.claude/commands/grill-with-docs.md](.claude/commands/grill-with-docs.md): a planning interview that runs before any code is written, and records decisions as they are made.
- [.claude/commands/ship.md](.claude/commands/ship.md): the shipping checklist, in order, stopping if any step fails.
- [.claude/skills/feature-playbook/SKILL.md](.claude/skills/feature-playbook/SKILL.md): the repeatable recipe for building a new feature.

The assistant I call Nadi is just Claude after it has read these files. I wrote them down because I would rather teach the tool how I work once than repeat myself every session.
