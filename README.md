# ISEC6000 Assessment 2 — Jenkins CI/CD Environment

Infrastructure configuration for the Jenkins continuous integration environment
used to build, test, scan and publish the Node.js sample application.

The application source lives in a **separate repository** (a fork of
`aws-samples/aws-elastic-beanstalk-express-js-sample`). Splitting them is
deliberate: the application is built many times a day by anyone with commit
access, while this stack defines who may run those builds and with what
privileges. Different change rate, different review requirements, different
audience.

| Repository | Contains |
|---|---|
| this one | `docker-compose.yml`, controller image, environment documentation |
| app repo (fork) | application code, unit tests, `Dockerfile`, `Jenkinsfile` |

## Stack

Two services on a private bridge network:

- **`dind`** — a Docker daemon in its own container, serving TLS on 2376.
  Privileged, because a daemon must be; isolated, unpublished, and holding no
  credentials.
- **`jenkins`** — the controller. Built from `jenkins/Dockerfile`, which adds
  the Docker CLI and the plugins the pipeline needs. Talks to `dind` over
  verified TLS.

Jenkins never sees the host's Docker socket. That is the security property the
whole layout exists to provide: with `-v /var/run/docker.sock`, any pipeline
step becomes root on the host, because it can launch a privileged container
with `/` mounted.

## Running it

```bash
cp .env.example .env
docker compose up -d --build

# initial admin password (one time)
docker compose logs jenkins | grep -A2 "Please use the following password"

# reach the UI from your workstation
ssh -L 8080:127.0.0.1:8080 <user>@<server>
# then open http://localhost:8080
```

Verify the controller really is driving the isolated daemon:

```bash
docker compose exec jenkins docker info | head -20
docker compose exec jenkins env | grep DOCKER_
```

`DOCKER_HOST=tcp://docker:2376` with `DOCKER_TLS_VERIFY=1` — and no
`/var/run/docker.sock` anywhere — is the evidence for the hardening section of
the report.

## Post-install hardening checklist

Done through the UI after first login; screenshot each one.

- [ ] Complete the setup wizard and create a named admin account
- [ ] **Manage Jenkins → Security**: security realm = Jenkins' own user
      database; **disable** sign-up
- [ ] Authorization = *Matrix-based security* (or role-based). Remove all
      permissions from `Anonymous`; grant `Overall/Read` only to authenticated
      users who need it
- [ ] Confirm **CSRF Protection** is enabled (default) and
      **Agent → Controller Access Control** is enabled
- [ ] **Manage Jenkins → Nodes → Built-In Node**: keep executors low (`1`–`2`).
      Note for the report: the textbook hardening advice is `0` executors on the
      controller, but that requires a separate agent node — with a single-node
      stack it would deadlock every build. The equivalent protection here is
      that no build *tooling* runs on the controller: `npm`, the test runner and
      the scanner all execute inside ephemeral `node:16` containers, and image
      builds are delegated to the isolated DinD daemon. The controller executor
      only orchestrates. Make that argument explicitly — it shows you understood
      the control rather than copied it
- [ ] Add credentials (Manage Jenkins → Credentials):
      - `dockerhub-credentials` — *Username with password*, using a Docker Hub
        **access token**, not your account password
      - `snyk-api-token` — *Secret text*
- [ ] Configure build log retention on the job (keep last 15 builds)
- [ ] Keep Jenkins and plugins updated; check the Manage Jenkins warnings page

## Teardown

```bash
docker compose down          # stops containers, KEEPS volumes
docker compose down -v       # destroys jenkins_home: every job, build and
                             # credential. Do not run this.
```
