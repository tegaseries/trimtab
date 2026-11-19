# Contributing

Thanks for helping. A few ground rules keep the skills small and safe.

## Ground rules

- **Markdown only.** No scripts, hooks or settings changes in the core plugins. A proposal that needs code should live in its own plugin.
- **Frontmatter:** only `name`, `description`, `license`, `compatibility`, `metadata` and `allowed-tools`, because claude.ai rejects other keys on upload.
- **No personal data.** Do not commit real names, emails, file paths from your machine, account or folder ids, or anything from your own memory folder. Use `example.com` and invented examples.
- **Claims need evidence.** If you add a savings figure, say how you measured it. Mark guesses as guesses.
- **Keep `SKILL.md` short.** Fewer than 500 lines, and no detail that belongs in a supporting file.

## Checking a change

1. Open the plugin folder with `claude --plugin-dir plugins/trimtab` and try the skill (`/trimtab:token-lean`).
2. Run `claude plugin validate plugins/trimtab` and `claude plugin validate .` (the second checks the marketplace).
3. Search your diff for anything personal before you open a pull request.

## Rebuilding the upload files

The files in `dist/` are ZIP archives with the skill folder at the top level. Rebuild them after any change to a `SKILL.md` or to `templates/`.

macOS or Linux:

```bash
cd plugins/trimtab/skills
zip -r ../../../dist/token-lean.skill token-lean
zip -r ../../../dist/memory-map.skill memory-map
```

Windows: don't use `Compress-Archive`; it can write backslashes into entry names, which claude.ai rejects. Use any ZIP tool that writes forward slashes, such as `tar -a -cf` from the `skills` folder, or 7-Zip.

Bump the version in `plugin.json` and the `SKILL.md` metadata, and add a line to `CHANGELOG.md`.
