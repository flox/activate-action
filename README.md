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
        uses: flox/activate-action@v2
        with:
          command: npm run build
          dir: ./frontend

      - name: Activate remote environment
        uses: flox/activate-action@v2
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

`environment` and `dir` reach the step as environment variables rather than as
text pasted into a shell script, so a value containing quotes or spaces arrives
unchanged. `command` is different: Flox runs it as a shell command, so whatever
is put in it runs as shell. Keep untrusted text out of it by passing the value
through `env:` on the step and naming it in the command:

```yml
- uses: flox/activate-action@v2
  env:
    PR_TITLE: ${{ github.event.pull_request.title }}
  with:
    command: echo "$PR_TITLE"
```

> [!IMPORTANT]
> **Upgrading from v1.** v1 wrapped `command` in single quotes, so every single
> quote inside it had to be written as `'\''`. v2 runs `command` as written, so
> replace each `'\''` with a plain `'`:
>
> | v1 `command` | v2 `command` | v2 result if left unchanged |
> | --- | --- | --- |
> | `echo '\''quoted'\''` | `echo 'quoted'` | syntax error |
> | `echo "it'\''s"` | `echo "it's"` | prints `it'\''s` |
>
> Search your workflows for `'\''` to find every command that needs the edit.
>
> v1 also pasted `environment` and `dir` into the script unquoted, so a shell
> variable in either was expanded. v2 passes both literally, so
> `dir: $GITHUB_WORKSPACE/frontend` now fails with `Did not find an environment
> in '$GITHUB_WORKSPACE/frontend'`. Use a relative path, or let the workflow
> expand it: `dir: ${{ github.workspace }}/frontend`.

## 🔐 Trusting FloxHub environments

Activation hooks run arbitrary code, so Flox refuses to activate a FloxHub
environment it does not already trust and the step fails:

```
✘ ERROR: The environment my-org/my-env is not trusted.
```

The check covers the FloxHub environment named with `environment`, and every
FloxHub environment the activated environment pulls in through `[include]`,
whether that environment is local or on FloxHub. Environments owned by `flox`
are always trusted. For anything else, pick one of these:

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
  uses: flox/activate-action@v2
  with:
    environment: my-org/my-env
    command: netlify deploy
    trust: true
```

> [!NOTE]
> Authenticating as the environment's owner is not a substitute for either. A
> `flox_pat_` or `flox_sat_` token gives a job *access* to an environment but
> not *trust*, so set `FLOX_FLOXHUB_TOKEN` to reach a private environment and
> still pick one of the options above.

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
