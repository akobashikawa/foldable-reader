# SPEC 002 — Establecer la frase alternativa de un nodo

> **Estado:** Implementado
> **Depende de:** SPEC 001
> **Fecha:** 2026-10-30
> **Objetivo:** Permitir que el usuario escriba, edite o borre la frase alternativa (`summary`) de cada nodo desde el propio lector, sin tocar el texto original.

## Por qué existe

El formato ya reserva `summary` para la frase alternativa y el demo lo trae poblado, pero el lector todavía no lo muestra ni permite cambiarlo. Sin esta pieza no hay forma de crear las frases que el plegado va a usar después.

## Alcance

**Incluye:**

- Mostrar la frase alternativa existente antes del texto propio de cada nodo.
- Un control con emoji (`✏️` para editar, `📝` para añadir) después de la frase, o al inicio del nodo cuando todavía no hay frase.
- Iconos ocultos por defecto: se muestran al pasar el mouse por el texto propio del nodo, al hacer clic/tap en el nodo o al recibir foco por teclado; se ocultan al hacer clic fuera.
- Editor inline con `Guardar` y `Cancelar`.
- Borrar la frase dejando el campo vacío.
- Feedback inmediato: la frase guardada se ve en cursiva sin recargar.
- Estado local de edición por nodo, compatible con el componente recursivo.

**Fuera de alcance (para specs futuras):**

- Plegado: mostrar la frase en lugar del original, con el original desplegable debajo.
- Exportar o guardar en disco el JSON con las frases editadas.
- Persistencia entre recargas o entre archivos.
- Editar `text`, añadir nodos, borrar nodos o reordenarlos.
- Edición del documento completo, deshacer/rehacer y atajos globales.

## Modelo de datos

Esta spec no introduce estructuras nuevas. Reutiliza el modelo de SPEC 001 y le da significado editable a `summary`:

```json
{
  "id": "cap-1-p1",
  "text": "El viento golpeaba las ventanas desde el mediodía...",
  "summary": "Elías sube al faro al comenzar la tormenta."
}
```

Convenciones:

- `summary` es la frase alternativa del nodo. `''` (campo completamente vacío) significa “sin frase alternativa”.
- Al guardar se conserva el texto tal cual, sin `trim()`. Un valor de solo espacios es una frase válida.
- `node.text` es inmutable en esta funcionalidad: solo se asigna `node.summary`.
- Orden de render por nodo: frase alternativa con el control `✏️` al final (o solo el control `📝` si no hay frase) o editor, texto original, hijos.
- El texto original nunca se oculta: esto es solo orden visual, no plegado.
- No hay store paralelo: el nodo vive dentro de `doc` (`ref` profundo) y mutar `node.summary` actualiza la vista.
- El estado del editor (`editing`, `draft`) es local a cada instancia de `FoldNode`; no se guarda en el documento.
- El nodo activo no se duplica por componente: el root guarda un único `activeId` y cada `FoldNode` recibe `activeId` como prop y calcula `isActive`.
- El clic dentro de un nodo usa `.stop`: activa ese nodo y evita que el listener de `document` apague el estado.
- Un clic fuera de cualquier nodo vuelve a poner `activeId = null`.
- El hover se resuelve solo con CSS (`:has()` sobre los hijos directos del `section`); no hay estado de hover en Vue.

## Plan de implementación

1. Renderizar la frase alternativa existente: en `FoldNode`, antes del encabezado o párrafo propio, mostrar `node.summary` como línea en cursiva con clase `.summary-line`. Verificación manual: con `public/texts/demo/demo.json`, los nodos con `summary` muestran su frase arriba del texto y los que no, nada.
2. Añadir el control y el editor: el control va al final de la frase cuando existe (`✏️`) y solo, antes del texto, cuando no (`📝`); su `aria-label` y su `title` son `Editar frase alternativa` o `Añadir frase alternativa` según corresponda. Al pulsarlo mostrar, en ese mismo bloque superior, un `textarea` con el valor actual y botones `Guardar` / `Cancelar`. Verificación manual: añadir una frase a un nodo sin `summary` la muestra en cursiva arriba del texto al guardar.
3. Cerrar la edición: `Guardar` asigna el texto tal cual (sin `trim()`), `Cancelar` descarta el borrador, `Esc` equivale a `Cancelar`, y al cerrar el foco vuelve al control que abrió el editor. Para quitar la frase hay que dejar el campo completamente vacío. Verificación manual: vaciar del todo y guardar elimina la frase; un texto de solo espacios se conserva como frase.
4. Visibilidad progresiva: los iconos nacen con `opacity: 0`; se revelan con hover sobre el texto propio del nodo (`:has()` y `@media (hover: hover)`), con el foco de teclado (`:focus-visible`) o cuando el nodo está activo (`is-active`). El clic en el `section` emite `activate` con el id y detiene la propagación; un listener de `document` limpia `activeId` en los clics fuera. Verificación manual: pasar el mouse por un párrafo, hacer clic y luego clic fuera.
5. Actualizar `LESSONS.md` con la entrada de la spec 002.

## Criterios de aceptación

