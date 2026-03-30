# Plantilla limpia LTX 2.3 para shorts verticales con audio externo

## Objetivo
Crear un workflow limpio en ComfyUI para generar shorts verticales usando LTX 2.3 con audio externo ya grabado.

Audio objetivo:
`C:\Users\vhgal\Documents\desarrollo\ia\AI-video-automation\video-dataset\jobs\000010\audio\000010_narration.wav`

## Idea general
Usar el audio real como guía del modelo LTX:
1. Cargar audio WAV.
2. Cargar imagen base o imagen vacía/placeholder.
3. Preprocesar imagen para vertical short.
4. Codificar audio con `LTXVAudioVAEEncode`.
5. Crear latent de video con `EmptyLTXVLatentVideo`.
6. Concatenar audio+video con `LTXVConcatAVLatent`.
7. Muestreo stage 1.
8. Separar audio+video con `LTXVSeparateAVLatent`.
9. Upscale latent de video con `LTXVLatentUpsampler`.
10. Volver a concatenar audio+video.
11. Muestreo stage 2.
12. Decodificar video con `VAEDecodeTiled`.
13. Decodificar audio con `LTXVAudioVAEDecode`.
14. Combinar en `CreateVideo` / `VHS_VideoCombine`.

---

## Modelos recomendados para tu PC (RTX 4070 12 GB)

### Recomendado para estabilidad
- UNET GGUF: `ltx-2-3-22b-dev-Q4_K_M.gguf`
- Audio VAE: `LTX23_audio_vae_bf16.safetensors`
- Video VAE: `LTX23_video_vae_bf16.safetensors`
- Text encoder: `gemma_3_12B_it_fp4_mixed.safetensors`
- Text projection: `ltx-2.3_text_projection_bf16.safetensors`
- Distilled LoRA: `ltx-2.3-22b-distilled-lora-dynamic_fro09_avg_rank_105_bf16.safetensors`
- Spatial upscaler: `ltx-2.3-spatial-upscaler-x2-1.0.safetensors`

### Si quieres más calidad y aceptas más riesgo
- UNET GGUF: `ltx-2-3-22b-dev-Q5_K_S.gguf`

---

## Rutas recomendadas

Base ComfyUI:
`C:\Users\vhgal\AppData\Local\Programs\ComfyUI\resources\ComfyUI\ComfyUI`

Modelos:
- `...\models\unet\ltx-2.3\ltx-2-3-22b-dev-Q4_K_M.gguf`
- `...\models\vae\ltx2.3\LTX23_audio_vae_bf16.safetensors`
- `...\models\vae\ltx2.3\LTX23_video_vae_bf16.safetensors`
- `...\models\text_encoders\gemma_3_12B_it_fp4_mixed.safetensors`
- `...\models\text_encoders\ltx-2.3_text_projection_bf16.safetensors`
- `...\models\loras\ltx2\ltx-2.3-22b-distilled-lora-dynamic_fro09_avg_rank_105_bf16.safetensors`
- `...\models\latent_upscale_models\ltx-2.3-spatial-upscaler-x2-1.0.safetensors`

---

## Workflow recomendado (nodos)

### A. Entrada
1. **LoadAudio**
   - Carga el WAV del job 000010.
2. **Load Image** o **EmptyImage**
   - Si tienes una imagen base del personaje, mejor usar `Load Image`.
   - Si todavía no la tienes, usa `EmptyImage` como placeholder.
3. **ResizeImagesByLongerEdge**
   - `longer_edge = 1536`
4. **GetImageSize**
5. **LTXVPreprocess**
   - `img_compression = 33`

### B. Modelos
6. **UnetLoaderGGUF**
   - `ltx-2-3-22b-dev-Q4_K_M.gguf`
7. **LoraLoaderModelOnly**
   - distilled LoRA
   - `strength_model = 0.6`
8. **DualCLIPLoader**
   - `gemma_3_12B_it_fp4_mixed.safetensors`
   - `ltx-2.3_text_projection_bf16.safetensors`
   - `type = ltxv`
9. **VAELoaderKJ** (solo video)
   - `LTX23_video_vae_bf16.safetensors`
10. **VAELoaderKJ** o loader equivalente para el audio VAE SOLO si el workflow usa encode/decode con audio real y te funciona bien en tu build
   - `LTX23_audio_vae_bf16.safetensors`

> Nota: si tu build tiene problemas con el loader genérico en audio, usa el loader LTX específico disponible en tu instalación para audio.

### C. Audio + video latents
11. **LTXVAudioVAEEncode**
   - `audio = LoadAudio output`
   - `audio_vae = audio VAE`
12. **MathExpression / PrimitiveInt**
   - `fps = 24`
