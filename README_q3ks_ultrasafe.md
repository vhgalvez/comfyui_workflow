# README q3ks ultrasafe

## Que se cambio

- UNET principal cambiado de `ltx-2-3-22b-dev-Q3_K_M.gguf` a `ltx-2-3-22b-dev-Q3_K_S.gguf`.
- `Length` reducido de `49` a `17`.
- Audio VAE cambiado de `device = main_device` a `device = cpu`.
- Se mantuvo el workflow lean de una sola etapa, sin latent upscaler ni segunda etapa de sampler.

## Por que se cambio

- El backend ya estaba fallando por presion real de VRAM durante staging y ejecucion.
- `Q3_K_S` reduce la carga del modelo principal frente a `Q3_K_M`.
- `17` frames sigue siendo valido para LTX porque cumple `8n+1`, y baja memoria de forma directa.
- Mover el Audio VAE a CPU libera VRAM para el UNET y el path principal de video.

## Por que esta version es mas segura para RTX 4070 12 GB

- Reduce el peso del UNET.
- Reduce el tamaño temporal del clip.
- Saca el Audio VAE de la GPU.
- Mantiene `576x1024`, `24 fps` y el pipeline simple de una sola etapa para minimizar picos de memoria.

## Estado del Audio VAE

- El Audio VAE quedo en `cpu`.
- Este cambio es valido para `VAELoaderKJ` en tu instalacion, que admite `device = main_device` o `device = cpu`.

## Que probar despues si esta version funciona

1. Ejecutar esta version tal cual y confirmar que el backend ya no muere.
2. Si va estable, probar otro seed antes de subir carga.
3. Si sigue estable, subir primero a `25` frames.
4. Solo despues considerar volver el Audio VAE a GPU o pasar a `Q3_K_M`.