- [x] Cada nodo muestra un control con emoji, también los que tienen hijos.
- [x] Con `demo.json` cargado, los nodos con `summary` muestran la frase en cursiva antes del texto, con el `✏️` después de la frase; los que no tienen frase muestran solo el `📝` antes del texto.
- [x] El `aria-label` y el `title` del control dicen `Editar frase alternativa` cuando hay frase y `Añadir frase alternativa` cuando no.
- [x] Añadir una frase la muestra en cursiva sin recargar la página.
- [x] Editar abre el editor con la frase actual precargada.
- [x] Vaciar por completo y guardar elimina la frase; queda solo el `📝` antes del texto.
- [x] Una frase de solo espacios se guarda como frase válida: aparece el `summary-line` (vacío a la vista) con su `✏️`.
- [x] `Cancelar` y `Esc` descartan los cambios del borrador.
- [x] El texto original (`text`) no cambia en ningún caso.
- [x] Reabrir el mismo JSON restaura los `summary` del archivo: lo editado no persiste.
- [x] Un `summary` con `<script>` o `&lt;` se muestra literal, sin ejecutarse.
- [x] Funciona abriendo `public/index.html` directamente (`file://`), sin servidor.
- [x] En reposo todos los iconos tienen `opacity: 0`.
- [x] El hover sobre el texto propio del nodo muestra su icono; el hover sobre un nodo hijo no enciende el icono del padre.
- [x] Un clic/tap en el nodo lo deja activo aunque el mouse se vaya; un clic en otro nodo mueve el activo y un clic fuera apaga todos.
- [x] El foco por teclado muestra el icono.
- [x] Cargar otro documento apaga el nodo activo.

## Decisiones

- **Sí:** editar `node.summary` en lugar de crear un mapa auxiliar `summaries[nodeId]`. El campo ya existe en el modelo y un store paralelo sería abstracción antes de tiempo.
- **Sí:** mutar el nodo dentro de `doc`. Es un `ref` profundo de Vue y evita propagar eventos o duplicar estado.
- **Sí:** control `✏️` al final de la frase, o solo antes del texto si no hay frase. Queda junto a lo que edita y no compite con el texto.
- **Sí:** dos emojis con `aria-label` y `title`: `✏️` para editar una frase existente y `📝` para añadir una nueva. El dibujo comunica la acción a simple vista y el nombre accesible conserva el significado para lectores de pantalla y mouseover.
- **Sí:** control y frase antes del texto propio, en un bloque superior del nodo. La frase resume lo que viene y el original queda visible debajo.
- **Sí:** mostrar la frase guardada como línea en cursiva. Da confirmación visual y prepara el plegado sin implementarlo.
- **No:** ocultar o colapsar el texto original en esta spec. Eso es plegado y va en la spec siguiente.
- **Sí:** guardar el borrador tal cual, sin `trim()`. Una frase de solo espacios es válida; la cadena vacía queda reservada para borrar.
- **Sí:** iconos ocultos por defecto con `opacity: 0` en vez de `display: none` o `visibility: hidden`. Conservan el espacio, el orden de tabulación y el nombre accesible.
- **Sí:** un único `activeId` en el root en vez de un `revealed` por componente. Evita que varios nodos queden encendidos y hace trivial el “clic fuera apaga todos”.
- **Sí:** `.stop` en el clic del nodo. Sin él, el mismo clic llegaría al listener de `document` y apagaría el nodo recién activado.
- **Sí:** hover con `:has()` en CSS. No necesita estado reactivo ni eventos por nodo, y el selector con hijos directos evita encender los iconos de los descendientes.
- **No:** transición de `opacity`. El cambio inmediato es suficiente y no depende de frames ni animaciones; además el estado se puede verificar con `getComputedStyle`.
- **Sí:** `Guardar` y `Cancelar` explícitos, con `Esc` para cancelar. Evita que un clic accidental fuera del campo guarde cambios no deseados.
- **Sí:** `textarea`. La frase puede ser más larga que una línea y el ajuste de texto es más legible que en un `input`.
- **No:** autoguardado al perder el foco. Complica explicar el descarte y el borrado.
- **No:** persistencia o exportación. Requiere definir el archivo auxiliar y su descarga; va en otra spec.
- **No:** `v-html`. La interpolación de Vue ya escapa el contenido del JSON.

## Riesgos

| Riesgo                                                        | Mitigación                                                                       |
| ------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| El usuario espera que las frases se guarden al recargar        | Es una limitación declarada en el alcance; la exportación se define en otra spec. |
| Un clic en el control de otro nodo deja un editor abierto       | Cada `FoldNode` maneja su propio estado `editing`; no hay estado compartido.      |
| El editor inline rompe la jerarquía visual de `.children`      | El editor se renderiza dentro del bloque propio del nodo y los estilos no tocan `.children`. |

## Lo que **no** está en esta spec

- Plegado y despliegue de frases alternativas.
- Exportar, importar o persistir las frases editadas.
- Edición del texto original, alta y baja de nodos.
- Deshacer/rehacer, atajos globales y edición del documento completo.

Cada uno, si llega, va en su propia spec.
