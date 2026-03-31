# README text2video pure fix

## Que se quito

- La rama falsa image-to-video completa:
  `EmptyImage`, `ResizeImagesByLongerEdge`, `GetImageSize`, `LTXVPreprocess` y `LTXVImgToVideoConditionOnly`.

## Por que se quito

- Ese camino no aportaba nada real en este caso porque el workflow no partia de una imagen util, sino de una imagen vacia artificial.
- Forzaba trabajo extra de VAE, conditioning y memoria antes del sampler.
- Para este caso el flujo correcto y mas barato es text-to-video puro con latent vacio de video.

## Por que `LTXVImgToVideoConditionOnly` era incorrecto aqui

- Ese nodo tiene sentido cuando existe una imagen inicial real que debe condicionar el video.
- Con una imagen vacia solo introduce complejidad y coste de memoria sin aportar informacion visual.

## Que nodos quedaron

- `LoadAudio`
- `UnetLoaderGGUF`
- `LoraLoaderModelOnly`
- `DualCLIPLoader`
- `VAELoaderKJ` audio
- `VAELoaderKJ` video
- `LTXVAudioVAEEncode`
- `EmptyLTXVLatentVideo`
- `LTXVConcatAVLatent`
- `CLIPTextEncode` positivo y negativo
- `LTXVConditioning`
- `CFGGuider`
- `RandomNoise`
- `KSamplerSelect`
- `ManualSigmas`
- `SamplerCustomAdvanced`
- `LTXVSeparateAVLatent`
- `VAEDecodeTiled`
- `LTXVAudioVAEDecode`
- `CreateVideo`
- `SaveVideo`
- `SaveImage`

## Parametros de memoria reducidos

- `DualCLIPLoader.device` cambiado a `cpu`.
- `Audio VAE.device` mantenido en `cpu`.
- `Video VAE.device` cambiado a `cpu`.
- `VAEDecodeTiled` reducido a `tile_size = 256`, `overlap = 64`, `temporal_size = 16`, `temporal_overlap = 4`.

## Que probar despues si esta version ya arranca

1. Ejecutar esta version sin tocar nada y confirmar que el backend sobrevive.
2. Si arranca, probar otro seed manteniendo el resto fijo.
3. Si sigue estable, devolver primero el `Video VAE` a `main_device` antes de subir frames.
4. Solo despues considerar subir decode o volver a un modelo mas pesado.
