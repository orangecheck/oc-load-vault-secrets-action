# Load OC Vault secrets — GitHub Action

Resolve `ocv://` secret references from [OC Vault](https://vault.ochk.io)
into a GitHub Actions job. Your code holds only references; the secrets are
fetched, decrypted, and loaded at run time.

**Zero-knowledge.** Decryption happens on the runner. The access token
authorizes transport only — it carries no key material — and the passphrase
(which derives the vault key) never leaves the runner. OrangeCheck only ever
serves ciphertext.

## Usage

Put `ocv://` references in the step's `env:`; the action resolves them and
loads the values into the job environment for every later step.

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: orangecheck/oc-load-vault-secrets-action@v1
        with:
          token: ${{ secrets.OCV_TOKEN }}
          passphrase: ${{ secrets.OCV_PASSPHRASE }}
        env:
          DATABASE_URL: ocv://personal/prod-postgres/url
          STRIPE_KEY: ocv://personal/Stripe/key

      - run: ./deploy.sh   # DATABASE_URL and STRIPE_KEY are set, and masked in logs
```

Or point at a committed `.env` file of references:

```yaml
      - uses: orangecheck/oc-load-vault-secrets-action@v1
        with:
          token: ${{ secrets.OCV_TOKEN }}
          passphrase: ${{ secrets.OCV_PASSPHRASE }}
          env-file: .github/secrets.ocv
```

## Inputs

| Input | Required | Description |
|---|---|---|
| `token` | yes | An OC Vault access token (`ocvt_…`). Mint one at **vault.ochk.io/vault/developer**. |
| `passphrase` | yes | Your vault passphrase. Decrypts the vault on the runner. |
| `env-file` | no | A `.env` file of `KEY=ocv://…` references. Without it, any job env var whose value is an `ocv://` reference is resolved. |
| `cli-version` | no | The `@orangecheck/vault-cli` range to install. Default `^0.2`. See [Multi-line secrets](#multi-line-secrets) before relying on it. |

## Setup

1. At **vault.ochk.io/vault/developer**, mint a **read-only** access token.
2. Add it as the `OCV_TOKEN` repository secret, and your vault passphrase as
   `OCV_PASSPHRASE`.
3. Reference secrets as `ocv://personal/<item>/<field>` — see the
   [`ocv://` syntax](https://ochk.io/docs/vault/developer).

Resolved values are registered as masked secrets, so they are redacted in
job logs. A multi-line value is the exception until vault-cli 0.3.3.

## Multi-line secrets

GitHub masks one line at a time. vault-cli masks every line of a multi-line
value, such as a PEM or SSH key, only from 0.3.3. Versions 0.2.0 through
0.3.1, including the 0.2.0 that this action's default `^0.2` installs, mask
the first line and print the rest into the job log in clear.

This action's default moves to 0.3.3 in its next release. Until then, don't
load multi-line secrets with it. Once vault-cli 0.3.3 is on npm, you can set
`cli-version: '^0.3.3'` to get the fix sooner.

## License

MIT © OrangeCheck
