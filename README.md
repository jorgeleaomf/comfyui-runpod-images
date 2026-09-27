# Imagens ComfyUI prontas para RunPod

Imagens Docker que sobem um pod **já pronto**: ComfyUI na versão certa, custom nodes instalados, modelos baixados e o workflow salvo dentro do ComfyUI. Sem `git pull`, sem "install missing nodes", sem download de 17 GB a cada sessão.

Cada família de modelo tem a sua imagem, todas partindo da mesma base oficial da RunPod:

| Imagem | Modelo | Tamanho aprox. |
|---|---|---|
| `comfyui-wan22` | Wan 2.2 I2V (Rapid AllInOne Q6_K + LoRA) | ~22 GB |
| `comfyui-minimax-h3` | MiniMax H3 (fast/vídeo+áudio) | a montar |
| `comfyui-ltx23` | LTX 2.3 | a montar |
| `comfyui-ltx25` | LTX 2.5 | a montar |

## Como usar

1. No RunPod, ao criar o pod: **Custom Image** →
   `ghcr.io/<sua-conta>/comfyui-wan22:latest`
2. Container disk: **≥ 60 GB** (a imagem ocupa ~22 GB e as saídas ficam em `/workspace`).
3. Portas expostas: **8188** (ComfyUI), 8888 (JupyterLab), 8080 (FileBrowser), 22 (SSH).
4. **Não** anexe network volume em `/workspace` — os modelos ficam na própria imagem; um volume montado em `/workspace` esconde a pasta e o pod sobe sem modelos.

O workflow já vem salvo em `Workflow → Open → wan22-i2v-rapid-aio`.

## Como as imagens são construídas

GitHub Actions (`.github/workflows/build.yml`) constrói e publica no GHCR. Nada roda na sua máquina:

- `docker/wan22/Dockerfile` — Wan 2.2 I2V
- `docker/<familia>/Dockerfile` — as demais famílias

Base: `runpod/comfyui:1.3.3-comfyuiv0.30.0-cuda13.0` (traz ComfyUI 0.30.0, PyTorch 2.10.0+cu130, Python 3.12.3, ComfyUI-Manager, KJNodes, Civicomfy, RunpodDirect, JupyterLab e FileBrowser).

Por cima dela cada imagem acrescenta apenas:

1. os custom nodes que o workflow pede e que **não** vêm na base (com commit fixado);
2. os arquivos de modelo, baixados das fontes públicas;
3. o workflow `.json`, salvo em `user/default/workflows/`.

A pasta de destino dos modelos reproduz exatamente o layout do template oficial (`/workspace/runpod-slim/ComfyUI`), então o `start.sh` da RunPod encontra tudo no lugar e não copia nem baixa nada.

## Versões fixadas (proveniência)

| Componente | Versão / commit |
|---|---|
| runpod/comfyui | `1.3.3-comfyuiv0.30.0-cuda13.0` |
| ComfyUI | v0.30.0 (na base) |
| ComfyUI-KJNodes | `bc8e4ce4254b` (na base) |
| ComfyUI-GGUF | `6ea2651e7df66d7585f6ffee804b20e92fb38b8a` |
| ComfyUI-VideoHelperSuite | `4d907bee61e92c2e65af3bd6383a4e4d356126d1` |

## Manutenção

- **Trocar/atualizar modelo**: editar o Dockerfile da família, commit, e o Actions reconstrói.
- **Nova versão do ComfyUI**: trocar a tag da base no `FROM` (as tags ficam em `runpod/comfyui` no Docker Hub) e reconstruir.
- **Tornar público**: cada pacote publicado no GHCR nasce privado; basta marcá-lo como público em *Package settings* (uma vez por imagem).
