# SSH Key

I store my primary SSH key as a file in 1Password and use the CLI to add it to
the SSH auth agent.

```sh
eval $(op signin $1PASSWORD_ACCOUNT)

# The document is named `id_ed25519` in my Private vault, and the lifetime of
# the key is set to 3 hours.
op get document id_ed25519 --vault Private | ssh-add -t 3h -
```
