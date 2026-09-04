# The host's half of a recipe-owned file

Some recipes ship a config file whose last word cannot be theirs — the map's
satellite provider needs a key only one deployment holds. There the recipe ships
the **instruction as a comment** and the host follows it.

Ask first whether it really cannot be theirs. **Whoever knows the answer states
the resolution**: the seam's `resolve_target_entities` block used to live here as
a comment for the host to uncomment, and it is gone — the package that provides
an area (`uhifadhi/area-module`) prepends it, exactly as `uhifadhi/team-module`
prepends the answer to the user contract, and an installation now writes neither.
A comment telling a host to paste a line that only ever had one right value is a
design that has not finished.

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
