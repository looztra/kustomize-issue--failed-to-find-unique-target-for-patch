# atlantis

## Context

This component deploy an Atlantis server.

Source:

- [Official atlantis website](https://www.runatlantis.io/)
- [Atlantis on Github](https://github.com/runatlantis/atlantis)

## Patches

### atlantis--deployment

The following environment variables needs to be patched:

- VAULT_ADDR: Address of the vault server to contact.
- VAULT_SKIP_VERIFY: Allow vault agent to skip ssl verification.
- ATLANTIS_ATLANTIS_URL: Specify the URL that Atlantis is accessible from. Used in the Atlantis UI and in links from pull request comments.
- ATLANTIS_REPO_WHITELIST: Atlantis requires you to specify a whitelist of repositories it will accept webhooks from.

```yaml
  - target:
    group: apps
    version: v1
    kind: Deployment
    name: atlantis
  path: patches/atlantis/atlantis--deployment.yml
```
