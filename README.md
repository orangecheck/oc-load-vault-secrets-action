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
| `cli-version` | no | The `@orangecheck/vault-cli` range to install. Default `^0.2`. |

## Setup

1. At **vault.ochk.io/vault/developer**, mint a **read-only** access token.
2. Add it as the `OCV_TOKEN` repository secret, and your vault passphrase as
   `OCV_PASSPHRASE`.
3. Reference secrets as `ocv://personal/<item>/<field>` — see the
   [`ocv://` syntax](https://ochk.io/docs/vault/developer).

Resolved values are registered as masked secrets, so they are redacted in
job logs.

## License

MIT © OrangeCheck
