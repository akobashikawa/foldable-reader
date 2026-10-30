# Lecciones

## 001 — Cargar texto desde JSON

- El build global de Vue incluye el compilador de plantillas: un componente recursivo puede definirse con un `template` en string y resolverse por su `name`, sin SFC ni bundler.
- `<input type="file">` + `file.text()` evita `fetch`, CORS y la necesidad de servidor: la app funciona con `file://`.
- Matcha.css estiliza `input[type=file]` y pisa el `display: none` que aporta el atributo `hidden`. Por eso el input usa la clase `.file-input { display: none }`; si se quita, aparece un segundo botón nativo "Choose File".
- Conviene separar la validación pura (`validateDoc` → `{ doc }` o `{ error }`) de la normalización (ids estables derivados de la ruta). Así los mensajes pueden señalar el nodo exacto (`nodes[0].children[1].text`) sin mezclar esa lógica con el manejo del DOM.
- Los nodos con hijos se renderizan como encabezado y los niveles se topan en `h3`; las hojas siempre como párrafo. Es una decisión de legibilidad, no del formato.
- La interpolación de Vue escapa el HTML automáticamente; `v-html` queda prohibido porque el JSON es contenido no confiable.
- Resetear `event.target.value` en el `finally` permite volver a elegir el mismo archivo sin cerrar la página.
- `public/texts/demo/demo.json` sirve como fixture manual: dos capítulos con hijos, summaries y una nota final sin hijos.

### Verificación

- El navegador de Playwright bloquea navegar a `file://` desde la herramienta de navegación, pero `page.goto('file://...')` dentro de `browser_run_code_unsafe` funciona y permite comprobar ese criterio.
- Para probar la carga, usar `page.setInputFiles('input.file-input', ruta)`; hacer clic en "Abrir JSON" abre el diálogo nativo del sistema y la herramienta expira esperando el selector.

## 002 — Frase alternativa editable

- `summary` ya existía en el modelo de la spec 001; esta spec no agregó estructuras, solo le dio comportamiento editable. Antes de inventar un mapa `summaries[nodeId]`, conviene revisar qué campos ya trae el JSON.
- El nodo se muta directo (`this.node.summary = ...`). `doc` es un `ref` profundo, así que editar el objeto actualiza la vista sin emitir eventos ni duplicar estado en el componente raíz.
- Cada `FoldNode` guarda su propio `editing` y `draft`. La recursión del componente ya resuelve el “un editor por nodo” sin store global ni identificadores.
- El control se dibuja en dos ramas mutuamente excluyentes (`summary-line` con botón dentro, o botón suelto sin frase) y ambas pueden compartir `ref="toggle"`: Vue solo monta una.
- El botón que abre el editor se desmonta mientras se edita. Por eso el foco se mueve en `$nextTick`, cuando el nuevo DOM ya existe; antes de eso `$refs` apunta a nodos viejos.
- Un botón con solo emoji necesita nombre accesible: `aria-label` y `title` con el texto real. Sin eso el lector de pantalla anuncia el nombre Unicode del emoji.
- `✏️` para editar y `📝` para añadir no son intercambiables: el dibujo comunica la acción antes de leer el `title`.
- Matcha.css estiliza `button`; un botón de emoji sin caja se logra pisando `background`, `border`, `padding` y `font`, no con `appearance: none`.
- La regla `.fold-node > p` tiene más especificidad que `.summary-line` y le ganaba el margen. La solución fue `.fold-node > .summary-line` en lugar de subir con `!important`.
- Sin `trim()` a propósito: una frase de solo espacios es válida y la cadena vacía queda reservada para borrar. Cualquier recorte silencioso haría desaparecer frases que el usuario quiso guardar así.
- El orden “frase arriba, original abajo” es solo visual. Ocultar el original ya es plegado y pertenece a otra spec; mezclarlo acá habría cambiado el alcance a mitad de camino.
- Cambiar de opinión durante la implementación se paga barato si el cambio se refleja en la spec en el mismo commit: orden, emojis y `trim` quedaron documentados junto al código.

### Verificación

- `$nextTick` y el foco se comprueban con `document.activeElement` (`tagName` y `aria-label`), no con capturas.
- En este entorno Playwright no dispara `requestAnimationFrame` de forma estable y sus clics normales se quedan esperando “estable”. Se verifica con `click({ force: true })` sobre elementos que ya se comprobó que están visibles y en el tope del hit-test.
- Para probar el HTML literal, cargar un JSON de prueba cuyo `summary` contenga `<script>` y comprobar que aparece el texto y que no se ejecuta nada.

