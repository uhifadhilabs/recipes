# uhifadhilabs recipes

Private [Symfony Flex](https://github.com/symfony/recipes) recipe endpoint for
the uhifadhi module bundles. Not a server: the endpoint is static JSON
generated into the `flex/main` branch by the GitHub Action on every push to
`main`.

Recipes are keyed by **composer package name**, which is why the directories
here are `uhifadhi/…` while the repository itself lives under the `uhifadhilabs`
GitHub organisation.

## Use in an app

```bash
composer config extra.symfony.endpoint \
  '["https://api.github.com/repos/uhifadhilabs/recipes/contents/index.json?ref=flex/main", "flex://defaults"]'
```

Composer's GitHub token authenticates access to this private repo.

## Add / update a recipe

Edit the files under `<vendor>/<package>/<version>/`, push to `main`; the
Action regenerates `flex/main`. Recipe contents must stay client-agnostic
(synthetic example domains only) — same rule as the bundles themselves.

A recipe carries what a host would otherwise have to write by hand: the bundle
registration (`bundles`), the module's `config/packages/<alias>.yaml` and
`config/routes/<alias>.yaml` (shipped under the recipe's `config/` and copied by
`copy-from-recipe`), and any `.env` block the module documents (`env`; keys
named `#1`, `#2`, … become comment lines, in order).

## The host's half of a recipe-owned file

Some recipes ship a config file whose last word cannot be theirs — the seam's
`resolve_target_entities` needs the name of an entity only the host has, and
the map's satellite provider needs a key only one deployment holds. In both
cases the recipe ships the **instruction as a comment** and the host follows it.

The rule that keeps that reconcilable: **a recipe-owned file in a host is the
recipe's bytes verbatim, followed by exactly the blocks the recipe's own
comments told the host to add, under a marker line that says so.** Nothing is
edited in place, nothing is deleted, and one deployment's circumstances never
leak upward into the recipe every other installation gets.

The reason is mechanical. `composer recipes:install <pkg> --force` overwrites
recipe-owned files with the recipe's version — that is what the flag is for —
and prints "use `git diff` … `git checkout -p`". If the host's additions are
interleaved with the recipe's text, that review is an archaeology exercise; if
they are one contiguous block at the bottom, restoring them is one hunk.

## post-install.txt: hand-steps only

A recipe may ship a `post-install.txt`, printed by Flex under the package's name
after `composer require`. The rule here is narrower than Symfony's own:

**It exists only when the module has a required hand-step, and it contains only
that hand-step.** Not a list of the files the recipe wrote — `git diff` shows
those, and `composer recipes <pkg>` shows which file belongs to which recipe. Not
a tour of the module's features; the module's README is where those live.

The reason is arithmetic. An installation with five modules prints five blocks,
one after another, in a terminal somebody is reading for the first time. Every
line that is not something they must do makes the lines that are less likely to
be read. A module with nothing for the operator to do ships no post-install.txt
at all.

Format follows the Symfony recipes: bullets starting with `*`, `<fg=green>` for
paths and code, short.

## Two things Flex does that a recipe must not try to

**`assets/controllers.json`.** Flex synchronises it from each installed package's
own `assets/package.json`, on every `composer require`/`update`, and it rewrites
the file wholesale from the packages it finds. It only looks at a package that
declares the **`symfony-ux` keyword in its `composer.json`** (see
`Symfony\Flex\PackageJsonSynchronizer::resolvePackageJson`).

A module missing that keyword therefore has its `controllers.json` entries
silently deleted by the next `composer update`, however carefully they were
added by hand. The fix belongs in the module, not here: add `symfony-ux` to its
`keywords` and ship `assets/package.json` with a `symfony.controllers` map.

**`importmap.php`.** The same file's `symfony.importmap` block is read the same
way, and Flex runs `importmap:require` for each entry — including path entries
pointing into the package's own AssetMapper namespace. A module whose JavaScript
is imported by bare specifier can therefore declare those specifiers itself
instead of asking the host to hand-edit `importmap.php`.

## License

MIT — see [LICENSE](LICENSE). Recipes are configuration a host copies into its
own project, so the permissive licence the Symfony recipes repositories use is
the right one here: the modules themselves carry their own licences, and
nothing about installing one should be encumbered by the terms of the file that
wired it up.
