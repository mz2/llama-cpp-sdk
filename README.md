# llama.cpp SDK for Workshop

llama.cpp runs large language model inference in C/C++. This SDK:

- puts the llama.cpp tools (`llama-server`, `llama-cli`, `llama-bench`, and
  related utilities) on `PATH`;
- persists downloaded models and server configuration across workshop updates;
- provides an opt-in, OpenAI-compatible `llama-server` on port 8080.

Each backend (CPU, CUDA, ROCm, Vulkan) is a separate channel.

---

## Reference workshop

```yaml
# workshop.yaml
name: llama-app
base: ubuntu@24.04
sdks:
  - name: llama-cpp-sdk
    channel: latest/stable

actions:
  serve: |
    llama-server -hf "$@"
  chat: |
    llama-cli -hf "$@"
  bench: |
    llama-bench "$@"
```

This runs a model from the command line with persistent model storage. For GPU
acceleration, use a GPU channel in place of `latest/stable` (see
[Channels](#channels)).

### Channels

| Channel | Backend | Platforms |
|---|---|---|
| `latest/stable` | CPU | ubuntu@22.04, ubuntu@24.04 (amd64, arm64) |
| `latest/stable/cuda` | NVIDIA CUDA 12 | ubuntu@24.04 (amd64) |
| `latest/stable/rocm` | AMD ROCm 7.2 | ubuntu@24.04 (amd64) |
| `latest/stable/vulkan` | Vulkan (cross-vendor) | ubuntu@22.04, ubuntu@24.04 (amd64, arm64) |

The GPU backends are published as *branches* under `latest/stable`
(`latest/stable/cuda`, and so on). This is temporary, until dedicated tracks
(`cuda/stable`, `rocm/stable`, `vulkan/stable`) are approved for this SDK. The
channels will move to that form once the tracks exist.

---

## Using the SDK

### Prerequisites

Most channels have no prerequisites.

The `cuda` channel is the exception. It expects the CUDA runtime
(`libcudart`, `libcublas`) under `/usr/local/cuda`, supplied by the
`cuda-toolkit` SDK in the same workshop. A CUDA workshop therefore combines the
two SDKs:

```yaml
# workshop.yaml
name: llama-cuda
base: ubuntu@24.04
sdks:
  - name: cuda-toolkit
    channel: 12.9/stable
  - name: llama-cpp-sdk
    channel: latest/stable/cuda
    plugs:
      gpu: {}
```

### Run a model

Models can be pulled from Hugging Face with `-hf`, or mounted into the workshop
and referenced by path.

```bash
workshop shell
# Pull from Hugging Face and chat (cached under ~/.cache/llama.cpp):
llama-cli -hf ggml-org/gemma-3-1b-it-GGUF
# On a GPU channel, offload layers to the GPU:
llama-cli -hf ggml-org/gemma-3-1b-it-GGUF -ngl 99
```

Downloaded models live in `~/.cache/llama.cpp` (the `models` mount), so they are
reused across workshop updates.

### Run the server

The `llama-server` service is installed but off by default. Set a model, and it
starts on the next refresh (or enable it directly):

```bash
workshop shell
echo 'LLAMA_SERVER_ARGS="-hf ggml-org/gemma-3-1b-it-GGUF -ngl 99"' \
  > ~/.config/llama-cpp/server.env
systemctl --user enable --now llama-server
```

It serves an OpenAI-compatible API on port 8080, reachable through the
`llama-server` tunnel slot:

```bash
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"messages":[{"role":"user","content":"Hello"}]}'
```

### Verify

```bash
workshop shell
llama-cli --version
workshop info   # shows the SDK health status
```

---

## Plugs (resources this SDK consumes)

### `gpu`

- Interface: `gpu`
- Channels: `cuda`, `rocm`, `vulkan` (not `latest`)
- Purpose: GPU access on the host for accelerated inference.

### `models`

- Interface: `mount`
- Workshop target: `/home/workshop/.cache/llama.cpp`
- Purpose: Persists models downloaded with `-hf` across workshop updates.

### `config`

- Interface: `mount`
- Workshop target: `/home/workshop/.config/llama-cpp`
- Purpose: Persists the `llama-server` configuration (`server.env`).

## Slots (resources this SDK provides)

### `llama-server`

- Interface: `tunnel`
- Endpoint: `8080`
- Purpose: Exposes the `llama-server` API once the server is enabled.

---

## Documentation and guidance

- [llama.cpp repository](https://github.com/ggml-org/llama.cpp)
- [llama-server documentation](https://github.com/ggml-org/llama.cpp/blob/master/tools/server/README.md)
- [Workshop documentation](https://documentation.ubuntu.com/canonical-workshop/latest/)

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
