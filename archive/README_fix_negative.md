# Fix de `negative` en `CFGGuider`

- Error original: los dos nodos `CFGGuider` tenían el input requerido `negative` con `link: null`, por lo que el workflow cargaba pero fallaba al ejecutar en ComfyUI.
- Nodos corregidos: `CFGGuider` con id `21` y `CFGGuider` con id `29`.
- Estrategia usada: se utilizó la salida `negative` de `LTXVConditioning` (nodo `16`), que ya forma parte del grafo y es la opción preferida para este workflow.
- Enlaces nuevos añadidos: `51` = `LTXVConditioning[negative] -> CFGGuider#21[negative]`; `52` = `LTXVConditioning[negative] -> CFGGuider#29[negative]`.
- Cambios técnicos: se actualizaron `links[]`, las referencias `inputs[].link` de ambos `CFGGuider`, y `last_link_id` pasó de `50` a `52`.
- Qué comprobar en ComfyUI: reimporta `workflow_LTX_000010_real_fixed_negative.json`, confirma que ambos `CFGGuider` ya no muestran `Required input is missing` en `negative`, y ejecuta el workflow verificando que no haya errores de enlace o validación al iniciar el sampler.
