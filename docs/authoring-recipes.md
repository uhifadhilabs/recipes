# Add / update a recipe

Edit the files under `<vendor>/<package>/<version>/`, push to `main`; the
Action regenerates `flex/main`. Recipe contents must stay client-agnostic
(synthetic example domains only) — same rule as the bundles themselves.

A recipe carries what a host would otherwise have to write by hand: the bundle
registration (`bundles`), the module's `config/packages/<alias>.yaml` and
`config/routes/<alias>.yaml` (shipped under the recipe's `config/` and copied by
`copy-from-recipe`), and any `.env` block the module documents (`env`; keys
named `#1`, `#2`, … become comment lines, in order).

## Recipes the skeleton itself installs

`uhifadhi/seam-module` and `uhifadhi/shell-module` are shipped by the skeleton
([`uhifadhi/uhifadhi`](https://github.com/uhifadhilabs/uhifadhi)), so their
recipes have a second half: the skeleton's `symfony.lock` records the recipe
version, its hash and the files it tracks, and that ledger is what tells a fresh
`create-project` there is nothing to update. Change one of those two recipes —
editing bytes in place or adding a version — and re-sync the ledger in the
skeleton:

```bash
composer recipes:update uhifadhi/seam-module
```

Skip it and every new installation is told "update available" on first boot,
with nothing to apply, because the stored hash no longer matches the recipe.
A file hand-committed into the skeleton without that command is the same bug.
