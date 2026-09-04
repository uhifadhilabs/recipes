# What a recipe must NOT try to do: assets/controllers.json

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
