# =============================================================================
# ComfyUI pronto para RunPod - Wan 2.2 I2V
#
# Base : runpod/comfyui:1.3.3-comfyuiv0.30.0-cuda13.0
#        (ComfyUI 0.30.0, PyTorch 2.10.0+cu130, Python 3.12.3, JupyterLab,
#         ComfyUI-Manager, KJNodes, Civicomfy, RunpodDirect - tudo ja na imagem)
# Extra: ComfyUI-GGUF + ComfyUI-VideoHelperSuite (commits fixados) e os arquivos
#        do Wan 2.2 (~17,8 GB) + o workflow salvo dentro do ComfyUI.
#
# Como o boot funciona (start.sh oficial da RunPod):
#   - se /workspace/runpod-slim/ComfyUI nao existe -> copia /opt/comfyui-baked
#   - cria a venv (--system-site-packages, sem rede) e sobe o ComfyUI na 8188.
# Como esta imagem JA traz /workspace/runpod-slim/ComfyUI pronto (com modelos),
# o boot pula a copia e sobe direto. A rotina de upgrade da RunPod faz rsync do
# core EXCLUINDO /models, /user, /custom_nodes, /input e /output: os modelos
# sobrevivem a qualquer atualizacao futura da base.
#
# ORDEM IMPORTA: a copia de /opt/comfyui-baked para o workspace vem ANTES de
# gravar os modelos, senao o `cp -r` seguinte copiaria a arvore para dentro de
# uma pasta ja existente (ComfyUI/comfyui-baked) e perderia os modelos.
#
# NAO anexar network volume em /workspace: um volume montado ali esconde a
# pasta de modelos que veio na imagem.
# =============================================================================
FROM runpod/comfyui:1.3.3-comfyuiv0.30.0-cuda13.0

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

WORKDIR /workspace/runpod-slim

# ---- commits fixados dos nodes que a base NAO traz -------------------------
ARG GGUF_SHA=6ea2651e7df66d7585f6ffee804b20e92fb38b8a
ARG VHS_SHA=4d907bee61e92c2e65af3bd6383a4e4d356126d1

LABEL org.opencontainers.image.title="comfyui-wan22"
LABEL org.opencontainers.image.description="ComfyUI 0.30.0 + Wan 2.2 I2V (Rapid AllInOne Q6_K + LoRA) para RunPod"
LABEL br.hermes.base="runpod/comfyui:1.3.3-comfyuiv0.30.0-cuda13.0"
LABEL br.hermes.comfyui-gguf="${GGUF_SHA}"
LABEL br.hermes.videohelpersuite="${VHS_SHA}"

