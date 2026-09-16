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
          # Required before a FloxHub environment can be activated. See below.
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
| `environment` | A FloxHub environment to activate, as `owner/name`. Omit to activate a local one. | |
| `dir` | Directory containing the `.flox/` directory to activate. | current directory |
| `trust` | Trust this one activation of a FloxHub environment, rather than trusting it for the whole job. | `false` |

## 🔐 Trusting FloxHub environments

Activating an environment runs its `hook.on-activate` script, which is arbitrary
code, so Flox asks before activating a FloxHub environment it does not already
trust. A runner has no terminal to answer on, so the step fails instead:

```
✘ ERROR: The environment my-org/my-env is not trusted.
```

A local environment activated with `dir` hits the same check for every FloxHub
environment its manifest pulls in through `[include]`.

Flox already trusts environments owned by `flox`, and your own while you are
signed in interactively. Anything else needs one of the two options below.

**For the whole job**, use the `trusted-environments` input of
[`flox/install-flox-action`][install-flox-action]. It records the decision
before anything activates, so every later step is covered, including `[include]`s.
Name several environments with commas, or a whole organization with `my-org/*`
(Flox 1.14.0 or later):

```yml
- name: "Install Flox"
  uses: flox/install-flox-action@v2
  with:
    trusted-environments: my-org/my-env
```

**For a single step**, set `trust`. It passes `--trust` to `flox activate`,
covering that activation and its `[include]`s without writing anything to the
runner's config, and overriding a `deny` already recorded there:

```yml
- name: "Deploy"
  uses: flox/activate-action@v1
  with:
    environment: my-org/my-env
    command: netlify deploy
    trust: true
```

> [!IMPORTANT]
> Authenticating as the environment's owner is **not** a third option. Flox does
> trust an environment whose owner matches your handle, but it reads that handle
> only from the token itself — which works for the JWT an interactive
> `flox auth login` stores, and not for the opaque `flox_pat_` (personal access)
> and `flox_sat_` (service account) tokens a workflow would use. Set
> `FLOX_FLOXHUB_TOKEN` to get *access* to a private environment, and one of the
> options above to get *trust*.

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
