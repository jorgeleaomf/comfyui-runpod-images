# =============================================================================
# ComfyUI pronto para RunPod - MiniMax H3 (FastVideo FastH3, 8 passos)
#
# Base : runpod/comfyui:1.3.3-comfyuiv0.30.0-cuda13.0
# Extra: ComfyUI ATUALIZADO para a v0.37.4 (a versao que o pod de referencia usa
#        depois do "Update ComfyUI" do Manager) + ComfyUI-GGUF + VideoHelperSuite
#        (mesmos commits das outras imagens) + os modelos do H3 (40,8 GB) + o
#        workflow testado.
#
# POR QUE ATUALIZAR O COMFYUI AQUI: o template do H3 usa nos NATIVOS que nao
# existem na 0.30.0 da base - BlockSparseAttention, ModelAttentionBackend,
# MiniMaxH3ImageToVideo, MiniMaxH3SigmaShift. Conferido no pod: os modulos de
# custom node continuam sendo so KJNodes/VideoHelperSuite/GGUF, ou seja, TODO o
# resto vem do proprio ComfyUI. Tag v0.37.4 = commit 8ff6dc384ba5.
#
# Modelos (conferidos em 27/09/2026):
#   diffusion_models/fastvideo_fasth3_8step_v2_pruned_int8_convrot  20,61 GB
#   text_encoders/qwen3vl_32b_minimax_h3_nvfp4_awq ................. 14,61 GB
#   vae/minimax_h3_video_vae_fp16 ..................................  4,85 GB
#   vae/minimax_h3_audio_vae_fp32 ..................................  0,56 GB
#   loras/H3-M3_Unlocked_V2.1 (CivitAI 3345855, sha256 52cb409b...) .. 0,16 GB
# NAO entram (variantes do pacote oficial que este workflow nao usa): difusor
# w4a8 11,68 GB, ref2va 10,96 GB, video VAE int8 2,62 GB.
#
# A LoRA entra com DOIS nomes (arquivo + link simbolico): o workflow testado
# espera "H3-M3_Unlocked_V2.1.safetensors" no widget promovido do subgrafo e o
# arquivo do CivitAI se chama "M3_Unlocked_V2.1.safetensors".
#
# ORDEM IMPORTA: copiar /opt/comfyui-baked para o workspace vem ANTES de gravar
# os modelos, senao o cp -r seguinte copiaria a arvore para dentro de uma pasta
# ja existente (ComfyUI/comfyui-baked) e perderia os modelos.
#
# NAO anexar network volume em /workspace (esconderia os modelos da imagem).
# =============================================================================
FROM runpod/comfyui:1.3.3-comfyuiv0.30.0-cuda13.0

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

WORKDIR /workspace/runpod-slim

ARG COMFYUI_TAG=v0.37.4
ARG GGUF_SHA=6ea2651e7df66d7585f6ffee804b20e92fb38b8a
ARG VHS_SHA=4d907bee61e92c2e65af3bd6383a4e4d356126d1

LABEL org.opencontainers.image.title="comfyui-h3"
LABEL org.opencontainers.image.description="ComfyUI 0.37.4 + MiniMax H3 FastVideo (8 passos int8) para RunPod"
LABEL br.hermes.base="runpod/comfyui:1.3.3-comfyuiv0.30.0-cuda13.0"
LABEL br.hermes.comfyui-tag="${COMFYUI_TAG}"
LABEL br.hermes.comfyui-gguf="${GGUF_SHA}"
LABEL br.hermes.videohelpersuite="${VHS_SHA}"

# ---- 1) atualizar o ComfyUI para a versao do pod (v0.37.4) -----------------
ARG COMFYUI_SHA=8ff6dc384ba5
RUN set -eux; \
    cd /opt/comfyui-baked; \
    git init -q . 2>/dev/null || true; \
    git remote remove origin 2>/dev/null || true; \
    git remote add origin https://github.com/comfyanonymous/ComfyUI.git; \
    git fetch -q --depth 1 origin tag "${COMFYUI_TAG}" || git fetch -q --depth 1 origin "${COMFYUI_SHA}"; \
    git checkout -f -q "${COMFYUI_TAG}" 2>/dev/null || git checkout -f -q FETCH_HEAD; \
    echo "== versao instalada =="; git log -1 --format='%H %cd' --date=short; \
    grep -o "[0-9][0-9.]*" comfyui_version.py 2>/dev/null | head -1 || true; \
    python3.12 -m pip install --no-cache-dir -r requirements.txt

# ---- 2) custom nodes extras (mesma dupla das outras imagens) ---------------
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

