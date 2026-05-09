# FHC ProconRulz Syntax Highlighting

This folder contains the Cursor/VS Code syntax highlighting extension for ProconRulz text files.

It recognizes these file patterns:

- `*procon*.txt`
- `*proconrulz*.txt`
- `.proconrulz`

Highlighted ProconRulz syntax includes triggers, conditions, actions, variables, operators, BF3 damage types, map keys and map modes.

## Install

From this folder:

```powershell
npx --yes @vscode/vsce package --allow-missing-repository
cursor --install-extension ".\fhc-proconrulz-syntax-0.0.1.vsix" --force
```

Then reload Cursor with `Developer: Reload Window`.

## Files

- `package.json`: extension manifest and file association patterns.
- `language-configuration.json`: comments, brackets and word matching.
- `syntaxes/proconrulz.tmLanguage.json`: TextMate grammar rules.
