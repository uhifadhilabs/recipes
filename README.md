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
