# ns8-coder

Coder is an open source cloud development environment (CDE) that you download and host in your cloud. Deploy a single solution in seconds that provisions the infrastructure, IDE, language, and tools your developers desire.

Module id: `Coder`. Images are pinned in `build-images.sh` and updated through reviewed changes.

## Install

```bash
add-module ghcr.io/geniusdynamics/coder:latest 1
```

The command returns the instance id, for example `Coder1`. Use the id NS8 returns.

## Configure

```bash
api-cli run configure-module --agent module/Coder1 --data - <<'EOF'
{
  "host": "coder.example.org",
  "http2https": true,
  "lets_encrypt": false
}
EOF
```

| Parameter | Required | Type | Description |
|---|---|---|---|
| `host` | yes | string | Host name for the application, like 'kickstart.domain.org' |
| `lets_encrypt` | yes | boolean | Request a valid Let's Encrypt certificate. |
| `http2https` | yes | boolean | Redirect all the HTTP requests to HTTPS |

## Configuration output

`get-configuration` returns: `host`, `lets_encrypt`, `http2https`.

## Images

| Image | Version |
|---|---|
| `docker.io/postgres` | `18.7` |
| `ghcr.io/coder/coder` | `v2.8.3` |
| `docker.io/codercom/code-server` | `ubuntu` |

## Network and access

- Reserves 2 TCP port(s) on the node. The web application is published through Traefik on the configured host name.
- Runs rootless.
- Declares these NS8 authorizations: `traefik@node:routeadm`.

## Backup

The following paths are included in the NS8 backup (`imageroot/etc/state-include.conf`):

- `state/coder.sql`
- `volumes/coder-app`

## Services

- `coder-app.service`
- `coder.service`
- `postgresql-app.service`

## Get the configuration

```bash
api-cli run get-configuration --agent module/Coder1
```

## Update and uninstall

```bash
api-cli run update-module --data '{"module_url":"ghcr.io/geniusdynamics/coder:latest","instances":["Coder1"],"force":true}'
remove-module --no-preserve Coder1
```

## Testing

Continuous integration:

- `Publish images` builds the module image on every push.
- `Test module` installs the image on a QEMU NS8 node and runs the Robot Framework suite in `tests/`. It runs after `Publish images` succeeds on `dev` and on `main`.
- `Build API JSON Schema docs` and the security scan run alongside.

The suite installs the module, configures a host name, checks that the web application answers through Traefik, takes UI screenshots and removes the module.

Run it locally with the NS8 standard infrastructure described in the [ns8-github-actions README](https://github.com/NethServer/ns8-github-actions/blob/v1/README.md#running-tests-locally).

## Open items

Found by the NS8 module standards check. These must be resolved before a stable release:

- coder-app.service: uses %S/state, kickstart uses %E/state
- postgresql-app.service: uses %S/state, kickstart uses %E/state
- coder.service: uses %S/state, kickstart uses %E/state
- discover-smarthost still present: remove if the app does not use SMTP
- README still has the template boilerplate
- coder-app.service: %S/state changed to %E/state (review the bind mounts that remain)
- postgresql-app.service: %S/state changed to %E/state (review the bind mounts that remain)
- coder.service: %S/state changed to %E/state (review the bind mounts that remain)
- imageroot/actions/configure-module/80start_services: smarthost lines removed, check the unit still starts
- imageroot/systemd/user/coder-app.service: smarthost lines removed, check the unit still starts
- CODER_IMAGE is used by the module but no image in the label provides it: leftover from the template?
- POSTGRES_IMAGE is used by the module but no image in the label provides it: leftover from the template?

## Upstream

Application documentation: https://coder.com/docs/v2/latest

## License

GPL-3.0-or-later. See [LICENSE](LICENSE).
