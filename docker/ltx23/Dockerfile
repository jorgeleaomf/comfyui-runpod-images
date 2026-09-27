# =============================================================================
# ComfyUI pronto para RunPod - LTX 2.3 (Image to Video, com audio)
#
# Base : runpod/comfyui:1.3.3-comfyuiv0.30.0-cuda13.0
#        (ComfyUI 0.30.0, PyTorch 2.10.0+cu130, Python 3.12.3, JupyterLab,
#         ComfyUI-Manager, KJNodes, Civicomfy, RunpodDirect)
# Extra: ComfyUI-GGUF + ComfyUI-VideoHelperSuite (commits fixados, mesma dupla
#        das outras imagens) e os arquivos do LTX 2.3 (~42,5 GB) + o workflow.
#
# Modelo instalado (exatamente o do pod de referencia):
#   checkpoints/ltx-2.3-22b-dev-fp8.safetensors ........ 27,14 GB  (Lightricks)
#   text_encoders/gemma_3_12B_it_fp4_mixed.safetensors .. 8,80 GB  (Comfy-Org)
#   loras/ltx_2.3_22b_distilled_1.1_lora_..._bf16.safetensors  2,55 GB (destilado)
#   loras/SexGod_LTX23_DoggyStyle_v2_5.safetensors ...... 2,51 GB
#   latent_upscale_models/ltx-2.3-spatial-upscaler-x2-1.1 0,93 GB
#   loras/gemma-3-12b-it-abliterated_lora_rank64_bf16 ... 0,59 GB
# O VAE de video e o vocoder de audio vem DENTRO do checkpoint fp8
# (o workflow aponta LTXVAudioVAELoader para o proprio checkpoint).
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

ARG GGUF_SHA=6ea2651e7df66d7585f6ffee804b20e92fb38b8a
ARG VHS_SHA=4d907bee61e92c2e65af3bd6383a4e4d356126d1

LABEL org.opencontainers.image.title="comfyui-ltx23"
LABEL org.opencontainers.image.description="ComfyUI 0.30.0 + LTX 2.3 (22b dev fp8 + destilado 1.1 + upscaler x2) para RunPod"
LABEL br.hermes.base="runpod/comfyui:1.3.3-comfyuiv0.30.0-cuda13.0"
LABEL br.hermes.comfyui-gguf="${GGUF_SHA}"
LABEL br.hermes.videohelpersuite="${VHS_SHA}"

# ---- 1) custom nodes extras (mesma dupla das outras imagens) ---------------
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

# ---- 2) arvore pronta do ComfyUI no workspace ------------------------------
RUN set -eux; \
    mkdir -p /workspace/runpod-slim; \
    cp -r /opt/comfyui-baked /workspace/runpod-slim/ComfyUI; \
    mkdir -p /workspace/runpod-slim/ComfyUI/user/default/workflows \
             /workspace/runpod-slim/ComfyUI/input \
             /workspace/runpod-slim/ComfyUI/output \
             /workspace/runpod-slim/ComfyUI/temp; \
    test -f /workspace/runpod-slim/ComfyUI/main.py

# ---- 3) o text encoder (8,8 GB) em camada propria --------------------------
RUN set -eux; \
    cd /workspace/runpod-slim/ComfyUI/models; \
    mkdir -p text_encoders loras checkpoints latent_upscale_models; \
    curl -fL --retry 5 --retry-delay 5 -o text_encoders/gemma_3_12B_it_fp4_mixed.safetensors \
        "https://huggingface.co/Comfy-Org/ltx-2/resolve/main/split_files/text_encoders/gemma_3_12B_it_fp4_mixed.safetensors"

# ---- 4) as tres LoRAs e o upscaler (camada propria) ------------------------
RUN set -eux; \
    cd /workspace/runpod-slim/ComfyUI/models; \
    curl -fL --retry 5 --retry-delay 5 -o loras/ltx_2.3_22b_distilled_1.1_lora_dynamic_fro09_avg_rank_111_bf16.safetensors \
        "https://huggingface.co/Comfy-Org/ltx-2.3/resolve/main/split_files/loras/ltx_2.3_22b_distilled_1.1_lora_dynamic_fro09_avg_rank_111_bf16.safetensors"; \
    curl -fL --retry 5 --retry-delay 5 -o loras/gemma-3-12b-it-abliterated_lora_rank64_bf16.safetensors \
        "https://huggingface.co/Comfy-Org/ltx-2/resolve/main/split_files/loras/gemma-3-12b-it-abliterated_lora_rank64_bf16.safetensors"; \
    curl -fL --retry 5 --retry-delay 5 -o loras/SexGod_LTX23_DoggyStyle_v2_5.safetensors \
        "https://huggingface.co/SexGod1979/LTX_2.3_Doggystyle/resolve/main/SexGod_LTX23_DoggyStyle_v2_5.safetensors"; \
    curl -fL --retry 5 --retry-delay 5 -o latent_upscale_models/ltx-2.3-spatial-upscaler-x2-1.1.safetensors \
        "https://huggingface.co/Lightricks/LTX-2.3/resolve/main/ltx-2.3-spatial-upscaler-x2-1.1.safetensors"

# ---- 5) o checkpoint principal (27 GB): sozinho na sua camada --------------
RUN set -eux; \
    mkdir -p /workspace/runpod-slim/ComfyUI/models/checkpoints; \
    curl -fL --retry 5 --retry-delay 5 \
        -o /workspace/runpod-slim/ComfyUI/models/checkpoints/ltx-2.3-22b-dev-fp8.safetensors \
        "https://huggingface.co/Lightricks/LTX-2.3-fp8/resolve/main/ltx-2.3-22b-dev-fp8.safetensors"

# ---- 6) o workflow salvo ---------------------------------------------------
COPY workflows/ltx23-i2v.json /workspace/runpod-slim/ComfyUI/user/default/workflows/ltx23-i2v.json

# ---- 7) portao -------------------------------------------------------------
RUN set -eux; \
    C=/workspace/runpod-slim/ComfyUI; \
    echo "== custom nodes =="; ls -1 "$C/custom_nodes"; \
    echo "== uso de disco dos modelos =="; du -sh "$C/models"/*; \
    for f in \
      "$C/models/checkpoints/ltx-2.3-22b-dev-fp8.safetensors" \
      "$C/models/text_encoders/gemma_3_12B_it_fp4_mixed.safetensors" \
      "$C/models/loras/ltx_2.3_22b_distilled_1.1_lora_dynamic_fro09_avg_rank_111_bf16.safetensors" \
      "$C/models/loras/gemma-3-12b-it-abliterated_lora_rank64_bf16.safetensors" \
      "$C/models/loras/SexGod_LTX23_DoggyStyle_v2_5.safetensors" \
      "$C/models/latent_upscale_models/ltx-2.3-spatial-upscaler-x2-1.1.safetensors" \
      "$C/user/default/workflows/ltx23-i2v.json" \
      "$C/main.py" ; do \
        if [ -s "$f" ]; then echo "OK   $(du -h "$f" | cut -f1)  $f"; else echo "FALTA $f"; exit 1; fi; \
    done; \
    python3.12 -c "import gguf, imageio_ffmpeg; print('dependencias dos nodes OK')"
