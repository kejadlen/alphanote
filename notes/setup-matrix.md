---

---

Use the Docker image to generate a configuration file. This should use the
`generate` command, but that results in the following error:

```
PermissionError: [Errno 13] Permission denied: '/data/homeserver.yaml'
```

So instead, use `migrate_config` (from [this comment][migrate-config] on #6303)
as a workaround:

[migrate-config]: https://github.com/matrix-org/synapse/issues/6303#issuecomment-548293934

```sh
docker run --rm \
  --env SYNAPSE_SERVER_NAME=matrix.kejadlen.dev
  --env SYNAPSE_REPORT_STATS=yes
  --volume `pwd`/data:/data
  matrixdotorg/synapse:latest migrate_config
```

