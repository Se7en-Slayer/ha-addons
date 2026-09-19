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

# Ownership repair, run before upstream's own init hook. See the script header
# for the full reasoning: Home Assistant restores add-on data with copytree,
# which drops ownership, and upstream only chowns targeted paths - leaving
# /data/platforms root-owned between two hermes-owned directories, which the
# supervised process cannot traverse.
COPY rootfs/etc/cont-init.d/00-fix-data-perms /etc/cont-init.d/00-fix-data-perms
RUN chmod +x /etc/cont-init.d/00-fix-data-perms

# Run the Telegram gateway rather than the interactive CLI. The upstream
# entrypoint dispatches through s6-overlay, which is why config.yaml sets
# init: false - Home Assistant must not add its own init on top, or the
# dispatcher is not PID 1 and silently skips the whole supervision tree.
CMD ["gateway", "run"]
