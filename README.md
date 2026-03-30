# comfyui_workflow

Plantilla técnica para **ComfyUI + LTX 2.3** orientada a la generación de **shorts verticales con audio externo real**, pensada para un flujo de trabajo de vídeo corto con sincronización audio‑video.

Este repositorio reúne dos piezas principales:

- `workflow_LTX.json`: blueprint técnico del workflow.
- `LTX_shorts_workflow_template.md`: guía resumida del flujo, modelos recomendados y parámetros base.

> **Importante:** este repositorio no contiene una exportación final totalmente lista para arrastrar al canvas de ComfyUI. El propio JSON indica que es una **plantilla/blueprint** cuya serialización final depende de la versión de ComfyUI, extensiones instaladas, IDs de nodos y disposición del canvas.

---

## ¿Qué hace este repositorio?

El objetivo es ayudar a construir un workflow limpio en ComfyUI para:

1. Cargar un archivo de audio WAV real.
2. Preparar una imagen base vertical o usar un placeholder.
3. Codificar el audio con el VAE de audio de LTX.
4. Crear el latent de vídeo.
5. Concatenar audio + vídeo en un latent AV.
6. Ejecutar muestreo de stage 1.
7. Separar audio y vídeo.
8. Reescalar/refinar el latent de vídeo.
9. Volver a concatenar audio + vídeo.
10. Ejecutar stage 2.
11. Decodificar vídeo y audio.
12. Exportar un short vertical final.

La plantilla está pensada específicamente para **vídeo corto vertical 9:16** con foco en:

- sincronización con narración externa,
- lip sync y motion coherentes,
- estabilidad en hardware tipo **RTX 4070 12 GB**,
- flujo iterativo primero estable y luego optimizado.

---

## Estructura del repositorio

```text
comfyui_workflow/
├── .gitattributes
├── LTX_shorts_workflow_template.md
└── workflow_LTX.json
```

### Archivos

#### `LTX_shorts_workflow_template.md`
Documento guía en lenguaje natural con:

- objetivo del workflow,
- idea general del pipeline,
- modelos recomendados,
- rutas sugeridas,
- lista de nodos por secciones,
- prompts base,
- parámetros de arranque,
- checklist final.

#### `workflow_LTX.json`
Describe el blueprint del workflow con:

- propósito del pipeline,
- notas técnicas,
- audio objetivo usado como ejemplo,
- modelos y rutas recomendadas,
- defaults para shorts verticales,
- prompt positivo y negativo,
- secciones del grafo del workflow,
- instrucciones de uso.

---

## Caso de uso principal

Este repo está orientado a crear shorts como:

- vídeos narrados para TikTok,
- Reels,
- Shorts de YouTube,
- clips verticales con retrato o personaje estable,
- vídeo corto guiado por una narración ya grabada.

El ejemplo de audio del repositorio apunta a un WAV concreto en Windows:

```text
C:\Users\vhgal\Documents\desarrollo\ia\AI-video-automation\video-dataset\jobs\000010\audio\000010_narration.wav
```

Y recomienda copiarlo al directorio `input` de ComfyUI:

```text
C:\Users\vhgal\AppData\Local\Programs\ComfyUI\resources\ComfyUI\ComfyUI\input\000010_narration.wav
```

Estas rutas aparecen explícitamente en la plantilla JSON. citeturn261208view4turn261208view5

---

## Modelos recomendados

La plantilla recomienda, para estabilidad en una RTX 4070 12 GB:

- **UNET GGUF:** `ltx-2-3-22b-dev-Q4_K_M.gguf`
- **Alternativa mayor calidad:** `ltx-2-3-22b-dev-Q5_K_S.gguf`
- **Video VAE:** `LTX23_video_vae_bf16.safetensors`
- **Audio VAE:** `LTX23_audio_vae_bf16.safetensors`
- **Text encoder:** `gemma_3_12B_it_fp4_mixed.safetensors`
- **Text projection:** `ltx-2.3_text_projection_bf16.safetensors`
- **Distilled LoRA:** `ltx-2.3-22b-distilled-lora-dynamic_fro09_avg_rank_105_bf16.safetensors`
- **Spatial upscaler:** `ltx-2.3-spatial-upscaler-x2-1.0.safetensors`