13. **Length**
   - `97` para prueba estable
   - luego `121` si va bien
14. **EmptyLTXVLatentVideo**
   - `width = from GetImageSize`
   - `height = from GetImageSize`
   - `length = Length`
15. **LTXVConcatAVLatent**
   - `video_latent = EmptyLTXVLatentVideo`
   - `audio_latent = LTXVAudioVAEEncode`

### D. Conditioning
16. **CLIPTextEncode** positivo
17. **CLIPTextEncode** negativo
18. **LTXVConditioning**
   - `frame_rate = 24.0`
19. **CFGGuider**
20. **RandomNoise**
21. **KSamplerSelect**
   - stage 1: `euler`
22. **ManualSigmas**
   - stage 1: `1., 0.99375, 0.9875, 0.98125, 0.975, 0.909375, 0.725, 0.421875, 0.0`
23. **SamplerCustomAdvanced**
   - usa el latent concatenado AV

### E. Refine / upscale
24. **LTXVSeparateAVLatent**
25. **LatentUpscaleModelLoader**
   - `ltx-2.3-spatial-upscaler-x2-1.0.safetensors`
26. **LTXVLatentUpsampler**
   - `samples = video_latent`
   - `vae = video VAE`
27. **LTXVConcatAVLatent**
   - `video_latent = upscaled video latent`
   - `audio_latent = separated audio latent`
28. **CFGGuider** stage 2
29. **RandomNoise** stage 2
30. **KSamplerSelect** stage 2
   - `euler`
31. **ManualSigmas** stage 2
   - `0.85, 0.7250, 0.4219, 0.0`
32. **SamplerCustomAdvanced** stage 2
33. **LTXVSeparateAVLatent**

### F. Decode y salida
34. **VAEDecodeTiled**
   - `vae = video VAE`
   - `tile_size = 512`
   - `overlap = 64`
35. **LTXVAudioVAEDecode**
   - `audio_vae = audio VAE`
36. **CreateVideo** o **VHS_VideoCombine**
   - `fps = 24`
   - `trim_to_audio = true` si usas VHS y quieres ajustar al audio
37. **SaveVideo**
   - prefijo ejemplo: `Video/LTX-2/short_000010`

---

## Prompt plantilla para shorts

### Prompt positivo (base)
Usa esto como plantilla:

"Vertical short video, cinematic close-up portrait, strong lip sync to the provided narration, realistic facial motion, expressive eyes, subtle head movement, natural shoulders and torso motion, shallow depth of field, soft key light, background simple and clean, high detail skin, realistic mouth articulation, stable framing, short-form social media style, portrait orientation, professional short video, consistent identity, no subtitle burn-in."

### Negativo
"blurry, low quality, duplicated face, extra fingers, bad anatomy, subtitles, text overlay, watermark, heavy compression artifacts, flicker, jumpy camera, frozen face, bad lipsync, distorted mouth, deformed eyes"

---

## Parámetros recomendados para empezar
- Resolución inicial base: `1080x1920` o imagen vertical equivalente
- Resize longer edge: `1536`
- FPS: `24`
- Length: `97`
- LoRA strength: `0.6`
- CFG: `1`
- Stage 1 sampler: `euler`
- Stage 2 sampler: `euler`

---

## Para tu archivo de audio 000010_narration.wav

### Opción A (recomendada)
Copia el WAV al directorio `input` de ComfyUI y cárgalo con `LoadAudio`.

Ejemplo:
`C:\Users\vhgal\AppData\Local\Programs\ComfyUI\resources\ComfyUI\ComfyUI\input\000010_narration.wav`

### Opción B
Si tu nodo `LoadAudio` permite navegar por archivo, selecciona directamente:
`C:\Users\vhgal\Documents\desarrollo\ia\AI-video-automation\video-dataset\jobs\000010\audio\000010_narration.wav`

---

## Diferencia clave frente a tu workflow roto anterior
No mezclar el camino de audio con un VAE de imagen/video incorrecto. El audio debe pasar por:
- `LoadAudio`
- `LTXVAudioVAEEncode`
- `LTXVAudioVAEDecode`

Y el video por:
- `EmptyLTXVLatentVideo`
- `LTXVLatentUpsampler`
- `VAEDecodeTiled`

---

## Checklist final
- [ ] El UNET GGUF carga desde `unet/ltx-2.3/`
- [ ] El audio VAE está en `vae/ltx2.3/`
- [ ] El video VAE está en `vae/ltx2.3/`
- [ ] El audio WAV se carga por `LoadAudio`
- [ ] El FPS está unificado en todo el workflow a `24`
- [ ] El `CreateVideo` también está en `24 fps`
- [ ] El primer test se hace con `Length = 97`
- [ ] Después se sube a `121` solo si va estable

