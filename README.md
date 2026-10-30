# Foldable Reader

## Ideas
- Foldable Reader permite presentar un texto con una interfaz que facilita su lectura y estudio.
- Puede cargar un texto, ya sea pegado por el usuario en un formulario, o de un archivo txt, json, epub, pdf, o de un url.
- Cierta parte del texto representa una idea que se podría representar con una frase alternativa. Foldable permite mostrar esa frase alternativa en lugar de la original y tener la original para desplegar debajo.
- Si cada parte del texto es representado con una frase alternativa, uno podría tener un resúmen constituido por esas frases, cada una de las cuales tiene un control para desplegar la original como un detalle.
- A su vez, el texto formado por las frases alternativas puede también ser representado con una frase alternativa. De ese modo, frases alternativas representan parrafos, secciones, capítulos o el libro completo.
- Llamemos a esta visualización foldable (plegable), debido a que podríamos leerlo rápidamente plegando contenido y leyendo las frases representativas, o profundizar según nuestro interés.
- El manejo de las frases alternativas se hace de manera no intrusiva, en un archivo auxiliar que el reader consolida con el texto original para obtener el foldable.

## Tecnologías

- Vue vía CDN
- HTML
- CSS

## Estructura

- `public/index.html`: toda la aplicación (HTML, CSS y JS embebidos).
- `public/` es la raíz de despliegue: solo contiene archivos estáticos listos para publicar. La documentación (`README.md`, `LESSONS.md`, `AGENTS.md`) queda fuera de esa carpeta.
- `specs/`: una especificación por funcionalidad (`NNN-nombre.md`), escrita antes de implementar. Las reglas del flujo están en `specs/.spec-config.yml` y en `.agents/skills/`.
- Para probar localmente, servir esa carpeta: `python3 -m http.server --directory public`.

## Metodología de desarrollo

- Progresivo, desde lo más simple, a través de pasos no complejos, iterando de manera orgánica para ir agregando características o haciendo correcciones, usando Spec Driven Development.
- El agente mantiene un archivo LESSONS.md destinado a la lectura del desarrollador, donde se le va contando acerca de las ideas usadas para la implementación, las estrategias, y las lecciones que se van aprendiendo.