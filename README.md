# GitHub Actions Best Practices
Best practices to follow when using GitHub Actions.

- [Security hardening for GitHub Actions](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions)

# Development

## IDE
Use [VSCode](https://code.visualstudio.com/).

### Language/Syntax support
Use RedHat's [YAML](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml) extension which utilizes [schemastore.org](https://www.schemastore.org/json/). This will provide tons of features such as syntax highlighting and formatting.

# Workflow Usage
When writing workflows there are some things to keep in mind.

## Reuse Workflows
You should call workflows from other workflows to avoid duplication. Practice innersourcing to promote best practices and reuse welldesigned/tested workflows.

### Why?
- Easier to maintain
- Create workflows more quickly
- Avoid duplication. DRY(don't repeat yourself).
- Build consistently across multiple, dozens, or even hundreds of repositories
- Require specific workflows for specific deployments
- Promotes best practices
- Abstract away complexity

### Example
#### Caller (reusable-caller.yml)
```yml
jobs:
  build:
    uses: ./.github/workflows/reusable-called.yml
    with:
      username: ${{ github.actor }}
```

#### Called (reusable-called.yml)
```yml
on:
  workflow_call:
    inputs:
      username:
        default: ${{ github.actor }}
        required: false
        type: string

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Run a one-line script
        run: echo Hello, ${{ inputs.username }}!
```

## Set timeouts
By default a job will be cancelled after 360 minutes or 6 hours. Consider setting a timeout to avoid running jobs for too long.

### Why?
Prevent jobs from hogging the GitHub Actions queue. There are usage limits that can cause other jobs to wait or perform poorly. See [Usage limits](https://docs.github.com/en/actions/learn-github-actions/usage-limits-billing-and-administration#usage-limits).

A timeout can also help catch issues early.

### Example
[`jobs.<job_id>.timeout-minutes`](https://docs.github.com/en/enterprise-cloud@latest/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idtimeout-minutes)

```yml
jobs:
  build:
    timeout-minutes: 30
    runs-on: ubuntu-latest
    steps:
      ...
```

## Limit Permissions
The default [`${{ GITHUB_TOKEN }}`](https://docs.github.com/en/actions/security-guides/automatic-token-authentication) has access to most things. Consider limiting permissions.

### Why?
- Prevent accidental access to sensitive information
- Avoid accidental use of destructive actions

### Example
[`permissions`](https://docs.github.com/en/enterprise-cloud@latest/actions/using-workflows/workflow-syntax-for-github-actions#permissions)
#### Permission types
```yml
permissions:
  actions: read|write|none
  checks: read|write|none
  contents: read|write|none
  deployments: read|write|none
  id-token: read|write|none
  issues: read|write|none
  discussions: read|write|none
  packages: read|write|none
  pages: read|write|none
  pull-requests: read|write|none
  repository-projects: read|write|none
  security-events: read|write|none
  statuses: read|write|none
```

#### Permission all
```yml
permissions: read-all|write-all
```

#### No permissions
```yml
permissions: {}
```

## Consider if (third-party) actions are really needed
Actions on the marketplace are written by third-party developers. You should careful consider if you need to use an action. GitHub grants the [verified creator badge](https://docs.github.com/en/developers/github-marketplace/github-marketplace-overview/about-marketplace-badges#for-github-actions) to creators who are [partner organizations](https://partner.github.com/).

### Why?

Third-party actions could break your devops pipeline or even perform malicious actions if precausion is not taken.

### Example
#### Bad
```yml
jobs:
  print:
    runs-on: ubuntu-latest
    steps:
      - uses: phwes/simple_hello_world@v1
```
#### Good
```yml
jobs:
  print:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Hello World"
```

### References
- [Actions Verified ✅](https://github.com/marketplace?type=actions&verification=verified_creator)
- [Actions Published by GitHub 😸](https://github.com/marketplace?type=actions&query=publisher%3Agithub+publisher%3Aactions)
- [Keeping your GitHub Actions and workflows secure Part 1: Preventing pwn requests](https://securitylab.github.com/research/github-actions-preventing-pwn-requests/)
- [Keeping your GitHub Actions and workflows secure Part 2: Untrusted input](https://securitylab.github.com/research/github-actions-untrusted-input/)
- [Keeping your GitHub Actions and workflows secure Part 3: How to trust your building blocks](https://securitylab.github.com/research/github-actions-building-blocks/)

## Pin actions to SHAs
Pin actions to SHAs to ensure that the action is always the same. Consider pinning to the SHA of a tested release.

### Why?
Authors can overwrite branches and tags to point at malicious or breaking code. A SHA is entirely unique and can be used to pin an action to a specific version.

### Example
#### Bad
```yml
      - uses: phwes/simple_hello_world@main
```
#### Good
```yml
      - uses: phwes/simple_hello_world@12d9258754d937b3d2500da69bf531924eaa567a
```

## Consider using concurrency
You can use `jobs.<job_id>.concurrency` to ensure that only one job is running at a time. The parameter `cancel-in-progress` will also cancel any other jobs in the same concurrency group.

### Why?
You only want one deployment to run at a time and you always want to deploy the latest code. Also you sometimes don't need to run CI on intermediate commits when newer code has been pushed. Any use case that requires a mutex is a good candidate for concurrency.

### Example
```yml
concurrency: 
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

### Refernces
- [Using concurrency](https://docs.github.com/en/actions/using-jobs/using-concurrency)

## Keep Secrets as Environment Variables
To help protect secrets, consider using environment variables, STDIN, or other mechanisms supported by the target process.

### Why?
Command-line processes may be visible to other users (using the ps command) or captured by security audit events.

### Example
#### Bad
```yml
steps:
  - name: Run a one-line script
    run: echo Hello, ${{ secret.name }}!
```

#### Good
```yml
steps:
  - name: Run a one-line script
    env:
      NAME: ${{ secret.name }}
    run: echo Hello, $NAME!
```
### References
[Using encrypted secrets in a workflow](https://docs.github.com/en/actions/security-guides/encrypted-secrets#using-encrypted-secrets-in-a-workflow)

## Prefer using OIDC Connect
Consider using OIDC Connect to interact with the cloud.

### Why?
Long-lived credentials are insecure and OIDC Connect allows the use of short-lived access tokens.

### References
- [Security hardening your deployments](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments)
- [Configuring OpenID Connect in cloud providers](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-cloud-providers)
- [Using OpenID Connect with reusable workflows](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/using-openid-connect-with-reusable-workflows)

## Careful with `pull_request_target` event trigger
You can check out external PRs when using the [`pull_request_target`](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#pull_request_target) event trigger. This means you are giving write access and secret access to untrusted code. This could compromise your entire repository and any resources connected to Actions.

### Example
#### Bad
> **Warning**
> We are checking out code on the target repo and executing code from the pull request.
```yml
name: my action
on: pull_request_target

jobs:
  pr-check: 
    name: Check PR
    runs-on: ubuntu-latest
    steps:
      - name: Setup Action
        uses: actions/checkout@v3
        with:
          ref: ${{github.event.pull_request.head.ref}}
          repository: ${{github.event.pull_request.head.repo.full_name}}
      ... execute code from the pull request ...
```
