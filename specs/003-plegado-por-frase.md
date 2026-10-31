# SPEC 003 — Plegado por frase alternativa y gestos con clic

> **Estado:** Aprobado
> **Depende de:** SPEC 001, SPEC 002
> **Fecha:** 2026-10-30
> **Objetivo:** Permitir que todo nodo con frase alternativa arranque plegado mostrando solo la frase, y alternar o editar con clic y doble clic.

## Por qué existe

El lector ya muestra y edita las frases alternativas, pero el texto original sigue siempre visible. El README describe una lectura “plegable”: primero las frases que resumen, y el original disponible al desplegar. Esta spec conecta esas dos piezas.

## Alcance

**Incluye:**

- Todo nodo con `summary` arranca plegado, en cualquier nivel.
- Plegado significa: se ve la frase y un caret `▸`; se oculta el texto propio y, si tiene, todo su subárbol.
- Un clic en la frase alterna plegado/desplegado. El caret pasa a `▾` al desplegar.
- Un doble clic en la frase la despliega y abre el editor de esa frase.
- Un doble clic en el texto propio de un nodo abre el editor de su frase: añadir si no tiene, editar si tiene.
- El control `✏️`/`📝` sigue editando con un clic y no alterna el plegado.
- Nodos sin `summary` se ven como hoy: sin caret y sin plegado.

**Fuera de alcance (para specs futuras):**

- Plegar o desplegar todo el documento de una vez.
- Persistir el estado plegado entre recargas o archivos.
- Exportar el JSON con las frases editadas.
- Editar `text`, añadir nodos, borrar nodos o reordenarlos.
- Animaciones de despliegue y transiciones de altura.
- Plegado por atajos de teclado distintos del foco en el botón.

## Modelo de datos

No cambia el JSON de entrada. El plegado es estado de sesión del componente:

```js
// Estado local de cada FoldNode, junto a editing y draft
expanded: false

// Derivados
isFolded   = node.summary !== '' && !expanded
showContent = !isFolded
```

Convenciones:

- `expanded` no se guarda en `doc` ni se serializa.
- `node.summary === ''` es no plegable: el texto propio y los hijos se muestran siempre.
- Al guardar una frase el nodo queda expandido, para ver el resultado de la edición.
- Al borrar la frase el nodo deja de ser plegable y muestra su contenido.
- La frase y su caret forman el botón de alternar. El `✏️` es un botón hermano, no anidado, y detiene la propagación para no alternar.
- Un nodo plegado no renderiza su texto ni sus hijos; no alcanza con ocultarlos con CSS, para no dejar controles enfocables dentro de contenido invisible.

## Plan de implementación

1. Plegado inicial: añadir `expanded: false` y los derivados `isFolded`/`showContent`; envolver texto propio y `.children` en `v-if="showContent"`; mostrar caret `▸` junto a la frase. Verificación manual: con `demo.json`, los 4 nodos con `summary` muestran solo su frase y los 3 sin `summary` muestran su texto.
2. Alternar con clic: convertir la frase y el caret en un `<button>` con `@click="toggleFold"`, `aria-expanded` y caret `▸`/`▾`; añadir `@click.stop` al `✏️`/`📝` para que no alterne. Verificación manual: clic en una frase pliega y despliega sin abrir el editor.
3. Doble clic: `@dblclick` en el botón de la frase hace `expanded = true` y `startEdit()`; `@dblclick` en el texto propio llama a `startEdit()` con el `summary` actual (o vacío si no tiene). Verificación manual: doble clic en una frase pliega y abre el editor; doble clic en un párrafo sin frase abre “añadir”.
4. Estilos y accesibilidad: caret sin caja, `cursor: pointer`, foco visible del botón de alternar y ajuste de la línea del summary con dos botones. Verificación manual con teclado: `Tab` hasta la frase, `Enter` alterna y `aria-expanded` cambia.
5. Actualizar `LESSONS.md` con la entrada de la spec 003.

## Criterios de aceptación

- [ ] Con `demo.json`, los 4 nodos con `summary` aparecen plegados y muestran solo su frase con caret `▸`; los 3 sin `summary` muestran su texto y no tienen caret.
- [ ] Un capítulo plegado no muestra su encabezado ni sus hijos.
- [ ] Al desplegar un capítulo aparecen su encabezado y sus hijos, y cada hijo con frase sigue plegado.
- [ ] Un clic en la frase alterna plegado/desplegado y el caret cambia entre `▸` y `▾`.
- [ ] Un clic en `✏️` o `📝` abre el editor y no alterna el plegado.
- [ ] Un doble clic en la frase la deja desplegada y abre el editor con su frase y foco en el `textarea`.
- [ ] Un doble clic en un párrafo sin `summary` abre el editor vacío para añadir la frase.
- [ ] Un doble clic en el texto de un nodo desplegado con frase abre el editor con esa frase.
- [ ] Guardar una frase deja el nodo expandido; borrarla lo vuelve no plegable.
- [ ] Con teclado, `Enter` sobre el botón de la frase alterna y `aria-expanded` refleja el estado.
- [ ] `text` no cambia en ningún caso.
- [ ] El plegado no persiste: al reabrir el JSON todo vuelve al estado plegado inicial.
- [ ] Funciona abriendo `public/index.html` directamente (`file://`), sin servidor.

## Decisiones

- **Sí:** plegar todos los niveles desde el inicio. Es la vista de resumen que describe el README y hace visible el valor de las frases.
- **Sí:** estado `expanded` local por `FoldNode`. La recursión ya aísla cada nodo y no hace falta guardarlo en `doc`.
- **Sí:** no renderizar el contenido plegado (`v-if`) en vez de ocultarlo con CSS. Evita botones y enlaces enfocables dentro de contenido invisible.
- **Sí:** caret `▸`/`▾` y `aria-expanded`. El caret hace descubrible el clic y el atributo lo comunica a lectores de pantalla.
- **Sí:** frase y caret como `<button>` real y `✏️` como botón hermano. Mantiene el teclado y evita botones anidados.
- **Sí:** clic inmediato y doble clic que fuerza “desplegar + editar”. El doble clic hace dos alternadas y luego deja el estado correcto; se acepta el parpadeo.
- **No:** retrasar el clic 250 ms para detectar el doble clic. Hace que el plegado se sienta lento para el uso principal.
- **No:** animación de altura. Agrega complejidad y no aporta a esta iteración.
- **No:** persistir el plegado. El JSON no tiene ese campo y el estado de lectura es de la sesión.
- **No:** plegar nodos sin frase. Sin frase no hay nada que mostrar en su lugar.

## Riesgos

| Riesgo                                                        | Mitigación                                                                 |
| ------------------------------------------------------------- | -------------------------------------------------------------------------- |
| El doble clic produce un parpadeo visible                      | `dblclick` es idempotente: deja `expanded = true` y abre el editor.        |
| Usuarios de teclado no tienen doble clic                       | El `✏️` sigue siendo un botón accesible con foco y `Enter`.                 |
| Un nodo plegado con hijos deja contenido enfocable si se oculta con CSS | Se usa `v-if`, no `display: none`, para no renderizar el subárbol. |
| Se pierde el estado de plegado al reabrir                       | Es comportamiento declarado; la persistencia va en otra spec.              |

## Lo que **no** está en esta spec

- Plegar todo el documento, persistencia y exportación.
- Edición del texto original o de la estructura de nodos.
- Animaciones y atajos globales.

Cada uno, si llega, va en su propia spec.
