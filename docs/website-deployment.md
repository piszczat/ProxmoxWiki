# Website Deployment

## Goal

Production changes to the portfolio website are promoted through controlled branches before deployment.

```text
dev
  ↓ Pull Request + CI
test
  ↓ Pull Request + CI
main
  ↓ successful CI
Deploy Production workflow
  ↓ self-hosted runner
website LXC
  ↓ local health check
public endpoint health check
```

## CI

The CI workflow runs on GitHub-hosted Linux runners and performs:

1. checkout;
2. Node.js setup;
3. dependency installation;
4. linting;
5. tests and production build.

The promotion validation additionally checks that:

- PRs into `test` come from `dev`;
- PRs into `main` come from `test`;
- promotion PRs originate from the same repository rather than a fork with a misleading branch name.

## Production deployment

After CI succeeds on `main`, the production deployment workflow runs on the dedicated self-hosted runner.

The runner:

1. validates deployment configuration;
2. connects to the production website LXC over restricted SSH;
3. fetches `main`;
4. resets the working tree to the remote `main` state;
5. installs dependencies;
6. performs a production build;
7. restarts the website systemd service;
8. verifies that the service is active;
9. performs a local HTTP health check;
10. verifies the public HTTPS endpoint.

## Deployment host configuration

Environment-specific deployment addressing is not committed to the website repository. GitHub repository variables are used for non-secret environment configuration such as the production target host.

Secrets, private keys and credentials remain outside Git.

## SSH model

The self-hosted runner has a dedicated SSH key for deployment. The website LXC authorizes that key with source restrictions and the deployment user has only the sudo commands required to restart and inspect the website service.

## Public exposure

The production website is exposed through Cloudflare Tunnel. The deployment pipeline therefore does not require inbound web port forwarding on the home router.

## Security design

General CI jobs run on GitHub-hosted runners. The self-hosted runner is reserved for trusted production deployment workloads so arbitrary pull-request code is not executed inside the homelab.
