<h1 align="center">
  <a href="https://flox.dev" target="_blank">
    <picture>
      <source media="(prefers-color-scheme: dark)"  srcset="img/flox-logo-white-on-black.png" />
      <source media="(prefers-color-scheme: light)" srcset="img/flox-logo-black-on-white.png" />
      <img src="img/flox-logo-black-on-white.png" alt="flox logo" />
    </picture>
  </a>
</h1>

<h2 align="center">
  Developer environments you can take with you
</h2>

<h3 align="center">
   &emsp;
   <a href="https://discourse.flox.dev"><b>Discourse</b></a>
   &emsp; | &emsp; 
   <a href="https://flox.dev/docs"><b>Documentation</b></a>
   &emsp; | &emsp; 
   <a href="https://flox.dev/blog"><b>Blog</b></a>
   &emsp; | &emsp;  
   <a href="https://twitter.com/floxdevelopment"><b>Twitter</b></a>
   &emsp;
</h3>

<p align="center">
  <a href="https://github.com/flox/activate-action/blob/main/LICENSE">
    <img alt="GitHub" src="https://img.shields.io/github/license/flox/activate-action?style=flat-square">
  </a>
  <a href="https://github.com/flox/flox/blob/main/CONTRIBUTING.md">
    <img alt="PRs Welcome" src="https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square"/>
  </a>
  <a href="https://github.com/flox/activate-action/releases">
    <img alt="GitHub tag (latest by date)" src="https://img.shields.io/github/v/tag/flox/activate-action?label=Version&style=flat-square">
  </a>
</p>

Runs a command in the context of a [Flox][flox-github] environment.


## ⭐ Getting Started

Create `.github/workflows/ci.yml` in your repo with the following contents:

```yml
name: "CI"

on:
  pull_request:
  push:

jobs:
  tests:
    runs-on: ubuntu-latest
    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Install Flox
        uses: flox/install-flox-action@v2
        with:
          # FloxHub environments must be trusted before they can be
          # activated. See "Trusting FloxHub environments" below.
          trusted-environments: my-username/my-netlify-env

      - name: Build website
        uses: flox/activate-action@v1
        with:
          command: npm run build
          dir: ./frontend

      - name: Activate remote environment
        uses: flox/activate-action@v1
        with:
          environment: my-username/my-netlify-env
          command: netlify deploy
```

## ⚙️ Inputs

| Input | Description | Default |
| --- | --- | --- |
| `command` | Command to run inside the environment. **Required.** | |
| `environment` | A FloxHub environment to activate, as `owner/name`. Omit to use a local `.flox/` directory. | |
| `dir` | Directory containing the `.flox/` directory to activate. | |
| `trust` | Trust the FloxHub environment for this activation only. See below. | `false` |

## 🔐 Trusting FloxHub environments

Activating an environment runs its `hook.on-activate` script, which is
arbitrary code. Flox therefore refuses to activate a FloxHub environment it
does not already trust, and normally asks you about it at the terminal. A
GitHub Actions runner has no terminal to answer on, so the prompt becomes a
hard failure:

```
✘ ERROR: The environment my-org/my-env is not trusted.

flox environments do not run in a sandbox.
Activation hooks can run arbitrary code on your machine.
Thus, environments need to be trusted to be activated.
```

This is not limited to the `environment` input. A **local** environment
activated with `dir` hits the same check for every FloxHub environment its
manifest pulls in via `[include]`, reported as `The included environment
my-org/my-env is not trusted.`

Two environments are trusted with no setup at all: those owned by `flox`
(such as `flox/python-pip`), and your own, when you are signed in
interactively. Everything else needs one of the following.

### Trust it once, at install time (recommended)

[`flox/install-flox-action`][install-flox-action] takes a
`trusted-environments` input, which records the trust decision in the
runner's Flox config before any activation happens. Every later step — this
action, a bare `flox activate`, an `[include]` — is then covered:

```yml
- name: "Install Flox"
  uses: flox/install-flox-action@v2
  with:
    trusted-environments: my-org/my-env,my-org/other-env
```

To trust every environment an organization owns, use a wildcard, which needs
Flox 1.14.0 or later:

```yml
    trusted-environments: my-org/*
```

This is the same thing as running
`flox config --set 'trusted_environments."my-org/my-env"' trust` on the
runner, and it is the option to reach for when a workflow activates more than
one environment, or activates an environment indirectly through `[include]`.

### Trust it for a single step

When one step needs an environment the rest of the workflow does not, set
`trust` on that step. It passes `--trust` to `flox activate`, which trusts the
environment for that activation only and writes nothing to the config:

```yml
- name: "Deploy"
  uses: flox/activate-action@v1
  with:
    environment: my-org/my-env
    command: netlify deploy
    trust: true
```

`trust` covers the environment's `[include]`s too, and overrides a `deny`
recorded in the config.

### Authenticate as the environment's owner

Flox always trusts an environment whose owner matches the signed-in handle, so
authenticating as the owner removes the need to trust it separately. Set
`FLOX_FLOXHUB_TOKEN` from a repository secret — you need this anyway for a
private environment, which cannot be fetched without it:

```yml
jobs:
  test:
    runs-on: ubuntu-latest
    env:
      FLOX_FLOXHUB_TOKEN: ${{ secrets.FLOX_SERVICE_TOKEN }}
```

> [!IMPORTANT]
> This shortcut only applies when the CLI can read your handle out of the
> token without calling FloxHub, which is true of the token an interactive
> `flox auth login` stores. It is **not** true of the opaque `flox_pat_`
> (personal access) and `flox_sat_` (service account) tokens, whose handle is
> resolved lazily and which `flox activate` never resolves. A workflow
> authenticating with one of those still needs `trusted-environments` or
> `trust`. Use the token to get *access* to the environment, and one of the
> two options above to get *trust*.

## 📫 Have a question? Want to chat? Ran into a problem?

We are happy to welcome you to our [Discourse forum][discourse] and answer your
questions! You can always reach out to us directly via the [flox twitter
account][twitter] or chat to us directly on [Matrix][matrix] or
[Discord][discord].


## 🤝 Found a bug? Missing a specific feature?

Feel free to [file a new issue][new-issue] with a respective title and
description on the the `flox/activate-action` repository. If you already
found a solution to your problem, we would love to review your pull request!


## 🪪 License

The `activate-action` is licensed under the MIT. See [LICENSE](./LICENSE).


[flox-github]: https://github.com/flox/flox 
[install-flox-action]: https://github.com/flox/install-flox-action
[flox-website]: https://flox.dev
[new-issue]: https://github.com/flox/activate-action/issues/new/choose
[discourse]: https://discourse.flox.dev
[twitter]: https://twitter.com/floxdevelopment
[matrix]: https://matrix.to/#/#flox:matrix.org
[discord]: https://discord.gg/5H7hN57eQR
[nix-website]: https://nixos.org
[nix-help-stores]: https://nixos.org/manual/nix/unstable/command-ref/new-cli/nix3-help-stores.html
[post-nixpkgs]: https://flox.dev/blog/nixpkgs
