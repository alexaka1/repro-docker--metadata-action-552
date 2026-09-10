# Reproduction for docker/metadata-action issue 552

Reproduction for [upstream issue 552](https://redirect.github.com/docker/metadata-action/issues/552).

Minimal reproduction of how `docker/metadata-action` v6.2.0 handles branch, tag, and commit-SHA checkout refs with `context: git` and `fetch-depth: 1`.

## Three scenarios

All three refs point to the same public commit, `374978565b5476f435282d53b130afb8553e8954`. Each job runs independently with the same pinned actions and fetch depth. Only the checkout ref and the corresponding metadata tag rule differ.

| Case | Checkout ref | Enabled metadata rule | Expected image tag |
| --- | --- | --- | --- |
| Branch | `repro-input` | `type=ref,event=branch` | `example/repro:repro-input` |
| Tag | `v0.0.1` | `type=ref,event=tag` | `example/repro:v0.0.1` |
| SHA | `374978565b5476f435282d53b130afb8553e8954` | `type=sha,prefix=sha-,format=short` | `example/repro:sha-3749785` |

Each job prints the input ref, shallow status, HEAD decoration, and available Git refs. It verifies that checkout selected the shared commit before invoking metadata-action, then verifies the generated tag if the action succeeds. A failure in one job does not cancel the other scenarios.

## Observed results

[Completed workflow run](https://github.com/alexaka1/repro-docker--metadata-action-552/actions/runs/34451938722)

| Case | Checkout | Metadata generation |
| --- | --- | --- |
| [Branch](https://github.com/alexaka1/repro-docker--metadata-action-552/actions/runs/34451938722/job/102789429143) | Passed | Passed |
| [Tag](https://github.com/alexaka1/repro-docker--metadata-action-552/actions/runs/34451938722/job/102789429246) | Passed | Passed |
| [SHA](https://github.com/alexaka1/repro-docker--metadata-action-552/actions/runs/34451938722/job/102789429391) | Passed | Failed with `Cannot infer ref from detached HEAD` |

## Run it

Run **Metadata action branch, tag, and SHA repro** from the Actions tab, or use:

```sh
gh workflow run repro.yml --repo alexaka1/repro-docker--metadata-action-552 --ref main
```

The workflow also runs when its file changes on `main`.

When running a fork, include the `repro-input` branch and `v0.0.1` tag. If they are missing, create them from the fixture commit in a clone of your fork:

```sh
git branch repro-input 374978565b5476f435282d53b130afb8553e8954
git tag v0.0.1 374978565b5476f435282d53b130afb8553e8954
git push origin repro-input refs/tags/v0.0.1
```

## Failure being reproduced

A shallow SHA checkout can leave a detached `HEAD` without a named branch or tag. The SHA tag rule only needs the commit SHA, but Git context resolution can fail before tag generation:

```text
Cannot infer ref from detached HEAD
```

See the [complete workflow](.github/workflows/repro.yml) for the three cases.
