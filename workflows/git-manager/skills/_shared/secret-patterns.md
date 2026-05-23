# Secret-File Patterns (Shared Reference)

Single source of truth for which filenames are treated as secrets across the git-agent and its skills. Enforcement lives in each skill's `disallowed-tools` frontmatter; this document explains **what** and **why**, so new skills and prose can stay consistent.

## Full list

**Environment and config secrets**

- `.env`, `.env.*`, `*.env` — application environment variables (tokens, DB URLs, API keys).
- `.envrc`, `.env.vault` — direnv exports and dotenv-vault packed files.
- `auth.json` — Composer and similar auth configs.
- `credentials`, `credentials.json` — generic credential files.
- `secrets.yml`, `secrets.yaml`, `secrets.json` — generic secrets manifests.
- `*.token`, `token` — single-token files.
- `*.tfvars`, `terraform.tfstate*` — Terraform state and variables with embedded secrets.
- `wp-config.php` — WordPress DB credentials.
- `config/master.key`, `config/credentials/*.key` — Rails encrypted secrets.
- `appsettings.*.json`, `local.settings.json` — .NET / Azure Functions app settings.

**Private keys and keystores**

- `*.pem`, `*.key` — private keys (PEM / generic).
- `id_rsa*`, `id_ed25519*`, `id_ecdsa*`, `id_dsa*` — SSH private keys.
- `*.p12`, `*.pfx`, `*.pkcs12` — PKCS#12 keystores.
- `*.kdbx` — KeePass databases.
- `*.jks`, `*.keystore` — Java keystores.
- `*.asc`, `*.gpg` — GPG-armored blobs (often contain private material).
- `firebase-adminsdk*.json`, `service-account*.json` — GCP / Firebase service-account keys.

**Host and registry credentials**

- `.ssh/**` — SSH configs, known_hosts, and private keys.
- `.netrc` — HTTP basic-auth credentials.
- `.git-credentials` — git-stored HTTPS credentials.
- `.npmrc`, `.yarnrc` — npm / Yarn registry tokens.
- `.pypirc` — PyPI upload credentials.
- `.cargo/credentials*` — crates.io tokens.
- `.gradle/gradle.properties` — may contain signing keys.

**Cloud configs**

- `.aws/credentials`, `.aws/config`, `.aws/**` — AWS CLI credentials.
- `.kube/config`, `.kube/**` — Kubernetes kubeconfigs.
- `.docker/config.json` — Docker registry auth.
- `gcloud/**` — Google Cloud SDK credentials.

## Behavior rules

The blocklist is not exhaustive. Apply these regardless of pattern coverage:

1. **Treat anything that looks like a secret as a secret** — judgment call, not grep.
2. **Reference by path only**, never by content. Never echo retrieved bytes.
3. **Stop and ask the user** if a workflow seems to need reading or staging one — do NOT try to work around the block.
4. **Rotation first, scrubbing second.** If a secret was committed or pushed, tell the user to rotate the credential immediately. History rewrite (`git filter-repo` / BFG) is useful but never assume the old version isn't cached somewhere.