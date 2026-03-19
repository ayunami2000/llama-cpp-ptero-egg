# llama-cpp-ptero

Wings-compatible Docker images for running [llama.cpp](https://github.com/ggml-org/llama.cpp) `llama-server` on [Pterodactyl Panel](https://pterodactyl.io/).

The upstream `ghcr.io/ggml-org/llama.cpp` images have an `ENTRYPOINT` baked in which prevents Pterodactyl Wings from controlling the startup command. These images are identical to upstream but with `ENTRYPOINT` and `CMD` cleared.

## Images

| Tag | Base | GPU |
|-----|------|-----|
| `cuda` | `ghcr.io/ggml-org/llama.cpp:server-cuda` | ✅ CUDA |
| `cpu` | `ghcr.io/ggml-org/llama.cpp:server` | ❌ CPU only |

Images rebuild automatically every day — only if the upstream digest has changed.

## Setup

### 1. Create the repo

Create a new GitHub repo (e.g. `llama-cpp-ptero`) and push these files to `main`.

### 2. Let Actions run

The workflow triggers on push to `main`, so the images will build and appear at:
```
ghcr.io/YOURGITHUBUSER/llama-cpp-ptero:cuda
ghcr.io/YOURGITHUBUSER/llama-cpp-ptero:cpu
```

### 3. Make the package public

GitHub profile → **Packages** → `llama-cpp-ptero` → **Package Settings** → Change visibility → **Public**

This lets Wings pull without any extra auth.

### 4. Import the egg

Edit `egg/egg-llama-cpp.json` and replace `YOURGITHUBUSER` with your GitHub username, then in Pterodactyl:

**Admin Panel → Nests → (your nest) → Import Egg**

### 5. Wings node — CUDA support

Install the nvidia container toolkit on the Wings host and set it as the default Docker runtime.

**Install toolkit:**
```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey \
  | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list \
  | sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit
```

**`/etc/docker/daemon.json`:**
```json
{
  "default-runtime": "nvidia",
  "runtimes": {
    "nvidia": {
      "path": "nvidia-container-runtime",
      "runtimeArgs": []
    }
  }
}
```

```bash
sudo systemctl restart docker
```

### 6. Create a server

- **Docker image:** `ghcr.io/YOURGITHUBUSER/llama-cpp-ptero:cuda`
- **MODEL_FILE:** e.g. `mistral-7b.Q4_K_M.gguf`
- **MODEL_URL:** direct HuggingFace link, or `none` to upload via SFTP
- **GPU_LAYERS:** `999` for full GPU offload, `0` for CPU-only

## Manual rebuild

Actions tab → **Build & Push** → **Run workflow**
