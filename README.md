# ComfyUI LTX Workflow for Job 000010

## Descripción

Esta carpeta contiene el workflow final de ComfyUI para generar un short vertical con LTX 2.3 usando audio real del job `000010`.

El workflow principal ya incluye la corrección necesaria para que los nodos `CFGGuider` tengan conectado el input obligatorio `negative`.

## Workflow principal

Archivo a usar:

- `workflow_LTX_000010_final.json`

Este es el único workflow que debes importar como versión final de trabajo.

## Audio utilizado

Audio fuente del proyecto:

- `C:\Users\vhgal\Documents\desarrollo\ia\AI-video-automation\video-dataset\jobs\000010\audio\000010_narration.wav`

Para que ComfyUI lo cargue correctamente, el archivo `000010_narration.wav` debe estar disponible dentro de `ComfyUI/input`.

## Modelos utilizados

- `ltx-2-3-22b-dev-Q4_K_M.gguf`
- `LTX23_audio_vae_bf16.safetensors`
- `LTX23_video_vae_bf16.safetensors`
- `gemma_3_12B_it_fp4_mixed.safetensors`
- `ltx-2.3_text_projection_bf16.safetensors`
- `ltx-2.3-22b-distilled-lora-dynamic_fro09_avg_rank_105_bf16.safetensors`
- `ltx-2.3-spatial-upscaler-x2-1.0.safetensors`

## Cómo importarlo en ComfyUI

1. Abre ComfyUI.
2. Importa `workflow_LTX_000010_final.json`.
3. Verifica que `000010_narration.wav` esté disponible en `ComfyUI/input`.
4. Confirma que los modelos listados arriba estén instalados y visibles para ComfyUI.
5. Ejecuta el workflow.

## Correcciones aplicadas

El workflow original tenía un error de validación en ambos nodos `CFGGuider` porque el input obligatorio `negative` estaba sin conectar.

Ese problema ya fue corregido en el workflow final conectando la salida `negative` de `LTXVConditioning` a los dos `CFGGuider`.

## Archivos archivados

La carpeta `archive` conserva material anterior o de referencia para no mezclarlo con la versión final:

- `workflow_LTX_000010_real.json`: versión anterior con el error de `CFGGuider negative`.
- `README_fix_negative.md`: nota técnica del arreglo aplicado, ya integrada a este README.
- `comfyui.txt`: referencia auxiliar histórica.
- `LTX_shorts_workflow_template.md`: plantilla o referencia de trabajo anterior.

## Estructura recomendada

- `README.md`
- `workflow_LTX_000010_final.json`
- `archive/`

## Workflow a importar

Importa este archivo:

- `workflow_LTX_000010_final.json`
