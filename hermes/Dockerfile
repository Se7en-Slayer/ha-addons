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

# Run the Telegram gateway rather than the interactive CLI. The upstream
# entrypoint dispatches through s6-overlay, which is why config.yaml sets
# init: false - Home Assistant must not add its own init on top.
CMD ["gateway", "run"]
