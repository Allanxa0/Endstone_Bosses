CLAUDE.md

Plugin de Endstone (Minecraft Bedrock, Python). Código limpio, modular y escalable.

Estilo (Python)

- Seguir la configuración de Ruff del repo: línea de 120, reglas "I" (isort) y "F" (pyflakes).
- Formato mecánico: "ruff format"; no formatear a mano contra el formatter.
- Clases "CamelCase"; funciones, métodos, variables y propiedades "snake_case"; constantes "UPPER_CASE"; privados con "_prefijo".
- Nombres en inglés, profesionales y cortos.
- Type hints en firmas públicas.
- Respetar convenciones de metadatos de plugins Endstone.
- Mantener las constantes junto al módulo responsable cuando su uso es local.
- Centralizar únicamente los valores compartidos por varios módulos o que representen una configuración común.
- Separar definiciones estáticas, catálogos y tablas de datos cuando su volumen o responsabilidad lo justifique.
- No convertir automáticamente textos, etiquetas, títulos, mensajes ni valores numéricos en constantes. Mantenerlos directamente en el lugar de uso cuando sean simples, locales y no requieran reutilización.
- Crear constantes únicamente cuando representen valores compartidos, configuraciones, límites importantes o datos cuyo mantenimiento centralizado aporte un beneficio real.
- Evitar archivos de constantes que acumulen textos de interfaz y valores triviales sin una necesidad concreta.
- Priorizar enums, constantes y tipos nativos de las APIs antes que duplicar sus valores manualmente.
- Evitar valores mágicos y estructuras de datos estáticas declaradas directamente en listeners, comandos y managers.
- No crear módulos, interfaces, contratos o abstracciones sin una necesidad concreta.
- Evitar duplicar definiciones y mantener una única fuente de verdad para los datos compartidos.
- Utilizar exclusivamente importaciones absolutas, incluidas las realizadas dentro de "TYPE_CHECKING".

Uso de constantes

- No crear constantes para valores locales, triviales o exclusivos de una implementación.
- No extraer automáticamente números, cadenas, rutas, identificadores, colores, formatos ni etiquetas a constantes.
- Los valores utilizados únicamente en un comando, formulario, listener o método deben permanecer directamente en su lugar de uso, salvo que exista una razón concreta para separarlos.
- No crear constantes para límites de sliders, intervalos, pasos numéricos o valores predeterminados de formularios cuando su uso sea local.
- No crear constantes para iconos, títulos, etiquetas, colores o formatos de interfaz que solo se utilicen en una pantalla o formulario.
- No crear constantes intermedias cuyo único propósito sea asignar un nombre a un valor literal.
- No agrupar valores relacionados en bloques de constantes únicamente por pertenecer a la misma funcionalidad.
- Crear constantes solo cuando exista reutilización real entre distintos módulos, configuración compartida o una necesidad clara de mantenimiento centralizado.
- Antes de crear una constante, evaluar si aporta una ventaja real frente a escribir el valor directamente donde se utiliza.
- Ante la duda, preferir valores locales y explícitos antes que crear nuevas constantes.

Comentarios

- Por defecto, sin comentarios.
- Si hace falta uno, una sola línea sobre una restricción no obvia.
- Prohibido: explicaciones largas, narrar decisiones, historial, apartes entre paréntesis.
- Prohibido dejar "notas de LLM": por qué se hizo un cambio, referencias al proceso de desarrollo/revisión, o repetir lo que el código ya dice.
- Igualar la densidad de comentarios del código vecino.

Docstrings (API pública)

- Solo cuando el contrato no se infiere de la firma.
- Documentar únicamente lo que el llamador puede asumir: propósito, parámetros, "None", retorno/fallo, hilo.
- No documentar detalles privados, estructuras internas ni historial.
- Breves; nunca documentos de diseño.

Errores

- Sin chequeos redundantes. Dejar que los errores se manifiesten.
- try/except solo cuando sea estrictamente necesario.

API

- No inventar ni suponer API. Verificar en la documentación proporcionada que cada método, evento o propiedad exista antes de usarlo.
- Si no existe, decirlo y proponer alternativa.

Arquitectura

- Separar por responsabilidad (config, commands, listeners, managers, models).
- "plugin.py" solo hace el bootstrap y delega.
- Respetar la estructura existente; cambios mínimos y enfocados.

Rendimiento

- Evitar operaciones repetitivas, cálculos innecesarios y consultas duplicadas.
- Reutilizar resultados mediante caché cuando exista un beneficio real.
- No bloquear el hilo principal del servidor con operaciones pesadas o de E/S.
- Utilizar ejecución asíncrona únicamente cuando la API lo permita y sea seguro para el estado del servidor.
- Evitar crear tareas, objetos y estructuras temporales innecesariamente.

Mantenimiento

- Evitar duplicación de lógica cuando exista una responsabilidad compartida.
- No introducir dependencias externas para resolver problemas que Python o Endstone ya pueden resolver.
- Eliminar código muerto, imports sin uso y dependencias innecesarias.
- No modificar código ajeno al objetivo solicitado.

Git

- Commits enfocados y fáciles de revertir.
- Nunca añadir "Co-Authored-By" de Claude.
- No hacer force-push ni reescribir historial publicado.

Changelog

- Cambios visibles para el usuario bajo "## [Unreleased]" en "CHANGELOG.md" (Keep a Changelog).
- Escribir para admins del servidor; omitir refactors internos.