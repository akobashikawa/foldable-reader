# SPEC 001 — Cargar texto estructurado desde JSON

> **Estado:** Aprobado
> **Depende de:** —
> **Fecha:** 2026-10-30
> **Objetivo:** Permitir que el usuario elija un JSON con texto estructurado y lo vea renderizado en pantalla, sin recargar la página ni usar servidor.

## Por qué existe

Es el primer paso del lector: sin una fuente de texto no hay nada que plegar. Se elige JSON porque es el mismo formato del archivo auxiliar de frases alternativas descrito en el `README.md`.

## Alcance

**Incluye:**

- Selector de archivo local `.json`.
- Lectura y parseo en el navegador.
- Validación del formato con mensajes en español que indican el nodo con problema.
- Render jerárquico del contenido.
- Estados de vacío, carga y error.
- Recarga de documento sin recargar la página.

**Fuera de alcance (para specs futuras):**

- Cargar por URL, arrastrar y soltar, `.txt`, `.epub`, `.pdf`.
- Plegado y despliegue de frases alternativas.
- Edición del texto o del JSON.
- Persistencia y múltiples documentos abiertos.
- Búsqueda, índice y atajos de teclado.

## Modelo de datos

```json
{
  "version": 1,
  "title": "La metamorfosis",
  "author": "Franz Kafka",
  "nodes": [
    {
      "id": "cap-1",
      "text": "Una mañana, al despertar de sueños intranquilos...",
      "summary": "Gregorio despierta convertido en insecto.",
      "children": [
        { "id": "cap-1-p1", "text": "Su habitación era ..." }
      ]
    }
  ]
}
```

| Campo      | Obligatorio | Regla                                                          |
| ---------- | ----------- | -------------------------------------------------------------- |
| `version`  | no          | Si existe debe ser `1`; otro valor es error de versión.         |
| `title`    | sí          | Texto no vacío.                                                 |
| `author`   | no          | Texto.                                                          |
| `nodes`    | sí          | Lista; puede estar vacía.                                       |
| `id`       | no          | Texto; si falta se genera `node-N` según la posición.           |
| `text`     | sí          | Texto no vacío; es lo que se muestra.                           |
| `summary`  | no          | Texto; se guarda para el plegado, todavía no se renderiza.      |
| `children` | no          | Lista de nodos con la misma forma (recursivo).                  |

Convenciones:

- Las claves desconocidas se ignoran para poder ampliar el formato sin romper.
- El documento normalizado vive en un `ref` llamado `doc`.
- Los ids generados son estables durante la sesión porque derivan de la ruta (`node-0`, `node-0-1`).

## Plan de implementación

1. Maquetar la vista base: botón “Abrir JSON”, input oculto, estado vacío y contenedor de error con `role="alert"`.
2. Implementar `validateDoc(raw)` como función pura que devuelve `{ doc }` o `{ error }`, con validación recursiva y rutas tipo `nodes[0].children[1].text`.
3. Implementar `normalizeNode(node, fallbackId)` para copiar solo los campos conocidos y asignar ids faltantes.
4. Crear el componente recursivo `fold-node`: encabezado si tiene hijos (tope `h3`), párrafo si es hoja.
5. Conectar el input con `file.text()`, manejar estados de carga y error, y reemplazar el documento al terminar.
6. Añadir estilos de lectura y probar los casos límite en navegador.

## Criterios de aceptación

- [ ] Cargar el ejemplo del spec muestra título, autor y el árbol de nodos en orden.
- [ ] Cambiar de archivo reemplaza el contenido anterior.
- [ ] Un JSON inválido muestra un error claro y conserva lo que ya estaba en pantalla.
- [ ] Un archivo con `version: 2` muestra error de versión.
- [ ] Un `nodes: []` muestra el título y un aviso de documento sin contenido.
- [ ] El texto con `<script>` o `&lt;` se muestra literal, sin ejecutarse.
- [ ] Funciona abriendo `public/index.html` directamente (`file://`), sin servidor.
- [ ] Un nodo con `text` vacío produce un error que indica su ruta exacta.

## Decisiones

- **Sí:** `<input type="file">` + `file.text()` en vez de `fetch`. Evita CORS y permite abrir la app con `file://`.
- **No:** cargar una URL en esta iteración. Requiere servidor o CORS y pertenece a otra spec.
- **Sí:** validación pura y separada de la normalización. Permite mensajes con ruta sin mezclar DOM.
- **Sí:** guardar `summary` aunque no se muestre. Deja el modelo listo para la spec de plegado sin migrar datos.
- **Sí:** hojas como `<p>` y nodos con hijos como encabezado con tope `h3`. Prioriza legibilidad sobre fidelidad estructural.
- **No:** `v-html`. El JSON es contenido no confiable y la interpolación de Vue ya escapa el HTML.
- **No:** drag & drop. Se pospone junto con los demás formatos de entrada.

## Riesgos

| Riesgo                                              | Mitigación                                                                 |
| --------------------------------------------------- | -------------------------------------------------------------------------- |
| Un archivo muy grande bloquea el render             | Sin optimización anticipada; si aparece, se medirá antes de virtualizar.    |
| Sin red no cargan Vue ni Matcha.css                 | La app no funciona offline; se acepta mientras sean CDN.                    |
| Matcha.css pisa el `hidden` de `input[type=file]`    | La clase `.file-input { display: none }` oculta el input.                   |

## Lo que **no** está en esta spec

- Plegado de frases alternativas.
- Carga por URL, arrastrar y soltar, `.txt`, `.epub`, `.pdf`.
- Edición, persistencia, búsqueda y atajos.

Cada uno, si llega, va en su propia spec.
