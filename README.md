# uhifadhilabs recipes

Private [Symfony Flex](https://github.com/symfony/recipes) recipe endpoint for
the uhifadhi modules. Not a server: the endpoint is static JSON
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

### Recipes the skeleton itself installs

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

## What a recipe must NOT try to do: assets/controllers.json

Stimulus controllers are not recipe business. Flex synchronises
`assets/controllers.json` from each installed package's own
`assets/package.json`, on every `composer require`/`update`, and it rewrites the
file wholesale from the packages it finds. It only looks at a package that
declares the **`symfony-ux` keyword in its `composer.json`** (see
`Symfony\Flex\PackageJsonSynchronizer::resolvePackageJson`).

A module missing that keyword therefore has its `controllers.json` entries
silently deleted by the next `composer update`, however carefully they were
added by hand. The fix belongs in the module, not here: add `symfony-ux` to its
`keywords` and ship `assets/package.json` with a `symfony.controllers` map.

The same file's `symfony.importmap` block is read the same way, and Flex runs
`importmap:require` for each entry — including path entries pointing into the
package's own AssetMapper namespace. A module whose JavaScript is imported by
bare specifier can declare those specifiers there instead of asking the host to
hand-edit `importmap.php`.

`uhifadhi/map-module` does exactly that, for `uhifadhi/basemaps`,
`uhifadhi/boundary` and `uhifadhi/map-chrome`. Its recipe's `map.yaml` therefore
documents the three lines without shipping them: they arrive from the package,
not from here. That is the general rule — **anything that has to end up in
`importmap.php` belongs in the module's `assets/package.json`, never in a
recipe** — and it applies to installations running AssetMapper, which is the only kind
of host that has an `importmap.php` for Flex to write into.

## License

MIT — see [LICENSE](LICENSE). Recipes are configuration a host copies into its
own project, so the permissive licence the Symfony recipes repositories use is
the right one here: the modules themselves carry their own licences, and
nothing about installing one should be encumbered by the terms of the file that
wired it up.
