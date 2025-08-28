# DRAWIO Exporter

A minimalist Docker image for batch-exporting draw.io diagrams (.drawio, .drawio.xml) to SVG and PNG without a GUI. When the container starts, it scans the `/data` directory and, for each page of each diagram, produces two files:

- name-pageN.svg
- name-pageN.png (150 DPI)

It works offline (network calls and auto-updates are disabled) and uses headless rendering via Xvfb and rsvg-convert. Handy for CI/CD and local batch conversion.

## What's inside

- Debian Bullseye
- draw.io Desktop 28.0.6 (CLI export)
- Xvfb for headless mode
- librsvg (rsvg-convert) to convert SVG → PNG
- A font set (DejaVu, Noto, Liberation)

## Supported inputs and outputs

- Input: files with `.drawio` and `.drawio.xml` extensions in the `/data` directory (the mount root).
- Output: for each page it creates `*-pageN.svg` and `*-pageN.png` next to the source file.
- Page indexing: N starts at 1 (page1, page2, …).

## Run and mount data

Place your `.drawio` files into the repository's `data/` folder and run the container by mounting it to `/data` (Linux):

```bash
docker run --rm -v "$(pwd)/data:/data" ghcr.io/rootshell-coder/drawio-exporter:latest
```

Notes:

- If the path contains spaces, keep the quotes as shown above.
- You can mount any other folder with diagrams: `-v "/path/to/diagrams:/data"`.

### Podman

If you're using Podman on Linux, the invocation is similar:

```bash
podman run --rm -v "$(pwd)/data:/data" ghcr.io/rootshell-coder/drawio-exporter:latest
```

On SELinux-enabled systems (e.g., Fedora/RHEL), add `:Z` to the volume to relabel the mount for the container context:

```bash
podman run --rm -v "$(pwd)/data:/data:Z" ghcr.io/rootshell-coder/drawio-exporter:latest
```

## gitlab-ci.yml

```yaml
stages:
  - export

services:
  - name: docker:dind
    alias: docker

variables:
  DOCKER_HOST: tcp://docker:2376
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: "/certs"

export:all:
  stage: export
  tags:
    - runner
  image:
    name: ghcr.io/rootshell-coder/drawio-exporter:latest
    entrypoint: [""]
  rules:
    - exists:
        - "*.drawio"
        - "*.drawio.xml"
  script:
    - cd "$CI_PROJECT_DIR"
    - /usr/local/bin/entrypoint
  artifacts:
    when: always
    expire_in: 1 day
    paths:
      - "*.svg"
      - "*.png"
```

## Example

Source: `data/test diagram.drawio`

Output (in the same `data` folder):

- `test diagram-page1.svg`
- `test diagram-page1.png`
- `test diagram-page2.svg`
- `test diagram-page2.png`

The number of pages is determined by `<diagram>` tags inside the file; if it's a single-page diagram, only `page1` will be created.

## Limitations and notes

- PNG is generated at 150 DPI. To change DPI, adjust `rsvg-convert` options in the `docker/entrypoint` script.
- Background network requests and Electron auto-updates are disabled for reproducibility.
- Export happens in two stages: first SVG (via draw.io), then PNG (via librsvg). The SVG is kept alongside the PNG and can be used directly.

## Licenses

draw.io (diagrams.net) — their license; Debian packages — their licenses. This Dockerfile/script is provided as-is.
