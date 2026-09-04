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

## Learn more

- [Add / update a recipe](docs/authoring-recipes.md) — where recipe files live,
  what a recipe carries, and the ledger re-sync the two recipes the skeleton
  ships with require.
- [The host's half of a recipe-owned file](docs/recipe-owned-files.md) — when a
  recipe ships an instruction instead of an answer, and the rule that keeps a
  host's additions reconcilable with `recipes:install --force`.
- [What a recipe must NOT try to do: assets/controllers.json](docs/stimulus-and-importmap.md)
  — why Stimulus controllers and importmap entries belong in the module's
  `assets/package.json`, never in a recipe.

## License

MIT — see [LICENSE](LICENSE). Recipes are configuration a host copies into its
own project, so the permissive licence the Symfony recipes repositories use is
the right one here: the modules themselves carry their own licences, and
nothing about installing one should be encumbered by the terms of the file that
wired it up.