# ---- 3) arvore pronta do ComfyUI no workspace -----------------------------
RUN set -eux; \
    mkdir -p /workspace/runpod-slim; \
    cp -r /opt/comfyui-baked /workspace/runpod-slim/ComfyUI; \
    mkdir -p /workspace/runpod-slim/ComfyUI/user/default/workflows \
             /workspace/runpod-slim/ComfyUI/input \
             /workspace/runpod-slim/ComfyUI/output \
             /workspace/runpod-slim/ComfyUI/temp; \
    test -f /workspace/runpod-slim/ComfyUI/main.py

# ---- 4) text encoder Qwen3-VL 32B nvfp4 (14,61 GB) - camada propria --------
RUN set -eux; \
    mkdir -p /workspace/runpod-slim/ComfyUI/models/text_encoders; \
    curl -fL --retry 5 --retry-delay 5 \
      -o /workspace/runpod-slim/ComfyUI/models/text_encoders/qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors \
      "https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/text_encoders/qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors"

# ---- 5) as duas VAEs (video fp16 + audio fp32) - camada propria ------------
RUN set -eux; \
    mkdir -p /workspace/runpod-slim/ComfyUI/models/vae; \
    cd /workspace/runpod-slim/ComfyUI/models/vae; \
    curl -fL --retry 5 --retry-delay 5 -o minimax_h3_video_vae_fp16.safetensors \
      "https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/vae/minimax_h3_video_vae_fp16.safetensors"; \
    curl -fL --retry 5 --retry-delay 5 -o minimax_h3_audio_vae_fp32.safetensors \
      "https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/vae/minimax_h3_audio_vae_fp32.safetensors"

# ---- 6) o difusor FastVideo 8 passos (20,61 GB) - sozinho na camada --------
RUN set -eux; \
    mkdir -p /workspace/runpod-slim/ComfyUI/models/diffusion_models; \
    curl -fL --retry 5 --retry-delay 5 \
      -o /workspace/runpod-slim/ComfyUI/models/diffusion_models/fastvideo_fasth3_8step_v2_pruned_int8_convrot.safetensors \
      "https://huggingface.co/FastVideo/FastVideo-FastH3-Comfy/resolve/main/diffusion_models/fastvideo_fasth3_8step_v2_pruned_int8_convrot.safetensors"

# ---- 7) a LoRA do CivitAI (164 MB) com os dois nomes ----------------------
# modelo "MiniMax H3 NSFW Unlocked" (versao H3_V1.1), fp32
# sha256 52cb409bd89bb5e9a69851c03a1197b3a5b5698c745b812e2f9813492cde7167
RUN set -eux; \
    mkdir -p /workspace/runpod-slim/ComfyUI/models/loras; \
    cd /workspace/runpod-slim/ComfyUI/models/loras; \
    curl -fL --retry 5 --retry-delay 5 -o H3-M3_Unlocked_V2.1.safetensors \
      "https://civitai.com/api/download/models/3345855?fileId=3232785"; \
    echo "52cb409bd89bb5e9a69851c03a1197b3a5b5698c745b812e2f9813492cde7167  H3-M3_Unlocked_V2.1.safetensors" | sha256sum -c -; \
    ln -sf H3-M3_Unlocked_V2.1.safetensors M3_Unlocked_V2.1.safetensors; \
    ls -la

# ---- 8) o workflow testado -------------------------------------------------
COPY workflows/h3-i2v.json /workspace/runpod-slim/ComfyUI/user/default/workflows/h3-i2v.json

# ---- 9) portao ------------------------------------------------------------
RUN set -eux; \
    C=/workspace/runpod-slim/ComfyUI; \
    echo "== versao do ComfyUI =="; grep -o "[0-9][0-9.]*" "$C/comfyui_version.py" 2>/dev/null | head -1 || true; \
    echo "== custom nodes =="; ls -1 "$C/custom_nodes"; \
    echo "== modelos =="; du -sh "$C/models"/*; \
    for f in \
      "$C/models/diffusion_models/fastvideo_fasth3_8step_v2_pruned_int8_convrot.safetensors" \
      "$C/models/text_encoders/qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors" \
      "$C/models/vae/minimax_h3_video_vae_fp16.safetensors" \
      "$C/models/vae/minimax_h3_audio_vae_fp32.safetensors" \
      "$C/models/loras/H3-M3_Unlocked_V2.1.safetensors" \
      "$C/models/loras/M3_Unlocked_V2.1.safetensors" \
      "$C/user/default/workflows/h3-i2v.json" \
      "$C/main.py" ; do \
        if [ -s "$f" ]; then echo "OK   $(du -h "$f" | cut -f1)  $f"; else echo "FALTA $f"; exit 1; fi; \
    done; \
    python3.12 -c "import comfy.model_management; print('ComfyUI 0.37.4 importavel')"
