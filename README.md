# llama.cpp SDK for Workshop

llama.cpp provides tooling for large language model inference in C/C++: the
`llama-server` OpenAI-compatible HTTP server, `llama-cli`, `llama-bench`, and
related utilities. The SDK ships the llama.cpp binaries on `PATH`, persists
downloaded models and server configuration across workshop updates, and
provides an opt-in `llama-server` service on port 8080. Separate channels carry
backend-specific builds.

---

## Reference workshop

A minimal workshop:

```yaml
# workshop.yaml
name: llama-app
base: ubuntu@24.04
sdks:
  - name: llama-cpp-mz2
    channel: latest/stable

actions:
  serve: |
    llama-server -hf "$@"
  chat: |
    llama-cli -hf "$@"
  bench: |
    llama-bench "$@"
```

This demonstrates running a model from the command line with persistent model
storage. Replace `latest/stable` with `cuda/stable`, `rocm/stable`, or
`vulkan/stable` for GPU acceleration.

### Available channels

| Channel | Backend | Source | Platforms |
|---|---|---|---|
| `latest/stable` | CPU | upstream ggml-org/llama.cpp | ubuntu@22.04, ubuntu@24.04 — amd64, arm64 |
| `cuda/stable` | NVIDIA CUDA 12 | canonical/llama.cpp-builds | ubuntu@24.04 — amd64, arm64 |
| `rocm/stable` | AMD ROCm 7.2 | canonical/llama.cpp-builds | ubuntu@24.04 — amd64 |
| `vulkan/stable` | Vulkan (cross-vendor) | upstream ggml-org/llama.cpp | ubuntu@22.04, ubuntu@24.04 — amd64, arm64 |

---

## Using the SDK

### Prerequisites, project layout

1. The `cuda` channel requires the `cuda-toolkit` SDK in the same workshop
   (for example `channel: 12.9/stable`); it supplies the CUDA runtime under
   `/usr/local/cuda`. The other channels have no prerequisite SDKs.
2. No project files are required. Models can be pulled from Hugging Face with
   `-hf`, or mounted into the workshop and referenced by path.
3. On launch, the SDK adds the llama.cpp tools to `PATH`, prepares the model
   cache, and installs (but does not start) the `llama-server` service.

A GPU-accelerated CUDA workshop combines the two SDKs:

```yaml
# workshop.yaml
name: llama-cuda
base: ubuntu@24.04
sdks:
  - name: cuda-toolkit
    channel: 12.9/stable
  - name: llama-cpp-mz2
    channel: cuda/stable
    plugs:
      gpu: {}
```

### Run a model

```bash
workshop shell
# Download from Hugging Face and chat (cached under ~/.cache/llama.cpp):
llama-cli -hf ggml-org/gemma-3-1b-it-GGUF
# Offload layers to the GPU on the cuda/rocm/vulkan channels:
llama-cli -hf ggml-org/gemma-3-1b-it-GGUF -ngl 99
```

Downloaded models are stored in `~/.cache/llama.cpp`, mapped to the host via the
`models` mount plug, so subsequent workshop updates reuse them.

### Run the server

The `llama-server` service is opt-in. Configure a model, then it starts on the
next refresh (or enable it directly):

```bash
workshop shell
# Set a model (and any flags) for the server:
echo 'LLAMA_SERVER_ARGS="-hf ggml-org/gemma-3-1b-it-GGUF -ngl 99"' \
  > ~/.config/llama-cpp/server.env
systemctl --user enable --now llama-server
```

The server exposes an OpenAI-compatible API on port 8080. Other SDKs or host
tools can reach it via the `llama-server` tunnel slot:

```bash
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"messages":[{"role":"user","content":"Hello"}]}'
```

### Verify from the command line

```bash
workshop shell
llama-cli --version
workshop info   # shows the SDK health status
```

---

## Plugs (resources this SDK consumes)

### `gpu`

- Interface: `gpu`
- Present on: `cuda`, `rocm`, `vulkan` channels (not `latest`).
- Purpose: Grants access to GPU hardware on the host for accelerated inference.

### `models`

- Interface: `mount`
- Workshop target: `/home/workshop/.cache/llama.cpp`
- Purpose: Persists models downloaded with `-hf` between workshop updates.

### `config`

- Interface: `mount`
- Workshop target: `/home/workshop/.config/llama-cpp`
- Purpose: Persists the `llama-server` configuration (`server.env`) so the
  opt-in server keeps its settings across workshop updates.

## Slots (resources this SDK provides)

### `llama-server`

- Interface: `tunnel`
- Endpoint: `8080`
- Purpose: Exposes the `llama-server` OpenAI-compatible API for use by other
  SDKs or host tools, once the server has been enabled.

---

## Documentation and guidance

- [llama.cpp repository](https://github.com/ggml-org/llama.cpp)
- [llama-server documentation](https://github.com/ggml-org/llama.cpp/blob/master/tools/server/README.md)
- [Workshop documentation](https://ubuntu.com/workshop/docs/)

---

## Community and support

- llama.cpp community:
  [GitHub Discussions](https://github.com/ggml-org/llama.cpp/discussions)
- Workshop forum:
  [Discourse](https://discourse.ubuntu.com/)
- Please review our
  [Code of Conduct](https://ubuntu.com/community/ethos/code-of-conduct) before
  participating.

---

## Contributions

All contributions, including code, documentation updates, and issue reports,
are welcome!

- See `CONTRIBUTING.md` for guidelines.
- Open issues or pull requests on the official repository.

---

## License and copyright

Copyright 2026 Canonical Ltd.

This SDK is licensed under the
[MIT License](https://opensource.org/licenses/MIT), the same license as
[llama.cpp](https://github.com/ggml-org/llama.cpp/blob/master/LICENSE).
