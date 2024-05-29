# The GitHub Actions Well-Architected Framework

## Introduction

The GitHub Actions Well-Architected Framework is a design framework that can improve the quality of a workload by helping it to:

* Be resilient, available, and recoverable.
* Be as secure as you need it to be.
* Deliver a sufficient return on investment.
* Support responsible development and operations.
* Accomplish its purpose within acceptable timeframes.
* The framework is founded on the five pillars of architectural excellence, which are mapped to those goals. They are: Reliability, Security, Cost Optimization, Operational Excellence, and Performance Efficiency.

Each pillar provides recommended practices, risk considerations, and tradeoffs. The design decisions must be balanced across all pillars, given the business requirements. The technical and actionable guidance is broad enough for all workloads and applies to a specific scenario. This guidance is centered on GitHub Actions.

Workload architecture isn't the same as its implementation. The Well-Architected Framework can set you up for success through architectural design, but the implementation choices depend on the business requirements and constraints of your organization.

## Pillars

The pillars dive into the five key areas of the Well-Architected Framework.

### 1. Reliability



### 2. Security

#### [Security hardening for GitHub Actions](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions)

### 3. Cost Optimization

#### Jobs round up to the nearest minute

When you run a job GitHub Actions spins up a new VM for the job to run on. The job is billed by the minute, and any fraction of a minute is rounded up to the nearest minute. For example, a job that runs for 61 seconds is billed for two minutes.

#### [Caching dependencies to speed up workflows](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows)

Caching dependencies can significantly reduce the time it takes to run a workflow. For example, if you're building a Node.js project, you can cache the `node_modules` directory to avoid reinstalling dependencies every time you run a workflow.

#### [Set timeouts for workflows](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idtimeout-minutes)

You can set a timeout for each [job](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idtimeout-minutes) or [step](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstepstimeout-minutes) in a workflow to prevent it from running for longer than it should. If the timeout is reached, the job or step is cancelled.

#### [Concurrency](https://docs.github.com/en/actions/using-jobs/using-concurrency)

Use concurrency to ensure that only a single job or workflow using the same concurrency group can run at a time.

```yml
concurrency:
  group: ${{ github.ref }}
```

You can also cancel a previous run when a new run is triggered.

```yml
  cancel-in-progress: true
```

#### Disable Actions

