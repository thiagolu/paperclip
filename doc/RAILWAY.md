# Railway Deployment Checklist

Deploying Paperclip on Railway requires a few platform-specific settings.

## 1) Dockerfile requirements

- Do not use `VOLUME` in Dockerfiles. Railway rejects builds containing the `VOLUME` keyword.
- This also applies to inherited `VOLUME` directives from base images.
- We include config-as-code in `railway.toml` (repo root), which pins:
  - Docker builder (`DOCKERFILE`)
  - default `dockerfilePath` (`Dockerfile`)
  - deploy healthcheck path (`/api/health`)
  - deploy healthcheck timeout (`300`)
- If you need a non-default Dockerfile name/path, either update `railway.toml` (`build.dockerfilePath`) or set:

```sh
RAILWAY_DOCKERFILE_PATH=<path-to-dockerfile>
```

Example:

```sh
RAILWAY_DOCKERFILE_PATH=Dockerfile.onboard-smoke
```

## 2) Persistent storage (required for stateful runs)

Paperclip persists instance data under `PAPERCLIP_HOME`, which defaults to `/paperclip` in our Docker images.

Attach a Railway Volume and set:

- Mount path: `/paperclip`

Important Railway behavior:

- Volumes are mounted at runtime, not build time.
- Volume mounts are owned by `root`.

Because our default image runs as a non-root user (`node`), set this Railway service variable when a volume is attached:

```sh
RAILWAY_RUN_UID=0
```

Note: `RAILWAY_RUN_UID` is a service variable and cannot be set via `railway.toml`.

## 3) Network + healthcheck

Railway injects `PORT`; your app must listen on that value. The Paperclip image already supports this.

Configure service healthcheck path to:

```sh
/api/health
```

If hostname allowlisting is enabled for your deployment mode, include:

```sh
healthcheck.railway.app
```

in your allowed hosts.

## 4) Recommended Railway service variables

These are the common values for hosted authenticated mode:

```sh
HOST=0.0.0.0
PAPERCLIP_DEPLOYMENT_MODE=authenticated
PAPERCLIP_DEPLOYMENT_EXPOSURE=private
PAPERCLIP_HOME=/paperclip
```

Also set:

```sh
PAPERCLIP_PUBLIC_URL=https://<your-railway-domain>
```

to keep invite/auth URLs aligned with your external endpoint.

## 5) Post-deploy smoke checks

After deploy (and after attaching volume), verify:

```sh
curl https://<your-railway-domain>/api/health
```

Expected response:

```json
{"status":"ok"}
```
