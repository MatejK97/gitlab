# gitlab

My self-hosted GitLab CE: the source of truth, CI/CD engine and Terraform
state backend for the whole homelab.

> A published snapshot of a repository that runs on my self-hosted GitLab.
> More about the lab: [homelab](https://github.com/MatejK97/homelab)

## Design

- **VPN-only.** The web UI is routed by Traefik to its WireGuard-only
  entrypoint, and Git over SSH is published on `10.10.0.1:2424`, the
  WireGuard address. Docker bypasses `ufw`, so binding to the VPN address is
  what keeps it private.
- **TLS at Traefik.** GitLab's own Let's Encrypt and HTTPS are off; it serves
  plain HTTP on the internal Docker network.
- **Tuned for one user** (`config/gitlab.rb`): smaller PostgreSQL buffers,
  one Puma worker, low Sidekiq concurrency, built-in Prometheus off.
- **Secrets stay out of Git:** data, logs, and `gitlab-secrets.json` are
  ignored; only `gitlab.rb` is tracked.

## Operations

- **Validate only.** CI checks the compose file, but GitLab is deployed by
  hand: a pipeline can't safely restart the platform it runs on.
- **Upgrades** follow GitLab's required upgrade stops, with a backup, a copy of
  `gitlab.rb` and `gitlab-secrets.json`, and `gitlab:check` afterwards.
- **Migration as a restore drill:** GitLab moved from a home VM to the VPS by
  restoring a backup onto a fresh instance of the same version, then
  verifying with `gitlab:doctor:secrets` that all encrypted data was readable.
