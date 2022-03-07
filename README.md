# GitHub Actions Best Practices
Best practices to follow when using GitHub Actions.


# Workflow Usage
When writing workflows there are some things to keep in mind.

## Keep Secrets as Environment Variables
To help protect secrets, consider using environment variables, STDIN, or other mechanisms supported by the target process.

### Why?
Command-line processes may be visible to other users (using the ps command) or captured by security audit events.

### Example
#### Good
```yml
steps:
  - name: Run a one-line script
    env:
      NAME: ${{ secret.name }}
    run: echo Hello, $NAME!
```

#### Bad
```yml
steps:
  - name: Run a one-line script
    run: echo Hello, ${{ secret.name }}!
```
### References
[Using encrypted secrets in a workflow](https://docs.github.com/en/actions/security-guides/encrypted-secrets#using-encrypted-secrets-in-a-workflow)
