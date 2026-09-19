# The whole wrapper. Everything else is the upstream image, unmodified.
#
# An explicit FROM is REQUIRED: since Supervisor 2026.04.0 there is no automatic
# BUILD_FROM fallback, so a Dockerfile that relies on one fails to build.
#
# Pinned to a release tag, never :latest. `version` in config.yaml must match
# this tag - that pairing is what makes Home Assistant offer an update button
# when we bump both.
FROM nousresearch/hermes-agent:v2026.9.14

# Hermes keeps its state in /opt/data. A Home Assistant add-on's persistent
# volume is /data, and that path is not configurable from our side.
#
# BOTH variables move, and they must move together. HERMES_HOME is where state
# lives; HERMES_WRITE_SAFE_ROOT is the sandbox boundary that stops the agent
# writing outside it. Moving only the first would leave the sandbox pointing at
# an empty /opt/data while the real state sat unprotected in /data.
ENV HERMES_HOME=/data
ENV HERMES_WRITE_SAFE_ROOT=/data

# Bring up the built-in web dashboard on 9119. Without this the s6 service
# starts and immediately has nothing to serve. It is the only route to edit
# config.yaml, because an add-on's /data is reachable by nothing but the add-on
# itself - not by Home Assistant, not by another add-on, not by a shell.
ENV HERMES_DASHBOARD=1

# Ownership repair, run before upstream's own init hook. See the script header
# for the full reasoning: Home Assistant restores add-on data with copytree,
# which drops ownership, and upstream only chowns targeted paths - leaving
# /data/platforms root-owned between two hermes-owned directories, which the
# supervised process cannot traverse.
COPY rootfs/etc/cont-init.d/00-fix-data-perms /etc/cont-init.d/00-fix-data-perms

# Bridges API keys from the add-on's Configuration tab into /data/.env. Sorts
# after the ownership fix (f < h) and before upstream's 01-hermes-setup, so the
# keys are in place before anything reads them. Logs names, never values.
COPY rootfs/etc/cont-init.d/00-hermes-env /etc/cont-init.d/00-hermes-env
RUN chmod +x /etc/cont-init.d/00-fix-data-perms /etc/cont-init.d/00-hermes-env

# Run the Telegram gateway rather than the interactive CLI. The upstream
# entrypoint dispatches through s6-overlay, which is why config.yaml sets
# init: false - Home Assistant must not add its own init on top, or the
# dispatcher is not PID 1 and silently skips the whole supervision tree.
CMD ["gateway", "run"]