Las rutas sugeridas dentro de ComfyUI también están definidas en el JSON del repo. citeturn261208view4turn633479view0

---

## Parámetros por defecto para shorts

El workflow viene configurado con estos valores base:

- **Orientación:** vertical
- **Aspect ratio objetivo:** `9:16`
- **FPS:** `24`
- **Longitud inicial recomendada:** `97` frames
- **Longitud máxima recomendada en RTX 4070:** `121` frames
- **Resize longer edge:** `1536`
- **Image compression:** `33`
- **Tile size:** `512`
- **Tile overlap:** `64`
- **CFG:** `1.0`
- **Sampler stage 1:** `euler`
- **Sampler stage 2:** `euler` citeturn261208view4

Esto encaja con la intención del repositorio: empezar por una configuración conservadora y estable, y luego subir longitud/calidad si el sistema aguanta.

---

## Arquitectura del workflow

Según `workflow_LTX.json`, el grafo está organizado por secciones.

### A. Entrada

- `LoadAudio`
- `LoadImage_or_EmptyImage`
- `ResizeImagesByLongerEdge`
- `GetImageSize`
- `LTXVPreprocess` citeturn261208view5

### B. Modelos

- `UnetLoaderGGUF`
- `LoraLoaderModelOnly`
- `DualCLIPLoader`
- `VAELoaderKJ_video`
- `VAELoaderKJ_audio_or_LTX_audio_loader` citeturn261208view5

### C. Audio + video latents

- `LTXVAudioVAEEncode`
- `PrimitiveInt_Length`
- `PrimitiveFloat_FPS`
- `EmptyLTXVLatentVideo`
- `LTXVConcatAVLatent` citeturn519052view3turn261208view5

### D. Conditioning + stage 1

Incluye conditioning de texto, CFG, ruido, selección de sampler y sigmas manuales para el primer pase. La plantilla `.md` recomienda sampler `euler` y una secuencia de sigmas concreta para stage 1. citeturn633479view0

### E. Refine / upscale

- `LTXVSeparateAVLatent`
- `LatentUpscaleModelLoader`
- `LTXVLatentUpsampler`
- `LTXVConcatAVLatent`
- segundo bloque de sampler/CFG para stage 2. citeturn633479view0

### F. Decode y salida

- `VAEDecodeTiled`
- `LTXVAudioVAEDecode`
- `CreateVideo` o `VHS_VideoCombine`
- `SaveVideo` con prefijo ejemplo `Video/LTX-2/short_000010` citeturn633479view0turn261208view5

---

## Prompt base incluido

La plantilla trae un prompt positivo ya pensado para shorts verticales con sincronización facial y presencia tipo retrato cinematográfico, y un prompt negativo centrado en evitar defectos visuales como blur, compresión fuerte, flicker, mala anatomía, texto incrustado o lipsync defectuoso. citeturn633479view0turn261208view5

Esto te da un punto de partida muy útil para:

- narrador a cámara,
- personaje consistente,
- contenido corto con foco en rostro,
- vídeos sociales sin subtítulos quemados en la imagen.

---

## Cómo usar este repositorio

## 1) Preparar ComfyUI

Necesitas una instalación de ComfyUI funcional con soporte para los nodos y loaders que usa LTX 2.3.

## 2) Colocar los modelos

Sigue las rutas sugeridas del repo para:

- `models/unet/ltx-2.3/`
- `models/vae/ltx2.3/`
- `models/text_encoders/`
- `models/loras/ltx2/`
- `models/latent_upscale_models/` citeturn261208view4turn633479view0

## 3) Copiar el audio

