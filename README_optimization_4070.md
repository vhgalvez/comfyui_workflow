# ComfyUI LTX 2.3 Optimization for RTX 4070 12 GB

## Files generated

- `workflow_LTX_000010_optimized_4070.json`
- `workflow_LTX_000010_safe_4070.json`

## What was changed

This was a surgical optimization of `workflow_LTX_000010_final.json`. The LTX pipeline structure, nodes, links, groups, prompts, models, audio, output nodes, and general canvas layout were preserved.

Only these runtime pressure points were reduced:

### Recommended workflow
File: `workflow_LTX_000010_optimized_4070.json`

- `ResizeImagesByLongerEdge`: `1536` -> `1024`
- `Length` (`PrimitiveInt`): `97` -> `32`
- `EmptyLTXVLatentVideo`: `[1080, 1920, 97, 1]` -> `[720, 1280, 32, 1]`
- `FPS`: kept at `24`
- `LTXVConditioning` frame rate: kept at `24`
- `CreateVideo` fps: kept at `24`

### Ultra safe workflow
File: `workflow_LTX_000010_safe_4070.json`

- `ResizeImagesByLongerEdge`: `1536` -> `768`
- `Length` (`PrimitiveInt`): `97` -> `16`
- `EmptyLTXVLatentVideo`: `[1080, 1920, 97, 1]` -> `[576, 1024, 16, 1]`
- `FPS`: kept at `24`
- `LTXVConditioning` frame rate: kept at `24`
- `CreateVideo` fps: kept at `24`

## Why these changes were made

- Lower base resolution reduces latent size and VAE decode pressure.
- Lower resize target reduces image preprocessing memory cost before video generation.
- Lower frame count reduces sampler workload, decode workload, and final video assembly pressure.
- Keeping `24 FPS` preserves the intended playback feel while making the clip shorter rather than slower.
- The double sampler, latent upscaler, tiled decode, `CreateVideo`, `SaveVideo`, and `SaveImage` were preserved exactly as requested.

## What was intentionally not changed

- Prompts
- Audio file and audio path inside ComfyUI input
- Model names
- Loaders
- `CFGGuider` negative conditioning connections
- `CreateVideo`
- `SaveVideo`
- `SaveImage`
- Workflow version `0.4`

## Models kept unchanged

- `ltx-2-3-22b-dev-Q4_K_M.gguf`
- `LTX23_audio_vae_bf16.safetensors`
- `LTX23_video_vae_bf16.safetensors`
- `gemma_3_12B_it_fp4_mixed.safetensors`
- `ltx-2.3_text_projection_bf16.safetensors`
- `ltx-2.3-22b-distilled-lora-dynamic_fro09_avg_rank_105_bf16.safetensors`
- `ltx-2.3-spatial-upscaler-x2-1.0.safetensors`

## Validation completed

- JSON remains valid in both generated files.
- Workflow `version` remains `0.4`.
- Both `CFGGuider` nodes still have `negative` connected.
- `LoadAudio` still points to `000010_narration.wav`.
- The GGUF model remains `ltx-2-3-22b-dev-Q4_K_M.gguf`.
- `CreateVideo`, `SaveVideo`, and `SaveImage` are still present.
- Both variants are materially lighter than the original workflow.

## Which file to use first

Use `workflow_LTX_000010_safe_4070.json` first if your immediate goal is to verify backend stability and avoid crashes during initial tests.

Once that runs reliably, move to `workflow_LTX_000010_optimized_4070.json` as the recommended daily workflow for your RTX 4070 12 GB.

## Recommended order

1. Import `workflow_LTX_000010_safe_4070.json`
2. Run a first validation render
3. If stable, switch to `workflow_LTX_000010_optimized_4070.json`
4. Keep the original `workflow_LTX_000010_final.json` only as reference, not as the default runtime workflow on this GPU
