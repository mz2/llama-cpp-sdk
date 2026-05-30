# Contributing

Thanks for your interest in improving the llama.cpp SDK for Workshop.

## Repository layout

- **`main`** — template branch. Holds `renovate.json` and the Renovate
  workflows. It has no `VERSION` file and is not built or published.
- **`latest`, `cuda`, `rocm`, `vulkan`** — channel branches. Each holds a
  `VERSION` file (a llama.cpp build tag such as `b9432`), a channel-specific
  `sdkcraft.yaml`, and the build/upload workflows. Renovate (running from
  `main`) opens version-bump PRs against these branches.

The hooks (`hooks/`) and the server unit (`services/`) are identical across all
channel branches; only `sdkcraft.yaml`, `VERSION`, and the `upload.yml`
platform list differ.

## Making changes

1. Edit on a feature branch and open a PR against the relevant channel branch.
2. Build locally before pushing:

   ```bash
   sdkcraft try --verbose
   workshop launch --verbose
   workshop shell
   llama-cli --version
   ```

3. Changes that should apply to every channel (hooks, service, README) must be
   propagated to each channel branch.

## Versioning

`VERSION` tracks the upstream llama.cpp build number (`bNNNN`). The `latest` and
`vulkan` channels follow `ggml-org/llama.cpp` releases; the `cuda` and `rocm`
channels follow `canonical/llama.cpp-builds` releases. Renovate manages these
bumps automatically.
