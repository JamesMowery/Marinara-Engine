# Onboarding interactivo

Esta guía explica el **onboarding interactivo** (incorporación interactiva), una función de las tarjetas de personaje que permite a los jugadores crear su persona respondiendo a las preguntas de quien creó la tarjeta cuando empiezan un chat de Roleplay. Cubre cómo se configura en el Character Editor y qué ven los jugadores en el asistente de chat nuevo.

## Qué hace

Una persona es el personaje que te representa en un chat. Consulta [Personas de usuario](personas.md). Normalmente escribes tú una persona antes de jugar. Con el onboarding interactivo, una tarjeta de personaje trae en su lugar una plantilla de persona. La plantilla está formada por los cinco campos de persona, **Description** (Descripción), **Personality** (Personalidad), **Backstory** (Trasfondo), **Appearance** (Apariencia) y **Scenario** (Escenario), con espacios en blanco que rellena el jugador, más las preguntas que los rellenan.

Cuando un jugador empieza un chat de Roleplay con la tarjeta, puede elegir **Create your persona** (Crear tu persona), responder a las preguntas y obtener una persona nueva y corriente. Las respuestas se escriben en el texto una sola vez. El resultado es una persona normal que el jugador puede editar, conservar o eliminar como cualquier otra. La tarjeta en sí no cambia.

El onboarding usa la sintaxis de macros existente. No ejecuta ningún script.

## Configurarlo en tu tarjeta

1. Abre el personaje en el Character Editor. Consulta [Crear y editar personajes](creating-and-editing-characters.md).
2. Abre la pestaña **Onboarding**, junto a **Card** (Tarjeta).
3. Activa **Interactive onboarding** (Onboarding interactivo). Debajo del interruptor aparece una breve guía, **How onboarding works** (Cómo funciona el onboarding).

Desactivar el interruptor oculta los campos de onboarding, pero conserva lo que escribiste.

### Añadir preguntas

En **Onboarding Questions** (Preguntas de onboarding), haz clic en **Add Question** (Añadir pregunta). Cada pregunta funciona como una variable de preset. Consulta [Presets](../prompts/presets.md). Una tarjeta puede tener hasta 100 preguntas.

- **Variable Name** es el nombre que usas en los campos, como `faction`. Elige cualquier nombre con letras, números y guiones bajos que empiece por una letra o un guion bajo. Los nombres distinguen mayúsculas de minúsculas: `Gender` y `gender` son distintos.
- **Question (shown to user)** es lo que lee el jugador. Puede usar respuestas anteriores, como `What does {{player}} look like?`. Aquí `{{char}}` es el nombre de la tarjeta y `{{user}}`, el nombre del jugador.
- Las **Options** convierten la pregunta en una elección. La primera opción está seleccionada de forma predeterminada. Sin opciones, el jugador escribe una respuesta libre.
- **Value** es lo que entra en la persona. Las mayúsculas y minúsculas se mantienen tal como las escribes; las macros que contenga se rellenan como el resto de los campos. Déjalo en blanco para usar la etiqueta de la opción. Escribe un valor en minúsculas, como `ranger`, si la respuesta va en mitad de una frase.
- **Help text for players** (Texto de ayuda para jugadores) es un texto opcional que se muestra debajo de la opción.
- **Allow own answer** (Permitir respuesta propia) añade una opción **Write your own** (Escribir tu propia respuesta) junto a tus opciones.
- **Multi-Select** (selección múltiple) permite al jugador elegir varias opciones. Se unen con el separador.

Una pregunta llamada `player` establece el nombre de la persona. Si no añades ninguna, primero se pregunta a los jugadores su nombre.

El editor te avisa cuando un nombre está vacío, no es válido, se usa dos veces o ya lo ocupa una macro integrada, y cuando ningún campo usa una pregunta.

### Escribir los campos de la persona

Escribe los cinco campos como si fuera una persona tuya, y pon `{{name}}` donde va una respuesta:

```
Name: {{player}}
Class: {{class}}
Role: {{player}} is a {{class}} sworn to {{faction}}.
```

Cuando haces clic en un campo, aparecen debajo sus **Available variables** (Variables disponibles). Haz clic en una para insertarla donde está el cursor.

Usa `{{#if}}` para mostrar texto solo con algunas respuestas:

```
{{#if knowsHer == yes}}Grew up next door to {{char}}. {{relationToAna}}{{/if}}
```

Una pregunta que solo aparece dentro de un `{{#if}}` solo se hace cuando esa condición se cumple. En el ejemplo, `relationToAna` es una pregunta de seguimiento que aparece cuando el jugador responde **yes** (sí). Las comparaciones no distinguen mayúsculas de minúsculas.

`{{user}}` y `{{char}}` se mantienen como macros en la persona creada, así que siguen funcionando como en cualquier persona. Otras macros, como `{{random}}`, `{{time}}` o `{{getvar}}`, se rellenan una sola vez al crear la persona, o quedan vacías si en ese momento no tienen valor.

El cuadro **Rendered preview** (Vista previa procesada) de la parte inferior muestra la persona creada a partir de respuestas de ejemplo y enumera las preguntas que se hacen, en orden.

## Crear una persona como jugador

1. Empieza un chat nuevo de Roleplay. En el paso **Persona & Characters** (Persona y personajes), las tarjetas con onboarding muestran un pequeño icono de una silueta con un signo más en la lista de personajes.
2. Añade el personaje. En la parte superior de **Your Persona** (Tu persona) aparece una fila **Create your persona**, con el nombre de la tarjeta a la que pertenece.
3. Selecciona esa fila y haz clic en **Next** (Siguiente).
4. Una ventana **Create your persona** muestra las preguntas de la tarjeta. Las preguntas de seguimiento aparecen en cuanto tus respuestas las hacen pertinentes.
5. Haz clic en **Create persona** (Crear persona). La persona nueva se guarda y se selecciona para el chat, y el asistente continúa.

Si eliges cualquier otra persona o **None** (Ninguna), la opción que dice "Stay anonymous", no se hace ninguna pregunta. **Back** (Atrás) en la ventana de preguntas vuelve al selector de persona. Cada tarjeta con onboarding del chat tiene su propia fila **Create your persona**.

El onboarding se ofrece en el asistente de chat nuevo de Roleplay. Quick Start, la configuración del modo Conversation y empezar un chat directamente desde una tarjeta de personaje todavía no lo ofrecen.

## Compartir tarjetas con onboarding

El onboarding se guarda con la tarjeta. Las exportaciones **Marinara Native** y **Compatible PNG Card** lo conservan, así que un jugador que importa la tarjeta también recibe las preguntas. La exportación **Compatible JSON** lo omite. Una tarjeta importada con más de 100 preguntas conserva las 100 primeras. Consulta [Importar y exportar tarjetas de personaje](import-export.md).

## Guías relacionadas

- [Crear y editar personajes](creating-and-editing-characters.md)
- [Personas de usuario: crear y editar](personas.md)
- [Elegir tu persona en un chat](choosing-your-persona.md)
- [Macros](../prompts/macros.md)
- [Importar y exportar tarjetas de personaje](import-export.md)
