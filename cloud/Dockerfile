# syntax=docker/dockerfile:1.7
#
# Edrin Travel — Coolify build wrapper for OpenClaw.
#
# Coolify invokes this Dockerfile with the repo root as build context.
# It reuses the upstream image as-is (no modifications to its Dockerfile)
# and layers our baked-in agents/skills on top.
#
# The upstream image is built by referencing it via a BUILDKIT stage that
# copies the upstream Dockerfile into a temporary builder. Since Coolify's
# buildpack only accepts one Dockerfile, we inline the upstream's final
# stage reference by asking Coolify to build the upstream Dockerfile first
# (base_directory: /, dockerfile_location: /Dockerfile) and produce an
# image tagged "openclaw-base", then this Dockerfile extends it.
#
# In practice Coolify builds a single Dockerfile, so we pick ONE of two
# strategies depending on which is more ergonomic:
#
#   (A) Point Coolify at the upstream /Dockerfile and accept that our
#       custom config is baked in via a RUN step appended at the end of
#       the upstream Dockerfile itself — minimally invasive, see the last
#       COPY line in ../Dockerfile.
#
#   (B) Use this cloud/Dockerfile and tell Coolify base_directory: /,
#       dockerfile_location: /cloud/Dockerfile. In that case the BASE
#       below must be buildable independently; easiest route is to push
#       an `openclaw-base` image to a registry from CI. Not set up yet.
#
# Default today: strategy (A). This file is a placeholder for strategy (B)
# when we need per-env variants of the baked config.
ARG BASE=openclaw-base:latest
FROM ${BASE}

COPY cloud/config/ /home/node/.openclaw/

# Preserve upstream CMD / ENTRYPOINT / HEALTHCHECK. Nothing to override.
