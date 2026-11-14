# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This repo mirrors `goreleaser/goreleaser:v1.2.2` to `ghcr.io/bradfordwagner/goreleaser` using GitHub Actions workflows from gh-template-mirror-image.

## Architecture

**`config.yaml`** is the single source of truth for what gets built. It declares:
- `target_repo`: the destination registry (ghcr.io/bradfordwagner/goreleaser)
- `flavor`: always `mirror` for this repo type
- `builds`: list of upstream images with repo, tag, and target architectures

**GitHub Actions** (`.github/workflows/`) drives two pipelines:
- `container_branches.yml` — triggers on branch pushes, tags images as `latest`
- `container_tags.yml` — triggers on git tags, tags images with the git ref name

Both are a single `container` job calling `bradfordwagner/gh-template-lib/.github/workflows/mirror.yml` (product matrix, per-arch `docker buildx` builds, multi-arch manifest).

## Making Changes

To add or update mirrored tags, edit `config.yaml`. Each entry under `builds` maps to one matrix job in CI. The `repo_override` field overrides the source registry path when non-empty.
