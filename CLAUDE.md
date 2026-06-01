# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This repo mirrors `goreleaser/goreleaser:v1.2.2` to `ghcr.io/bradfordwagner/goreleaser` using Dagger-based GitHub Actions workflows.

## Architecture

**`config.yaml`** is the single source of truth for what gets built. It declares:
- `target_repo`: the destination registry (ghcr.io/bradfordwagner/goreleaser)
- `flavor`: always `mirror` for this repo type
- `builds`: list of upstream images with repo, tag, and target architectures

**GitHub Actions** (`.github/workflows/`) drives two pipelines:
- `container_branches.yml` — triggers on branch pushes, tags images as `latest`
- `container_tags.yml` — triggers on git tags, tags images with the git ref name

Both workflows follow a 3-job pattern: `product` (matrix generation via Dagger) → `builds` (parallel per-matrix-entry builds via Dagger) → `manifest` (multi-arch manifest push via Dagger).

**Dagger module** used: `https://github.com/bradfordwagner/dagger-container-builds@0.2.0`
- `product-json --src=. --version=<ver>` — generates the build matrix from config.yaml
- `build --src=. --index=<i> --version=<ver>` — builds and pushes a single image
- `manifest --src=. --version=<ver> --actor=... --token=...` — pushes multi-arch manifests

Dagger version pinned at **0.15.2**, installed via the install script in each job.

## Making Changes

To add or update mirrored tags, edit `config.yaml`. Each entry under `builds` maps to one matrix job in CI. The `repo_override` field overrides the source registry path when non-empty.

To test locally with Dagger (after installing dagger 0.15.2 to `./bin/dagger`):
```sh
./bin/dagger -m https://github.com/bradfordwagner/dagger-container-builds@0.2.0 call product-json --src=. --version=latest
```
