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
- Al mover un elemento dentro de un wrapper cambia el alcance de los selectores: `.fold-node > .summary-line` dejó de coincidir cuando la frase pasó a estar dentro de `.node-head`, y la cursiva y el color atenuado desaparecieron sin error visible. El arreglo fue `.node-head > .summary-line`; no era un problema de especificidad.
- Sin `trim()` a propósito: una frase de solo espacios es válida y la cadena vacía queda reservada para borrar. Cualquier recorte silencioso haría desaparecer frases que el usuario quiso guardar así.
- El orden “frase arriba, original abajo” es solo visual. Ocultar el original ya es plegado y pertenece a otra spec; mezclarlo acá habría cambiado el alcance a mitad de camino.
- Cambiar de opinión durante la implementación se paga barato si el cambio se refleja en la spec en el mismo commit: orden, emojis y `trim` quedaron documentados junto al código.
- Ocultar un control con `opacity: 0` y no con `display: none` deja el espacio reservado, mantiene el orden de tabulación y permite que `:focus-visible` lo revele. El precio es que sigue existiendo para el lector de pantalla, que es justo lo que se busca.
- El hover de un ancestro se activa también al pasar por sus descendientes. Para encender solo el icono del nodo apuntado hay que mirar hijos directos: `:has(> .node-head:hover, > h2:hover, > p:hover)`. Un `.fold-node:hover` a secas encendería también los iconos de los hijos.
- Para el clic hay un ida y vuelta que se pisa si falta una pieza: el `section` activa y hace `.stop`, y un listener de `document` apaga todo. Sin `.stop`, el mismo clic que activa llega al documento y apaga el nodo en el acto.
- Un único `activeId` en el root es más simple que un booleano por componente: garantiza que haya como máximo un nodo encendido y que el clic fuera apague todo sin recorrer el árbol.
- Para la verificación conviene leer `getComputedStyle(boton).opacity` en cada estado: reposo (`0`), hover (`0.6`), activo (`0.6`) y foco de teclado (`1`).

### Verificación

- `$nextTick` y el foco se comprueban con `document.activeElement` (`tagName` y `aria-label`), no con capturas.
- En este entorno Playwright no dispara `requestAnimationFrame` de forma estable y sus clics normales se quedan esperando “estable”. Se verifica con `click({ force: true })` sobre elementos que ya se comprobó que están visibles y en el tope del hit-test.
- Las transiciones CSS tampoco avanzan en ese navegador: `getComputedStyle` devuelve el valor inicial aunque pasen segundos. Se quitó la transición de `opacity` para que el estado sea inmediato y verificable; con `transition: none` los valores coincidían, así que el problema era el frame, no el selector.
- Para probar el HTML literal, cargar un JSON de prueba cuyo `summary` contenga `<script>` y comprobar que aparece el texto y que no se ejecuta nada.

## 003 — Plegado por frase alternativa

- Plegar es estado de sesión: `expanded` local en cada `FoldNode`, y `isFolded = summary !== '' && !expanded`. No va al JSON ni a `doc`.
- Para el contenido plegado conviene `v-if` y no CSS: un subárbol que no se ve no debe existir en el DOM, o quedarían botones enfocables dentro de contenido invisible.
- El plegado inicial de todos los niveles reduce el árbol renderizado: con el demo quedan 3 secciones visibles, no 7. Los criterios de conteo tienen que hablar de lo que se renderiza, no de todos los nodos del JSON.
- La frase y el caret son un `<button>` real, y el `✏️` es un botón hermano. Eso evita botones anidados y da teclado gratis (`Enter`/espacio) sin listeners extra.
- El clic del `✏️` no dispara el de alternar porque son hermanos, no ancestro y descendiente. Agregar `.stop` “por las dudas” bloquea además la activación del nodo y esconde los iconos.
- `aria-expanded` en el botón de alternar y `aria-hidden` en el caret: el estado se anuncia y el nombre accesible queda limpio, solo con la frase.
- Un doble clic dispara antes dos `click`: el plegado se ejecuta dos veces y vuelve al estado inicial. Después, `dblclick` fuerza `expanded = true` y abre el editor; hacerlo idempotente evita depender de si el nodo estaba plegado.
- El foco visible del botón de alternar se declara a mano. El reset de `border`/`background` y el estilo de Matcha no garantizan un anillo legible.
- Los errores de alcance del CSS no avisan: `.fold-node > .summary-line` no coincidía desde que la frase vive dentro de `.node-head`, y la cursiva había desaparecido sin error. Conviene leer `getComputedStyle` cada vez que se mueve el DOM.

### Verificación

- `page.mouse` usa coordenadas del viewport. Para nodos fuera de pantalla hay que scrollear antes con `el.scrollIntoView()` por `evaluate`; `scrollIntoViewIfNeeded` también espera “estable” y se cuelga en este entorno.
- El doble clic se prueba con `page.mouse.dblclick` sobre el centro del `boundingBox`, no con `locator.dblclick`, por el mismo motivo.
- El plegado se verifica sobre el DOM: cantidad de `.fold-node`, presencia de `h2`/`.children`, `aria-expanded` y el carácter del caret.


