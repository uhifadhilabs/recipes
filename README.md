# uhifadhilabs recipes

Private [Symfony Flex](https://github.com/symfony/recipes) recipe endpoint for
uhifadhilabs bundles. Not a server: the endpoint is static JSON generated into
the `flex/main` branch by the GitHub Action on every push to `main`.

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