Coloca el WAV en el directorio `input` de ComfyUI o usa `LoadAudio` si tu build permite cargar la ruta directa. El propio JSON recomienda empezar así. citeturn261208view5

## 4) Construir el workflow real en canvas

Abre `LTX_shorts_workflow_template.md` y `workflow_LTX.json` como referencia y crea el grafo en ComfyUI respetando:

- nombres de nodos,
- orden lógico,
- parámetros base,
- rutas de modelos,
- valores de fps y longitud.

## 5) Hacer la primera prueba conservadora

Empieza con:

- `Q4_K_M`
- `97` frames
- `24 fps`
- resolución vertical controlada
- una imagen base simple o placeholder.

## 6) Escalar después

Cuando sea estable, sube a:

- `121` frames,
- mejor calidad del modelo GGUF,
- imagen base más cuidada,
- prompts más específicos. citeturn633479view0turn261208view5

---

## Recomendaciones prácticas

### Empezar simple

El repo está claramente diseñado para evitar errores comunes: primero audio correcto, luego latent AV, luego refinado. No intenta meter demasiadas piezas avanzadas desde el inicio.

### No mezclar rutas de audio y VAEs equivocados

La guía explica que uno de los errores clave del workflow roto anterior era mezclar el camino de audio con un VAE incorrecto de imagen/vídeo. Aquí el flujo correcto queda separado:

- audio → `LoadAudio` → `LTXVAudioVAEEncode` → `LTXVAudioVAEDecode`
- vídeo → `EmptyLTXVLatentVideo` → `LTXVLatentUpsampler` → `VAEDecodeTiled` citeturn633479view0

### Mantener FPS unificado

El repo insiste en usar `24 fps` de forma coherente en conditioning, decode y creación del vídeo final. citeturn261208view5

### Pensado para iteración

Este repositorio no busca ser una solución cerrada, sino una base ordenada para adaptar a:

- otros audios,
- otros prompts,
- otros personajes,
- otras rutas de trabajo dentro de tu pipeline.

---

## Limitaciones actuales

Este repositorio, tal como está publicado, tiene un alcance claro:

- **No incluye** una exportación final 100% plug-and-play de ComfyUI.
- **No incluye** assets de ejemplo aparte de las rutas referenciadas.
- **No incluye** automatización de ejecución externa.
- **No incluye** documentación extendida de instalación de nodos personalizados.

En otras palabras: es una **base técnica bien organizada**, no un producto terminado listo para un clic.

---

## ¿Para quién sirve?

Este repo te sirve especialmente si:

- ya usas ComfyUI,
- quieres montar un flujo LTX 2.3 con audio externo,
- trabajas shorts verticales,
- tienes una GPU intermedia como RTX 4070,
- quieres una referencia limpia para reconstruir un workflow estable.

---

## Mejores usos recomendados

- Prototipado de vídeos narrados cortos.
- Base para tu pipeline personal de contenido.
- Referencia para construir un workflow exportable más adelante.
- Plantilla para integrar después con automatización externa.
- Punto de partida para testing de lip sync y coherencia facial.

---

## Siguiente mejora recomendada para el repo

Si quieres llevar este repositorio a un nivel más profesional, las mejoras más útiles serían:

1. Añadir un `README.md` oficial en la raíz.
2. Incluir captura del canvas del workflow.
3. Documentar dependencias exactas de nodos/custom nodes.
4. Añadir una sección de troubleshooting.
5. Incluir una versión exportada real del workflow si tu instalación ya está estable.
6. Añadir ejemplos para varios jobs (`000010`, `000011`, etc.).
7. Documentar cómo adaptar el audio, prompt y personaje sin romper el grafo.

---

## Fuentes del análisis

Este README se ha redactado a partir del contenido público del repositorio y de sus dos archivos visibles en GitHub: la vista del repositorio, el archivo `LTX_shorts_workflow_template.md` y el archivo `workflow_LTX.json`. citeturn105099view0turn633479view0turn748764view0turn261208view4turn261208view5
