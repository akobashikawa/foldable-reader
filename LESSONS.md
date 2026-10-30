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