You can always disable actions for a specific [repository](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/managing-github-actions-settings-for-a-repository#managing-github-actions-permissions-for-your-repository) or [workflow](https://docs.github.com/en/actions/using-workflows/disabling-and-enabling-a-workflow) if you're not using them. You can always re-enable them later.

### 4. Deployment

#### [Environments](https://docs.github.com/en/actions/using-jobs/using-environments-for-jobs)

Environments allow you to specify a job as a deployment job. This allows you to better track deployments and enforce environment protection rules.

##### [Environment protection rules](https://docs.github.com/en/enterprise-cloud@latest/actions/deployment/targeting-different-environments/using-environments-for-deployment#deployment-protection-rules)


### 4. Operational Excellence

### 5. Performance Efficiency

#### Conditional on changed files

Sometimes you want to condition the jobs/steps that run based on what files changed in a push or pull request.

##### [Path filtering](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#onpushpull_requestpull_request_targetpathspaths-ignore)

You can use `on.<push|pull_request|pull_request_target>.<paths|paths-ignore>` to trigger a workflow based on the files changed in a push or pull request.

##### [tj-actions/changed-files](https://github.com/tj-actions/changed-files) and [dorny/paths-filter](https://github.com/dorny/paths-filter)

These actions allows you to conditionally run jobs or steps based on the files changed in a pull request.

### 6. Authoring

Use [VSCode](https://code.visualstudio.com/).

#### [GitHub Actions](https://marketplace.visualstudio.com/items?itemName=GitHub.vscode-github-actions)

The GitHub Actions extension lets you manage your workflows, view the workflow run history, and helps with authoring workflows. It provides syntax highlighting, integrated documentation, validation, code completion, and more.

#### [GitHub CLI](https://cli.github.com/)

The GitHub CLI is preinstalled on all GitHub-hosted runners. It's a great utility both during actions runtime and even for local development.

#### Running locally

While there isn't really a way to run GitHub Actions locally, there are a few ways to test your workflows locally.

##### Make your local machine as a [self-hosted runner](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/adding-self-hosted-runners)

You can use your local machine as a self-hosted runner to test your workflows locally. You do need to specify the `runs-on` label as the one you set your machine to. You still need to trigger the workflow through GitHub.

##### [Act](https://github.com/nektos/act)

Act is a command-line tool that allows you to run your GitHub Actions locally.

### 6. Scale and Reusability

You should call workflows from other workflows to avoid duplication. Practice innersourcing to promote best practices and reuse well designed and tested workflows.

- Easier to maintain
- Create workflows more quickly
- Avoid duplication. DRY(don't repeat yourself).
- Build consistently across multiple, dozens, or even hundreds of repositories
- Require specific workflows for specific deployments
- Promotes best practices
- Abstract away complexity

#### Reusable workflows & Composite Actions

[Reusable Workflows](https://docs.github.com/en/actions/learn-github-actions/reusing-workflows) and [Composite Actions](https://docs.github.com/en/actions/creating-actions/creating-a-composite-run-steps-action) can help you reuse logic across multiple workflows.

| Feature | [Reusable workflows](https://docs.github.com/en/actions/learn-github-actions/reusing-workflows) | [Composite Actions](https://docs.github.com/en/actions/creating-actions/creating-a-composite-run-steps-action) |
|---------|--------------------|-------------------|
| Definition | Reusable jobs | Reusable steps |
| Nesting Depth | 3 layers | 10 layers |
| Use of Secrets | Can use secrets from the caller workflow | Must be passed in as inputs |
| Can specify the `runs on` label | Yes | No |
| Can add additional steps to job | No | Yes |
| call | `uses: <owner><repo>/.github/workflows/<workflow>.yml@<ref>` | `uses: <owner>/<repo><?/path>@<ref>` |
| can be used in matrix strategy | Yes | Yes |

#### [Repository rulesets](https://docs.github.com/en/enterprise-cloud@latest/organizations/managing-organization-settings/managing-rulesets-for-repositories-in-your-organization)

Using the new version of branch protection rules, repository rulesets, you can actually [require workflows to pass before merging](https://docs.github.com/en/enterprise-cloud@latest/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets#require-workflows-to-pass-before-merging) a pull request for all(or some) of the repositories in your organization.

#### Keeping reusable workflows and actions up to date

It's important that you can cleanly update your workflows across all repositories that use them but how do we know that something won't break?

##### Versioning

> [!WARNING]
> Pinning to a branch such as `@main` is not a best practice because it can introduce breaking changes. It's recommended to pin to a specific version or use a version range.

By versioning your actions and reusable workflows you can ensure that you can update them in a controlled manner. This allows you to test the changes in a single repository before rolling them out to all repositories that use them.

###### [Dependabot](https://docs.github.com/en/code-security/supply-chain-security/keeping-your-dependencies-updated-automatically/about-dependabot-version-updates)

Dependabot supports GitHub Actions as a package manager. It can automatically create pull requests to update your workflows when new versions of actions are released. By introducing changes through PRs you can ensure that the changes are reviewed before merging and don't cause any issues.

#### [Starter workflows](https://docs.github.com/en/actions/using-workflows/creating-starter-workflows-for-your-organization)

> [!NOTE]  
> Starter workflows require a public .github repository, they are not available for private repositories.

Starter workflows are a great way to provide pre-written workflows to your users.

#### [Repository templates](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-template-repository)

Repository templates allow you to create a repository that can be used as a template for new repositories. This template can include workflows, actions, and other files that you want to be included in new repositories.

#### [Sharing secrets and variables within an organization](https://docs.github.com/en/actions/using-workflows/sharing-workflows-secrets-and-runners-with-your-organization#sharing-secrets-and-variables-within-an-organization)

## Conclusion

Summarize the key points of the framework and its benefits.
