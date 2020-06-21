# docker-cloudflared

<p>
  <a href="https://github.com/papanito/docker-cloudflared/blob/master/LICENSE"><img src="https://badgen.net/github/license/papanito/docker-cloudflared?color=cyan"/></a>
  <a href="https://github.com/papanito/docker-cloudflared"><img src="https://badgen.net/github/forks/papanito/docker-cloudflared?icon=github&label=forks"/></a>
  <a href="https://github.com/papanito/docker-cloudflared"><img src="https://badgen.net/github/stars/papanito/docker-cloudflared?icon=github&label=stars"/></a>
  <a href="https://cloud.docker.com/repository/docker/papanito/cloudflared"><img src="https://images.microbadger.com/badges/image/papanito/cloudflared.svg"/></a>
  <a href="https://cloud.docker.com/repository/docker/papanito/cloudflared"><img src="https://badgen.net/docker/pulls/papanito/cloudflared?icon=docker&label=pulls"/></a>
  <a href="https://cloud.docker.com/repository/docker/papanito/cloudflared"><img src="https://badgen.net/docker/stars/papanito/cloudflared?icon=docker&label=stars"/></a>
</p>

> Inspired by the [msnelling/docker-cloudflared](https://hub.docker.com/r/msnelling/docker-cloudflared) container.

A docker container for running [Cloudflare's Argo Tunnel](https://developers.cloudflare.com/argo-tunnel/quickstart/) to proxy a service.

You can activate a tunnel to a service by specifying the following environment variables:

* `TUNNEL_HOSTNAME` - The hostname on Cloudflare that you wish to register for the tunnel endpoint (i.e. mirror.example.com).
* `TUNNEL_ORIGIN_URL` - The local URL you wish to forward the above hostname to, for example http://localhost:8080.

Additionally you must mount your client origin certificate/private key/token file. By default `cloudflared` looks for it in `/etc/cloudflared/cert.pem` but this can be overridden with the environment variable `TUNNEL_ORIGIN_CERT`.

`cloudflared tunnel` is stared with `--no-autoupdate` option as auto-update does not work and your container will restart frequently with

```bash
[ERROR] Unable to restart server automatically: fork/exec /usr/local/bin/cloudflared: no such file or directory
[INFO] Initiating graceful shutdown due to failed to update cloudflared: fork/exec /usr/local/bin/cloudflared: no such file or directory ...
[ERROR] Quitting due to error: failed to update cloudflared: fork/exec /usr/local/bin/cloudflared: no such file or directory
```

Additional options can be configured by means of[] `config.yaml`](https://developers.cloudflare.com/argo-tunnel/reference/config/). Thus in kubernetes you create a config map:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cloudflare-cm
data:
  config.yaml: |
    #no-tls-verify: true
```

And then map the config map as volume to your container

```yaml
volumeMounts:
  - mountPath: /etc/cloudflared
    name: tunnel-secret
    readOnly: true
  - mountPath: ~/.cloudflared
    name: cloudflare-config
```

Ensure your volume is defined

```yaml
volumes:
- name: tunnel-secret
  secret:
    secretName: cloudflare-pem
- configMap:
    name: cloudflare-cm
  name: cloudflare-config
```

Above also shows how to map the `cert.pem` but this requires to create a secret `cloudflare-pem` as follows in the dedicated namespace `xxxxx`

```bash
kubectl create secret generic cloudflare-pem --from-file="$HOME/.cloudflared/cert.pem" -n xxxxx
```
