# analysis_finance_reports_input_database

Palestine Database — an input database for the [freeports](https://www.freeports.org) extraction engine.

`companies/` decides what **matches**: the identifiers, names and aliases by which a company is
recognised in a report. `lists/` decides only the **association** — which companies belong to
which target list. That difference is not a matter of filing; it is why a change to one is a bigger
change than a change to the other, and the version rule below follows from it.

## What a commit here has to clear

This database has no test suite, no linted source and no methodology grants, so there is no figure
here held to a minimum. The one thing `.githooks/pre-commit` enforces is that **the fingerprints
and the version move together**.

`metadata.yaml` declares `sha256.companies` and `sha256.lists`: the hash of the hashes of every
file under each directory, in `LC_ALL=C` order, with the path inside each hashed line — so moving a
file counts as much as editing it. Reproduce either by hand:

```sh
cd companies && find . -type f | sed 's|^\./||' | LC_ALL=C sort | xargs sha256sum | sha256sum
```

When one of them moves, `info.version` must move with it. Otherwise the manifest says version X
covers content Y, which is false, and the whole point of the field is that it is not. The bump is
mechanical, so the hook proposes it rather than making you work it out:

| What you changed | Proposed bump | Why |
|---|---|---|
| `lists/` only | **minor + 1**, patch reset | the association changed; what matches did not |
| `companies/` (with or without `lists/`) | **major + 1**, minor and patch reset | `companies/` changes the meaning of every run, and subsumes a lists change |

The patch component is never proposed: it is for a change that moves neither fingerprint — this
README, a description — and only you know you made one.

The hook asks on the terminal. **Declining, or having no terminal at all** — a rebase, a script, an
editor's commit button, a CI runner — **is no**: a prompt nobody can answer must never be read as a
yes.

`ci.yaml` says which branches are which. On `main` a fingerprint that moved without a version
bump refuses the commit; on a `dev` or `list/*` branch it warns and the commit stands; on an
`off` branch the hook does nothing. `freeports-dev branch-class` says which you are on and why.

By hand, at any time:

```sh
freeports-dev fingerprint             # compare and say
freeports-dev fingerprint --propose   # offer the bump
freeports-dev fingerprint --update    # write the manifest once the version has moved
```

Without `freeports-dev` on the PATH the hook states the rule and lets the commit through: this
database is edited by people who may have no Python environment at all.