# ---- 1) custom nodes que o workflow usa e a base nao tem -------------------
# Vao para /opt/comfyui-baked (a origem que o start.sh da RunPod copia) e as
# dependencias entram no python do SISTEMA, porque a venv do pod roda com
# --system-site-packages: assim o pod nao baixa nada da rede ao subir.
RUN set -eux; \
    cd /opt/comfyui-baked/custom_nodes; \
    git clone --filter=blob:none https://github.com/city96/ComfyUI-GGUF.git ComfyUI-GGUF; \
    git -C ComfyUI-GGUF checkout "${GGUF_SHA}"; \
    git clone --filter=blob:none https://github.com/Kosinkadink/ComfyUI-VideoHelperSuite.git ComfyUI-VideoHelperSuite; \
    git -C ComfyUI-VideoHelperSuite checkout "${VHS_SHA}"; \
    python3.12 -m pip install --no-cache-dir \
        -r /opt/comfyui-baked/custom_nodes/ComfyUI-GGUF/requirements.txt \
        -r /opt/comfyui-baked/custom_nodes/ComfyUI-VideoHelperSuite/requirements.txt; \
    rm -rf /opt/comfyui-baked/custom_nodes/*/.git

# ---- 2) a arvore pronta do ComfyUI dentro do workspace ---------------------
# (o alvo nao existe ainda, entao o cp cria /workspace/runpod-slim/ComfyUI)
RUN set -eux; \
    mkdir -p /workspace/runpod-slim; \
    cp -r /opt/comfyui-baked /workspace/runpod-slim/ComfyUI; \
    mkdir -p /workspace/runpod-slim/ComfyUI/user/default/workflows \
             /workspace/runpod-slim/ComfyUI/input \
             /workspace/runpod-slim/ComfyUI/output \
             /workspace/runpod-slim/ComfyUI/temp; \
    test -f /workspace/runpod-slim/ComfyUI/main.py

# ---- 3) modelos do Wan 2.2 -------------------------------------------------
# O lugar do GGUF importa: unet_gguf -> models/unet e clip_gguf -> models/clip
# (foi assim no pod de referencia e e assim que o build carrega).
RUN set -eux; \
    cd /workspace/runpod-slim/ComfyUI/models; \
    mkdir -p unet clip vae loras clip_vision; \
    curl -fL --retry 5 --retry-delay 5 -o unet/wan2.2-i2v-rapid-aio-v10-nsfw-Q6_K.gguf \
        "https://huggingface.co/befox/WAN2.2-14B-Rapid-AllInOne-GGUF/resolve/main/v10/wan2.2-i2v-rapid-aio-v10-nsfw-Q6_K.gguf"; \
    curl -fL --retry 5 --retry-delay 5 -o clip/umt5-xxl-encoder-Q4_K_M.gguf \
        "https://huggingface.co/city96/umt5-xxl-encoder-gguf/resolve/main/umt5-xxl-encoder-Q4_K_M.gguf"; \
    curl -fL --retry 5 --retry-delay 5 -o clip/spiece.model \
        "https://huggingface.co/google/umt5-xxl/resolve/main/spiece.model"; \
    curl -fL --retry 5 --retry-delay 5 -o vae/wan_2.1_vae.safetensors \
        "https://huggingface.co/Comfy-Org/Wan_2.1_ComfyUI_repackaged/resolve/main/split_files/vae/wan_2.1_vae.safetensors"; \
    curl -fL --retry 5 --retry-delay 5 -o clip_vision/clip_vision_h.safetensors \
        "https://huggingface.co/Comfy-Org/Wan_2.1_ComfyUI_repackaged/resolve/main/split_files/clip_vision/clip_vision_h.safetensors"; \
    curl -fL --retry 5 --retry-delay 5 -o loras/Blink_Doggystyle_BackView_HighNoise.safetensors \
        "https://huggingface.co/Se0ulSeeker/wan_2.2_i2v_nsfw_loras/resolve/main/loras/Blink_Doggystyle_BackView_HighNoise.safetensors"

# ---- 4) o workflow salvo ---------------------------------------------------
COPY workflows/wan22-i2v-rapid-aio.json /workspace/runpod-slim/ComfyUI/user/default/workflows/wan22-i2v-rapid-aio.json

# ---- 5) portao: nada sai daqui sem os arquivos no lugar e com tamanho ------
RUN set -eux; \
    C=/workspace/runpod-slim/ComfyUI; \
    echo "== custom nodes =="; ls -1 "$C/custom_nodes"; \
    echo "== modelos =="; du -sh "$C/models"; \
    for f in \
      "$C/models/unet/wan2.2-i2v-rapid-aio-v10-nsfw-Q6_K.gguf" \
      "$C/models/clip/umt5-xxl-encoder-Q4_K_M.gguf" \
      "$C/models/clip/spiece.model" \
      "$C/models/vae/wan_2.1_vae.safetensors" \
      "$C/models/clip_vision/clip_vision_h.safetensors" \
      "$C/models/loras/Blink_Doggystyle_BackView_HighNoise.safetensors" \
      "$C/user/default/workflows/wan22-i2v-rapid-aio.json" \
      "$C/main.py" ; do \
        if [ -s "$f" ]; then echo "OK   $(du -h "$f" | cut -f1)  $f"; else echo "FALTA $f"; exit 1; fi; \
    done; \
    python3.12 -c "import gguf, imageio_ffmpeg; print('dependencias dos nodes OK')"

# Em setembro de 2026 o GGUF tem ~12,8 GB e o total de modelos ~17,8 GB; o build
# falha se algum curl vier truncado (o portao acima pega arquivo vazio).
#
# O ENTRYPOINT (/start.sh), as portas (8188/8888/8080/22) e o WORKDIR
# (/workspace/runpod-slim) vem da base e nao devem ser alterados.
