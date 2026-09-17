# =============================================================================
#  Jenkins controller image for ISEC6000 Assessment 2.
#
#  The stock jenkins/jenkins image has no Docker CLI, so it cannot drive the
#  DinD daemon. Rather than installing tools at runtime (unreproducible, and
#  it would need a root shell inside a running container), the CLI and the
#  required plugins are baked in here.
# =============================================================================

# lts-jdk17 is the current long-term-support line. For a production system,
# pin a digest (jenkins/jenkins@sha256:...) so the image is byte-identical on
# every rebuild.
FROM jenkins/jenkins:lts-jdk17

# --- root section: install the Docker CLI from Docker's own signed repository
USER root

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        ca-certificates curl gnupg lsb-release && \
    install -m 0755 -d /etc/apt/keyrings && \
    curl -fsSL https://download.docker.com/linux/debian/gpg \
        -o /etc/apt/keyrings/docker.asc && \
    chmod a+r /etc/apt/keyrings/docker.asc && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
https://download.docker.com/linux/debian $(. /etc/os-release && echo "$VERSION_CODENAME") stable" \
        > /etc/apt/sources.list.d/docker.list && \
    apt-get update && \
    # CLI only. The daemon (docker-ce) is NOT installed - it lives in the
    # separate DinD container, which is the whole point of the design.
    apt-get install -y --no-install-recommends docker-ce-cli && \
    apt-get purge -y --auto-remove gnupg && \
    rm -rf /var/lib/apt/lists/*

# --- drop back to the unprivileged jenkins user for everything else
USER jenkins

# Plugins installed at build time so the instance comes up ready and
# reproducible, instead of relying on someone clicking through the UI.
#   workflow-aggregator ... declarative Pipeline
#   docker-workflow ...... docker{} / docker.build() steps
#   git, github .......... SCM checkout and polling
#   credentials-binding .. withCredentials() for the registry and scanner tokens
#   matrix-auth .......... per-user authorisation matrix (Task 2 hardening)
#   junit ................ test result trends
#   timestamper .......... timestamps in console logs (Task 4 logging)
#   audit-trail .......... who changed what, when (Task 4 monitoring)
#   warnings-ng .......... annotated static-analysis / scanner reports
#   ws-cleanup ........... workspace cleanup in post{}
RUN jenkins-plugin-cli --plugins \
      "workflow-aggregator \
       docker-workflow \
       git \
       github \
       credentials-binding \
       matrix-auth \
       junit \
       timestamper \
       audit-trail \
       warnings-ng \
       ws-cleanup"
