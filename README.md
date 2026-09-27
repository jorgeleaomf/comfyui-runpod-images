# Imagem ComfyUI pronta para RunPod — Wan 2.2

Imagem Docker que sobe um pod **já pronto**: ComfyUI na versão certa, custom nodes instalados, modelos baixados e o workflow salvo dentro do ComfyUI. Sem `git pull`, sem "install missing nodes", sem baixar 17 GB a cada sessão.

Hoje o repositório tem **uma** família:

| Imagem | Modelo | Tamanho |
|---|---|---|
| `comfyui-wan22` | Wan 2.2 I2V (Rapid AllInOne Q6_K + LoRA Blink) | 20,60 GB comprimida |

> Por que só o Wan: os modelos do LTX e do MiniMax H3 são os mesmos que o ComfyUI baixa sozinho pelos templates oficiais, rápido (~3 min). O Wan 2.2 não — ele usa o AllInOne NSFW (Q6_K, 12,8 GB) mais uma LoRA que **não estão** no catálogo do ComfyUI, e é justamente esse download manual que a imagem elimina.

## Como usar

1. Crie um **template** no RunPod (Templates → New Template) com os campos abaixo, ou edite um pod existente para usar a imagem direto.
2. Imagem: `ghcr.io/jorgeleaomf/comfyui-wan22:latest`
3. Container disk: **60 GB** (a imagem ocupa ~22 GB descomprimida; as saídas ficam em `/workspace`).
4. Portas **HTTP**: `8188` (ComfyUI) e `8888` (JupyterLab). Opcional TCP: `22` (SSH).
5. **Sem volume persistente** — e se o console obrigar a escolher, o caminho de montagem tem de ser diferente de `/workspace` (use `/dados`). Um volume montado em `/workspace` **esconde os modelos da imagem** e o pod sobe vazio.
6. Container Start Command: em branco (o `start.sh` da base já sobe o ComfyUI).

O workflow já vem salvo em `Workflow → Open → wan22-i2v-rapid-aio`.

## Como a imagem é construída

GitHub Actions (`.github/workflows/build.yml`) constrói e publica no GHCR. Nada roda na máquina de quem mantém:

- `docker/wan22/Dockerfile` — Wan 2.2 I2V
- `workflows/wan22-i2v-rapid-aio.json` — o workflow salvo dentro da imagem

Base: `runpod/comfyui:1.3.3-comfyuiv0.30.0-cuda13.0` (traz ComfyUI 0.30.0, PyTorch 2.10.0+cu130, Python 3.12.3, ComfyUI-Manager, KJNodes, Civicomfy, RunpodDirect, JupyterLab, FileBrowser, ffmpeg, git, curl, rsync).

Por cima dela a imagem acrescenta:

1. os custom nodes que o workflow pede e que **não** vêm na base (com commit fixado);
2. os arquivos de modelo, baixados das fontes públicas — **um `RUN` por arquivo grande**, porque cada `RUN` vira uma camada e o pod puxa camadas em paralelo (com tudo num `RUN` só, virava uma camada de 16,5 GB em fila indiana);
3. o workflow `.json`, salvo em `user/default/workflows/`.

A pasta de destino reproduz o layout do template oficial (`/workspace/runpod-slim/ComfyUI`), então o `start.sh` da RunPod encontra tudo no lugar e não copia nem baixa nada.

Cada push constrói **apenas** a família cujos arquivos mudaram (a lista sai do `git diff` do commit) — mexer no Wan não rebaixa as outras.

## Versões fixadas (proveniência)

| Componente | Versão / commit |
|---|---|
| runpod/comfyui | `1.3.3-comfyuiv0.30.0-cuda13.0` |
| ComfyUI | v0.30.0 (na base) |
| ComfyUI-KJNodes | `bc8e4ce4254b` (na base) |
| ComfyUI-GGUF | `6ea2651e7df66d7585f6ffee804b20e92fb38b8a` |
| ComfyUI-VideoHelperSuite | `4d907bee61e92c2e65af3bd6383a4e4d356126d1` |

Modelos: difusor `wan2.2-i2v-rapid-aio-v10-nsfw-Q6_K` (befox), texto `umt5-xxl-encoder-Q4_K_M` (city96) + `spiece.model`, VAE `wan_2.1_vae` e `clip_vision_h` (Comfy-Org), LoRA `Blink_Doggystyle_BackView_HighNoise` (Se0ulSeeker).

## Manutenção

- **Trocar/atualizar modelo**: editar `docker/wan22/Dockerfile`, commit, e o Actions reconstrói.
- **Trocar a LoRA**: mudar a URL e o nome do arquivo no Dockerfile **e** no `workflows/wan22-i2v-rapid-aio.json` (os dois têm de casar).
- **Nova versão do ComfyUI**: trocar a tag do `FROM` (tags em `runpod/comfyui` no Docker Hub) e reconstruir.
- **Rodar o build sem mudar arquivo**: aba Actions → *construir imagens* → *Run workflow* → informar a família (`wan22` ou `todas`).
- **Visibilidade**: o pacote no GHCR herda a visibilidade do repositório — com o repositório público, o `docker pull` anônimo funciona (conferido com token anônimo na imagem publicada).
