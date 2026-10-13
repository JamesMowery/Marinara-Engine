# Escribir conjuntos de reglas de Game Mode

Un conjunto de reglas explica a Game Mode cómo funciona un sistema de mesa: qué dados tira una prueba, qué contiene la ficha, qué recursos se gastan y qué recupera un descanso. Esta guía es para quienes quieren escribir uno y compartirlo. Para jugar con uno creado por otra persona, empieza por [Elegir las reglas](../game/getting-started.md#choosing-rules).

Un conjunto es un archivo JSON. Son datos, no código. Nada se ejecuta, así que importarlo no puede hacer nada en tu computadora. Antes de importar un archivo ajeno, lee con atención el texto del Game Master: se envía al modelo en todas las partidas que usan el conjunto.

## Lee esto primero: qué puede y qué no puede hacer un conjunto

Un conjunto solo puede rellenar los parámetros de una mecánica que el motor ya conoce. Actualmente conoce dos formas de resolver pruebas; el archivo elige una con `resolution.kind`:

- **`dice-sum`**: tira dados, suma números de la ficha e iguala o supera una dificultad. Abarca sistemas d20, sistemas de 2d6 más atributo y muchos otros.
- **`dice-pool`**: tira la cantidad de dados del personaje y cuenta los que alcanzan un objetivo. Abarca sistemas donde una puntuación representa un puñado de dados en vez de una bonificación.

Ambos se describen por completo en [Tipos de resolución](#resolution-kinds).

Una mecánica que no encaje en ninguna forma no se puede escribir en el archivo. Por ejemplo, quedarse con el dado más alto de una reserva, pruebas porcentuales por debajo de un valor, dados de símbolos y reservas enfrentadas. Cada una necesita un nuevo tipo de resolución dentro del motor: una contribución de código con pruebas, no un JSON. Si tu sistema lo necesita, abre una solicitud de función en el repositorio del motor y describe la mecánica con varias tiradas resueltas. Esos ejemplos se convierten en las pruebas.

Game Mode puede resolver una pelea con el combate propio de Marinara o con las reglas de tu conjunto. El bloque opcional `battle` presta al combate de Marinara las cifras de las fichas: consulta [Batallas](#battles-lending-the-sheet-to-marinaras-combat). El bloque opcional `combat` define cómo resuelve el conjunto la pelea: consulta [Combate](#combat-a-fight-your-own-rules-resolve). El motor ya aplica esas reglas. **Combat Preference** (Preferencia de combate) elige la presentación Classic o, si el conjunto define distancia, un campo Tactical.

## Inicio rápido

1. Copia el ejemplo que coincida con las tiradas de tu sistema. [`ember-roads.json`](https://github.com/Pasta-Devs/Marinara-Engine/blob/staging/docs/examples/rulesets/ember-roads.json) es un sistema pequeño de 2d6 con tres atributos que demuestra que el formato no presupone un d20 ni seis atributos. [`gravewatch.json`](https://github.com/Pasta-Devs/Marinara-Engine/blob/staging/docs/examples/rulesets/gravewatch.json) es una reserva pequeña de dados de diez caras con tres puntuaciones y seis oficios. Como ejemplo completo, consulta el archivo 5e (SRD 5.1) en [`ruleset-5e-2014.example.json`](https://github.com/Pasta-Devs/Marinara-Engine/blob/staging/docs/development/ruleset-5e-2014.example.json).
2. Cambia `id` por el tuyo. Usa minúsculas, dígitos y guiones simples, como `ember-roads`.
3. Edita la ficha, los descansos y el texto del Game Master.
4. Impórtalo; consulta [Probar tu conjunto](#trying-your-ruleset). La importación comprueba todo el archivo e indica los errores línea por línea antes de guardar nada.
5. Crea una partida, elige tu conjunto en **Rules** (Reglas) y juega unas cuantas pruebas.

Para recibir ayuda al escribir, configura el JSON Schema en tu editor añadiendo esta primera línea dentro de las llaves exteriores del archivo:

```json
"$schema": "https://raw.githubusercontent.com/Pasta-Devs/Marinara-Engine/staging/docs/extending/ruleset.schema.json",
```

El esquema detecta claves mal escritas y tipos incorrectos mientras escribes. No comprueba que los nombres apunten a elementos existentes, como una habilidad que nombra un atributo. Eso lo hace la importación.

Puedes añadir una línea `"$comment": "..."` a cualquier objeto para dejarte una nota. El motor la ignora.

## Las partes del archivo

| Clave | Qué contiene |
| --- | --- |
| `schemaVersion` | Siempre `1`. |
| `id`, `version` | El nombre del conjunto para el motor y un entero que incrementas cada vez que publicas un cambio. |
| `name` | Lo que ven los jugadores en el asistente. |
| `edition` | Opcional. Una línea sobre la edición o borrador. |
| `license` | Opcional. Un ID SPDX y la atribución exigida por la fuente. |
| `coverage` | Lo que cubre el conjunto y el resumen de una línea del asistente. |
| `resolution` | Cómo se tira una prueba o salvación. |
| `sheet` | Todo lo que hay en la ficha. |
| `rests` | Lo que restaura y elimina cada descanso. |
| `gm` | El texto que recibe el modelo del Game Master y los valores de ficha que ve por personaje. |
| `catalogs` | Opcional. Entradas preparadas que ofrece el editor para evitar escribir listas largas a mano. |
| `items` | Opcional. Las palabras con las que se escriben tus objetos: categorías, rarezas, estadísticas, ranuras y dinero. |
| `battle` | Opcional. Qué lee una batalla de la ficha y qué escribe al terminar. |
| `combat` | Opcional. Cómo resuelven tus reglas una pelea y qué ejecuta la pantalla de batalla. |
| `layers` | Opcional. Variantes del conjunto que se activan al crear una partida. |

El archivo puede ocupar hasta 256 KB. El texto que llega a un prompt (nombres, etiquetas, texto del Game Master) no puede contener saltos de línea, corchetes ni llaves dobles.

Los IDs de la ficha (atributos, habilidades, campos, reservas, etc.) usan minúsculas, dígitos y guiones bajos, y empiezan por letra, como `grit_max`.

<a id="resolution-kinds"></a>

### Tipos de resolución

`resolution.kind` elige cómo tirar. Ambos tipos leen la misma ficha y comparten tres claves; las partes del archivo posteriores a `resolution` no cambian al alternar:

- `abilityModifier`: cómo se convierte una puntuación en un número. `identity` usa la propia puntuación; `floorHalfMinusTen` es la regla de 5e; `stepTable` permite definir umbrales como `[[score, number], ...]`.
- `proficiencyTiers`: niveles de entrenamiento de una habilidad o salvación. El primero se aplica a habilidades no incluidas. Cada nivel añade `flat`, o `multiplier` por la bonificación de competencia, o ambos. Si el sistema tiene esa bonificación, indica su origen con `"proficiency": { "bonus": { "derived": "proficiency_bonus" } }`.
- `proficiency`: opcional; solo se necesita para un nivel que multiplica.

El significado del número depende del tipo: `dice-sum` lo suma a la tirada; `dice-pool` tira esa cantidad de dados.

#### `dice-sum`: sumar los dados

```json
"resolution": {
  "kind": "dice-sum",
  "dice": { "count": 2, "sides": 6 },
  "abilityModifier": { "op": "identity" },
  "proficiencyTiers": [
    { "id": "untrained", "label": "Untrained" },
    { "id": "trained", "label": "Trained", "flat": 1 }
  ],
  "advantage": false,
  "difficultyLadder": [
    { "label": "Easy", "dc": 6 },
    { "label": "Hard", "dc": 10 }
  ]
}
```

- `dice`: cantidad de dados y de caras. El total se compara con la dificultad.
- `advantage`: si el Game Master puede pedir dos tiradas y conservar una.
- `naturals`: efecto de las caras mayor y menor de un único dado en pruebas y salvaciones: `none`, `both`, `max-only` o `min-only`. Omítelo para usar solo aritmética. Requiere un dado único; un sistema 2d6 debe usar `none`.
- `difficultyLadder`: dificultades que se indica al GM que elija. `dc` es el número que debe alcanzar el total. El Game Master puede nombrar un peldaño en vez de escribir su número, como `difficulty="Hard"`, y el motor toma el número del peldaño.

#### `dice-pool`: tirar los dados y contarlos

El número de la ficha es el **tamaño de la reserva**, no una bonificación adicional. Una puntuación de 3 y un oficio que vale 2 tiran cinco dados. Esa es toda la idea: sin vocabulario ni editor de fichas nuevos, un sistema cuyas puntuaciones son puñados de dados usa las mismas `abilities`, `skills` y `proficiencyTiers` que cualquier otro.

```json
"resolution": {
  "kind": "dice-pool",
  "die": { "sides": 10 },
  "abilityModifier": { "op": "identity" },
  "proficiencyTiers": [
    { "id": "rating_0", "label": "Untried" },
    { "id": "rating_1", "label": "Shown once", "flat": 1 }
  ],
  "pool": { "min": 1, "max": 15, "abilityPlusAbility": true },
  "target": { "default": 7, "min": 5, "max": 9 },
  "explode": { "from": 10, "min": 8 },
  "cancel": { "upTo": 1 },
  "botch": { "upTo": 1 },
  "exceptional": { "successes": 5 },
  "situationalDice": { "min": -3, "max": 3 },
  "reroll": [{ "id": "careful", "upTo": 6, "mode": "once" }],
  "difficultyLadder": [
    { "label": "Plain work", "successes": 1, "target": 6 },
    { "label": "Grim", "successes": 3, "target": 8 }
  ]
}
```

- `die`: caras de cada dado de la reserva, de 2 a 100.
- `pool`: límites aplicados al número de la ficha antes de las explosiones. Un `min` de 0 permite que una reserva vacía falle sin tirar; `max` no puede superar 100. `abilityPlusAbility: true` permite que una prueba de atributo nombre un segundo atributo con `with=` y tire ambos, para sistemas que tiran dos atributos juntos; sin ello, `with=` no significa nada en una prueba de atributo.
- `target`: cara que debe alcanzar un dado para contar. Pon `min` por debajo de `max` para permitir al GM moverla por prueba con `threshold=`; iguala los tres valores para fijarla.
- `double`: opcional. Una cara igual o superior a `from` cuenta dos veces.
- `explode`: opcional. Una cara igual o superior a `from` tira otro dado, que también puede explotar. Los dados extra se limitan a `pool.max` además de la reserva inicial: una prueba tira como máximo el doble de `pool.max` y un `from` bajo no causa tiradas infinitas.
- Ambos pueden llevar un `min`: la cara más baja hasta la que una prueba puede bajar la regla, con `explode="N"` o `double="N"` en la etiqueta o en una entrada que usó el personaje. Con un `min` puedes omitir `from`, y entonces la regla solo se activa en una prueba que la pida: una especialidad que hace volver a tirar los dieces en una habilidad es `"explode": { "min": 10 }`. `from` no puede estar por debajo de `min`.
- `cancel`: opcional. Una cara igual o inferior a `upTo` quita un éxito. La cantidad nunca baja de cero.
- `botch`: opcional. Si **ningún** dado tuvo éxito y salió una cara igual o inferior a `upTo`, hay fracaso crítico. Una reserva cuyo único éxito se canceló ha fallado, pero no es una pifia. `"rule": "halfOrMore"` lo interpreta de otra forma: caras iguales o inferiores a `upTo` en al menos la mitad de los dados tirados al principio (sin contar explosiones) son un fracaso crítico si ningún dado tuvo éxito; si no, el resultado se mantiene y se marca con `complication="true"`: algo sale mal a la vez. `"noSuccesses"` es el predeterminado.
- `exceptional`: opcional. Esta cantidad de éxitos netos o más, en una prueba superada, es un éxito crítico.
- `situationalDice`: opcional. Límites de dados que el GM puede añadir o quitar con `bonus=` por acrobacias, heridas o mala iluminación.
- `reroll`: opcional. Repeticiones que concede el sistema sin que nadie las pague, hasta seis, cada una con un `id` que el GM nombra en una prueba con `reroll=`. Las caras iguales o inferiores a `upTo` se vuelven a tirar: con `"once"`, la cara nueva se queda sea cual sea; con `"until"`, se sigue tirando hasta que salgan más altas. `upTo` va de 1 a una cara menos que las del dado, porque repetir todas las caras no terminaría nunca, y un `until` se detiene de todos modos tras cien repeticiones en una prueba. No se tira ninguna salvo que la prueba la nombre. Una prueba hace una sola repetición: si un gasto o una entrada que usó el personaje también compró una, se tira la que alcanza más caras y, si alcanzan las mismas, `until` antes que `once`.
- `difficultyLadder`: `successes` indica cuántos éxitos se necesitan. Un peldaño también puede definir `target`, pero solo si es ajustable y dentro de sus límites. Ese es el objetivo con el que cuenta una prueba en ese peldaño: cuando el GM nombra el peldaño con `difficulty=` o escribe un `dc=` que solo necesita exactamente un peldaño. Si varios peldaños necesitan la misma cantidad de éxitos, `dc=` por sí solo no indica cuál se quería, así que el objetivo sigue siendo el predeterminado; nombra el peldaño.

Las caras de `cancel` y `botch` deben estar por debajo del objetivo mínimo, y todas las caras nombradas deben existir en el dado. Una regla imposible de activar se rechaza al importar, en vez de descubrirse durante el juego.

Un conjunto empaquetado de reservas requiere Capability API 1.24; un `min` en `explode` o `double`, `pool.abilityPlusAbility` y `botch.rule` requieren 1.37, y `reroll`, 1.38. Uno comunitario importado lo valida el motor que lo lee, así que no necesita nada más.

#### Qué puede escribir el Game Master en una prueba de reserva

```
[skill_check: skill="Ward" dc="2" who="Bram the Quiet" threshold="8" bonus="-2" with="Sinew"]
[skill_check: skill="Ward" difficulty="Grim" explode="9"]
[skill_check: skill="Nerve" dc="1" with="Warmth"]
[skill_check: skill="Ward" dc="2" reroll="careful"]
```

- `dc` es la cantidad de **éxitos** necesaria, no un objetivo por dado. Va de 1 al máximo que pueda contar una tirada: el máximo de la reserva, duplicado si hay explosiones y otra vez si las caras cuentan doble.
- `threshold=` mueve el objetivo por dado y solo se ofrece si `target.min` es menor que `target.max`.
- `bonus=` añade o quita dados y solo se ofrece si se declara `situationalDice`.
- `with=` tira una habilidad o salvación con otro atributo. Funciona con ambos tipos; 5e obtiene "Strength (Intimidation)" con el mismo atributo. En una prueba de atributo, en cambio, añade un segundo atributo si `pool.abilityPlusAbility` está activado.
- `difficulty=` nombra un peldaño de la escala en lugar de `dc=`, en ambos tipos: sus éxitos (o su dificultad en una suma) y, en una reserva, su objetivo. Un `dc=` escrito junto a él sigue teniendo prioridad para el número, y `threshold=` para el objetivo. Se ignora un nombre al que no responde ningún peldaño o que comparten dos.
- `explode=` y `double=` mueven las caras de esas reglas para esta prueba y solo se ofrecen si la regla tiene un `min`.
- `reroll=` nombra una de tus entradas `reroll` para esta prueba y solo se ofrece si declaraste alguna. Se ignora un nombre al que no responde ninguna.

Todo respeta el archivo: los valores fuera de rango se ajustan al extremo más cercano y un atributo no ofrecido se ignora, sin rechazar la prueba. El registro muestra lo realmente usado: la dificultad y el umbral como números, los dados adicionales después de los límites, `with=` solo si se sustituyó o añadió el atributo, una cara solo si la prueba la movió, `reroll=` solo si la repetición que nombra fue la que se tiró y volvió a tirar algo, y `complication="true"` cuando una pifia `halfOrMore` salió mal junto a un resultado. El motor siempre tira los dados. Sustituye resultados de reserva escritos por el modelo, ignora `mode="advantage"` porque este tipo no tiene ventaja y no usa dados tirados por el jugador antes del turno.

#### Qué queda fuera y por qué

Cada caso necesita otro tipo de resolución: ninguno se expresa contando dados que alcanzan un objetivo.

- **Conservar el dado más alto** (como Blades in the Dark) requiere un resultado de éxito parcial que las pruebas no tienen.
- **Reservas de postura comparadas con un atributo** (como Lasers and Feelings) deciden si superar o quedar por debajo en cada prueba, otra comparación.
- **Dados de símbolos** (como Genesys) no producen números.
- **Reservas enfrentadas** resuelven dos personajes a la vez; una prueba tiene un solo lanzador.
- **Porcentajes por debajo del valor y porcentajes abiertos** comparan en la dirección contraria.
- **Reservas sumadas con un dado especial** (como OpenD6) suman los dados y tratan uno de forma distinta.

Las dos cosas que antes quedaban fuera ya están modeladas; la sección sobre gastar para cambiar tiradas explica cómo. Una regla del propio sistema, "spend a point for a success" o "spend a point to throw the failures again", es `resolution.spend`. Una repetición que el sistema concede gratis cuando la situación lo pide es `resolution.reroll`, descrito arriba. Una que pertenece a algo elegido por el personaje es `mechanics.check` de una entrada de catálogo, pagado con el coste propio de esa entrada.

### Gastar para cambiar una tirada

Algunos sistemas permiten pagar por una tirada que está a punto de hacerse: un punto de voluntad por un éxito automático. `resolution.spend` lo define como regla permanente del sistema, no como algo comprado por un personaje:

```json
"spend": [{ "pool": "resolve", "amount": 1, "successes": 1, "perCheck": 2 }]
```

- `pool` es una de tus `live.pools`. No puede empezar vacía, pues no habría nada que gastar al comenzar a jugar.
- `amount` es el coste de UNA compra. `successes`, `dice` y `reroll` indican qué compra; debe comprar al menos uno. Los éxitos se añaden después de contar los dados y de las cancelaciones, porque nadie los tiró. Los dados se tiran con la reserva, dentro de sus límites. Un `reroll` es `{ "upTo": 6, "mode": "once" }`, se interpreta igual que `resolution.reroll` y se compra una sola vez por muchas compras que haga la prueba: un dado que se vuelve a tirar dos veces sigue siendo una repetición.
- `perCheck` indica cuántas compras permite una prueba; como máximo puede comprar por valor de `amount * perCheck` puntos. Ese límite impide que una reserva llena compre una tirada imposible de perder. Es un número de 1 a 10, un valor de la ficha con la misma forma que lee un valor derivado (`{ "abilityScore": "nerve" }`, `{ "derived": "focus" }`) o `"pool"`: tantas como dados tenga la prueba, es decir, el número de la ficha para ella antes de heridas, dados adicionales o modificadores. Un valor de la ficha se lee para quien tira, se redondea hacia abajo y se limita entre 0 y 100; un personaje cuyo límite resulta 0 no compra ni paga nada.
- Solo lo admite `dice-pool`: una tirada sumada no tiene éxitos ni reserva a los que añadir nada. Se rechaza al importar un conjunto `dice-sum` que declare `spend`.
- Hasta cuatro entradas, y dos no pueden nombrar la misma reserva, porque la prueba no podría distinguirlas.
- Un gasto que compra un `reroll`, un `perCheck` que no es un número y más de dos entradas requieren Capability API 1.38 en un conjunto empaquetado.

**Se escribe en la propia prueba.** El GM escribe `[skill_check: skill="Nerve" dc="2" spend="resolve:1"]`, no un comando `[sheet:]` separado: los dados se tiran antes de aplicar esos comandos y ya no quedaría nada que cambiar. Una sola resolución tira los dados y paga lo que los modificó.

### Un amuleto que cambia una tirada

`resolution.spend` es una regla del sistema. Una entrada que el personaje realmente ELIGIÓ también puede cambiar una prueba, mediante `mechanics.check` en la entrada de catálogo:

```json
"mechanics": {
  "kind": "utility",
  "cost": [{ "pool": "blood", "amount": 1 }],
  "perCostStep": { "flat": 1 },
  "check": { "reroll": { "upTo": 1, "mode": "once" }, "successes": 1 }
}
```

- `reroll` vuelve a tirar los dados iguales o inferiores a `upTo`. `once` sustituye cada uno una vez y conserva la cara nueva; `until` sigue tirando. `upTo` debe estar por debajo de la cara máxima o repetiría toda la reserva eternamente. El motor limita cuántos dados puede repetir una prueba, diga lo que diga el archivo.
- `dice` añade dados antes de tirar; `successes` añade éxitos después de contar; `threshold` fija el objetivo por dado para esa tirada, dentro del rango permitido por `target`.
- `explode` y `double` mueven las caras de esas reglas para esa tirada, solo si tu regla tiene un `min` y dentro de él: `"check": { "explode": 8 }` es una entrada que hace volver a tirar los ochos. Tienen prioridad sobre lo que escribió el GM en la etiqueta, igual que `threshold`. Capability API 1.37.
- Debe haber al menos uno de ellos; si no, la entrada no dice nada y se rechaza.
- Solo `dice-pool` puede aplicarlo; un conjunto `dice-sum` con `mechanics.check` se rechaza al importar.

**El coste es el `cost` de la entrada**, pagado con el mismo mecanismo que cualquier otro uso: la reserva y un uso de cada contador que escribió esa entrada. `perCostStep` indica que la entrada ESCALA: si lo declara, se compra tantas veces como se haya pagado su precio; si no, se compra una vez por mucho que se ofrezca.

**El GM la nombra en la prueba:** `[skill_check: skill="Brawl" dc="3" use="Potence" spend="blood:3"]`. No usa un comando de ficha separado por el mismo motivo: se tira antes de actualizar los registros.

**Todo o nada.** Si la reserva no alcanza, no se compra ni descuenta nada; la tirada queda como habría sido. Los puntos que no equivalen a un número entero de compras tampoco compran nada. Pedir más de `perCheck` se limita, no se rechaza; solo se paga hasta ese máximo. El motor lo calcula todo. El GM nombra lo que el jugador quiso gastar y nunca toca los dados. El registro indica lo realmente pagado, la entrada aplicada, los éxitos que no se tiraron y cuántos dados se repitieron. Un amuleto no elegido por el personaje, o cuyo catálogo no pueda leer el motor, no hace nada; no se aplica por confianza.

### La ficha

- `sections` agrupa elementos en el editor. Los campos, valores derivados y listas nombran con `section` la sección en la que están, y también pueden hacerlo los atributos, habilidades y salvaciones: el editor, la ficha de la partida y el bloque de ficha del GM los muestran entonces bajo ese encabezado (una ficha que no les asigna sección los muestra exactamente como antes). Una sección también puede llevar una regla `untrained` (abajo).
- `abilities` son las puntuaciones principales. `skills` y `saves` pueden nombrar el atributo con el que tiran, un `cap` y una regla `untrained` (abajo).
- `fields` contiene valores individuales. Tipos: `number`, `text`, `longtext`, `boolean`, `enum` (lista fija de opciones) y `dice` (texto como `1d8`).
- Los valores `derived` se calculan a partir de otros y no pueden sobrescribirse a mano. Las operaciones son `sum`, `min`, `max`, `scale` (multiplicar y redondear), `stepTable` (consultar umbrales, como cuando un nivel determina competencia) y `enumTable` (un número para cada valor de un campo enum o de un estado vivo; consulta Estados vivos más abajo).
- `lists` son tablas con columnas propias, como equipo, conjuros o rasgos. Una lista con `pools` convierte cada fila en un recurso con máximo propio, para rasgos de clase de usos limitados.
- `live` es lo que cambia al jugar: `pools` (puntos de golpe, espacios de conjuro, Grit), `tracks` (número en una escala, como agotamiento, o contador de heridas con casillas), `text` (notas breves, como la concentración), `conditions` y `states` (un valor de un conjunto cerrado, como una forma o una postura).

Todo lector de números los nombra con una referencia de valor: un objeto con exactamente una clave entre `const`, `field`, `derived`, `abilityScore`, `abilityMod`, `abilityModFromField`, `skillMod`, `saveMod`, `listSum`, `livePool`, `liveTrack` e `itemStat`. Por ejemplo, una reserva con máximo derivado: `"max": { "derived": "grit_max" }`.

- `listSum` suma una columna numérica de una lista: `{ "listSum": { "list": "gear", "column": "bulk", "onlyWhen": "packed" } }`. `onlyWhen` es opcional y nombra una columna booleana; solo cuentan las filas donde está activada. Una celda vacía cuenta como el valor predeterminado de su columna, y una lista oculta por `hideWhen` no suma nada. Ember Roads calcula así Burden a partir del equipo que lleva cargado el personaje, de modo que nunca puede desviarse de la lista como lo haría un número escrito a mano. Una columna escalada de catálogo no puede leer una suma de lista, ni siquiera mediante un valor derivado: una lista puede contener celdas escaladas, propias o que vuelven a leerla, y el recálculo nunca terminaría.
- `livePool` lee lo que queda en una de tus `live.pools` (la reserva de una fila de lista no es una de ellas). `liveTrack` lee uno de tus `live.tracks`, con un `read` opcional: `"value"`, dónde está (predeterminado); `"filled"`, cuánto supera su `min`; `"remaining"`, cuánto le falta para su `max`; o, solo en un contador de heridas, `"penalty"`, la penalización vigente. Una reserva o contador oculto lee 0. "Harm still to take" de Gravewatch es `{ "liveTrack": "harm", "read": "remaining" }`.
- Una lectura viva toma el estado vivo tal como está cuando se tira una prueba, empieza un combate o se escribe el bloque de ficha del GM. Si aún no hay ninguno (el editor de fichas, la revisión de una importación), lee el estado con el que empieza el juego: una reserva llena o vacía según su `start`, un contador en su `default`, un contador de heridas limpio.
- Nada que se calcule antes de que exista un estado vivo puede leer uno: el `max` de una reserva o de un contador, la bonificación de competencia, o la columna escalada o el escalado de un catálogo. Eso se mantiene a través de un valor derivado que lea uno y de una habilidad limitada por un valor vivo, y la importación nombra el valor que lo hace.
- Estos tres requieren Capability API 1.39 en un conjunto empaquetado.
- `itemStat` lee los objetos que tiene el personaje, como la guardia de la armadura que lleva puesta, y sigue la misma regla que una lectura viva. Consulta [Objetos en la ficha](#items-on-the-sheet). Capability API 1.52.

**Qué hace una prueba sin entrenamiento.** Una habilidad o salvación en la que el personaje no tiene entrenamiento (su nivel es el primero) se tira como siempre, salvo que el conjunto diga otra cosa con `untrained` en la habilidad o salvación o en la sección donde está (la regla propia de la habilidad o salvación tiene prioridad):

```json
"sections": [{ "id": "labour", "label": "Labour", "untrained": { "by": -1 } }],
"skills": [
  { "id": "dig", "label": "Dig", "ability": "sinew", "section": "labour", "untrained": "refuse" },
  { "id": "listen", "label": "Listen", "ability": "nerve", "untrained": "harder" }
]
```

- `"normal"`: como siempre (predeterminado).
- `{ "by": -3 }`: se suma al número de la prueba, de -20 a 20: dados en una reserva, una cantidad fija en una suma. Forma parte del número que muestra la ficha y que tira cada prueba, antes de cualquier `cap`.
- `"harder"`: un paso más difícil. En una reserva, el objetivo por dado sube uno (sin salir del rango del objetivo), así que necesita un conjunto de reservas cuyo objetivo pueda moverse; en cualquier otro caso se rechaza al importar, porque nunca podría cambiar una tirada.
- `"refuse"`: la prueba no se tira. El motor devuelve la petición del GM con `reason="untrained"`, lo que la zanja: nada la tira después, se informa al GM de que el personaje no pudo intentarlo y la narración lo refleja. El endpoint de pruebas de habilidad responde a esa prueba con un 400 (`skill_check_untrained`), y una rama creada a partir de ella no conserva ninguna de las dos mitades. El editor de fichas no muestra ningún número para ella.
- Un personaje sin ficha no tiene entrenamiento que leer, así que no se le aplica ninguna regla de falta de entrenamiento.
- El recordatorio del GM enumera las reglas de falta de entrenamiento, por sección o por habilidad, para que pida pruebas que el personaje pueda hacer.

Las secciones en atributos, habilidades y salvaciones, y `untrained` en cualquier lugar, requieren Capability API 1.41 en un conjunto empaquetado.

Una habilidad o salvación puede llevar un `cap`, una referencia de valor que es lo máximo a lo que puede llegar su prueba: una puntuación que no puede superar o un contador que la frena. Gravewatch limita Soothe a la Resolve que le queda a un guardián, `"cap": { "livePool": "resolve" }`. El número limitado es el que muestra la ficha y el que tira cada prueba, y un cambio con `with=` se calcula sobre el número anterior al límite y después se vuelve a limitar. Un límite no puede leer un modificador de habilidad o salvación, ni tampoco el valor derivado que lee ni ningún valor derivado declarado antes que ese, para que la ficha se siga calculando una sola vez, de arriba abajo. Capability API 1.39.

`hideWhen` oculta un campo, un valor derivado, una lista, una reserva o un contador simple según el valor de otro campo, exactamente de una de tres formas: `equals` un valor, `notEquals` un valor (oculto siempre que el campo tenga cualquier otro, que es como se dice "solo para este tipo de personaje") o `in` una lista de valores. Cada valor nombrado debe ser uno que el campo pueda contener, o la regla nunca coincidiría, y se comprueba de uno en uno; para `notEquals` e `in` eso incluye el rango de un campo numérico y la longitud de un campo de texto. La regla lee el valor que muestra el editor de fichas: un campo sin valor es el valor con el que empieza una ficha en blanco, y un valor enum que el conjunto ya no ofrece es el predeterminado del campo. El archivo 5e usa `equals` para ocultar espacios de conjuro a quien no lanza conjuros; Gravewatch quita el aceite de linterna de la ficha de todo guardián que no esté en la guardia nocturna con `"hideWhen": { "field": "watch", "notEquals": "night" }`. Una capa no puede retirar un valor con el que compare cualquiera de ellas. Un contador de heridas no puede ocultarse, porque las tiradas y los combates lo leen muestre lo que muestre la ficha. `notEquals` e `in` requieren Capability API 1.39.

El `max` de un contador simple puede ser una referencia de valor en vez de un número, para que una puntuación del personaje determine hasta dónde llega: `"max": { "field": "willpower_rating" }`. Su `min` y su `default` siguen siendo números. Un contador de heridas con niveles con nombre tiene un `max` igual a su número de niveles; un contador de casillas numeradas tiene la longitud que indica su propio `max` (consulta Casillas numeradas más abajo). `alwaysShow: true` muestra un contador en el bloque de ficha del GM incluso en su valor predeterminado, para una puntuación que importa en cada turno; sin él, un contador en su valor predeterminado se omite, porque tres salvaciones contra la muerte a cero no dicen nada. Estos tres requieren Capability API 1.37 en un conjunto empaquetado.

### Estados vivos: una forma, una postura, una luz

Un campo enum se elige al crear el personaje y no cambia, y un estado solo está activado o desactivado. Un **estado vivo** es un valor de un conjunto cerrado que cambia durante el juego: la forma de un cambiaformas, una postura de combate, cuánto alumbra una linterna.

```json
"live": {
  "states": [
    {
      "id": "stance",
      "label": "Stance",
      "values": ["guarded", "steady", "reckless"],
      "valueLabels": { "guarded": "Guarded", "steady": "Steady", "reckless": "Reckless" },
      "default": "steady"
    }
  ]
}
```

- `values` son de dos a cuarenta, y cada uno se muestra como su entrada de `valueLabels` o tal cual. Siguen la regla de toda etiqueta (una línea, sin corchetes ni llaves de macro) y nunca contienen comillas dobles, porque el comando de ficha los entrecomilla. Hasta doce estados.
- `default` es el valor con el que empieza toda ficha y, si lo omites, el primer valor. Solo se guarda un estado que se ha movido de su valor predeterminado, y un valor guardado que el estado ya no ofrece se lee como el predeterminado.
- `hideWhen` quita un estado de una ficha según el valor de un campo, igual que con una reserva: no se muestra, ningún comando lo cambia y una tabla (abajo) lo lee como si no tuviera valor.
- El GM fija uno con `[sheet: who="Name" op="state" state="Stance" value="Reckless"]`, nombrando el estado por su ID o etiqueta y el valor por sí mismo o por su etiqueta. Se rechaza un estado o un valor que la ficha no tiene (`unknown-state`, `unknown-value`). El motor enseña el comando y enumera todos los valores solo si tu conjunto tiene un estado, y cada bloque de ficha indica el valor de cada estado, incluido el predeterminado, porque "en forma humana" es un dato que la narración necesita en cada turno.
- La pantalla de ficha de la partida muestra un selector para cada estado.
- Un descanso restablece uno con un paso de restauración que lo nombra: `{ "state": "stance", "to": "default" }`, o `"to"` uno de sus valores.

**Números que lo siguen.** Un valor derivado con `"op": "enumTable"` da un número para cada valor de un campo enum o de un estado vivo:

```json
{
  "id": "stance_brawn",
  "label": "Stance on Brawn",
  "op": "enumTable",
  "from": { "liveState": "stance" },
  "table": { "guarded": -1, "reckless": 2 },
  "default": 0
}
```

- `from` nombra exactamente uno entre `field` (un campo enum) y `liveState`. Cada fila de `table` es un valor que este puede tener, de uno a cuarenta. Un valor que la tabla omite, y un estado que la ficha oculta, leen `default` (0 si lo omites).
- Un campo queda fijado al crear el personaje, así que una tabla basada en uno puede alimentar un máximo: Gravewatch da una Resolve más a un guardián de la guardia del alba. Un estado vivo cambia durante el juego, así que una tabla basada en uno es una lectura viva, como `liveTrack`: nada que se calcule antes de que exista un estado vivo puede leerla, ni directamente ni mediante otro valor derivado. Por eso la clave es `liveState` y no `state`: toda lectura del estado vivo se llama `live`, y así tanto tú como la importación pueden distinguir lo que un máximo no puede usar.
- Para llevar el número a los dados, apunta `resolution.adjust` hacia él (consulta Modificadores fuera de la ficha). Ember Roads suma su Stance a cada tirada de Brawn; Gravewatch resta una linterna cerrada u oscura de cada tirada de Nerve.
- Una capa puede retirar un valor de un campo para el que una tabla tiene una fila. Esa fila simplemente no se lee mientras la capa está activa.

Los estados vivos, `enumTable` y un paso de descanso que nombra un `state` requieren Capability API 1.42 en un conjunto empaquetado.

### Contadores de heridas: salud en una escala de marcas

Muchos sistemas no cuentan puntos de golpe. Tienen una columna de casillas, cada una peor que la anterior, y marcas una al recibir daño. Añade `kinds` y, o bien `levels` con nombre, o bien `boxes` numeradas (abajo), a una entrada `live.tracks` para convertir un número en una de estas escalas:

**¿Qué forma necesita tu sistema?** Una reserva guarda CUÁNTO daño llegó; un contador guarda cuánto Y el tipo de cada parte. Si una herida es contundente, letal o agravada y el tipo sigue importando después del golpe (porque lo agravado cura más despacio, no puede absorberse o acaba matando), debe conservarse tras la tirada. Solo una marca lo conserva. Una reserva no puede: al restar daño solo queda un número menor, sin memoria de qué puntos eran de cada tipo. Por eso `combat.damageKinds` se rechaza cuando la salud es una reserva, en vez de ignorarse. Una reserva sí puede tener `damageTypes`, y los enemigos pueden resistirlos o ser inmunes: eso determina cuánto daño llega, no qué herida queda después.

```json
{
  "id": "harm",
  "label": "Harm",
  "min": 0,
  "max": 4,
  "levels": [
    { "label": "Scuffed", "penalty": 0 },
    { "label": "Winded", "penalty": -1 },
    { "label": "Bleeding", "penalty": -3 },
    { "label": "Down", "penalty": -99 }
  ],
  "kinds": [
    { "id": "knock", "label": "K", "severity": 0 },
    { "id": "tear", "label": "T", "severity": 1 }
  ]
}
```

- `levels` tiene 1–16 peldaños, del mejor al peor. Cada uno tiene `label` y `penalty` entero igual o menor que 0. Un negativo grande expresa quedar fuera de combate, así que `-99` es válido.
- `kinds` tiene 1–6 tipos de daño que admite el contador, cada uno con `id`, `label` breve para la casilla y `severity`. Las severidades deben ser distintas; solo importa su orden, no los números, así que sepáralas como quieras.
- Ambos van juntos. Se rechaza `kinds` sin `levels` ni `boxes`, porque no habría dónde marcar, y `levels` o `boxes` sin `kinds`, porque la marca debe ser de algún tipo. Un contador tiene `levels` o `boxes`, nunca ambos.
- **Distingue los términos.** `kinds` define lo que PUEDE SER una marca. Una MARCA es uno de esos tipos situado en el contador durante la partida. La definición contiene tipos; la ficha contiene marcas.
- Un contador de heridas empieza sin marcas, así que su `min` es 0. Un contador con niveles con nombre tiene la longitud de sus niveles, así que su `max` es `levels.length`, y cualquier otra declaración se rechaza en vez de corregirse, para que nunca contenga dos longitudes contradictorias. Un contador de casillas tiene la longitud que indica su propio `max`, que puede ser un valor que calcula la ficha (consulta Casillas numeradas más abajo).

**Las reglas exactas**, porque una interpretación vaga produce un contador incorrecto:

- Las marcas se ordenan con **las más graves primero**. Siete niveles admiten como máximo siete marcas.
- Cada marca se **inserta por orden de gravedad**, nunca simplemente al final. Ocupa el nivel más alto que le corresponde y desplaza las más leves hacia abajo.
- La penalización activa es la del **nivel marcado más bajo**, nunca la suma. Tres marcas en el ejemplo dan `-3`, no `0 + -1 + -3`.
- `amount` es la cantidad de marcas de un tipo, **aplicadas una a una**. Así, llenarse a mitad del proceso sigue la misma regla que empezar lleno.
- Marcar un contador **lleno** **aumenta un paso la gravedad de su marca más leve**, en vez de añadir otra. Sube un peldaño en tu escala de tipos, sea cual sea el tipo nuevo.
- Una marca que superaría la gravedad máxima se queda en ella; la que no pudo entrar cuenta como **desbordamiento**. Este se guarda para que recargar no olvide el daño recibido.
- **Curar usa el mismo comando con cantidad negativa.** Antes de cualquier marca elimina el desbordamiento. Una curación que nombra uno de los tipos del contador solo elimina marcas de ese tipo, así que "lo contundente se cura, lo letal se queda" es `kind="bashing"` con cantidad negativa; una que no nombra ningún tipo del contador elimina primero las marcas más leves.

**Marcar durante el juego.** El GM escribe `[sheet: op="damage" track="harm" kind="knock" amount="1"]` y cura con `amount` negativo. En un contador que se llena por casilla, añade `box="3"` para la casilla donde cae el golpe. La forma de reserva de `damage`, que nombra `pool=`, no cambia. El comando `track` ordinario se rechaza para heridas: un número no identifica los tipos de marcas nuevas. El jugador también puede marcar y limpiar casillas a mano, como esperan estos sistemas.

**Un combate también puede marcarlo.** Apunta `combat.health` al contador en vez de a una reserva: un golpe acertado marca las casillas que indica `combat.damageKinds.marks`, del tipo al que ese bloque asigna su daño. Un personaje con el contador lleno está caído, el estado que consulta la regla de muerte. En un contador que se llena por casilla, el daño `per-point` nombra la casilla donde cae y `per-blow` apunta a la primera, y un golpe que ninguna casilla puede recibir derriba al personaje, que es lo que significa quedar fuera de combate en los sistemas que usan un contador así. Curar elimina una marca, la más leve, de cualquier tipo. Los puntos temporales se rechazan porque el contador no tiene un espacio para ellos. El motor lo lee como niveles RESTANTES; el resto del combate, caer, revivir, el registro y el resumen no cambia.

**Un descanso puede curar heridas.** Un paso de restauración que use `"to"` lo reduce a esa cantidad de marcas, incluido el desbordamiento; con un `"by"` NEGATIVO elimina esa cantidad, empezando por el desbordamiento, así que `"by": { "const": -2 }` elimina dos. Un paso que además nombra un `kind` solo elimina marcas de ese tipo: "Catch your breath" de Gravewatch es `{ "track": "harm", "kind": "knock", "to": "min" }`, que cura los golpes y deja los desgarros. Un paso que AÑADIRÍA marcas no hace nada, porque un descanso no causa daño, y eso incluye un `"by"` positivo: se omite sin aviso, así que escribe el signo menos.

#### Casillas numeradas en vez de niveles con nombre

Si un contador mide tanto como la resistencia del personaje, dale `boxes` en vez de `levels`. Tiene tantas casillas como su propio `max`, que puede ser un valor que calcula la ficha (nunca uno que lea el estado vivo), hasta 64:

```json
{
  "id": "strain",
  "label": "Strain",
  "min": 0,
  "max": { "derived": "strain_boxes" },
  "boxes": {
    "penalty": {
      "by": "remaining",
      "table": [
        [0, -1],
        [1, 0]
      ]
    }
  },
  "kinds": [{ "id": "strain", "label": "S", "severity": 0 }],
  "fill": "indexed",
  "onFull": "refuse"
}
```

- La penalización vigente se lee en `table` (una tabla por peldaños, como la de un valor derivado, con penalizaciones iguales o menores que 0) según el número de casillas `filled` o `remaining`, sea cual sea ese número, así que una tabla puede costar algo sin ninguna marca. No depende de qué casillas estén marcadas.
- Las casillas están numeradas: la ficha muestra "Box 3", y el bloque de ficha del GM indica cuántas hay marcadas de cuántas en vez de nombrar un nivel.

#### Llenar por casilla y contadores que rechazan

- `"fill": "indexed"` coloca una marca en la casilla que nombra el comando con `box=` (la primera si no nombra ninguna), o en la siguiente libre por encima de ella, y nunca mueve una marca ya colocada. Una curación elimina la más alta de las marcas más leves. La pantalla de ficha permite al jugador hacer clic en cualquier casilla libre.
- `"onFull": "refuse"` rechaza una marca que no tiene casilla libre, en vez de aumentar la gravedad de la marca más leve. Se rechaza el comando entero, así que tres marcas con dos casillas libres no colocan ninguna. Un contador indexado siempre rechaza; indícalo con `"onFull": "refuse"` a su lado.

#### Niveles que añade una lista

`"extra": { "list": "scars", "countColumn": "levels", "penaltyColumn": "penalty" }` en un contador con niveles con nombre permite que una lista de la ficha lo alargue para cada personaje. Cada fila, en orden, añade su cantidad de niveles (16 como máximo) con su penalización, después del último nivel cuya penalización sea al menos igual de buena, con el nombre de ese nivel si las penalizaciones coinciden y, si no, con el de la penalización. Las cicatrices de Gravewatch funcionan así. Una lista que la ficha oculta no añade nada, y si se borra una fila mientras sus niveles están marcados, las marcas que ya no caben se conservan como desbordamiento: editar la ficha no deshace el daño.

`boxes`, `fill`, `onFull`, `extra` y el `kind` de un paso de descanso requieren Capability API 1.40 en un conjunto empaquetado.

### La penalización de las tiradas

`resolution.penaltyFrom` nombra el contador de heridas que penaliza todas las pruebas del conjunto. Se declara, no se presupone: sin él, las tiradas funcionan igual que antes de que existieran los contadores.

Lo que HACE la penalización depende del tipo de resolución, igual que el número de la ficha:

- En `dice-pool`, **quita dados de la reserva**, con mínimo `pool.min`. Un `pool.min` de 1 permite tirar un dado incluso desde el último peldaño; un `pool.min` de 0 permite no tirar ninguno y fallar sin lanzar.
- En `dice-sum`, es un **modificador fijo**, incorporado al mismo número al que ya contribuyen atributo y entrenamiento.

Debe nombrar un contador de heridas. Uno ordinario no tiene penalización y se rechaza al importar. El resultado indica la aplicada, para que el jugador entienda por qué tiró menos dados; el bloque de ficha del GM también muestra el peldaño y su coste.

### Modificadores fuera de la ficha

Algunos números de una ficha acompañan a cada tirada a la que afectan: una mochila pesada en cada escalada, una penalización de armadura en cada intento de sigilo, una bendición en todo. `resolution.adjust` lo indica, en cualquiera de los dos tipos:

```json
"adjust": [{ "value": { "derived": "burdened" }, "abilities": ["brawn"] }]
```

- `value` es un valor de la ficha, con la misma forma que lee un valor derivado, incluido el estado vivo. Un número negativo resta. Ember Roads suma el volumen del equipo que lleva cargado un personaje en un Burden, y una tabla por peldaños convierte cada tres puntos en uno menos en cada tirada de Brawn. Una tabla basada en un estado vivo es la forma en que una forma o una postura llega a los dados (consulta Estados vivos).
- `abilities` es opcional. Con él, el modificador solo se aplica a una prueba que tira con uno de esos atributos: el propio atributo, una habilidad o salvación que lo usa, una habilidad tirada con él mediante `with=` o una pareja de atributos que lo incluye. Sin él, se aplica a todas las pruebas.
- Se aplica donde se aplica la penalización por heridas: dados en una reserva, con el mismo mínimo `pool.min`, y un número fijo en una suma, dentro del modificador que suma el registro. Hasta ocho entradas, que se suman en cada prueba.
- Un combate del conjunto lo añade a cada tirada que construye a partir de una ficha, al empezar el combate: un ataque tirado con un atributo o una habilidad, una salvación, una prueba de enfrentamiento y la iniciativa leídas de un atributo, una habilidad o una salvación toman las entradas de ese atributo, y cualquier otra tirada solo toma las entradas que se aplican a todas las tiradas.
- Un personaje sin ficha no obtiene nada de él, igual que no obtiene ningún modificador.

El resultado indica lo que añadió la ficha (`adjust="-2"` en el registro y una línea en la tarjeta de dados), para que el jugador vea por qué una tirada salió así. Requiere Capability API 1.38 en un conjunto empaquetado.

### Descansos

Un descanso es una lista de restauraciones y elementos que borrar. Cada paso nombra un objetivo (`pool`, `poolGroup`, `listPools`, `track` o `state`) y lo fija (`"to": "max"`, `"to": "min"` o un número) o modifica (`"by": { "const": 1 }` o `"by": { "fractionOfMax": 0.5 }`). Si nombra un contador de heridas, solo puede curarlo, así que su `by` es negativo; consulta arriba. Un paso que nombra un estado vivo solo lo fija, en `"default"` o en uno de sus valores.

### Texto del Game Master

- `checkGuidance` sustituye el párrafo integrado que enseña a pedir pruebas. Indica el sistema y cuándo tirar. El GM solo nombra habilidad y dificultad. El motor tira y calcula desde la ficha: no pidas al modelo que haga matemáticas.
- `sheetGuidance` presenta las fichas en el prompt. Explica qué recursos importan y cuándo gastarlos.
- `worldGuidance` es opcional y se lee una vez al generar el mundo para adaptar la ambientación a las reglas: sin pólvora, magia escasa, muertos que caminan. Nunca llega a un turno.
- `sheetSummary` elige campos, valores derivados y filas de listas que ve el GM de cada personaje. El motor siempre muestra modificadores de atributo, habilidades y salvaciones entrenadas y valores vivos. Mantén breve lo demás, pues se envía en cada turno. El `nameColumn` de una lista del resumen puede ser una columna de texto o enum (un enum muestra la etiqueta de su valor), y `columns` nombra hasta tres columnas más de la propia fila que se muestran tras el nombre, para que el bloque diga `Gear: Crowbar 1d6` en vez de solo nombres. Una columna booleana muestra su etiqueta cuando está activada. Ambos requieren Capability API 1.37 en un conjunto empaquetado.
- No hace falta enseñar el comando de ficha en `sheetGuidance`. El motor enseña cada comando por sí mismo, con los nombres que declara tu archivo, y si tu conjunto tiene un contador de heridas añade el comando `op="damage"` para marcarlo y enumera sus niveles y tipos de daño. Si tiene estados vivos, añade el comando `op="state"` y enumera los valores que puede tomar cada uno.

## Catálogos: entradas preparadas para las listas de la ficha

Escribir una lista de conjuros, una tabla de equipo o una página de rasgos fila por fila es tedioso. Un catálogo es una colección con nombre de entradas preparadas que distribuyes con el conjunto. El editor las ofrece en un selector de cada lista que alimenta el catálogo; elegir una rellena la fila.

Los catálogos son opcionales. Puede haber hasta doce por conjunto; el motor no conoce su temática: todos los IDs, columnas, filtros y palabras proceden del archivo.

### La cabecera

La cabecera va en `catalogs`, en el nivel superior del archivo, junto a `gm`.

```json
"catalogs": [
  {
    "id": "knacks",
    "label": "Knacks",
    "feeds": ["knacks", "tricks"],
    "filters": [
      { "id": "grit", "label": "Grit cost", "type": "number" },
      { "id": "road", "label": "Road", "type": "text" },
      { "id": "callings", "label": "Calling", "type": "tags", "startFrom": { "field": "calling" } }
    ],
    "units": { "distance": { "label": "paces", "perCell": 2 } },
    "entries": []
  }
]
```

- `id` y `label`: el ID sigue las reglas de IDs de ficha; la etiqueta da nombre al selector.
- `holds`: `"rows"` (predeterminado y usado por todos los catálogos anteriores a esta versión), `"creatures"` o `"items"`. Un catálogo de criaturas es un bestiario que lee el combate, y un catálogo de objetos enumera cosas que lleva el grupo. Ninguno de los dos escribe en fichas, declara `feeds` ni aparece nunca en el selector. Consulta [Criaturas](#creatures-a-bestiary-a-fight-reads) y [Objetos](#items-what-a-party-carries) más abajo.
- `feeds`: de una a ocho listas de ficha donde pueden escribir las entradas. Obligatorio para filas y rechazado para criaturas u objetos. Una entrada no puede escribir en otra lista ni colocar un valor que sus columnas no admitan.
- `filters`: opcional, hasta ocho. Define cómo acotar el selector: `number`, un valor `text` o `tags` (varias palabras). `startFrom` nombra un campo con el que se abre el selector: un personaje con Calling Tinker ve primero las entradas Tinker.
- `units`: opcional. Define qué significa en el sistema un alcance o tamaño de área del bloque `mechanics`.

### Una entrada

```json
{
  "id": "road-sense",
  "label": "Road Sense",
  "summary": "You read a road the way other people read a face.",
  "filters": { "grit": 0, "road": "Ash Flats", "callings": ["Scout", "Courier"] },
  "rows": [
    {
      "list": "knacks",
      "values": { "name": "Road Sense", "notes": "Sneak to notice where a road turns bad." }
    }
  ]
}
```

- `id`: minúsculas, dígitos y guiones simples, único dentro del catálogo.
- `label` y `summary`: lo que muestra el selector. El resumen es opcional, de una línea y hasta 300 caracteres.
- `filters`: valores de los filtros de la cabecera. Un filtro `number` recibe un número; `text`, una cadena; `tags`, una lista de cadenas.
- `rows`: lo que escribe la selección, entre una y seis filas. `list` pertenece a `feeds`; las claves de `values` son los IDs de columnas de esa lista.
- `creature`: un adversario en vez de filas, para catálogos cuyo `holds` indica criaturas. Cada entrada tiene exactamente uno entre `rows` y `creature`; una criatura no lleva `mechanics`, pues define lo que hace en sus acciones.

Cada valor se valida contra las columnas de destino. Los nombres mal escritos o números fuera de rango se notifican con su entrada de origen. Las entradas incluidas en el archivo de reglas se validan al cargarlo, que para un archivo importado es durante la importación. Un catálogo separado de paquete se valida la primera vez que lo solicita el selector; si contiene un error, muestra sus motivos en vez de entradas.

### Una entrada, varias listas

Un rasgo de usos limitados ocupa dos filas: el rasgo y su contador. Sigue siendo una sola selección.

```json
{
  "id": "last-ember",
  "label": "Last Ember",
  "rows": [
    {
      "list": "knacks",
      "values": { "name": "Last Ember", "notes": "Spend 1 Grit to give a downed friend 3 Grit back." }
    },
    { "list": "tricks", "values": { "name": "Last Ember", "uses": 1, "recharge": "camp" } }
  ]
}
```

### Valores que mantiene actualizados el conjunto

Los números de una fila pertenecen al jugador desde que la elige. Hay una excepción útil: máximos que dependen del personaje, como usos iguales a un atributo o un recurso de clase que crece por nivel. Una fila puede nombrar hasta cuatro columnas numéricas propias en un mapa `scaled`; el editor mantiene esas celdas actualizadas.

```json
{
  "list": "tricks",
  "values": { "name": "Last Ember", "uses": 1, "recharge": "camp" },
  "scaled": { "uses": { "from": { "abilityScore": "heart" } } }
}
```

- La clave es una columna `number` de la lista.
- `from` es una referencia de valor ordinaria, con el mismo vocabulario cerrado del resto del formato. Para algo más complejo, declara un valor `derived` y apúntalo desde `from` (`"from": { "derived": "lay_on_hands_max" }`). Aquí no se añade aritmética nueva.
- `table` es opcional. Consulta el valor de la referencia en una tabla por peldaños, como cuando un nivel determina un número: `"scaled": { "max": { "from": { "field": "level" }, "table": [[1, 2], [3, 3], [6, 4]] } }`.
- `values` debe seguir incluyendo un número ordinario para la columna; omitirlo se rechaza. Es el valor antes de conocer la ficha y el que conserva una ficha sin esa referencia.
- Una fila con `scaled` debe ser la única de la entrada para esa lista; así, una fila marcada siempre coincide con una sola especificación.

El valor se calcula al editar la ficha, nunca al leerla: una fila guardada siempre contiene el número que declara. Se adapta a su columna: se limita entre `min` y `max` y se redondea hacia abajo si admite enteros. En el ejemplo, Heart 3 da tres usos y Heart 0 o menor, ninguno. La fila permanece con 0 usos; un contador con máximo 0 no es una reserva, por lo que no hay nada que gastar.

Las columnas escaladas requieren Capability API 1.23 para conjuntos empaquetados. Los comunitarios importados los valida el motor que los lee, sin requisitos adicionales.

### Las filas elegidas son copias

Cada fila elegida se copia a la ficha con la clave adicional `_catalog`, que contiene `<catalog id>/<entry id>`. Los IDs de columna empiezan por letra, así que esta clave nunca puede ser una tuya.

La copia pertenece al personaje. El jugador puede editarla, la ficha sigue funcionando sin el conjunto instalado y publicar una versión nueva nunca reescribe personajes. La marca permite al selector mostrar lo que ya tiene la ficha y a la función de actualización identificarlo.

### Actualizar desde el conjunto

La marca permite al editor avisar si el texto nuevo difiere de la fila. Una línea bajo la lista indica cuántas filas tienen texto nuevo; **Review** (Revisar) muestra el contenido guardado junto al del conjunto, con una casilla por fila. Nada se escribe hasta pulsar **Update selected** (Actualizar seleccionadas); solo cambian las columnas diferentes de las filas marcadas. Todo lo demás se conserva, incluida la marca.

La comparación es deliberadamente limitada:

- Solo compara valores existentes de columnas `text`, `longtext`, `dice` y `enum`. Los valores `number` y `boolean` pertenecen al jugador y se conservan, incluidos 0 y false. Una columna aún ausente puede ofrecerse con su valor tipado, incluidos números e interruptores. Se excluyen las columnas escaladas, porque ya siguen la ficha.
- Solo compara columnas definidas por la entrada. Las omitidas no se tocan, contengan lo que contengan.
- Se omiten valores que la propia columna rechazaría, como un `enum` retirado o texto superior a `maxLength`.
- La fila se relaciona con su fila de origen por posición entre las que comparten marca en esa lista, mientras la ficha conserve tantas como escribe la entrada. Si no, solo funciona si la entrada escribe una sola fila en esa lista. Si el jugador borra una de dos filas, se deja la entrada intacta en vez de adivinar.
- Si el catálogo ya no tiene la entrada, la fila se deja intacta sin avisos.

Así, reformular o renombrar puede llegar a personajes que ya eligieron la entrada, si lo aceptan. Cambiar el significado de un número no llega ni llegará: la columna pertenece al jugador desde que la fila es suya.

### `mechanics`: qué hace una entrada en números

Una entrada puede incluir `mechanics` opcional para expresar efectos numéricos: `kind` (`attack`, `heal`, `buff`, `debuff`, `utility`, `rider`), `range`, `area`, `targets`, `targetCount`, `friendlyFire`, `amount` (dados como `2d6` o número fijo), `damageType`, `attackRoll`, `autoHit`, `save` (una salvación de la ficha y el efecto de superarla), `applies` (estados que aplica), `temporary` (puntos temporales de salud), `scales` (cantidad que crece con la ficha), `cost` (reserva que gasta), `perCostStep`, `budget` (parte de la economía de acciones que gasta), `concentration`, `reaction`, `plus`, `free`, `gives`, `standard`, `rider` y `check`.

El selector muestra el bloque en una línea. Quién lee el resto depende del bloque elegido por el conjunto:

- Con [`combat`](#combat-a-fight-your-own-rules-resolve), la pelea lee sus efectos de combate. `range`, `area` y `friendlyFire` se aplican en campos con posiciones; `reaction` marca una entrada que responde a algo en vez de usarse en un turno y nombra el momento que espera (consulta [Ventanas](#windows-holding-the-fight-open)); una marcada simplemente con `true` no nombra ninguno y no aparece en ningún menú. `check` afecta a pruebas de habilidad, como se explica arriba.
- Con solo [`battle`](#battles-lending-the-sheet-to-marinaras-combat), la batalla lee `kind`, `range`, `area`, `friendlyFire`, `amount`, `damageType` y `cost`, pues el combate propio de Marinara tiene dónde aplicarlos.

El vocabulario es cerrado: las claves o valores que no están en la lista se rechazan en vez de ignorarse.

`cost` también es lo que paga el comando `use` del GM fuera de batalla, descrito a continuación.

### El comando `use`: permitir al Game Master pagar el coste que escribiste

Mientras narra, el GM actualiza fichas con comandos `[sheet: ...]`: `spend`, `restore` (`heal` significa lo mismo), `damage`, `temp`, `track`, `condition`, `note` y `rest`. Los conjuntos con catálogos reciben otro:

```
[sheet: who="Mira" op="use" name="Fireball"]
[sheet: who="Mira" op="use" name="Fireball" pool="3rd-level slots"]
```

`op="cast"` equivale a `op="use"` y `spell=` a `name=`; así funciona la expresión que elige el GM sin que el formato deba conocer la palabra "spell".

El nombre se compara sin distinguir mayúsculas con las filas del personaje procedentes de tus catálogos. La fila responde al nombre mostrado al GM (columna de nombre de `sheetSummary`, después `pools.nameColumn` y después la primera columna de texto) y al `label` de su entrada de origen. Así, renombrar una fila no hace perder su acceso. Se rechazan tanto nombres sin coincidencia como nombres compartidos por dos entradas distintas.

Lo que gasta:

- Cada término de `mechanics.cost`. Si nombra una reserva viva, paga de ella. Si nombra un GRUPO, paga de la primera reserva del grupo que pueda costearlo, por orden de declaración. No sube automáticamente a otra reserva superior, porque un grupo no siempre es una escala.
- Además, uno de cada reserva de fila escrita por la misma entrada, como el contador de usos de un rasgo: la segunda fila de `Last Ember`. Un contador con máximo 0 no tiene usos; se rechaza el comando en vez de ejecutarlo gratis.

`pool=` es el lanzamiento a nivel superior: el mismo precio único, pagado desde otra reserva del grupo. Solo se acepta si el coste tiene un único término y la reserva nombrada comparte su grupo. Lo demás se rechaza sin reinterpretarlo.

Es todo o nada. Si no puede pagarse alguna parte, se rechaza el comando entero, no cambia nada y se avisa al jugador. Una entrada sin coste, como un truco o rasgo pasivo, se acepta sin cambiar nada.

### Incluido en el archivo o en uno propio

Un catálogo pequeño se incluye en `ruleset.json`, dentro de `entries` de la cabecera. Uno largo vive en su propio archivo y la cabecera lo nombra con `asset`. Cada catálogo tiene exactamente una de esas opciones.

```json
{ "id": "knacks", "label": "Knacks", "feeds": ["knacks"], "asset": "catalogs/knacks.json" }
```

La ruta siempre es `catalogs/<the catalog's id>.json`. El archivo tiene esta forma:

```json
{ "schemaVersion": 1, "catalog": "knacks", "entries": [] }
```

Los archivos separados son para paquetes publicados mediante el catálogo oficial: el paquete los incluye en `contributions.assets.paths` junto a `ruleset.json` y necesita Capability API 1.21. Un catálogo de criaturas, incluido o separado, necesita Capability API 1.27, y uno de objetos, 1.49. **Un conjunto importado como archivo único o compartido mediante GitHub incluye sus catálogos dentro del archivo**, de modo que deben caber en el límite total de 256 KB. Eso permite unos cientos de entradas breves.

Los límites son doce catálogos por conjunto, 2000 entradas por catálogo en ambas formas y 1 MB por archivo de catálogo.

<a id="items-what-a-party-carries"></a>

## Objetos: lo que lleva el grupo

Las armaduras, las armas, las pociones, el equipo, la munición y el dinero son objetos. Un bloque opcional `items` declara las palabras con las que se escribe cada objeto de tu conjunto, y un catálogo con `holds: "items"` enumera los propios objetos. Ambos necesitan Capability API 1.49; `rarityCaps` y `propose` del bloque, que rigen los objetos que inventa el Game Master, necesitan 1.51; un valor que lee los objetos de un personaje (`itemStat`) necesita 1.52; lo que hace un objeto en las pruebas mientras se lleva puesto o encima, con los límites de `bonus` del bloque, necesita 1.53; lo que un objeto exige a quien lo lleva puesto y los atributos que cambia necesitan 1.54; el `attack` de un arma necesita 1.55; lo que hace un objeto en un combate necesita 1.56; lo que dispara un arma y lo que tiene cargado necesitan 1.57; los modos de un arma, el ataque con la mano secundaria, el mínimo y los estados al impactar necesitan 1.58; lo que hace usar un objeto en un combate, con las cargas que contiene, necesita 1.59; un uso que restaura una reserva necesita 1.60; las cargas recuperadas al descansar o un objeto que se rompe al vaciarse necesitan 1.61; un uso que pide antes una prueba (`gate`) necesita 1.62; las tablas de botín y el botín de una criatura necesitan 1.63; una línea de botín que suelta monedas o una capa que retira monedas necesita 1.64; un mercado, el lugar `sold` de un objeto y un `service` necesitan 1.65; un día de mercado (unos fondos diarios, `buyBack`, el `stock` de un vendedor, una cantidad `limited` o un `sold` que no nombra ningún lugar) necesita 1.69, y un efecto adicional que se activa con las armas que se empuñan (`weapons`) necesita 1.70.

### El bloque de objetos

El bloque va en `items`, en el nivel superior del archivo. Este es el de Ember Roads, algo abreviado:

```json
"items": {
  "categories": [
    { "id": "weapon", "label": "Weapon" },
    { "id": "armor", "label": "Armor" },
    { "id": "ammunition", "label": "Ammunition" }
  ],
  "rarities": [
    { "id": "common", "label": "Common" },
    { "id": "storied", "label": "Storied" }
  ],
  "tags": [
    { "id": "thrown", "label": "Thrown" },
    { "id": "ranged", "label": "Ranged" },
    { "id": "two_handed", "label": "Two-handed" },
    { "id": "arrow", "label": "Arrow" }
  ],
  "stats": [
    { "id": "bulk", "label": "Bulk", "type": "number", "min": 0, "max": 10, "default": 0 },
    { "id": "guard", "label": "Guard", "type": "number", "min": 0, "max": 4, "default": 0 },
    { "id": "damage", "label": "Damage", "type": "dice", "example": "1d6" },
    { "id": "swing", "label": "Rolls with", "type": "enum", "values": ["brawn", "wits", "heart"], "default": "brawn" },
    { "id": "reach", "label": "Reach", "type": "enum", "values": ["close", "near", "far"], "default": "close" }
  ],
  "slots": [
    { "id": "body", "label": "Body", "count": 1 },
    { "id": "hands", "label": "Hands", "count": 2 }
  ],
  "carry": { "stat": "bulk", "encumberedAbove": { "derived": "load" }, "limit": { "const": 12 } },
  "rarityCaps": [
    { "rarity": "common", "stats": { "guard": 1 }, "bonus": 1 },
    { "rarity": "storied", "stats": { "guard": 3 }, "bonus": 2 }
  ],
  "currencies": [
    {
      "id": "coin",
      "label": "Coin",
      "perWeight": 100,
      "units": [
        { "id": "bit", "label": "bits", "value": 1 },
        { "id": "mark", "label": "marks", "value": 10 },
        { "id": "sovereign", "label": "sovereigns", "value": 100 }
      ]
    },
    {
      "id": "salt",
      "label": "Salt",
      "perWeight": 10,
      "units": [
        { "id": "pinch", "label": "pinches", "value": 1 },
        { "id": "cake", "label": "cakes", "value": 20 }
      ]
    }
  ]
}
```

- `categories`: de una a 24. Cada objeto tiene exactamente una, como armadura, arma o poción.
- `rarities`: opcional, hasta 12. Enuméralas de la más común a la más rara.
- `tags`: opcional, hasta 48. Propiedades que puede tener un objeto, como arrojadizo o plateado.
- `stats`: opcional, hasta 24. Los números y palabras que lleva un objeto, declarados exactamente como una columna de lista: `number`, `text`, `boolean`, `enum` o `dice`, con rango, valores y valor predeterminado cuando el tipo los tiene. `promptVisible` (`true` de forma predeterminada) indica si se muestra al Game Master la estadística junto al objeto. Ponlo en `false` para algo que solo debe ver el jugador, como dónde está escondido un objeto.
- `slots`: opcional, hasta 12. Dónde se lleva puesto o se empuña un objeto, y cuántas ranuras de ese tipo tiene un personaje, de 1 a 20.
- `binding`: opcional. Sintonización, investidura o cualquier otra cosa que limite cuántos objetos puede tener vinculados a la vez un personaje. `label` es su nombre y `max` es un valor leído de la ficha del personaje, como `{ "abilityScore": "nerve" }` o un valor derivado. No puede leer el estado vivo, porque cuántos objetos puede vincular un personaje no cambia con cada golpe.
- `carry`: opcional. `stat` nombra la estadística numérica que es el peso de un objeto, cuyo `min` es 0 o más. `encumberedAbove` es cuánto lleva un personaje antes de quedar sobrecargado, y `limit` (opcional), lo máximo que puede llevar. Ambos se leen de la ficha como `binding.max`. Sin `carry`, el peso no significa nada y nadie queda nunca sobrecargado.
- `currencies`: opcional, hasta seis familias de una a diez monedas cada una.
  - Las monedas de una familia se cambian entre sí según `value`, que cuenta la moneda más pequeña de la familia. Así, la moneda más pequeña vale 1 y no hay dos monedas de una familia que valgan lo mismo.
  - Dos familias nunca se cambian entre sí. La moneda de otra nación, o los favores de una ambientación, son una familia propia.
  - El ID de una moneda es único entre todas las familias, porque el coste de un objeto nombra solo la moneda.
  - `perWeight` (opcional, y solo junto a `carry`) indica cuántas monedas de la familia pesan una unidad de la estadística de carga.
  - Las monedas son el dinero del grupo y se llevan en las bolsas como cualquier objeto (consulta Dinero, más abajo).
- `rarityCaps`: opcional, como máximo una por rareza. Lo máximo que puede dar un objeto que inventa el Game Master en esa rareza: el valor más alto de cada estadística numérica nombrada en `stats`, dentro del rango de esa estadística y como número entero si la estadística admite enteros, y `bonus` (Capability API 1.53), la bonificación fija más alta que puede sumar uno de sus modificadores al llevarse puesto o encima. En una rareza con `bonus`, se omite una bonificación en dados, porque los dados no pueden limitarse a un número; una penalización nunca se limita. Un objeto inventado queda sujeto a ello; los objetos que enumeran tus catálogos son tuyos y nunca se limitan.
- `propose`: `true` de forma predeterminada. `false` prohíbe al Game Master inventar objetos de tu conjunto.
- `native`: `true` de forma predeterminada. `false` desactiva los objetos sin tipo propios de Game Mode en las partidas de tu conjunto (consulta **Dónde se leen los objetos**). El Game Master puede seguir inventando objetos, escritos con las palabras de tu conjunto.
- `freeform`: en qué se convierte un objeto que escribe el jugador. `"plain"` (predeterminado) lo conserva como objeto sin reglas, como hasta ahora. `"refuse"` solo permite objetos de tu conjunto.
- `lootTables` (Capability API 1.63): opcional, hasta 24. Lo que sueltan las criaturas de un combate ganado y el `[loot:]` del Game Master (consulta Botín, más abajo).

### Un objeto

Un catálogo de objetos no declara `feeds`:

```json
{ "id": "outfitter", "label": "Outfitter", "holds": "items", "entries": [] }
```

Cada entrada lleva un `item` en vez de `rows` o una `creature`:

```json
{
  "id": "hunting-bow",
  "label": "Hunting bow",
  "summary": "Yew, waxed string, and a grip worn smooth by someone else's hand.",
  "item": {
    "category": "weapon",
    "rarity": "common",
    "tags": ["ranged", "two_handed"],
    "stats": { "bulk": 2, "damage": "1d8", "swing": "wits", "reach": "far" },
    "slots": { "hands": 2 },
    "cost": { "amount": 3, "unit": "sovereign" }
  }
}
```

- `id`, `label`, `summary` y `filters` funcionan igual que en cualquier entrada. La etiqueta es el nombre del objeto.
- `category` es obligatorio. `rarity` y `tags` son opcionales, y los tres nombran IDs propios del bloque. Un objeto solo puede tener rareza si el bloque declara alguna.
- `stats`: valores para las estadísticas del bloque. Cada uno es un valor que su estadística podría contener.
- `slots`: cuántas ranuras de cada tipo ocupa el objeto, nunca más de las que tiene un personaje.
- `stack`: lo máximo que contiene una pila, de 1 a 999.999. Sin él, una pila contiene tantos como cualquier pila de Game Mode. Un objeto que contiene `charges` va solo en su pila.
- `cost`: una cantidad entera (`amount`) de una moneda, nombrada por su ID en `unit`. El selector la muestra, y el Game Master la ve junto al objeto (consulta Dinero, más abajo).
- `sold` (Capability API 1.65): `{ "place": "city" }`, el más pequeño de los lugares de tu mercado que vende el objeto, por encima de cualquier regla que diga otra cosa; con 1.69 puede, en su lugar o además, indicar cuántos tiene disponibles un lugar en un día (`"limited": 2` o `"limited": "1d3"`) (consulta Mercados, más abajo).
- `service` (Capability API 1.65): `true` para alojamiento, un pasaje o una bendición: se compra como un objeto y nunca se lleva encima. Comprarlo solo paga; nada más lo mete en una bolsa (añadir uno se rechaza como `service`), y el selector omite los servicios. Un servicio tiene un `cost`, y nada relacionado con llevarlo o usarlo (`slots`, `stack`, `binds`, `worn`, `carried`, `requires`, `attack`, `use`, `charges`).
- `binds`: el objeto debe vincularse antes de hacer nada mientras se lleva puesto. `restriction` (opcional) indica con palabras quién puede vincularlo, y `cursed: true` marca uno que no se deja soltar. Solo un conjunto con `binding` puede tener objetos que se vinculan.
- `worn` y `carried` (opcionales, Capability API 1.53): lo que hace el objeto en las pruebas y salvaciones de quien lo tiene mientras lo lleva puesto y mientras solo lo lleva encima, (1.54) los atributos que fija o aumenta y (1.56) lo que hace en un combate. Consulta [Pruebas fuera de un combate](#checks-outside-a-fight) y [Armadura y efectos puestos en un combate](#armor-and-worn-effects-in-a-fight).
- `requires` (opcional, Capability API 1.54): lo que exige el objeto a quien lo lleva puesto y lo que se aplica mientras no lo cumple. Consulta [Pruebas fuera de un combate](#checks-outside-a-fight).
- `attack` (opcional, Capability API 1.55): lo que hace el objeto como arma en un combate mientras se lleva puesto, (1.57) lo que dispara y tiene cargado, y (1.58) sus otros modos, un ataque con la mano secundaria, un mínimo para su daño y los estados que aplica al impactar. Consulta [Armas en un combate](#weapons-in-a-fight), [Munición y recarga](#ammunition-and-reloading) y [Modos, una segunda arma, un mínimo y estados al impactar](#modes-a-second-weapon-a-floor-and-conditions-on-a-hit).
- `use` y `charges` (opcionales, Capability API 1.59): lo que hace usar el objeto, en un combate y fuera de él, y las cargas que contiene para que ese uso las gaste, y (1.60) una reserva que restaura. Consulta [Usar objetos en un combate](#using-items-in-a-fight) y [Usar objetos fuera de un combate](#using-items-outside-a-fight).

Un objeto no lleva `mechanics`: lo que hace se escribe en su bloque `item`.

### Dónde se leen los objetos

Todo lo anterior se comprueba al importar el conjunto, y tus catálogos de objetos se distribuyen con él. En una partida, los lee el inventario de Game Mode:

- **Tus objetos están en la bolsa.** Una pila puede ser uno de tus objetos y sigue siendo ese objeto, lo llame como lo llame el jugador. El botón **From the ruleset** (Del conjunto de reglas) del inventario abre un selector con todos los objetos que enumeran tus catálogos, con la búsqueda y los filtros del catálogo, y una capa que oculta una entrada también la oculta ahí.
- **Los nombres encuentran tus objetos.** Un nombre que escribe el jugador, o uno que escribe el Game Master en `[inventory: action="add"]`, que sea la `label` de uno de tus objetos, con cualquier combinación de mayúsculas, añade ese objeto. Si dos objetos comparten etiqueta, un nombre encuentra el que tus catálogos enumeran primero.
- **`stack`** se respeta: añadir, fijar, combinar o dar por encima de él llena la pila y empieza una nueva.
- **`freeform: "refuse"`** deja al jugador solo tus objetos: el selector y los nombres que son tus objetos.
- **`native: false`** deja al Game Master solo tus objetos y los que inventa: se rechaza un nombre que no es ninguna de las dos cosas (`not-ruleset-item`), aunque sigue pudiendo añadirse más de algo que ya se tiene, y sus instrucciones lo indican. Un combate ya no pregunta a un modelo qué hacen los objetos del inventario, así que solo combaten tus objetos: en un combate del conjunto, tus armas empuñadas y los usos de tus objetos (consulta Armas en un combate, más abajo), y en una batalla Classic o Tactical, tus objetos con un `use` (consulta Objetos en batallas Classic y Tactical, más abajo). Lo que escribe el jugador sigue rigiéndose por `freeform`, y lo que llevaba el grupo vuelve en una sesión nueva en cualquier caso.
- **Qué es un objeto** se muestra en la pila seleccionada: su categoría, rareza y propiedades por sus etiquetas, las estadísticas que da, su resumen y cuántos contiene una pila. El selector también muestra su `cost`. El Game Master ve cada uno de tus objetos que tiene con su categoría, rareza, propiedades y las estadísticas que dejaste en `promptVisible`, como `Hand axe [Weapon, Common, Thrown; Damage 1d6, Reach close]`.

- **`slots`**: un objeto que ocupa ranuras puede equiparlo quien lo lleva mientras tenga esas ranuras libres, y un objeto de una pila mayor se separa en su propia pila para llevarlo puesto. El inventario muestra cada ranura en uso por personaje.
- **`binding`**: un objeto que `binds` puede vincularse, hasta `binding.max` leído de la ficha de su portador (un personaje sin ficha lee una en blanco). Un objeto `cursed`, una vez vinculado, sigue vinculado: el jugador no puede desvincularlo, quitárselo, regalarlo ni eliminarlo, y solo el Game Master puede terminar la maldición.
- **`carry`**: un objeto pesa su valor de `carry.stat` (un objeto sin él no pesa nada), y la carga de un personaje es lo que pesa su bolsa, comparada con `encumberedAbove` y `limit` leídos de su propia ficha. Un objeto añadido a la vista compartida del inventario, por el jugador o por el Game Master sin `who=`, va a quien pueda llevarlo sin quedar sobrecargado (primero el jugador y después el grupo en orden), repartido según el espacio que le queda a cada uno cuando nadie puede llevarlo entero. Nada supera el `limit` de nadie: lo que nadie puede llevar se queda atrás y se informa al Game Master. Dale a una estadística de peso `"integer": false` para pesos como un cuarto de libra.
- **El Game Master inventa objetos con tus palabras.** Salvo que pongas `propose: false`, su `[inventory: action="add"]` puede describir un objeto nuevo: `like=` uno de tus objetos como punto de partida y después cualquiera de `category=`, `rarity=`, `tags=`, `stats=` (pares `id=value`), `slots=` (`id=count`), `binds=` (`yes`, `cursed` o `no`), `worn=` y `carried=` (cambios separados por `;`, cada uno `+N`, `-N`, `advantage`, `disadvantage` o `fails` para salvaciones, en habilidades o salvaciones por nombre, o en `checks` o `saves` para todas ellas, `+N` o `-N` en el nombre de un atributo para subirlo o bajarlo y, si tienes combates, `+N`, `-N`, ventaja o desventaja en los ataques y `+N` o `-N` en la defensa o en tu palabra para ella: `worn="+1 Sneak; disadvantage on Sway checks"`, `carried="+1 Brawn"`, `worn="+1 Guard"` o `none`) y `summary=`, cada parte por su ID o etiqueta. Una estadística de objeto que ya cuenta tu defensa (Guard de Ember Roads suma el `guard` de lo que se lleva puesto) es la forma en que una armadura la aumenta, así que se omite un cambio puesto en la defensa junto a esa estadística y la respuesta lo indica: un modelo pequeño escribe `guard=1` y `+1 Guard` para un único +1. También lee lo que suele escribir en su lugar un modelo pequeño: un número tras el nombre (`Brawn +1`), "bonus" tras los ataques o la defensa (`+1 attack roll bonus`), `none` para una lista vacía de propiedades, estadísticas o ranuras, y `worn=`, `carried=` o `summary=` dentro de `stats=` cuando no tienes ninguna estadística con ese nombre. El motor solo conserva lo que tiene tu bloque: se omite una categoría, propiedad, estadística, ranura, habilidad o salvación desconocida, una rareza que no tienes se convierte en la más baja, un número se limita al rango de su estadística y después a `rarityCaps` para su rareza, una bonificación puesta o encima a la `bonus` de su rareza (incluida la parte de la que partió `like`), y un nombre que es uno de tus objetos es simplemente ese objeto. La respuesta indica al Game Master qué se cambió (nunca sobre una estadística que no le muestras), y los detalles del objeto muestran cada cambio al jugador. La partida conserva el objeto, así que el mismo nombre es ese objeto el resto de la partida, y una sesión nueva lo conserva mientras alguien lo siga teniendo.
- El Game Master puede hacer `equip` y `unequip` de tus objetos con su comando de inventario si tienes `slots`, y `bind` y `unbind` si tienes `binding`: solo se le informa de los que tiene tu conjunto. Ve la carga de cada personaje, sus objetos vinculados y sus ranuras, y lo que lleva puesto o vinculado.

- **Lo que hace un objeto mientras se lleva puesto o encima** se muestra en la pila seleccionada y en el selector ("While worn: -1 on checks (Sneak)"), y el Game Master lo ve junto al objeto (`worn: -1 on checks (Sneak)`). Las pruebas fuera de un combate lo aplican (consulta Pruebas fuera de un combate, más abajo).
- **Lo que hace un arma** se muestra igual ("Attack (Action): Brawn to hit, 1d6 + Brawn cut damage"), y un combate la ofrece mientras se empuña (consulta Armas en un combate, más abajo).
- **Lo que hace usar un objeto** se muestra igual ("Use (Action): heals 1d4 + 1, range 0 paces, used up"), con las cargas que le quedan. Lo ofrece un combate, y también el botón **Use** (Usar) del inventario y el `[inventory: action="use"]` del Game Master (consulta Usar objetos en un combate y Usar objetos fuera de un combate, más abajo).

En una batalla Classic o Tactical, cada uno de tus objetos con un `use` hace ahí lo que indica, y uno sin él no se ofrece (consulta Objetos en batallas Classic y Tactical, más abajo). Diga lo que diga `native`, la ficha puede leer tus objetos (abajo), un arma empuñada es un ataque en un combate del conjunto y dispara lo que saca de la bolsa, lo que hace un objeto mientras se lleva puesto o encima cuenta en él (consulta Armadura y efectos puestos en un combate, más abajo), un objeto con un `use` se usa según sus propias reglas, en un combate y fuera de él (consulta Usar objetos en un combate y Usar objetos fuera de un combate, más abajo), y tus `currencies` son el dinero del grupo (consulta Dinero, más abajo).

<a id="items-on-the-sheet"></a>

### Objetos en la ficha

Una referencia de valor puede leer los objetos que tiene un personaje con `itemStat` (Capability API 1.52). Ember Roads suma la guardia de la armadura que lleva puesta un viajero a su Guard, que es lo que debe superar un golpe:

```json
{
  "id": "guard",
  "label": "Guard",
  "op": "sum",
  "of": [{ "const": 6 }, { "abilityMod": "wits" }, { "itemStat": { "stat": "guard", "from": "worn", "pick": "sum" } }]
}
```

- `from`: `"worn"` son los objetos que lleva puestos el personaje: un objeto que ocupa ranuras mientras está equipado, uno que `binds` mientras está vinculado, y uno que hace ambas cosas mientras cumple ambas. Un objeto que no hace ninguna nunca se lleva puesto. `"carried"` es el resto de lo que tiene, y `"all"`, ambos.
- `pick`: `"sum"` suma el valor de cada objeto multiplicado por cuántos hay; `"max"` y `"min"` leen el valor individual más alto o más bajo, y `"count"` cuenta los objetos según cuántos hay.
- `stat` nombra una de tus estadísticas de objeto. `sum`, `max` y `min` necesitan una estadística `number`. `count` puede omitirla para contar todos los objetos, o nombrar cualquier estadística para contar solo los objetos que la dan.
- `slot`, `category` y `tag` (opcionales) conservan solo los objetos que ocupan esa ranura, son de esa categoría o tienen esa propiedad.
- `default` (opcional, 0 si se omite) es lo que lee cuando no se elige ningún objeto o ninguno da la estadística.

Los objetos cambian durante el juego, así que `itemStat` sigue la misma regla que una lectura viva: el `max` de una reserva o de un contador, la bonificación de competencia, `binding.max`, los números de `carry` y la columna escalada o el escalado de un catálogo no pueden leerlo, ni siquiera mediante un valor derivado. En cualquier otro lugar donde se lea un número, sí puede: un valor derivado, el `adjust` de una prueba, el `cap` de una habilidad, la defensa, la absorción o la iniciativa de un combate.

Los objetos de un personaje son los de su propia bolsa. La tarjeta del jugador (la que lleva el nombre del personaje con el que juega el chat o, si no, la primera) lee la bolsa del jugador, y cada una de las demás tarjetas, la suya. Solo cuentan los objetos de tu conjunto, porque un objeto simple no tiene estadísticas que leer. La ficha en pantalla, una prueba, el bloque de ficha del Game Master y un combate al empezar leen lo que tiene cada personaje en ese momento. Fuera de una partida, en el editor de fichas o en la revisión de una importación, nadie tiene nada e `itemStat` lee su `default`.

<a id="checks-outside-a-fight"></a>

### Pruebas fuera de un combate

Una prueba o salvación que el Game Master pide fuera de un combate lee más que la ficha: los estados activos del propio personaje, los niveles que ha alcanzado, lo que hacen sus objetos mientras los lleva puestos o encima, y lo que cuesta el requisito de un objeto puesto mientras no lo cumple. Es el mismo vocabulario que usan los estados de un combate (Capability API 1.53 para las claves de objeto y la restricción).

Un objeto indica lo que hace con `worn`, que se aplica mientras lo lleva puesto (y vinculado, si se vincula), y `carried`, que se aplica mientras solo lo lleva encima. El abrigo de cuero de Ember Roads cruje cuando te mueves a hurtadillas, y una piedra de camino hace que la gente de las caravanas confíe en ti:

```json
"worn": { "modifiers": [{ "to": "checks", "skills": ["sneak"], "flat": -1 }] }
```

```json
"carried": { "modifiers": [{ "to": "checks", "skills": ["sway"], "flat": 1 }] }
```

Cada uno toma las partes de un estado que lee una prueba:

- `effects`: `own-checks-advantage`, `own-checks-disadvantage`, `own-saves-advantage` y `own-saves-disadvantage`.
- `modifiers`: cambios `to` pruebas o salvaciones, con `flat`, `dice` (y `minus`), `mode` y sus propios `skills` o `saves`, exactamente como los de un estado.
- `failsSaves`: salvaciones que su portador falla sin tirar.
- `skills` y `saves`: a qué habilidades o salvaciones se refieren los efectos y los modificadores que no nombran las suyas.
- `abilities` (Capability API 1.54): atributos que cambia, por ID, cada uno `{ "set": N }` (al menos N: una puntuación más alta se mantiene) o `{ "add": N }` (un número negativo lo reduce). Se aplican antes de calcular la ficha, así que todo lo que lee el atributo lee el valor cambiado: la ficha en pantalla, los valores derivados, las pruebas, el bloque de ficha del Game Master y un combate al empezar. Primero se aplican las sumas, después el `set` más alto, y se respetan el `min` y el `max` del propio atributo. El máximo de una reserva o de un contador y la bonificación de competencia se calculan solo a partir de la ficha, así que un objeto no los mueve.

Los guanteletes de piel de buey de Ember Roads prestan a un agarre débil el de un boyero:

```json
"worn": { "abilities": { "brawn": { "set": 2 } } }
```

Lo que hace un objeto en un combate (defensa, ataques, velocidad, los demás efectos del combate y el daño y los estados que evita) está en [Armadura y efectos puestos en un combate](#armor-and-worn-effects-in-a-fight); una prueba fuera de un combate solo lee las partes anteriores. Un objeto se aplica una vez, por muchas pilas de él que tenga alguien.

Un objeto también puede exigir algo a quien lo lleva puesto, con `requires`: un valor de su ficha (`value`, cualquier referencia de valor, leída como todo lo que está en juego), lo mínimo que puede ser (`atLeast`) y lo que se aplica mientras no lo alcanza (`otherwise`, con el mismo vocabulario, salvo que no puede cambiar un atributo, porque lo que exige puede leer uno). Hasta cuatro. La pala de sepulturero de Gravewatch es un trabajo pesado para un guardián débil:

```json
"requires": [
  {
    "value": { "abilityScore": "sinew" },
    "atLeast": 3,
    "otherwise": { "modifiers": [{ "to": "checks", "skills": ["dig"], "flat": -1 }] }
  }
]
```

Un requisito lee el atributo tal como lo dejan los objetos de quien lo lleva puesto, así que unos guanteletes que fijan Sinew en 3 cumplen el de la pala.

En una prueba:

- Cada modificador que se refiere a ella suma su número, y sus dados se tiran: un número fijo en una prueba sumada, dados en una reserva. Una prueba de atributo, y una prueba que el conjunto no puede nombrar, solo leen lo que no está restringido a nada, y una salvación solo lee lo que se refiere a salvaciones.
- La ventaja y la desventaja de todas las fuentes, incluido el `mode=` del Game Master, se anulan entre sí: con cualquiera de cada una, la prueba se tira una sola vez. Solo se inclina un conjunto que tira dos veces (`resolution.advantage`).
- Una salvación que un estado u objeto hace fallar falla sin tirar, y no se gasta nada en ella.
- El registro indica lo que la cambió: `effects="-1"` para el número, `from="Leather coat"` para lo que la cambió y `automatic="true"` para una salvación fallada sin tirar. La tarjeta de dados muestra lo mismo, y el Game Master lo lee en el turno siguiente. Se le indica que el motor aplica estos cambios, para que no vuelva a sumarlos.

Gravewatch muestra lo mismo en una reserva: la Dawn bell vinculada añade un dado a Ward, y Rattled quita uno de Soothe y de Barter.

<a id="weapons-in-a-fight"></a>

### Armas en un combate

Un objeto puede ser un arma (Capability API 1.55). Su `attack` tiene la forma de las [filas de ataque](#combat-a-fight-your-own-rules-resolve) de un bloque de combate, con valores en lugar de columnas, y un [combate del conjunto](#combat-a-fight-your-own-rules-resolve) lo ofrece como ataque mientras el objeto se lleva puesto: empuñado con las manos que ocupa y vinculado si se vincula. Uno guardado o que solo se lleva encima no ofrece nada, y las filas de ataque y las acciones propias de una criatura funcionan exactamente como antes. El hacha de mano de Ember Roads:

```json
"attack": {
  "budget": "act",
  "toHit": { "abilities": { "stat": "swing" } },
  "damage": { "dice": { "stat": "damage" }, "abilities": { "stat": "swing" }, "type": "cut" },
  "reach": 2,
  "range": { "normal": 10, "long": 20 }
}
```

- `budget`: el presupuesto que gasta, uno de `combat.economy.budgets`.
- `toHit`: lo que suma para acertar. `abilities` son uno o más IDs de atributo, y cuenta el mejor de ellos (un arma de destreza enumera dos). `skill` suma lo que suma una prueba de esa habilidad, con el atributo del ataque en lugar del propio de la habilidad, igual que en una fila de ataque. `proficiency` es un valor de la ficha del portador: si lee más de 0, se suma la bonificación de competencia del conjunto (`{ "const": 1 }` para siempre). `bonus` es un número. `target` es el objetivo por dado propio de un combate de reservas para el ataque de esta arma (la pala de Gravewatch cuenta cada dado desde 6 en vez del 7 de la reserva), y solo puede darlo un conjunto `dice-pool` cuyo `target` pueda moverse; en un combate sumado, su `bonus` dice lo mismo.
- `damage`: los `dice` que inflige, con el mejor de sus `abilities` de daño sumado, un `bonus` y un `type`. Un combate sumado necesita `dice`; en un combate de reservas, un impacto inflige sus éxitos y `dice` añade esa cantidad más del dado propio de la reserva.
- `reach` y `range` (`normal`, y `long` para lo que sigue alcanzando más allá) están en tu unidad de `combat.distance` y la necesitan. Un arma con ambos es arrojadiza, igual que una fila de ataque: un golpe de cerca, un lanzamiento más allá. Sin ninguno, alcanza una casilla.
- `versatile`: los `dice` que inflige en su lugar mientras cada ranura que ocupa tenga sitio para otro tanto entre lo que lleva puesto su portador: una lanza en una mano con la otra libre.
- `strikes`: cuántos ataques compra un gasto, leído de la ficha del portador, igual que el `strikes` de una fila de ataque. Un arma sin él da un ataque por gasto, por muchos ataques que tenga quien la empuña.

Cualquiera de esos números o palabras puede leer en su lugar una estadística del propio objeto: `{ "stat": "damage" }` es el `damage` del objeto, y una estadística a la que el objeto no da nada es como si el valor no estuviera escrito. `abilities` y `skill` leen una estadística enum cuyas palabras son IDs de atributo o de habilidad, y `type`, una estadística de texto o enum. Así, un objeto que el Game Master inventa `like=` un arma también es un arma y combate con sus propias estadísticas, limitadas por `rarityCaps` como cualquier estadística (un valor que escribes se copia tal cual). Uno inventado sin ranura ni vinculación nunca se lleva puesto, así que se crea sin el ataque. Un modelo pequeño suele describir un arma y omitir `like=`, así que uno inventado en una categoría de tus armas sin nada de lo que partir combate como el arma de esa categoría a la que más se parece por nombre (una palabra compartida en cualquier sentido, así que una ballesta se parece a un arco) o, si no, como la primera de ellas, tomando cualquier estadística que lea ese ataque y que la propuesta omitió, y la respuesta lo indica ("It fights as Hand axe does."). El formulario de propuesta del Game Master también le indica que un arma creada como otra combate como ella.

Las propiedades de un arma son lo que llevan sus golpes. Una entrada `resist` o `immune` de una criatura puede indicar qué la atraviesa: `{ "type": "tearing", "except": ["silver"] }` se recibe completo de un arma con la propiedad `silver` y se resiste de cualquier otra cosa. El grave wight de Gravewatch está escrito así, y su silver coffin nail lo atraviesa.

Los detalles del arma y la línea del Game Master indican lo que hace: `attack (Action): Brawn to hit, 1d6 + Brawn cut, reach 2 paces, range 10 to 20 paces`, con `1d8 with a hand free` para la boar spear de Ember Roads y `at 6` junto a la pala de Gravewatch.

<a id="ammunition-and-reloading"></a>

#### Munición y recarga

Un arma puede disparar algo y puede tener una cantidad cargada propia (Capability API 1.57). El arco de caza de Ember Roads dispara flechas, y la pistola de guardia de Gravewatch lleva una bala, cargada con la munición y la pólvora del guardián:

```json
"ammo": { "tag": "arrow", "recover": 0.5 }
```

```json
"ammo": { "tag": "shot" },
"clip": { "max": 1, "reload": "act" }
```

- `ammo`: lo que dispara el arma, según una de tus `tags` de objeto. Cada ataque toma `perAttack` (1 si no dice nada) de los objetos con esa propiedad que lleva su portador, puestos o no, de la primera pila de ese tipo de su bolsa y después de la siguiente. El arma solo se ofrece mientras lleve suficientes, y el menú del combate indica cuántos quedan. Tras un combate que gana el grupo, `recover` (de 0 a 1) de lo que disparó cada uno vuelve a la pila de la que salió: la proporción se suma a lo largo de todo el combate y se redondea hacia abajo una vez por pila, así que la mitad de cinco flechas disparadas desde un carcaj son dos. Un combate perdido o del que se huye no devuelve nada.
- `clip`: una cantidad cargada que el arma lleva consigo. `max` es cuánto contiene, escrito o leído de una estadística numérica del objeto, y `reload` es el presupuesto que gasta una recarga. Un ataque gasta lo cargado en vez de tomar de la bolsa, y el arma solo se ofrece mientras contenga suficiente para uno. Una opción **Reload** (Recargar) en el menú del combate gasta `reload` y la llena: desde la bolsa si tiene `ammo` (todo lo que aún contenga la bolsa si es menos) y por completo si no tiene. La cantidad se guarda en la pila de inventario del arma, así que una pistola vaciada en un combate sigue vacía en el siguiente. Un arma que nadie ha disparado aún está cargada, y también una que un jugador combina en una pila de varias, porque una cantidad cargada pertenece a un arma. Las balas de un cargador no se recogen tras un combate, así que `recover` es para un arma sin cargador.

Lo que un combate dispara, carga y recoge se escribe en el inventario a medida que se da cada paso, igual que la salud del grupo, y el diario indica lo que se usó. Se rechaza un paso cuya pila ya no existe o contiene menos de lo que contaba el combate, igual que un objeto gastado que el combate no encuentra. El registro del combate indica cada disparo, recarga y recogida ("Hunting bow: 2 left to shoot.", "Ada loads 1 into Watch pistol: 1 of 1 loaded."), y los detalles del objeto y la línea del Game Master indican lo que dispara y contiene el arma: `ammunition Arrow (1 an attack, 50% picked up after a won fight)` y `holds 1, reload (Act)`.

<a id="modes-a-second-weapon-a-floor-and-conditions-on-a-hit"></a>

#### Modos, una segunda arma, un mínimo y estados al impactar

Un arma puede tener otras formas de hacer su ataque, una compañera en la mano secundaria, un daño mínimo que inflige y estados que aplica a lo que golpea (Capability API 1.58). El arco de caza de Ember Roads puede disparar una andanada, y el silver coffin nail de Gravewatch puede empuñarse en cada mano y marca aquello en lo que se clava hondo:

```json
"modes": [{ "id": "volley", "label": "Volley", "ammo": 2, "toHit": -2, "targets": 2 }]
```

```json
"offHand": true,
"onHit": [{ "condition": "marked", "atLeast": 2, "rounds": 2 }]
```

- `modes`: hasta seis formas más de hacer el ataque, cada una con un `id`, una `label` y lo que cambia. `ammo` es cuántos disparos hace un ataque con ese modo, así que el arma tiene `ammo` o un `clip` desde el que disparar. `toHit` es lo que suma para acertar: dados en un combate de reservas y un número en uno sumado. `target` mueve en esa cantidad el objetivo por dado de un combate de reservas (desde el propio del arma o, si no, el de la reserva; `+1` es más difícil), si el objetivo de la reserva puede moverse. `targets` es a cuántos puede apuntar, cada uno con su propia tirada de ataque. El menú del combate pregunta de qué forma una vez elegido el ataque (después de su estilo de iniciativa, si hay estilos), cada una con lo que se espera que haga, y solo ofrece los modos para los que su portador tiene disparos. Un miembro del grupo que interpreta el motor evalúa cada modo dirigido a un objetivo, y deja a un jugador los modos para varios. En una ventana, un ataque se hace tal cual.
- `offHand`: un arma para la mano secundaria. El bloque de combate nombra el presupuesto que gasta un segundo ataque y si su daño conserva un atributo positivo:

  ```json
  "offHand": { "budget": "quick", "ability": "full" }
  ```

  Una vez que su portador ha atacado este turno con un arma marcada `offHand`, cada una de las demás armas de ese tipo que lleva puestas ofrece un segundo ataque con ese presupuesto, llamado "Silver coffin nail, off hand" en el menú. `ability` es `full` (predeterminado) o `penalty-only`, que suma el atributo de daño solo cuando resta algo, como hace el combate con dos armas de 5e. Un ataque con la mano secundaria es un único golpe, compre los ataques que compre el ataque principal, y nunca un golpe contra alguien que se aleja. Un arma marcada `offHand` necesita el `offHand` del bloque de combate.
- `floor`: lo mínimo que inflige un impacto, escrito o leído de una estadística numérica del objeto. El daño de un combate de reservas después de la absorción, o el daño de un combate sumado, se sube hasta él antes de que una resistencia lo reduzca a la mitad, y el registro lo indica. Un golpe que gasta tira el número de quien lo hace y nunca se sube.
- `onHit`: hasta cuatro de tus estados que recibe el objetivo cuando el daño que infligió el golpe alcanzó `atLeast`, durante `rounds` de sus propios turnos o, sin él, hasta que algo lo quite. El daño es lo que le llegó después de la absorción y de sus resistencias; un golpe que toma quita iniciativa en vez de causar daño, así que no aplica ninguno.

Los detalles del arma y la línea del Game Master lo indican todo: `modes Volley (2 shots, -2 to hit, up to 2 targets)`, `off hand (Quick)`, `at least 1 on a hit before resistance` y `Marked for 2 rounds when a hit deals 2 or more`.

<a id="armor-and-worn-effects-in-a-fight"></a>

### Armadura y efectos puestos en un combate

Lo que hace un objeto mientras se lleva puesto o encima también puede cambiar un combate (Capability API 1.56) para su portador, junto a sus estados y niveles. Cada uno toma las partes de un estado que lee un combate:

- `effects`: cualquiera de los de un estado salvo los cuatro que no puede tener un nivel (`half-move-to-stand`, `ends-on-damage`, `cannot-target-source`, `cannot-approach-source`), porque nadie le puso un objeto a su portador y nunca termina por sí solo: `own-attacks-advantage`, `attacks-against-disadvantage`, `resist-all`, `speed-zero` y los demás.
- `modifiers`: cambios `to` la defensa, los ataques y la velocidad, además de las pruebas y salvaciones, exactamente como los de un estado; en un combate `dice-pool`, un cambio a los ataques es un número fijo de dados, igual que el de un estado.
- `resist`, `vulnerable` e `immune`: tipos de daño de los que su portador recibe la mitad, el doble o nada, igual que los de una criatura (y comprobados contra `combat.damageTypes` si declaras alguno). `conditionImmunities`: tus propios estados que nunca se aplican a su portador.

El widow's ring maldito de Gravewatch cuesta un dado en cada ataque, su Dawn bell vinculada impide que a su portador se le aplique Rattled, y una piedra de camino de Ember Roads llevada encima protege del calor:

```json
"worn": { "modifiers": [{ "to": "attacks", "flat": -1 }] }
```

```json
"worn": { "modifiers": [{ "to": "checks", "skills": ["ward"], "flat": 1 }], "conditionImmunities": ["rattled"] }
```

```json
"carried": { "modifiers": [{ "to": "checks", "skills": ["sway"], "flat": 1 }], "resist": ["burn"] }
```

Un combate lee cada objeto una vez, con el nombre de la pila ("Widow's ring" junto a una tirada en el registro, y en una defensa), igual que una prueba; el `otherwise` de un requisito no cumplido también cuenta en un combate, así que una armadura demasiado pesada para quien la lleva puede costar velocidad. Los detalles de un objeto y la línea del Game Master lo indican todo ("While worn: -1 on attacks", "worn: +1 on checks (Ward), immune to Rattled"). Una armadura que simplemente suma a una defensa o a una absorción es una estadística de objeto leída con `itemStat` (consulta Objetos en la ficha), igual que el Guard de Ember Roads lee el abrigo que se lleva puesto.

**Dureza.** Si la iniciativa es un número que mueven los ataques, `combat.pool.hardness` es la dureza de un combatiente, leída de su ficha al empezar el combate (la de su armadura, mediante `itemStat`), y una criatura da su propia `hardness`. Un golpe que gasta cuyos dados están por debajo de ella impacta y no hace nada: el número de quien lo hace sigue volviendo a la base, el registro lo indica ("Ada's Grave spade lands on the warden with 5 dice, below a hardness of 6, and does nothing.") y el menú no le prevé ningún daño. Solo puede tenerla un conjunto con un estilo que gasta, y una criatura escrita como ficha la toma de la ficha.

<a id="using-items-in-a-fight"></a>

### Usar objetos en un combate

Un objeto puede usarse en un combate (Capability API 1.59): una cataplasma sobre un corte, un tónico que se traga, una campana que se toca. Su `use` indica lo que hace usarlo con las palabras que usa el `mechanics` de una entrada de catálogo (consulta [Qué lee un combate de `mechanics`](#what-a-fight-reads-from-mechanics)): `kind` (`heal`, `attack`, `buff` o `debuff`), `amount`, `damageType`, `plus`, `attackRoll`, `save`, `applies`, `temporary`, `range`, `area`, `targets`, `targetCount` y `friendlyFire`, con el `budget` que gasta, o `free`. La cataplasma de Ember Roads y el tónico de calor y la campana del alba de Gravewatch:

```json
"use": { "kind": "heal", "budget": "act", "range": 0, "targets": "ally", "amount": { "dice": "1d4", "flat": 1 }, "consumes": true }
```

```json
"use": { "kind": "heal", "budget": "quick", "targets": "self", "amount": { "flat": 1 }, "consumes": true }
```

```json
"stack": 1,
"use": {
  "kind": "debuff",
  "budget": "act",
  "targets": "enemy",
  "save": { "save": "steel", "onSuccess": "negates" },
  "saveDifficulty": 7,
  "applies": [{ "condition": "rattled", "duration": { "rounds": 2 } }],
  "charges": 1
},
"charges": { "max": 3 }
```

- Un uso aparece en el menú del combate bajo **Items** (Objetos), con el nombre del objeto ("Poultice") se llame como se llame la pila. Un objeto que ocupa ranuras o se vincula se usa mientras se lleva puesto (y vinculado, si se vincula), como la campana; cualquier otro, mientras se lleva encima.
- Gasta un presupuesto, o es `free`, y paga consigo mismo: un `cost`, `perCostStep`, `check`, `concentration`, `reaction`, `scales`, `gives`, `standard` o `rider` es para una entrada de catálogo que tiene una ficha, y se rechaza en un uso.
- `toHit`, en un uso con `attackRoll`, es lo que suma para acertar, tal como lo indica el `toHit` de un arma (consulta Armas en un combate): atributos, una habilidad, una bonificación y, en un combate de reservas, un objetivo por dado. Sin él, la tirada no suma nada.
- `saveDifficulty`: el número contra el que se tira una salvación que pide (la suya propia, una que termina un estado que aplica, o la de una cláusula sin `difficulty`), escrito o leído de una estadística numérica del objeto. Una entrada de una ficha lee ese número de la fuente de su catálogo; un objeto no tiene ninguna, así que indica el suyo.
- `consumes: true` quita uno de la pila del objeto cada vez que se usa, y el menú indica cuántos quedan. El último que se usa se lleva la pila con él.
- `restore` (Capability API 1.60) devuelve parte de una reserva de cada ficha a la que llega, como una bolsa de sangre devuelve sangre: `{ "pool": "resolve", "amount": { "flat": 1 } }`, con `dice` y `flat` como los tiene un `amount`. Nombra una de tus reservas vivas, nunca la reserva de salud (la salud vuelve con una curación), y va en un uso que ayuda, un `heal` o un `buff`. Una criatura no tiene reservas, así que no recibe nada. El registro lo indica ("Ada gets back 1 Resolve, and is on 3 of 4."), y un miembro del grupo que interpreta el motor deja en paz una reserva llena. El tónico de calor de Gravewatch restaura un punto de Resolve junto a su curación.
- `charges` gasta esa cantidad de las `charges` propias del objeto, cuyo `max` es cuántas contiene, de 1 a 100, escrito o leído de una estadística numérica del objeto (se rechaza un objeto que da esa estadística como menos de 1, y uno que pasa de 100 contiene 100). La cantidad se guarda en la pila de inventario del objeto, así que una campana tocada dos veces en un combate tiene una carga en el siguiente, y una que nadie ha tocado está llena. Las cargas pertenecen a un objeto, así que un objeto que las contiene tiene un `stack` de 1. Un uso se consume o gasta cargas, nunca ambas cosas, y las cargas de un objeto siempre las gasta su uso. Dos claves de `charges` van más allá (Capability API 1.61), como las tiene la campana del alba de Gravewatch:

  ```json
  "charges": { "max": 3, "recharge": { "rests": ["vigil"], "amount": "max" }, "breaksOn": { "die": 20, "atMost": 1 } }
  ```

  - `recharge`: cuáles de tus `rests` devuelven las cargas y cuántas: `"max"` llena el objeto, y una cantidad (`{ "dice": "1d6", "flat": 1 }`) devuelve esa cantidad, nunca por encima de `max`. Un descanso devuelve las cargas de cada objeto que lleva su personaje y que lo nombra, tanto si lo hace el botón Rest de la ficha como el `[sheet: op="rest"]` del Game Master; la ficha y la bolsa se escriben juntas, y una respuesta regenerada solo recarga una vez. La ficha indica lo que volvió tras el descanso ("Dawn bell 3/3 charges"), y el Game Master ve las cargas que le quedan a cada objeto junto a él en el inventario ("2 of 3 charges left").
  - `breaksOn`: cuando un uso gasta la última carga, el motor tira un d`die`, y con `atMost` o menos el objeto se rompe y desaparece de la bolsa, en un combate y fuera de él. El registro del combate lo indica ("Dawn bell breaks (a 1 on its die)."), también el informe del botón Use ("Its last charge spent, it breaks."), y el diario lo cuenta como perdido.
- `gate` (Capability API 1.62) es una prueba que su usuario supera antes de que el objeto funcione, como pide un pergamino de un conjuro por encima del nivel de quien lo lee. `check` nombra lo que se tira: `{ "skill": "ward" }`, `{ "ability": "wits" }` o un valor de la ficha (`{ "value": { "derived": "spell_mod" } }`, para un número que varía de un personaje a otro, como el modificador de lanzamiento de conjuros de un lanzador de 5e). `difficulty` va de 1 a 100, escrito o leído de una estadística numérica del objeto (un objeto que no lo da queda fuera del menú, y su botón Use lo dice claramente). `unless` omite la prueba cuando un valor de la ficha del usuario es al menos `atLeast`. Una prueba fallida consume el objeto (o gasta sus cargas) para nada. En un combate, la prueba se tira al gastar el objeto, con los dados propios del combate (una reserva en un combate de reservas, donde `difficulty` son los éxitos que necesita) y el número de la ficha leído al empezar el combate, y cuenta lo que suman a esa prueba los estados y objetos puestos del usuario, incluido lo propio de una habilidad. El registro lo indica ("Ada rolls Ward to use Page of the vigil litany: 2 successes from 2 dice at 7 or more (8, 9), needing 2 successes, a success."). La página de la letanía de vigilia de Gravewatch tiene una:

  ```json
  "gate": { "check": { "skill": "ward" }, "difficulty": 2, "unless": { "value": { "abilityScore": "nerve" }, "atLeast": 3 } }
  ```

Lo que un combate consume y las cargas que deja se escriben en el inventario a medida que se da cada paso, igual que los disparos de un arma, y el diario indica lo que se usó. El registro del combate indica cada uso ("Poultice: 1 of 2 left.", "Dawn bell: 2 of 3 left."). Un miembro del grupo que interpreta el motor solo usa una curación en alguien herido, y nadie usa un objeto para golpear a alguien que se aleja. Los detalles del objeto y la línea del Game Master indican lo que hace usarlo: "Use (Action): heals 1d4 + 1, range 0 paces, used up" y `use (Act): Steel 7 save negates it, Rattled, 1 of 3 charges`, y los detalles muestran las cargas que quedan. Un objeto que el Game Master inventa `like=` uno de los tuyos se usa como ese, cargas incluidas.

### Objetos en batallas Classic y Tactical

Un conjunto que no resuelve sus propios combates los deja a las batallas Classic y Tactical propias de Game Mode, donde se pregunta a un modelo qué hacen los objetos del inventario cuando empieza un combate. Tus objetos no se adivinan: cada uno con un `use` hace lo que este indica, con los números propios del motor, y la suposición del Game Master solo se pide para los objetos que no son tuyos (y para ninguno con `native: false`).

- Un `heal` cura y un `attack` daña en una proporción de la salud máxima del objetivo, según la media de su `amount`: una media de 7, el `1d8+3` de un arma básica, es algo más de una quinta parte, nunca menos de una vigésima parte y como mucho toda. Esa es la escala con la que el puente de combate lee los números de una entrada de catálogo. El `damageType` es el elemento.
- El primer estado de `applies` se aplica como estado con su propio nombre, durante sus `rounds` (2 si no las tiene), y un `buff` o `debuff` es solo ese estado, que sube o baja la defensa como lo hacen los estados propios del motor.
- `targets` indica en quién se usa. Sin él, una curación o una mejora va a un aliado, y un ataque o un perjuicio, a un enemigo.
- `consumes: true` quita uno de la pila, y un uso sin él deja el objeto donde está.
- Una tirada para acertar, una salvación, un área, los puntos temporales y una reserva restaurada no tienen cabida en estas batallas, igual que ahí con el `mechanics` de una entrada de catálogo: la descripción del objeto en el menú sigue indicándolos.
- Un objeto que ocupa ranuras solo se usa mientras se lleva puesto, y uno que se vincula, solo mientras está vinculado, igual que en un combate del conjunto. Uno que se consume se toma de lo que se lleva puesto, nunca de uno de repuesto de la bolsa.
- Un objeto cuyo uso gasta `charges` se ofrece mientras le quede un uso, y el menú Items cuenta sus usos. Cada uso las gasta de la pila del objeto, primero la del propio jugador, y el último gasto tira su `breaksOn`: si se rompe, desaparece de la bolsa y el diario lo cuenta como perdido.
- Un objeto que pide antes una prueba (`gate`) la tira en el servidor cuando lo usa un miembro del grupo, con la ficha de ese miembro y lo que lleva puesto (un acompañante sin tarjeta propia tira con una ficha en blanco, y solo la unidad propia del jugador recurre a la tarjeta del jugador), igual que la tira el botón Use, y se omite cuando se cumple `unless`. Si falla, el objeto se gasta (o se gastan sus cargas) y no hace nada, y el registro lo indica: "Ada rolls Ward to use Page of the vigil litany: 0 against 2, failed, and it is used up for nothing."
- Un objeto sin `use` no se ofrece en estas batallas, ni tampoco una curación sin `amount`.

El motor calcula por sí mismo el efecto de cada objeto, así que lo que envía la pantalla nunca decide lo que hace uno de tus objetos.

### Botín

Un combate ganado suelta botín en toda partida de Game Mode, una vez, en las bolsas del grupo. Sin conjunto, procede de las tablas propias de Game Mode (más y más raro en una partida más difícil), y lo mismo ocurre en un conjunto que no declara tablas de botín y deja `native` activado. Un conjunto que declara tablas de botín suelta en su lugar sus propios objetos, y uno con `native: false` y sin tablas no suelta nada.

`lootTables` en el bloque `items` (Capability API 1.63) las enumera, como hacen los ajuares funerarios de Gravewatch:

```json
"lootTables": [
  {
    "id": "grave_goods",
    "label": "Grave goods",
    "rolls": "1d2",
    "entries": [
      { "item": "kit/shot-and-powder", "weight": 4, "count": "1d4" },
      { "item": "kit/warming-tonic", "weight": 3 },
      { "item": "kit/litany-page", "weight": 2 },
      { "filter": { "category": "arm" }, "weight": 1 },
      { "coins": "shilling", "weight": 2, "count": "1d6" }
    ]
  }
]
```

- `rolls` es cuántas elecciones hace la tabla: un número de 0 a 20, o dados (`"1d2"`), 1 de forma predeterminada.
- Cada elección saca una línea según su `weight` (1 de forma predeterminada) frente a las demás. Una línea nombra uno de tus objetos como `<catalog>/<entry>`, un `filter` por `rarity`, `category` y `tag`, que elige por igual entre todos los objetos que coinciden, o (Capability API 1.64) `coins`, una de tus monedas por su ID. `count` es cuántos se sueltan, un número o dados, 1 de forma predeterminada. Un objeto o una moneda que retira una capa no suelta nada, ni tampoco un filtro que no encuentra ninguno.
- Una criatura del bestiario nombra la tabla que lleva con `loot` (consulta Criaturas, más abajo). Un combate del conjunto ganado tira la tabla de cada criatura derrotada, y un combate que el motor no resuelve con tus reglas no tiene criaturas del bestiario, así que no suelta nada.
- Lo que se suelta va a la vista compartida igual que una adición, preguntando primero a la bolsa del jugador, y lo que nadie puede llevar se queda atrás. El resumen que recibe el Game Master indica lo que se soltó y que ya está en las bolsas, el diario lo indica y una notificación lo muestra.
- El Game Master tira una tabla en la historia con `[loot: table="grave_goods" who="Ada"]` (`who=` para la bolsa de un personaje), medido desde donde empezó el turno, igual que sus etiquetas de inventario, así que una respuesta regenerada suelta una sola vez. Cada objeto que se suelta se responde como una adición, y sus instrucciones enumeran tus tablas.

### Dinero

Tus `currencies` son el dinero del grupo. Una moneda es un objeto en las bolsas, una pila de cada moneda como cualquier otra, con el nombre de su `label`:

- Pesa una unidad de tu estadística de carga por cada `perWeight` monedas de su familia, cuenta para la carga de su portador y se coloca según la regla de carga cuando entra. Una familia sin `perWeight` no pesa nada.
- El jugador añade monedas desde la lista **Coins** (Monedas) del selector, y las divide, regala y combina como cualquier pila.
- Una línea sobre las pilas del inventario muestra las monedas de cada familia a la vista y su valor en la moneda más pequeña de la familia. El Game Master ve ese valor junto a cada bolsa (`Coin worth 432 pennies`, o `no coins` para un monedero vacío, con la bolsa del jugador incluida aunque no contenga nada), y el `cost` de un objeto junto al objeto (`costs 3 shillings`).
- El Game Master paga con `[inventory: action="pay" amount="3 shillings" who="Ada"]` y cobra con `action="earn"`. `amount` es una cantidad y una moneda, por su ID o etiqueta, en singular o plural, con cualquier combinación de mayúsculas (`1 penny`, `12 pennies`). Un pago sale de una bolsa (la del jugador sin `who=`), solo en la familia propia de la moneda: primero van las monedas más grandes que caben, después se cambia la moneda más pequeña restante que cubre lo que aún se debe, y el cambio vuelve en las monedas más pequeñas de la familia, empezando por las más grandes. La respuesta indica lo que se pagó y lo que volvió: pagar 3 shillings de un monedero que contiene una crown es `Paid with crowns ×1; shillings ×2 back.` Se rechaza un pago que la bolsa no puede cubrir (`cannot-afford`), una moneda que no tienes (`unknown-coin`) y cualquier pago en un conjunto sin monedas (`no-currencies`). Un cobro es una adición de esa moneda, a la bolsa de `who` o repartida según la regla de carga. El diario enumera lo gastado y lo cobrado, y una notificación lo muestra. Sus instrucciones enumeran tus monedas, y solo las tiene un conjunto que tenga alguna.
- Comprar es un pago seguido de una adición o, en un conjunto con mercado, un `buy` (abajo). No hay pantalla de tienda.

### Mercados

Un `market` en el bloque `items` (Capability API 1.65) indica qué vende un lugar y a qué precio, en vez de que todas las tiendas vendan de todo a su precio de lista. El de Gravewatch, algo abreviado:

```json
"market": {
  "prices": [
    { "id": "cheap", "label": "cheap", "times": 0.75 },
    { "id": "fair", "label": "fair", "times": 1, "default": true },
    { "id": "dear", "label": "dear", "times": 1.5 }
  ],
  "places": [
    { "id": "hamlet", "label": "hamlet" },
    { "id": "village", "label": "village" },
    { "id": "town", "label": "market town" },
    { "id": "city", "label": "city" }
  ],
  "sold": [
    { "filter": { "rarity": "rare" }, "place": "town" },
    { "filter": { "category": "arm" }, "place": "village" }
  ],
  "sellers": [
    { "id": "chandler", "label": "chandler", "sells": [{ "category": "coat" }, { "category": "tonic" }] },
    { "id": "smith", "label": "smith", "sells": [{ "category": "arm" }], "place": "village" },
    {
      "id": "chapel",
      "label": "chapel of the Vigil",
      "sells": [{ "category": "page" }],
      "place": "town",
      "only": { "value": { "abilityScore": "nerve" }, "atLeast": 3, "label": "wardens of Nerve 3 or more" }
    }
  ]
}
```

- `prices`: de uno a ocho niveles, cada uno un multiplicador (`times`, mayor que 0) sobre el `cost` de un objeto, y uno de ellos el `default`. Un precio es el coste en la moneda más pequeña de su familia multiplicado por el nivel y redondeado en ella (nunca por debajo de una moneda para algo que cuesta algo), y se expresa en la moneda en la que está escrito su coste si esa moneda lo paga exactamente o, si no, en la moneda más grande que dejan tus capas y que lo hace: en `dear`, un abrigo que cuesta 8 shillings cuesta 12 shillings, y una poción de 5e que cuesta 50 gp cuesta 50 gold pieces, nunca 5 platinum. Regatear o el humor de un vendedor mueven el nivel, nunca el número.
- `places`: de uno a doce tamaños de lugar, del más pequeño al más grande, con tus propias palabras.
- `sold`: reglas para el lugar más pequeño que vende lo que elige un filtro. Un filtro nombra una `rarity`, una `category` y una `tag`, cualquiera de ellas, y coincide con un objeto que tenga todas las que nombra; decide la primera regla que coincide con un objeto. El `sold` propio de un objeto tiene prioridad sobre las reglas, y un objeto que nada nombra se vende en cualquier lugar.
- `sellers`: tipos de vendedor, cada uno con lo que `sells` (filtros, cualquiera de los cuales elige un objeto), el `place` más pequeño que tiene uno (cualquier lugar, sin él) y, opcionalmente, a quién vende `only`: un valor de la ficha del comprador, con su estado vivo y lo que lleva encima, al menos `atLeast`, que se indica al Game Master como `label`. Sin `sellers`, cualquiera en un lugar vende lo que vende ese lugar.
- Un objeto sin `cost` no está a la venta, ni tampoco uno que oculta una capa o uno cuyas monedas retiró una capa.

**Dónde está el grupo.** El Game Master indica en qué lugar está una escena y su tamaño con `[place: name="Millbrook" size="market town"]`, el tamaño por su ID o etiqueta, y omite el tamaño para un lugar sin mercado (un camino, las tierras salvajes). El motor responde a cada etiqueta en su sitio, rechazando un tamaño que no tienes, y el último lugar al que respondió en sus propias respuestas que ve el jugador, desde el inicio de conversación más reciente, es el lugar vigente hasta que se indique otro. Una etiqueta de lugar que escribe un jugador nunca cuenta. Una respuesta regenerada solo lee lo que vino antes de la indicación que sustituye.

**Comprar.** El Game Master compra con `[inventory: action="buy" item="Hand axe" count="1" level="dear" seller="smith" who="Ada"]`, con `level` (el predeterminado sin él) y `seller` (cualquier vendedor de aquí que venda el objeto, sin él) opcionales. El motor le pone precio en el lugar vigente cuando termina la respuesta, saca el precio del monedero del comprador (el del jugador sin `who=`) igual que un pago, con cambio, y mete el objeto en su bolsa, o solo paga un servicio. Ocurren ambas cosas o ninguna. La respuesta indica lo que costó (`price="12 shillings"`) y lo que se pagó y se devolvió, y una notificación lo muestra. Una compra se rechaza, sin cambiar nada, si no se ha indicado lugar (`no-place`), si un nivel, objeto o vendedor no existe (`unknown-level`, `unknown-item`), si no hay precio (`not-for-sale`), si el lugar es demasiado pequeño para el objeto (`not-here`), si no hay aquí ningún vendedor que lo venda (`no-seller`), si el comprador no cumple el `only` de un vendedor (`not-to-you`), si el monedero no puede cubrirlo (`cannot-afford`) o si una bolsa no puede llevarlo.

**Lo que ve el Game Master.** Mientras una escena está en un lugar con mercado, un bloque MARKET nombra el lugar y su tamaño, los niveles de precio y cada vendedor que hay allí con una docena de lo que vende al nivel predeterminado, de lo más barato a lo más caro; cualquier cosa que venda puede comprarse por su nombre. Sus instrucciones enseñan `[place:]` y `buy`, y solo las tiene un conjunto con mercado.

#### Un día de mercado

Un día de mercado (Capability API 1.69) hace que las existencias de un lugar cambien y se agoten, y permite al grupo vender. El de Gravewatch, algo abreviado:

```json
"places": [
  { "id": "hamlet", "label": "hamlet", "purse": { "amount": 10, "unit": "shilling" } },
  { "id": "town", "label": "market town", "purse": { "amount": 100, "unit": "shilling" } }
],
"buyBack": 0.5,
"sold": [{ "filter": { "rarity": "rare" }, "place": "town", "limited": "1d2" }],
"sellers": [
  { "id": "smith", "label": "smith", "sells": [{ "category": "arm" }], "stock": { "table": "smiths_odd_lot" } }
]
```

- **El día.** Un día en un lugar dura hasta que el Game Master nombra otro lugar; empieza un día nuevo en el mismo lugar volviendo a indicarlo con `day="new"` (`[place: name="Millbrook" size="market town" day="new"]`). Todo lo siguiente se calcula por día a partir de las respuestas del Game Master que ve el jugador, así que un swipe, una respuesta regenerada o una rama no necesitan guardar nada, y un día siempre tira las mismas existencias.
- **`limited`** en una regla `sold` o en el `sold` propio de un objeto: cuántos tiene disponibles un lugar en un día, un número o dados tirados para el día. Una regla puede nombrar un límite sin lugar (se vende en cualquier lugar, unos pocos cada vez); el lugar de un objeto procede de la primera regla que nombra un lugar, y su límite, de la primera que nombra un límite. Una vez comprados, una compra se rechaza (`sold-out`) hasta un día nuevo.
- **`stock`** en un vendedor: una de tus tablas de botín, tirada una vez al día para lo que ese vendedor tiene en existencias además de lo que vende. Lo que sale (solo objetos, nunca monedas) lo vende únicamente ese vendedor, tantos como salieron, digan lo que digan `sells` y `sold`; el `only` de un vendedor sigue aplicándose.
- **`buyBack`**: la proporción (mayor que 0, como máximo 1) del precio predeterminado de un objeto que paga un vendedor por él. Lo compra un vendedor de aquí que venda ese tipo de cosa (cualquiera, sin vendedores). Sin `buyBack`, nadie compra nada.
- **`purse`** en un tamaño de lugar: lo máximo que pagan en total sus vendedores en un día, en una de tus monedas. Las ventas pagadas en la familia de esa moneda cuentan contra ese máximo; sin fondos, compran sin límite.
- **Vender.** El Game Master vende con `[inventory: action="sell" item="Silver coffin nail" count="1" seller="smith" who="Ada"]`. El motor le pone precio, lo saca de la bolsa de quien vende (la del jugador sin `who=`; nunca un objeto maldito vinculado) y mete sus monedas, empezando por las más grandes, o lo rechaza sin cambiar nada: si no se ha indicado lugar, si nadie aquí lo compra (`not-bought`), si no hay precio (`not-for-sale`), si los fondos del día se han agotado (`purse-spent`), si no se tiene ninguno o si la bolsa no puede llevar las monedas. La respuesta indica lo que se obtuvo (`price="4 shillings"`), el diario lo registra como vendido y una notificación lo muestra.
- El bloque MARKET añade lo que tiene cada vendedor en existencias hoy, cuántos quedan de un objeto limitado, lo que pagan los vendedores de aquí y cuánto queda de los fondos del día; sus instrucciones enseñan `sell` y `day="new"`.

<a id="using-items-outside-a-fight"></a>

### Usar objetos fuera de un combate

El mismo `use` funciona fuera de un combate (Capability API 1.60 para `restore`; el resto no necesita nada nuevo). El botón **Use** del inventario, en uno de tus objetos con un `use`, y el `[inventory: action="use" item="Poultice" who="Juno"]` del Game Master hacen lo que hace el objeto a quien lo lleva, con los dados del motor:

- Una curación devuelve salud: la cantidad en una reserva de salud, o una marca eliminada en un contador de heridas, igual que una curación en un combate. Los puntos `temporary` van a una reserva de salud, `restore` devuelve su reserva y cada estado que `applies` se aplica hasta que algo lo quite.
- Un uso que daña, o que apunta al otro bando (`kind` `attack` o `debuff`, o `targets` `enemy`), no alcanza a nadie: fuera de un combate no hay nadie en el tablero a quien golpear. El objeto se usa igualmente, y se informa al Game Master de lo que hace para que lo narre.
- Quita uno de la pila o gasta sus cargas exactamente igual que un combate, y uno al que no le queda ninguna, o uno que ocupa una ranura o se vincula y no se lleva puesto, se rechaza sin cambiar nada. La ficha y la bolsa se escriben juntas.
- Un `gate` se tira primero, igual que una prueba: los dados propios del conjunto con el número de la ficha, la penalización por heridas del usuario y lo que hacen a esa prueba sus estados y sus objetos puestos o encima. Uno fallido consume el objeto para nada, y la línea lo indica ("Ada uses Page of the vigil litany: Ward check 1 against 2, failed; it is used up for nothing. 1 left.").

Lo que ocurrió se indica en una línea: "Juno uses Poultice: heals 4 (Grit 6/11). 1 left." El botón Use lo envía al Game Master en un bloque `[item_used]` después de "I use my Poultice.", que el chat muestra como una insignia; la propia etiqueta del Game Master se responde con él, medida desde donde empezó el turno igual que sus otras etiquetas, así que una respuesta regenerada nunca usa un objeto dos veces. Un objeto sin `use`, o en una partida sin conjunto, se sigue indicando sin más: "I use my rope."

<a id="battles-lending-the-sheet-to-marinaras-combat"></a>

## Batallas: prestar la ficha al combate de Marinara

De forma predeterminada, una batalla no conoce la ficha. Construye los combatientes como siempre; un personaje puede salir de una pelea sin cambios en los puntos de golpe de su ficha.

El bloque opcional `battle` cambia eso en una sola dirección: presta las cifras al combate y escribe su resultado de vuelta. **No hace que el combate siga tus reglas.** Los dados, quién acierta y cuánto daño hace siguen siendo de Marinara. Por eso la salud se transfiere como proporción del máximo, no como tu cifra: media salud en la ficha equivale a media barra de la que Marinara creó. Tu reserva de 9 puntos no entra directamente en una pelea donde un golpe hace 12.

```json
"battle": {
  "health": { "pool": "grit" },
  "energy": { "pool": "luck" },
  "skills": [{ "list": "knacks" }]
}
```

- `health`: obligatorio. Reserva viva que representa los puntos de golpe. Debe pertenecer a `sheet.live.pools`, no ser una lista cuyas filas son reservas.
- `energy`: opcional. Reserva que puede gastar el combate y que se convierte en la barra MP. Debe ser distinta de `health`, pues no se pueden gastar puntos de golpe como energía.
- `slots`: opcional. Reservas gastadas de una en una, cada una con `level` de 1 a 9: `[{ "pool": "slots_1", "level": 1 }]`. Cada nivel y reserva se usa una sola vez.
- `skills`: opcional, hasta ocho. Listas cuyas filas pasan a ser habilidades de combate. Solo cuentan filas de tus catálogos cuya entrada tenga `mechanics`: una fila escrita a mano no expresa efectos numéricos. `onlyWhen` nombra una columna booleana que debe estar activada, como un conjuro preparado. `alwaysWhen` nombra una columna y un valor que permiten entrar de todos modos, como conjuros que no requieren preparación. Es la excepción a `onlyWhen` y se rechaza sin él.

### Qué entra y qué vuelve

**Al entrar**, para cada miembro cuya ficha tiene la partida: la proporción de salud determina el punto inicial en la barra de Marinara; la reserva de energía se convierte en MP; cada reserva de espacios se convierte en espacios de ese nivel; las filas marcadas se convierten en habilidades. Salud máxima, ataque, defensa, velocidad y nivel siguen siendo cifras de Marinara. Una reserva de salud a cero hace empezar caído; por encima de cero nunca se empieza con menos de un punto, para que el redondeo no derribe a alguien por una proporción pequeña.

**Al salir**, una vez terminada la pelea: la proporción final de la barra se convierte a la escala de la reserva y se aplica como daño o curación la diferencia respecto al inicio. Energía y espacios son cantidades, no proporciones, y vuelven tal cual. Todo sigue las reglas de los botones de la ficha; los cambios rechazados se omiten y notifican, no se fuerzan. Una pelea que no cambió la salud no escribe ningún cambio, para que las conversiones por sí solas nunca alteren la ficha.

**En ninguna dirección**: tiradas de ataque, salvaciones, concentración y efectos de pagar más. Están en `mechanics` para que los lea un sistema de combate completo en el futuro; este puente no los aplica y el conjunto no debe afirmar que sí.

Una batalla abandonada no escribe nada. Si borras el mensaje que la inició o nunca llega al final, la ficha queda igual: la pelea no ocurrió.

### Cómo se convierte una entrada en habilidad

El bloque `mechanics` se interpreta así:

- `kind` determina el tipo de habilidad. Se omiten entradas `utility` y las marcadas `reaction`, pues el combate de Marinara no tiene dónde aplicarlas.
- `amount` determina la fuerza como multiplicador del ataque del combatiente, no como daño fijo. Dados mayores nunca golpean más suave; el multiplicador permanece en el rango usado por habilidades generadas.
- `range` y `area.size` se dividen por `units.distance.perCell` del catálogo para obtener casillas, sin redondear nunca a cero. Una explosión usa su radio, un cono la mitad y una línea una casilla. Un área afecta a todos los enemigos cubiertos y respeta `friendlyFire`.
- `damageType` se convierte en elemento. No se transfiere `targets`: Marinara determina los objetivos válidos de curaciones, mejoras y ataques por el tipo de habilidad.
- Un `cost` de energía se convierte en coste MP; varios se suman. Un `cost` de exactamente un espacio gasta uno de ese nivel. Marinara cobra una cantidad de energía o un espacio, nunca ambos: omite entradas con dos espacios, espacios de dos niveles o espacio más energía. También omite costes de otras reservas, como salud o recursos de clase, para no concederlos gratis.
- `buff` y `debuff` se convierten en los efectos propios de Marinara. Otras promesas del texto, como quitar un estado de la ficha, no se aplican. Omite `mechanics` si el efecto solo tiene sentido fuera de batalla.

`coverage.combat` sigue siendo independiente y conserva su significado: actívalo solo si las batallas siguen realmente las reglas de tu sistema.

<a id="combat-a-fight-your-own-rules-resolve"></a>

## Combate: una pelea que resuelven tus propias reglas

El bloque `battle` presta al combate las cifras de la ficha, manteniendo la aritmética de Marinara. El bloque opcional `combat` define cómo RESUELVEN tus reglas la pelea. Parametriza un tipo de combate del motor, igual que `resolution` parametriza pruebas; todos sus nombres son tuyos. Hay dos tipos: `attack-vs-defense`, donde los dados se suman y se comparan con un número, y `dice-pool`, donde un combate tira las reservas propias de tu conjunto `dice-pool` y cuenta éxitos (consulta Un combate tirado con reservas, más abajo).

**Una partida cuyo conjunto declara `combat` pelea según ese bloque.** Las cifras del grupo vienen de sus fichas, los adversarios de tu bestiario o escala de amenaza, y cada turno se resuelve con tus dados. Todo lo gastado o perdido se guarda en la ficha en el momento, así que cerrar la pestaña a mitad no pierde nada. La pantalla usa tus palabras: ataques y capacidades en el menú, presupuestos, estados y registro con los cálculos reales. Las carencias se enumeran en la sección sobre lo que falta.

```json
"combat": {
  "kind": "attack-vs-defense",
  "health": { "pool": "grit" },
  "defense": { "derived": "guard" },
  "initiative": { "dice": { "count": 2, "sides": 6 }, "modifier": { "abilityMod": "wits" } },
  "attackRoll": { "dice": { "count": 2, "sides": 6 } },
  "economy": { "budgets": [{ "id": "act", "label": "Action", "per": "turn", "count": 1 }] },
  "attacks": [
    {
      "list": "gear",
      "budget": "act",
      "name": "name",
      "toHit": { "ability": { "column": "swing" } },
      "damage": { "dice": { "column": "damage" }, "ability": { "column": "swing" }, "type": { "column": "harm" } }
    }
  ],
  "abilities": [{ "list": "knacks", "budget": "act" }],
  "standard": ["dodge", "help"],
  "conditions": [
    { "condition": "shaken", "effects": ["own-attacks-disadvantage", "ends-on-damage"] },
    { "condition": "pinned", "effects": ["cannot-act", "speed-zero"] }
  ]
}
```

Ese es todo el bloque Ember Roads y sus partidas lo usan para combatir. El borrador 5e usa las mismas claves para d20:

```json
"combat": {
  "kind": "attack-vs-defense",
  "health": { "pool": "hp" },
  "defense": { "field": "ac" },
  "initiative": { "dice": { "count": 1, "sides": 20 }, "modifier": { "derived": "initiative" } },
  "attackRoll": {
    "dice": { "count": 1, "sides": 20 },
    "advantage": true,
    "naturals": { "max": "critical", "min": "miss" },
    "critical": "double-dice"
  },
  "economy": {
    "budgets": [
      { "id": "action", "label": "Action", "per": "turn", "count": 1 },
      { "id": "bonus", "label": "Bonus action", "per": "turn", "count": 1 },
      { "id": "reaction", "label": "Reaction", "per": "turn", "count": 1 }
    ],
    "movement": { "field": "speed" }
  },
  "abilities": [
    {
      "list": "spells",
      "onlyWhen": "prepared",
      "alwaysWhen": { "column": "level", "equals": 0 },
      "budget": "action",
      "toHit": { "derived": "spell_attack" },
      "saveDifficulty": { "derived": "spell_save_dc" }
    }
  ],
  "concentration": { "text": "concentration", "save": "con_save", "floor": 10, "fromDamage": 0.5 }
}
```

### Todas las claves

- `kind`: `"attack-vs-defense"` (un bando tira contra la defensa del otro; acertar causa daño) o `"dice-pool"` (consulta Un combate tirado con reservas). Cada clave siguiente significa lo mismo en ambos, leída como ese tipo lee los números.
- `health`: obligatorio. Lo que quita la pelea: `{ "pool": "grit" }`, una reserva que disminuye y cuyo margen temporal absorbe primero el daño si existe; o `{ "track": "harm" }`, un contador de heridas que MARCA. El contador necesita `damageKinds` y no concede puntos temporales.
- `defense`: obligatorio, referencia de valor. Un campo introducido por el jugador o un valor derivado que calculas.
- `initiative`: obligatorio. Dados tirados una vez al inicio y referencia opcional de modificador. Los empates favorecen el modificador mayor y después el orden de preparación del combate. `"each": "round"` vuelve a tirar la iniciativa de todos al empezar cada ronda nueva, con el modificador tal como esté entonces (así, un modificador que lee un contador de heridas es más lento una vez herido), y la ronda empieza por quien quede primero en el nuevo orden. Capability API 1.47. Un combate `dice-pool` puede tirarla en su lugar como reserva: `pool` (una referencia de valor) es cuántos dados, y sus éxitos más `plus` son el número, sin `dice` ni `modifier` al lado. `resource`, que necesita `pool`, conserva ese número y permite que los ataques lo muevan (consulta "Iniciativa que mueven los ataques", más abajo). Capability API 1.48.
- `attackRoll`: obligatorio en `attack-vs-defense` y rechazado en `dice-pool`, que tira las reservas propias de tu resolución. Define dados, si se tira dos veces y se conserva una (`advantage`), qué hacen las caras extremas de un dado (`naturals.max`: `critical`, `hit` o `none`; `naturals.min`: `miss` o `none`) y qué hace el crítico (`critical`: `double-dice` vuelve a tirar los dados de daño; `max-dice` añade una vez sus caras máximas; `none` es un impacto normal). Las caras especiales requieren un único dado, igual que en pruebas. Las salvaciones de combate usan estos mismos dados.
- `economy`: obligatorio. `budgets` define lo que cabe en un turno: ID, etiqueta, `per` (`turn` se repone al inicio del turno del portador; `round`, al empezar una ronda) y `count`. El PRIMER presupuesto es el principal, gastado por acciones estándar. `movement` es una referencia opcional de distancia recorrida por turno, en tu unidad. La leen combates sobre tablero; consulta Posiciones.
- `attacks`: opcional. Listas de ficha cuyas filas son armas. `name` es la columna de nombre; `damage.dice`, la de dados; `toHit.ability`, `toHit.proficiency`, `toHit.bonus`, `damage.ability`, `damage.bonus` y `damage.type` nombran columnas de la misma lista. Una columna `ability` es un `enum` con un ID de atributo; otros valores no añaden nada. `proficiency` es `boolean`: activarlo añade competencia. Sin dados legibles, la fila no es un ataque; una cuerda sigue siendo cuerda. `toHit.skill` nombra una columna `enum` con uno de tus IDs de habilidad: la fila suma lo que suma una prueba de esa habilidad, con el `ability` propio de la fila en lugar del de la habilidad si nombra uno, exactamente como lo cambia el `with=` de una prueba (Capability API 1.47).

  `strikes` referencia cuántos ataques compra UN gasto del presupuesto. Elegir una fila sin ataques disponibles gasta el presupuesto y deja el resto disponibles. Mientras quede alguno, todas las filas que declaran `strikes` son gratuitas: cambiar arma, objetivo o caminar entre ataques surge del propio menú. `strikesCappedBy` nombra una columna booleana que limita SU fila a un único ataque, por muchos que compre la lista: sirve para armas que disparan una vez por turno, como la propiedad Loading de SRD 5.1. No tiene sentido, y se rechaza, en listas que ya compran un ataque por gasto. Los ataques disponibles pertenecen al COMBATIENTE, no a una lista: dos listas con `strikes` comparten la misma cantidad, use la fila que use. Se borran al terminar el turno que los compró. Sin declaración se compra uno por gasto, como antes.

  ```json
  {
    "list": "attacks",
    "budget": "action",
    "name": "name",
    "strikes": { "field": "attacks_per_action" },
    "damage": { "dice": { "column": "damage" } }
  }
  ```

- `abilities`: opcional. Listas cuyas filas marcadas por catálogo son capacidades, filtradas como `battle.skills` mediante `onlyWhen` y `alwaysWhen`. Sus efectos son el `mechanics` de la entrada; el bloque establece el `budget` predeterminado, el `toHit` añadido si tira para acertar y la `saveDifficulty` de las salvaciones. Se rechaza una entrada que pide salvación, propia o para terminar un estado aplicado, si su lista no tiene `saveDifficulty`: una salvación contra nada siempre tendría éxito.
- `standard`: opcional, lista cerrada `dash`, `disengage`, `dodge`, `help`, `hide`, `ready`. Siempre se resuelven `dodge` (los ataques recibidos se tiran dos veces y conservan el peor) y `help` (el siguiente ataque del aliado se tira dos veces y conserva el mejor). `dash` (otro movimiento completo) y `disengage` (nadie ataca por alejarte este turno) se resuelven en tablero y solo se registran sin él. `hide` y `ready` se aceptan, pero aún no hacen nada.
- `standardEffects`: opcional, efectos de acciones estándar que su indicador no contiene. Hoy solo `dodge`: `{ "dodge": { "saves": ["dex_save"] } }` indica qué salvaciones tira dos veces conservando la mejor mientras dura. Solo nombra salvaciones declaradas, y solo si `standard` incluye `dodge`. Sin esto, esquivar sigue dificultando los impactos y nada más.
- `conditions`: opcional. Asigna TUS IDs de estado a sus efectos para que ficha y combate compartan un registro: un personaje envenenado sigue así después. Lista cerrada: `own-attacks-advantage`, `own-attacks-disadvantage`, `attacks-against-advantage`, `attacks-against-disadvantage`, `attacks-against-adjacent-advantage`, `attacks-against-far-disadvantage`, `attacks-from-adjacent-critical`, `cannot-act`, `cannot-react`, `speed-zero`, `half-move-to-stand`, `ends-on-damage`, `own-saves-advantage`, `own-saves-disadvantage`, `own-checks-advantage`, `own-checks-disadvantage`, `resist-all`, `cannot-target-source` y `cannot-approach-source`. `failsSaves` nombra las salvaciones que falla sin tirar. Los seis efectos que requieren distancia o movimiento (`attacks-against-adjacent-advantage`, `attacks-against-far-disadvantage`, `attacks-from-adjacent-critical`, `speed-zero`, `half-move-to-stand`, `cannot-approach-source`) solo tienen efecto en tablero; consulta Posiciones. `cannot-react` excluye al portador de las ventanas abiertas por movimiento, por lo que no se le pregunta. Hay otras cuatro claves:
  - `modifiers`: los números que cambia mientras está activo. Consulta Números que cambia un estado, más abajo.
  - `saves`: salvaciones a las que se refieren los dos efectos de salvación y cualquier modificador a las salvaciones. Sin ella, todas; declararla sin ninguno de ellos se rechaza.
  - `skills`: habilidades a las que se refieren los dos efectos de prueba y cualquier modificador a las pruebas que no nombra habilidades propias. Sin ella, todas; declararla sin ninguno de ellos se rechaza. Los enfrentamientos de un combate tiran las pruebas propias del combate, no habilidades, así que algo restringido a habilidades nunca llega a uno; una prueba fuera de un combate con una de esas habilidades sí lo lee (consulta Pruebas fuera de un combate, más abajo). Capability API 1.53.
  - `whileSourceInSight`: lo que solo cuenta mientras quien lo aplicó esté a la vista del portador. `true` condiciona todo el estado; una lista de sus efectos condiciona solo esos y conserva el resto, como un miedo que impide acercarte aunque no veas el origen. Se rechazan efectos que el estado no tenga. Sin tablero no hay línea de visión que romper y todo cuenta igualmente.
  - `endsWhenSourceDown`: se quita en cuanto cae quien lo aplicó.

  `own-saves-advantage` y su opuesto tiran dos veces y conservan una, como los ataques; se cancelan entre sí. `own-checks-advantage` y su opuesto hacen lo mismo con el lado del portador en un enfrentamiento, que es donde un combate hace sus pruebas de atributo, y con sus pruebas fuera de un combate. `resist-all` reduce a la mitad todo daño además de las defensas propias del objetivo y se cancela con una vulnerabilidad de la misma manera. `cannot-target-source` impide apuntar cualquier cosa a quien lo aplicó. `cannot-approach-source` impide acercarse más que la casilla actual, incluido el camino: permite rodear hasta otra igual de distante, pero no pasar cerca y salir por el otro lado.

  ```json
  { "condition": "restrained", "effects": ["own-saves-disadvantage"], "saves": ["dex_save"] }
  ```

  **Números que cambia un estado.** Cada entrada de `modifiers` nombra lo que cambia, `to`, y en cuánto:
  - `defense`: el número que debe alcanzar un ataque contra el portador. Solo un número fijo.
  - `attacks`, `saves`, `checks`: las tiradas de ataque, salvaciones y pruebas de enfrentamiento del propio portador. Un número fijo, o `dice` tirados cada vez que se hace la tirada (`"1d4"`); `minus: true` resta los dados en vez de sumarlos.
  - `speed`: cuánto camina el portador, en tu propia unidad de distancia (`flat`), o `times` 0.5 o 2 para la mitad o el doble, aplicado después de cada cambio fijo. Solo se lee en un tablero, igual que `speed-zero`.

  Un modificador a `checks` puede nombrar sus propias `skills`, y uno a `saves` sus propias `saves`, que lo restringen en lugar de las del estado. Cualquiera de ellos puede tirar dos veces con `mode: "advantage"` o `"disadvantage"`, que se cuenta junto con los efectos del mismo nombre y se cancela igual que ellos, con o sin un número al lado; así, un mismo estado puede hacer Sneak más difícil y Climb más fácil a la vez. Estas tres claves necesitan Capability API 1.53.

  ```json
  { "condition": "muddy", "modifiers": [{ "to": "checks", "skills": ["sneak"], "mode": "disadvantage" }] }
  ```

  Un número fijo lleva su propio signo, y 0 se rechaza. Cada modificador se muestra en el registro junto a la tirada que cambió, con el nombre del estado ("12 + 5 + 3 (Blessed) = 20"), y la probabilidad de acertar y la de ganar del menú lo tienen en cuenta. Los estados se acumulan entre sí; el mismo estado dos veces sigue siendo un estado.

  ```json
  "conditions": [
    { "condition": "blessed", "modifiers": [{ "to": "attacks", "dice": "1d4" }, { "to": "saves", "dice": "1d4" }] },
    { "condition": "shielded", "modifiers": [{ "to": "defense", "flat": 5 }] },
    { "condition": "slowed", "modifiers": [{ "to": "speed", "times": 0.5 }, { "to": "defense", "flat": -2 }] }
  ]
  ```

- `levels`: opcional. Niveles de un contador vivo que cuentan como estados mientras el contador esté lo bastante alto, que es como se expresa un estado que empeora por pasos, como el agotamiento. Cada entrada nombra un `track` simple (no un contador de heridas), el nivel `at` en el que empieza y lo que hace, con los mismos `effects`, `modifiers`, `failsSaves`, `saves` y `skills` que tiene un estado. Cuenta cada nivel que ha alcanzado el contador, así que se suman a medida que sube. Un nivel no tiene origen y solo termina cuando baja lo que lee, así que `half-move-to-stand`, `ends-on-damage`, `cannot-target-source` y `cannot-approach-source` se rechazan en uno. Solo un combatiente con ficha tiene contadores. El registro nombra un nivel por su contador y número ("- 1 (Heat 3)").

  En lugar de `track`, un nivel puede leer un valor `derived` (Capability API 1.54), calculado con el estado vivo y los objetos del personaje, para que nadie tenga que marcar nada: Ember Roads ralentiza a cualquiera que lleve 10 de volumen o más.

  ```json
  { "derived": "bulk_carried", "at": 10, "modifiers": [{ "to": "speed", "flat": -2 }, { "to": "checks", "skills": ["sneak"], "flat": -1 }] }
  ```

  Un umbral que varía de un personaje a otro ("more than five times Strength") es un valor derivado que lo resta: `scale` por -1 da la parte que hay que restar, y una `sum` de ambos es lo que lee el nivel.

  ```json
  "levels": [
    { "track": "exhaustion", "at": 1, "effects": ["own-checks-disadvantage"] },
    { "track": "exhaustion", "at": 2, "modifiers": [{ "to": "speed", "times": 0.5 }] },
    { "track": "exhaustion", "at": 3, "effects": ["own-attacks-disadvantage", "own-saves-disadvantage"] }
  ]
  ```

  Los modificadores, los niveles y los efectos de prueba necesitan Capability API 1.45.

- `checks` y `contests`: opcionales. Lo que lee un enfrentamiento y los enfrentamientos que cualquiera puede iniciar en un combate: agarrar, empujar, liberarse. Consulta Enfrentamientos, más abajo. Capability API 1.43.
- `concentration`: opcional. Campo vivo `text` que registra lo mantenido, `save` exigida por el daño, `floor` mínimo de dificultad y `fromDamage`, proporción del daño que la fija si es mayor. Empezar otra capacidad de concentración termina la anterior; fallar la salvación la termina y quita los estados que mantenía.
- `dying`: opcional, `kind: "saves"`. Dos contadores de tiradas (la cantidad necesaria es el máximo de cada uno), `dice`, `succeedAt`, efectos de caras extremas (`naturals.max`: `revive-1` o `success`; `naturals.min`: `one-failure` o `two-failures`), coste de daño estando caído (`damageWhileDown`, `criticalWhileDown`) y `condition` del personaje caído. Sin bloque, salud cero solo lo deja caído y curar lo devuelve.
- `damageTypes`: opcional. Tipos del sistema, comparados sin distinguir mayúsculas.
- `damageKinds`: obligatorio si `health` es un contador de heridas y rechazado si es una reserva; solo una marca conserva un tipo. Define qué `kinds` marca cada golpe y cuántas casillas. `default` recibe lo no asignado, incluido daño sin tipo; `byType` asigna `damageTypes` a tipos de marca, ignorando mayúsculas: `"Fire"` y `"fire"` son una clave y declarar ambas se rechaza. `marks` no tiene predeterminado porque las respuestas son opuestas: `"per-point"` cuenta niveles de salud según daño, así que tres de daño marcan tres casillas y reducirlo ayuda; `"per-blow"` marca una casilla si el golpe acierta, sea cual sea su fuerza. Un golpe con varias cláusulas marca una casilla usando el tipo más grave que llegó. Indica cuál usa el sistema: `{ "default": "bashing", "byType": { "fire": "aggravated" }, "marks": "per-point" }`.
- `pool`: obligatorio en `dice-pool` y rechazado en `attack-vs-defense`. Consulta Un combate tirado con reservas.
- `spendLimits`: opcional. Cuánto de una reserva viva puede gastar un combatiente por `turn` o por `round` en un combate, sea del tipo que sea: `[{ "pool": "blood", "max": { "derived": "blood_per_turn" }, "per": "turn" }]`. `max` se lee una vez al empezar el combate. Un coste que supera lo que queda del límite no se puede pagar, así que queda fuera del menú, por mucho que haya en la reserva. Un límite por turno vuelve a empezar al inicio del turno de su portador, y uno por ronda, al empezar una ronda. Un oponente escrito con números simples no paga nada de una ficha, así que un límite nunca le afecta; un oponente escrito como ficha paga de sus propias reservas y queda sujeto al límite como cualquier otro. Capability API 1.47.
- `threat`: opcional y necesario para bestiarios. `tiers` es la escala de adversarios: ID, etiqueta, intervalo `health`, `defense`, `toHit`, intervalo `damagePerRound` y `saveDifficulty`. Cada criatura nombra un nivel; un adversario no escrito se adapta al que pidió el GM, sin salirse de tu escala. `damagePerRound` expresa lo que hace a UN objetivo por ronda, incluida toda su secuencia.

### Un combate tirado con reservas: `dice-pool`

Un conjunto `dice-pool` cuenta éxitos en vez de sumar dados, y sus combates también pueden hacerlo. `"kind": "dice-pool"` necesita que `resolution.kind` sea `"dice-pool"`, y lee la ficha como ya lo hacen tus pruebas de reserva: **cada número que suma una tirada es una cantidad de dados, y cada número con el que se compara es una cantidad de éxitos.** Así que no se renombra nada:

- Un número para acertar (el de una fila de ataque, el `toHit` de una lista de capacidades, el de una acción de criatura) es la reserva que tira un ataque. El dado, el objetivo que debe alcanzar un dado y las caras con las que se duplica, explota, cancela y pifia son las de tu `resolution`. El contador de heridas que nombra tu `resolution.penaltyFrom` resta su penalización de la reserva, igual que en una prueba.
- `defense` es cuántos éxitos necesita un ataque, y nunca menos de uno. El modificador de un estado a `defense`, y `cover.bonus`, se suman a ella.
- Cada éxito por encima de los necesarios añade un dado de daño. Una pifia falla, cuente lo que cuente. No hay críticos: un impacto fuerte vale sus dados extra.
- El número de una salvación es su reserva, y su dificultad, los éxitos que necesita. Un enfrentamiento tira la prueba de cada bando como reserva, y gana el bando con más éxitos.
- El modificador de un estado a ataques, salvaciones o pruebas son dados que se añaden o se quitan, así que es un número `flat`: un modificador tirado (`dice`) se rechaza.
- El daño son dados de tu dado: los `dice` de una cantidad (de una criatura, de una entrada) indican cuántos y deben ser de tu dado, y su parte `flat` son éxitos automáticos, que nunca se tiran. La columna de dados de daño de una fila de ataque se lee por su cantidad, `damage.ability` añade como dados la puntuación de ese atributo y `damage.bonus` añade éxitos automáticos. `perCostStep` y `scales` añaden dados, como siempre. La curación y los puntos temporales son cantidades, y se suman como antes.
- La iniciativa es una suma (`initiative.dice` y `modifier`), o una reserva si la pides (`initiative.pool`, abajo), y una regla de muerte conserva sus propios dados.

Lo que tira un combate de reservas más allá de eso es su bloque `pool`:

```json
"pool": {
  "advantage": true,
  "damageTarget": 6,
  "soak": { "roll": true, "byKind": { "knock": { "abilityMod": "sinew" } } }
}
```

- `advantage`: si una tirada que se inclina se tira dos veces y se conserva la que tiene más éxitos (una pifia cuenta como menos que cualquier tirada que no pifió). Sin él, un combate tira una vez diga lo que diga un estado.
- `damageTarget`: el objetivo por dado contra el que se tiran los dados de daño y de absorción. El objetivo predeterminado de tu resolución si lo omites. Esos dados cuentan una vez cada cara igual o superior a él y nada más: con ellos nada se duplica, explota, cancela ni pifia.
- `soak`: lo que un objetivo resta del daño que hace un impacto, por tipo de daño. `all` es una referencia de valor para cualquier tipo, y `byKind`, una por cada tipo de tu contador de salud, que tiene prioridad sobre `all` para su tipo; un tipo que no nombra ninguno de los dos no se absorbe. Con `"roll": true`, el objetivo tira esa cantidad de dados contra el objetivo de daño y cada éxito resta uno. Con `"roll": false`, el número se resta de los dados de daño antes de tirarlos, así que nunca afecta a los éxitos automáticos, que la absorción tirada resta como cualquier otro. Nada baja de cero, y no se tira nada para absorber un golpe que no contó nada. Una criatura da sus propios números como `soak: { "all": 1, "byKind": { "knock": 3 } }`, lo que necesita que el `soak` del bloque indique cómo se absorbe. La resistencia, la vulnerabilidad, la inmunidad y `resist-all` se aplican después de la absorción, a lo que queda.
- `hardness`: una referencia de valor para la dureza de un combatiente, si la iniciativa es un número que mueven los ataques. Consulta [Armadura y efectos puestos en un combate](#armor-and-worn-effects-in-a-fight).

El daño de cada objetivo se tira para ese objetivo, porque lo que ganó un impacto por encima de los éxitos necesarios, y lo que absorbe, es suyo. La segunda cláusula de un golpe y un efecto adicional son cada uno su propia reserva de daño de su propio tipo, sin dados extra del impacto. El menú prevé una reserva con exactitud: la probabilidad de alcanzar los éxitos necesarios y lo que valen los dados de daño después de lo que absorbe el primer objetivo.

Gravewatch está escrito así: sus guardianes tiran una puntuación y un oficio, absorben los golpes contundentes con Sinew y no absorben en absoluto los desgarros, pueden gastar un punto de Resolve por turno y vuelven a tirar la iniciativa cada ronda.

**Declarar en orden inverso no está implementado.** Algunos sistemas hacen que todos declaren su acción antes de que nadie actúe, empezando por el más lento. En un combate donde cada combatiente elige una acción cuando llega su turno, declarar primero no cambia nada que lea ninguna regla, así que no hay nada que pueda decir una clave.

### Iniciativa que mueven los ataques

Algunos sistemas de reservas mantienen la iniciativa como un número durante todo el combate y permiten que los ataques la muevan: una forma de atacar se la quita al objetivo, otra gasta la del propio atacante como daño, y quien cae hasta una línea fijada se derrumba. Un combate `dice-pool` puede expresarlo (Capability API 1.48):

```json
"initiative": {
  "pool": { "abilityMod": "nerve" },
  "plus": 3,
  "resource": {
    "base": 3,
    "styles": [
      { "id": "press", "label": "Press", "takes": { "gain": 1 } },
      { "id": "telling", "label": "Telling blow", "spends": { "onMiss": [[0, 1], [6, 2], [11, 3]] } }
    ],
    "crash": { "at": 0, "condition": "reeling", "bonus": 5, "recoverAfter": 3 }
  }
}
```

- **La apertura.** Todos tiran `pool` dados al empezar el combate, y los éxitos más `plus` son su número, así que `resource` necesita `pool`: los dados sumados son un orden, no una cantidad de dados. El `initiativeModifier` de una criatura es su reserva, porque cada número que suma un combate de reservas son dados. El número se conserva: nunca se vuelve a tirar (`each` se rechaza junto a `resource`), y al empezar cada ronda el orden se ordena según los números tal como estén.
- **Estilos.** Cada ataque (una fila de ataque, una capacidad o entrada de catálogo que tira para acertar y causa daño, una acción de criatura que lo hace y una acción formada por otras acciones) se ofrece una vez por estilo, y un jugador elige el estilo antes que el objetivo. Hasta cuatro, cada uno toma o gasta, y al menos uno toma. Una elección que no nombra ningún estilo se hace en el primero; se rechaza una en la que no se ofrece el ataque. Un ataque conserva el estilo con el que se hizo hasta que termina, aunque una respuesta cambie el número de quien lo hace antes de que impacte.
- **Un estilo que toma.** Al impactar, el daño se tira como siempre (los dados extra del impacto, los dados del arma, la absorción por tipo, los éxitos automáticos), y lo que cuenta se resta del número del objetivo en vez de su salud. El atacante gana todo eso más `gain`.
- **Un estilo que gasta.** Solo se ofrece mientras el número del atacante esté por encima de la línea de derrumbe (0 si no das `crash`), y nunca para una acción formada por otras acciones, porque un número se gasta en un solo golpe. Al impactar, el daño es el número del atacante en el momento de hacer el ataque, en dados contra el objetivo de daño, y nada más: sin dados del arma, sin dados extra del impacto, sin éxitos automáticos, sin absorción. Marca la salud con el propio tipo de daño del ataque. Una vez terminado el ataque, un número que impactó en alguien vuelve a `base`, y uno que no impactó en nadie pierde lo que indica `onMiss` según el número con el que se hizo (una tabla por peldaños: pares `[at least, lose]`, ascendentes; nada por debajo del primer peldaño).
- **Derrumbarse.** Un número que cae hasta `crash.at` o por debajo se ha derrumbado: se le aplica la `condition` (uno de tus propios estados, y opcional), y quien lo llevó hasta ahí gana `bonus`. Un fallo que derrumba a quien lo hizo no da bonificación a nadie. Un combatiente derrumbado no puede gastar. El derrumbe se levanta, estado incluido, cuando su número vuelve a superar la línea, después de `recoverAfter` de sus propios turnos (su número vuelve a `base` al empezar ese turno) y cuando termina el combate, termine como termine. Alguien cuya tirada de apertura esté en la línea o por debajo empieza el combate derrumbado. `base` debe estar por encima de `crash.at`, o volver a él lo derrumbaría de nuevo.

El menú muestra la previsión propia de cada estilo: la probabilidad de impactar, y lo que tomaría un estilo que toma o el daño que haría uno que gasta. El panel de estado muestra el número de todos, y el registro indica cada cambio y su motivo: "Ada gains 5 initiative for crashing Rats, and is on 14." El selector del motor evalúa un golpe que toma con un turno de antelación: tomar y después gastar frente a gastar ahora y de nuevo desde la base.

<a id="what-a-fight-reads-from-mechanics"></a>

### Qué lee un combate de `mechanics`

`kind` determina si `amount` es daño o curación; cualquier entrada marcada como `reaction` queda fuera del menú, al igual que una entrada `utility`, salvo que cambie lo que puede contener el propio turno (véase más abajo). `attackRoll` realiza una tirada contra la defensa del objetivo con el `toHit` de la lista; `autoHit` omite por completo ese paso. `save` tira la salvación del propio objetivo contra la `saveDifficulty` de la lista, y `onSuccess` determina si un éxito recibe la mitad o nada. `targetCount` indica a cuántos objetivos puede dirigirse. Una capacidad que no tira para atacar (un área contra la que todos realizan una salvación, algo que impacta directamente) tira sus dados UNA VEZ para todos ellos; una que tira para impactar a cada objetivo vuelve a tirar sus dados por cada impacto. `applies` impone condiciones a lo que afecta, cada una con una `duration` de `instant` (sin reloj propio: permanece hasta que algo la retire), `until-save` (que necesita `saveEnds` a su lado) o `{ "rounds": n }`, y un `saveEnds` opcional que identifica la salvación y si se repite en `turn-end` o `turn-start`. Las rondas descuentan los turnos propios del portador a medida que termina cada uno; `{ "rounds": 1, "at": "turn-start" }` los cuenta a medida que empieza cada uno, que es como se dice "hasta el inicio de tu siguiente turno". `endsAfter` quita el estado tras el primer `own-attack` (la siguiente tirada de ataque del portador), `attacked` (la siguiente tirada de ataque contra el portador) u `own-save` (la siguiente salvación del portador), diga lo que diga su reloj, así que `{ "duration": { "rounds": 1 }, "endsAfter": "own-attack" }` es "en su siguiente ataque antes del final de su siguiente turno". Uno que dura un ataque dura todo el ataque, daño incluido, y uno nuevo que aplica el mismo golpe se queda. Ambos necesitan Capability API 1.45. `temporary` concede puntos temporales a la reserva de salud, que nunca se acumulan: se conserva la protección mayor. `scales` aumenta la cantidad con los DADOS adicionales que su tabla indica para el valor que lee. `cost` se paga mediante el comando `use` de la propia ficha, y `budget` sustituye qué parte de la economía se consume.

`plus` contiene hasta tres cantidades MÁS en el mismo golpe, junto a `amount`, cada una con su propia tirada y tipo ("and 2d6 fire"). Una cláusula es `{ "dice": "2d6", "flat": 1, "type": "fire" }` y puede llevar su propia `save`, `{ "save": "con_save", "difficulty": 13, "onSuccess": "none" | "half" }`, que tira el OBJETIVO con independencia de lo que ya le haya exigido la acción: `none` elimina toda esa cláusula con un éxito y `half` deja la mitad, mientras que el resto del golpe no cambia en ninguno de los casos. Sin `difficulty`, utiliza el número de la salvación de la propia acción y, después, la `saveDifficulty` de la lista. Un crítico duplica los dados de cada cláusula con la misma regla que duplica los de la primera cantidad; una cláusula sin `type` utiliza el tipo de daño del propio golpe, y el golpe completo sigue siendo UNA prueba de concentración, con el daño sumado, y una prueba para caer. Una cláusula necesita un `amount` al que acompañar, y un `heal` no lleva ninguna.

```json
{
  "kind": "attack",
  "attackRoll": true,
  "amount": { "dice": "1d8" },
  "damageType": "piercing",
  "plus": [{ "dice": "2d6", "type": "fire" }]
}
```

Tres claves indican qué hace una entrada con la economía del propio turno, y una entrada `utility` que declare cualquiera de ellas se ofrece en lugar de descartarse:

- `free`: no consume ningún presupuesto. Sigue pagando el `cost` que indique y no puede indicar también un `budget`.
- `gives`: `[{ "budget": "action", "count": 1 }]`, hasta cuatro. Usarla añade esos presupuestos en ese mismo momento, con un límite igual a lo que contiene un turno más lo concedido, para que nada pueda ahorrarse para un turno posterior.
- `standard`: `{ "actions": ["dash", "disengage", "hide"], "budget": "bonus" }`. Su poseedor puede realizar esas acciones estándar con ESE presupuesto. Se ofrecen junto a las ordinarias como `standard:<id>@<budget>`, y la propia entrada queda fuera del menú cuando solo concede ese permiso, porque un permiso no es algo que se ejecute.

Una entrada del nuevo `kind: "rider"` es PASIVA: nadie la ejecuta, nunca aparece en el menú y añade automáticamente una cláusula de daño al primer impacto que cumpla los requisitos en un período. Lleva `rider` y nada más que pueda ejecutarse:

```json
{
  "kind": "rider",
  "rider": {
    "on": "hit",
    "sources": ["attacks"],
    "requires": { "column": "finesse" },
    "when": ["advantage", "ally-adjacent"],
    "oncePer": "turn",
    "amount": { "dice": "1d6" }
  },
  "scales": {
    "from": { "field": "level" },
    "table": [
      [1, 0],
      [3, 1]
    ]
  }
}
```

`sources` identifica las listas de ataques de las que procede y `requires` una columna de sus filas con valor verdadero, de modo que un efecto adicional que solo se activa con ciertas armas puede especificarlas sin que Engine sepa qué es un arma; no indicar ninguna significa cualquier impacto de su poseedor. Un arma empuñada desde la bolsa no es una fila, así que `weapons` (Capability API 1.70) nombra además las propiedades de objeto con las que se activa: `"weapons": { "tags": ["finesse",
"ranged"] }` se activa con un impacto de un arma empuñada que tenga cualquiera de esas propiedades, incluido su golpe con la mano secundaria. Nombrar solo `weapons` no se activa con ninguna fila, y nombrar solo filas no se activa con ninguna arma empuñada. `when` acepta CUALQUIERA de sus opciones: `advantage` es la inclinación final de la tirada de ataque, y `ally-adjacent` es un aliado del atacante que esté en pie y pueda actuar, a una casilla del objetivo en un tablero o en cualquier lugar si no hay tablero. `oncePer` es `turn` (se renueva al inicio de todos los turnos, de modo que un golpe realizado mientras actúa otra persona todavía puede incluir uno) o `round`. `amount` crece con el `scales` de la propia entrada, y `type` es el tipo de daño, cuyo valor predeterminado es el del propio golpe.

<a id="creatures-a-bestiary-a-fight-reads"></a>

### Criaturas: un bestiario que lee el combate

Un catálogo que declara `"holds": "creatures"` contiene oponentes en lugar de filas de ficha. No alimenta ninguna lista, el selector del editor de fichas nunca lo ofrece y todos sus números se escriben con las claves que ya declara tu bloque `combat`. Necesita un bloque `combat` y una escala `threat`, porque cada criatura se clasifica en uno de tus propios niveles.

```json
{
  "id": "road_trouble",
  "label": "Road trouble",
  "holds": "creatures",
  "filters": [{ "id": "tier", "label": "How bad", "type": "text" }],
  "entries": [
    {
      "id": "rust-jackal",
      "label": "Rust Jackal",
      "summary": "A lean thing that lives on the metal roads.",
      "filters": { "tier": "Pack trouble" },
      "creature": {
        "health": { "dice": "3d6" },
        "defense": 6,
        "initiativeModifier": 1,
        "speed": 16,
        "abilities": { "brawn": 1, "wits": 0, "heart": -1 },
        "tier": "pack",
        "actions": [
          {
            "id": "bite",
            "name": "Bite",
            "budget": "act",
            "toHit": 2,
            "damage": { "dice": "1d6", "flat": 1, "type": "cut" },
            "reach": 2
          },
          {
            "id": "worry",
            "name": "Worry",
            "budget": "act",
            "toHit": 2,
            "damage": { "dice": "1d4", "type": "cut" },
            "applies": [{ "condition": "shaken", "duration": { "rounds": 2 } }]
          },
          {
            "id": "snap_and_worry",
            "name": "Snap and worry",
            "budget": "act",
            "sequence": [
              { "action": "bite", "times": 1 },
              { "action": "worry", "times": 1 }
            ]
          }
        ]
      }
    }
  ]
}
```

Los números siguientes son la forma directa de escribir una criatura. Una escrita en los términos de tu propio conjunto de reglas, como `sheet`, obtiene de esa ficha `health`, `defense`, `initiativeModifier`, `speed`, `abilities`, `saves`, `checks`, `soak` y `hardness` (véase "Una criatura escrita en los términos de tu conjunto de reglas", más abajo).

- `health`: un número, o `{ "dice": "3d6", "flat": 2 }`, que se tira una vez al crear el combate. Una previsión lee la media, de modo que un menú nunca promete un dado que nadie ha tirado.
- `defense`, `initiativeModifier`, `speed`: contra qué se tira un ataque, qué se suma a la iniciativa (o, si la iniciativa se tira como reserva, cuántos dados tira) y cuánto camina en un turno, en tu propia unidad de distancia.
- `abilities` y `saves`: usan como claves los identificadores de características y salvaciones que declara tu ficha. Una salvación no indicada se lee como cero.
- `checks`: lo que suma en un enfrentamiento, con los IDs de `combat.checks` como claves. Uno no indicado se lee como cero. Capability API 1.43.
- `soak`: solo en un combate `dice-pool`, lo que absorbe: `all` para cualquier daño, `byKind` para un tipo de tu contador de salud. Capability API 1.47.
- `hardness`: si la iniciativa es un número que mueven los ataques, un golpe que gasta cuyos dados están por debajo de ella no hace nada. Capability API 1.56.
- `resist`, `vulnerable`, `immune`: tipos de daño, comparados sin distinguir mayúsculas, que se validan contra `combat.damageTypes` cuando declaras alguno. Una entrada `resist` o `immune` también puede indicar qué la atraviesa, `{ "type": "tearing", "except": ["silver"] }`: un golpe de un objeto arma con una de esas propiedades de objeto se recibe tal cual (Capability API 1.55, consulta [Armas en un combate](#weapons-in-a-fight)). `conditionImmunities` identifica tus propias condiciones.
- `tier`: a qué peldaño de `combat.threat` pertenece.
- `loot` (Capability API 1.63): el ID de la tabla de botín que tira un combate ganado por ella (consulta Botín, más arriba).
- `traits`: pares breves de nombre y texto que se muestran al Game Master. Nunca se resuelven, por lo que cualquier cosa con números pertenece a una acción.
- `signaturePoints`: puntos que se recuperan al inicio de su propio turno y se gastan en acciones `signature`.
- `riders`: hasta cuatro, equivalentes al `rider` de una entrada de catálogo, escritos en el bloque. Cada uno es `{ "id": "pack", "name": "Pack", "on": "hit", "oncePer": "turn" | "round", "amount": { "dice": "1d6" } }`, con un `type` opcional y un `actions` opcional que indica con cuáles de las acciones del propio bloque se activa. El efecto adicional de un bloque no lee ninguna lista de ficha, por lo que no tiene las claves `sources` y `requires`. Una criatura escrita como ficha obtiene los efectos adicionales de sus listas, exactamente igual que un personaje.
- `actions`: hasta doce, cada una con su propio `id`. Una acción lleva lo mismo que un bloque de estadísticas escrito a mano (`toHit`, `autoHit`, `damage`, `save`, `applies`, `targetCount`, `reach`, `range`, `area`), más cuatro cosas exclusivas de las criaturas. `reach` indica hasta dónde golpea, `range` hasta dónde se lanza o dispara y `area` la forma en la que impacta, todo en tu propia unidad de distancia; `range` puede ser un número simple o `{ "normal": 30, "long": 120 }` si llega más lejos con una penalización, y `area` es `{ "shape": "burst" | "cone" | "line", "size": n, "friendlyFire": false }` (véase Posiciones):
  - `uses`: `{ "per": "encounter" | "day", "count": n }`. Cuando se agotan, la acción sale del menú.
  - `recharge`: `{ "dice": { "count": 1, "sides": 6 }, "from": 5 }`. Empieza el combate disponible, se gasta al usarla y se tira al inicio del propio turno de la criatura: `from` o más la recupera. El registro incluye los dados en ambos casos.
  - `sequence`: otras acciones del mismo bloque, en orden, cada una con su propio objetivo. **Así se escribe una criatura que golpea dos veces con una sola acción.** Un presupuesto paga toda la secuencia. Una secuencia no lleva nada propio y nunca puede identificar otra secuencia.
  - `signature`: `{ "cost": n }`, se compra con los puntos de la propia criatura en lugar de un presupuesto y solo mientras actúa otra persona: el combate la ofrece en la ventana entre un turno y el siguiente (véase Ventanas).
  - `reaction`: el mismo objeto que el `mechanics.reaction` de una entrada de catálogo (consulta Ventanas): la acción no está en el menú de ningún turno y se ofrece en su lugar en el momento que nombra. Una reacción no es también una acción distintiva, y ninguna secuencia puede crear una.
  - `self: true`: recae sobre la propia criatura en vez de sobre otra persona, así que no lleva `targetCount` ni `area`. Ambos necesitan Capability API 1.46. Una Parry es ambas cosas, con un estado `parrying` que suma 2 a la defensa:

    ```json
    {
      "id": "parry",
      "name": "Parry",
      "budget": "reaction",
      "self": true,
      "reaction": { "on": "hit" },
      "applies": [{ "condition": "parrying", "duration": { "rounds": 1 }, "endsAfter": "attacked" }]
    }
    ```
- Una salvación necesita una dificultad en la propia acción: `save.difficulty` para una salvación que impone la acción, o `saveDifficulty` para una condición que termina con una salvación cuando la acción no tiene la suya propia. Una acción de bloque se escribe con números simples incluso en una criatura con ficha, así que ese número reside en la acción. La salvación de una cláusula puede omitir su `difficulty` y recurrir al mismo número.
- `damage.plus` es la misma lista de cláusulas que el `plus` de una entrada de catálogo y se lee exactamente igual: `"damage": { "dice": "1d6", "flat": 2, "type": "piercing", "plus": [{ "dice": "1d4", "type": "fire" }] }` es un mordisco cuyo calor tiene su propia cantidad, a la que se aplica resistencia y que se duplica por separado.

El bestiario del borrador de 5e contiene cinco criaturas escritas a mano en `docs/development/ruleset-5e-2014.example.json`, que cubren una secuencia, una recarga, una salvación con una condición, resistencias e inmunidades, usos limitados, puntos de acciones distintivas y una criatura escrita como ficha.

#### Una criatura escrita en los términos de tu conjunto de reglas

Una criatura no tiene que escribirse con números simples. Dale una `sheet`, con exactamente la misma estructura que la ficha de un personaje, y el combate la construye igual que a un miembro del grupo: su salud, defensa, salvaciones, iniciativa, velocidad y todos los ataques y capacidades de sus listas son lo que determinen tus propias fórmulas de ficha. Así expresa un conjunto de reglas que sus oponentes tienen las mismas características, habilidades y listas que sus personajes, sean cuales sean. El Toll Warden de Ember Roads:

```json
{
  "id": "toll-warden",
  "label": "Toll Warden",
  "creature": {
    "tier": "pack",
    "traits": [{ "name": "Knows the road", "text": "It will not follow anyone past the last milestone." }],
    "sheet": {
      "abilities": { "brawn": 2, "wits": 1, "heart": 1 },
      "skills": { "sway": "trained" },
      "fields": { "calling": "Hauler", "toughness": 3 },
      "lists": {
        "gear": [{ "name": "Toll hook", "swing": "brawn", "damage": "1d6", "harm": "cut" }],
        "knacks": [{ "name": "Hold the Line", "_catalog": "knacks/hold-the-line" }]
      }
    }
  }
}
```

- **Todas las partes son opcionales**: `abilities`, `skills`, `saves`, `bonuses`, `fields` y `lists`, con los identificadores que declara tu ficha como claves. Lo omitido se lee con el valor predeterminado de tu ficha, exactamente como en un personaje en blanco. El Grit del guardián es 9 porque tu `grit_max` suma 4, su Toughness y su Brawn, y su Guard es 7 por una razón semejante.
- **Cada número tiene un único origen.** Una criatura con ficha no proporciona también `health`, `defense`, `initiativeModifier`, `speed`, `abilities`, `saves` o `checks`, y Engine rechaza el archivo si lo hace. Puede no tener `actions` propias, porque sus listas son lo que hace. Una criatura sin ficha sigue proporcionando los tres primeros y al menos una acción.
- **Se valida como los datos de autoría que es.** Cada identificador debe estar declarado en tu ficha; una habilidad o salvación se establece en uno de los niveles de competencia que ofreces para ella; un campo, puntuación, bonificación o columna contiene lo que declara admitir (un entero dentro de su intervalo, uno de sus valores, etc.), y una lista no contiene más filas de las permitidas. No hay parte `live`, porque el combate guarda lo que la criatura ha gastado.
- **Una fila puede proceder de un catálogo.** `_catalog: "<catalog>/<entry>"` identifica la entrada de la que se seleccionó una fila, como en la ficha de un personaje, y el combate lee de esa entrada qué cuesta y qué hace la fila. El catálogo debe alimentar esa lista. Cuando el catálogo está escrito en línea, la entrada debe existir en él; cuando reside en su propio archivo, una fila que identifica una entrada ausente del archivo simplemente no concede nada a la criatura. Los catálogos que identifican las fichas de un bestiario se cargan para el combate junto con el bestiario.
- **Lo que la entrada declara junto a la ficha sigue contando**: `tier`, `traits`, `actions`, `signaturePoints`, `riders`, `resist`, `vulnerable`, `immune` y `conditionImmunities`.
- **Paga de sus propias reservas.** Empiezan llenas, las gasta en lo que le proporcionan sus listas y se le ofrecen las formas de pago mayores (un conjuro con un espacio superior) exactamente como a un miembro del grupo, tanto si decide Engine como si decide el Game Master. Hold the Line le cuesta Luck al guardián.
- **Con un contador de heridas, su salud es el contador.** Un golpe marca el contador propio de la criatura según tus `damageKinds`, después de aplicar sus `resist`, `vulnerable` e `immune`, por lo que una criatura inmune a un tipo de daño no recibe ninguna marca de él.
- **Sigue siendo un oponente.** A cero queda fuera de combate en lugar de agonizar; nunca tira contra la muerte; la pantalla muestra lo que siempre mostraba de un oponente y nada de su ficha, y no se escribe lo que haya gastado en ningún otro lugar, ni siquiera cuando un personaje comparte su nombre.
- **Una ficha cuya salud total sea cero** queda fuera del combate, y el registro inicial explica el motivo, en lugar de introducir algo a lo que nadie puede dañar.
- Una capa que retire un valor de uno de tus campos enumerados nunca elimina a una criatura que lo utilice: mientras la capa esté activa, ese campo toma su valor predeterminado para la criatura, exactamente como para un personaje, y no se rechaza a la criatura por ello.
- Un Game Master también puede inventar una, que queda limitada a su nivel (véase Oponentes que nadie escribió).
- Un paquete que incluya una declara Capability API 1.34.

El Toll Sergeant del borrador de 5e es lo mismo en una ficha d20: su Armor Class, puntos de golpe, salvaciones y dos golpes por acción proceden de sus propios campos y su lista de ataques.

#### Oponentes que nadie escribió

Cuando un Game Master inventa un oponente, Engine ajusta la propuesta a tu escala `threat` antes de cualquier tirada: la salud entra en la franja del nivel; la defensa, el ataque y las dificultades de salvación se limitan a dos por encima de los del nivel, y el daño se reduce hasta que la mejor ronda de la criatura (su secuencia más potente o su acción individual más potente, medida contra un objetivo) quepa en el `damagePerRound` del nivel. Primero reduce la cantidad de dados, luego la parte fija, después un golpe de una secuencia y, solo entonces, el tamaño del dado, sin reducir nunca nada a cero. Se descartan los nombres que no contiene tu conjunto de reglas: tipos de daño, condiciones y salvaciones desconocidos, y cualquier acción después de las seis primeras. Un nivel que nunca declaraste recurre al más bajo de tu escala. Cada cambio se devuelve como una frase sencilla para que el registro pueda explicar qué se hizo.

Una invención también puede escribirse como `sheet`, igual que una criatura de bestiario; así obtiene espacios y conjuros un mago inventado. El Game Master ve los identificadores de tu ficha y lo que puede contener cada uno, las listas que lee un combate y los nombres que ofrecen tus catálogos para ellas, de modo que un conjuro se identifica en lugar de describirse: una fila que identifica una entrada de catálogo, con cualquier combinación de mayúsculas, se convierte en esa entrada, y los valores propios del Game Master (como que un conjuro esté preparado) se aplican encima. La ficha se lee con tolerancia porque la escribió un modelo: se descarta un nombre que no exista en tu conjunto de reglas, un valor se ajusta a su campo o columna y no se usan los números escritos junto a la ficha.

Una criatura inventada que no sea un jefe queda limitada a lo que le abre tu conjunto de reglas. Un filtro de catálogo con `startFrom` identifica el campo de ficha por el que se organizan sus entradas (la lista de conjuros del paquete 5e por `class`), y una criatura inventada conserva solo las entradas cuyo filtro coincide con su propio valor de ese campo, comparado como lo hace el selector al abrirse. Así, un Sorcerer nunca tiene toda la lista de conjuros, y quien no indique clase no obtiene nada de un catálogo organizado por clase. Después se completan las elecciones que dejó abiertas sin volver a consultar al Game Master: para cada lista cuyas filas solo cuentan una vez elegidas (`onlyWhen` en una fuente de capacidades de combate), entre las entradas disponibles que puede pagar con sus propias reservas, se completa cada reserva con un pequeño número de entradas (más para una criatura más competente), así como aquello que puede usar a voluntad. Lo que recibe depende de su temperamento y competencia, los mismos con los que combate: una criatura protectora o de apoyo busca lo que sostiene a su bando; una temeraria, daño; una metódica o paciente, lo que frena al enemigo; y cuanto más competente sea, más probable es que lleve una reacción, una contra o cualquier otra cosa que altere el turno. La selección utiliza la semilla del propio combate, por lo que un mismo combate siempre se completa de la misma manera. Una fila que el Game Master haya identificado de una lista así cuenta como elegida.

Un jefe queda enteramente en manos del Game Master, como excepción que puede ser: no se le quita nada ni se le completa nada.

Después, ambos quedan limitados a su nivel:

- La salud entra en la franja del nivel mediante el único campo del que se lee: el máximo de la reserva es ese campo o una `sum` con exactamente un campo (el máximo de puntos de golpe de 5e, Toughness de Ember Roads). Una fórmula de salud sin un único campo se deja como está y el registro lo indica. La longitud de un contador de heridas es tuya y nunca cambia.
- Una vez construida la criatura, la defensa, el ataque y las dificultades de salvación se limitan a dos por encima de los del nivel, y el daño se reduce hasta que su mejor ronda quepa en el `damagePerRound` del nivel, contando el mayor pago que pueda permitirse. Primero se reduce lo que compra un pago mayor, luego los dados, la parte fija, un golpe y, solo entonces, el tamaño del dado.

Tu propio bestiario nunca se limita. Son datos que escribiste tú, así que Engine los acepta tal como están.

### Posiciones: un combate sobre un tablero

Un combate se desarrolla en el teatro de la mente hasta que tu bloque indica cuánto vale una casilla de tablero. Declara `distance` y podrá librarse sobre una cuadrícula; entonces movimiento, alcance, distancias, áreas, línea de visión, cobertura y golpes contra quien se aleja empiezan a tener significado. Todos son números que escribiste tú; Engine proporciona el tablero y nada más.

```json
"distance": { "label": "ft", "perCell": 5 },
"ranged": { "long": "disadvantage", "adjacentFoe": "disadvantage" },
"cover": { "bonus": 2 },
"opportunity": { "budget": "reaction" }
```

Ember Roads declara una sola línea y nada más; ese es el propósito: nada del resto es obligatorio.

```json
"distance": { "label": "paces", "perCell": 2 }
```

**La casilla.** `distance.perCell` indica cuánta cantidad de TU unidad vale una casilla, y `label` es el nombre de esa unidad. Todas las distancias del mundo del bloque la utilizan: `economy.movement`, la `speed` de una criatura, el `reach` y el `range` de un arma, y el `reach` y el `range` de una acción de criatura. Un catálogo que declare su propio `units.distance` convierte sus propios `mechanics.range` y `area.size` con su `perCell`; uno que no lo declare usa este. Una distancia mayor que cero se redondea a la casilla más cercana, nunca a ninguna, por lo que cualquier cosa a la que hayas dado un número alcanza al menos una. Cero no es una distancia corta, sino que conserva su propio significado: un `mechanics.range` de 0 es uno mismo o contacto (y tocar a otra persona alcanza la casilla siguiente), y una columna `reach` o `range` de arma con valor 0 en una fila significa que esa fila no tiene esa distancia.

**Si el combate usa un tablero.** Deben coincidir dos cosas: tu bloque declara `distance` y la partida del jugador tiene el estilo de combate Tactical. Con el estilo Classic, o con un conjunto de reglas sin `distance`, el combate sigue siendo teatro de la mente: cualquiera puede apuntar a cualquiera y no se lee nada de lo siguiente.

**Lo que ve el jugador.** Se dibuja el tablero con el terreno del propio estilo táctico. Cada casilla es un botón, accesible con el puntero o las teclas de dirección, e indica qué es, quién está en ella y qué significa para la elección parcialmente realizada. Caminar ilumina las casillas ofrecidas por el menú, cada una con su coste EN TU UNIDAD, dibuja el camino hasta ellas y marca en ámbar cualquier casilla cuyo camino provocaría un golpe de alguien, identificándolo debajo del tablero. Una opción que requiere un objetivo ilumina a quienes pueden elegirse, simultáneamente en el tablero y en la lista. Una opción con `area` apunta a una casilla, y la casilla bajo el puntero indica a quién atraparía, incluidos los aliados. Lo que queda del movimiento permitido se muestra junto a tus presupuestos, también en tu unidad. La pantalla no mide nada: el servidor envía cada casilla, coste, camino, objetivo y punto de mira.

**Movimiento.** El movimiento permitido de un turno es `economy.movement` para un miembro del grupo, o la `speed` propia de la criatura, dividido por `perCell` y redondeado HACIA ABAJO, nunca menos de una casilla mientras pueda moverse. Se repone al inicio del turno de su poseedor y puede gastarse antes, entre y después de las acciones: caminar, golpear, volver a caminar. Entrar en una casilla cuesta uno, o más si el terreno es difícil. Ocho direcciones, todas con el mismo coste, porque así se juegan las cuadrículas de mesa a las que se dirige. Se puede pasar por un aliado, pero no detenerse sobre nadie; un oponente es un muro; no se puede entrar en nada sólido ni cortar una esquina entre dos casillas sólidas.

**Alcance y distancia.** Una fila de arma los obtiene de `combat.attacks[].reach` y `.range`, cada uno una columna de esa misma lista o el mismo número en todas las filas:

```json
"attacks": [
  {
    "list": "attacks",
    "budget": "action",
    "name": "name",
    "toHit": { "ability": { "column": "ability" } },
    "damage": { "dice": { "column": "damage" } },
    "reach": { "column": "reach" },
    "range": { "normal": { "column": "range" }, "long": { "column": "long_range" } }
  }
]
```

Una columna con valor 0 en una fila indica que esa fila no tiene esa distancia; así puede convivir una espada ordinaria con un hacha arrojadiza en la misma lista. Una fila sin alcance llega a una casilla. Una acción de criatura utiliza su propio `reach` o `range`, y una capacidad de catálogo utiliza `mechanics.range` (0 es uno mismo o contacto, que equivale a una casilla cuando se dirige a otra persona).

Una fila con AMBOS es un arma arrojadiza: dentro de su alcance golpea cuerpo a cuerpo y fuera dispara. Por tanto, las reglas siguientes para disparos no le afectan en la mano de alguien, y sirve para golpear a quien pasa cerca, cosa que no hace un arco.

Una acción de criatura también puede llevar el `area` en la que impacta, en tu propia unidad: `{ "shape": "cone",
"size": 15 }`, con `"friendlyFire": false` para perdonar a su propio bando. Así, un arma de aliento es un cono real sobre el tablero en lugar de un número de objetivos. Una secuencia no tiene forma propia; las acciones que identifica tienen las suyas. Un combate sin tablero ignora la forma y utiliza `targetCount`, por lo que una entrada de criatura puede llevar ambos y ser fiel en los dos casos.

**Hasta dónde se puede enviar una forma.** Lo indica `range`: una bola lanzada a cien pies lleva uno. Sin distancia, una explosión se produce donde se coloca, en la casilla del propio actor, y un cono o una línea pueden apuntar a cualquier lugar dentro de la longitud que dibujan, porque ahí la casilla solo indica su dirección. Esto se aplica tanto a `mechanics.area` de una entrada de catálogo como al área de una criatura.

`ranged` indica qué cuesta un disparo efectuado más allá de su distancia ordinaria `normal`, o con alguien del bando contrario en la casilla siguiente. Cada uno es `"disadvantage"` o `"normal"`; omite el bloque y ninguno cuesta nada. Un golpe cuerpo a cuerpo nunca es un disparo, así que ninguna regla le afecta, ni tampoco a un arma arrojadiza usada dentro de su propio alcance.

**Áreas.** El `mechanics.area` de una entrada se convierte en una forma real en el tablero, dirigida a una casilla en lugar de a alguien, y `targetCount` no determina nada: la forma decide a cuántos alcanza. Atrapa a todos los que estén en esas casillas, amigos y enemigos, salvo que la entrada indique `"friendlyFire": false`.

```
burst, size 2, aimed at X        cone, size 3, aimed right      line, size 3, aimed right
. . . . .                        . . . .                        . . . .
. # # # .                        . . # .                        A # # #
. # X # .                        A # # #                        . . . .
. # # # .                        . . # .
. . . . .                        . . . .
```

Una explosión abarca todas las casillas dentro de su tamaño alrededor de la casilla a la que se apuntó. Un cono parte del actor hacia esa casilla, con una anchura en cada paso igual a la distancia recorrida. Una línea avanza igual, con una casilla de ancho. Las tres se detienen ante cualquier cosa sólida.

**Línea de visión y cobertura.** Una línea recta de casillas entre ambos: cualquier cosa sólida en ella bloquea un disparo e impide que un área se extienda más allá, y el objetivo simplemente no aparece en el menú. El terreno que sirve de cobertura suma `cover.bonus` a la defensa contra la que se tira el ataque, y el registro lo indica. No hay cobertura de tres cuartos, cobertura total ni elevación.

**Golpes contra quien se aleja.** Declara `opportunity.budget` y, cuando un combatiente salga caminando del alcance de un enemigo en pie que pueda actuar, tenga ese presupuesto y disponga de algo cuerpo a cuerpo con lo que golpear, el movimiento se DETIENE donde está y se pregunta al enemigo si desea golpear. Aceptar gasta el presupuesto y se resuelve exactamente como el mismo ataque en su propio turno; dejarlo pasar no cuesta nada. En ambos casos, el movimiento se reanuda donde se detuvo, pagando cada casilla realmente cruzada, y un golpe que derribe a quien se mueve termina su movimiento donde cayó. Una oportunidad por enemigo para todo el recorrido, por muchas veces que el camino salga del mismo alcance. `disengage` lo impide durante el resto del turno, y un conjunto de reglas sin `opportunity` no tiene nada de esto.

La pregunta es una VENTANA y detiene todo el combate: nada más se mueve hasta que respondan todos los consultados. La ventana de un miembro del grupo la responde el jugador, con el golpe o **Pass** (pasar) a su lado; la de los demás la responde quien los interpreta, y un jefe del Game Master mediante la decisión del propio Game Master. Véase Ventanas más abajo.

**Qué hace un oponente con el tablero.** Un oponente que nadie interpreta evalúa cada casilla alcanzable frente a todas las opciones que podría tomar desde allí, resta valor por cada golpe que recibiría durante el recorrido y prefiere no moverse si ya puede hacer lo mejor desde donde está. Si no tiene nada al alcance, se acerca, y primero esprinta si tu lista `standard` incluye `dash`.

**Rechazos que puedes ver.** `out-of-reach` (más lejos de lo que alcanza), `no-line-of-sight` (algo sólido en medio), `unreachable` (una casilla cuyo coste no puede pagar el movimiento o en la que no puede terminar) y `bad-cell` (una forma dirigida a un lugar al que no puede apuntar).

### Qué hace un combate con tu bloque en el servidor

Una partida cuyo conjunto de reglas declare `combat` obtiene un combate resuelto por él, sobre la misma batalla guardada que siempre ha usado Engine:

- **Quién participa.** El Game Master indica quién combate; Engine lee los números de cada miembro del grupo en su propia ficha. Un miembro sin ficha para tu conjunto de reglas se rechaza por nombre, en lugar de recibir números que no escribiste.
- **De dónde proceden los números de un oponente**, por este orden: la criatura que el Game Master identificó en tu bestiario; luego una cuya etiqueta coincida con el nombre del oponente; después un bloque de estadísticas propuesto por el Game Master para este combate, ajustado a tu escala de amenaza; y, por último, una criatura simple construida con los números del propio peldaño. Cada alternativa y cada ajuste se registran con palabras sencillas para que el combate pueda explicar qué hizo. Un conjunto de reglas sin entrada de bestiario, propuesta ni escala de amenaza rechaza el combate en lugar de inventar uno.
- **Tus fichas son el registro.** La salud, las reservas, las condiciones, la concentración y los contadores de tu regla de agonía se escriben mediante las reglas de la propia ficha tras cada acción aceptada, de modo que recargar a mitad del combate muestra exactamente lo que dejó, sin un recuento final que pueda contradecirlo.
- **Tu menú es la única fuente de legalidad.** Todo aquel que actúa, jugador u oponente, elige un identificador del mismo menú que produce tu bloque. Un oponente controlado por Engine elige de él con las tácticas del propio Engine; a uno controlado por el Game Master se le pide elegir un identificador de ese mismo menú, mostrándole tus números sin decirle nunca qué harán los dados.
- **Tus dados.** Un combate lleva su propia semilla y un cursor, por lo que uno recuperado del disco continúa con los dados que habría tirado.

### En pantalla

El combate se desarrolla en la pantalla de batalla con tus palabras. El menú contiene tus ataques, tus capacidades, tus enfrentamientos y las acciones estándar que enumeraste, indicando cada una qué gasta de tus presupuestos y reservas. Se muestran el orden de turnos, la ronda, cada condición que identificaste con sus rondas restantes, los puntos temporales, la concentración y los dos contadores de tu regla de agonía. El registro imprime la aritmética real en tus términos: "Juno attacks Rust jackal with Road axe: 8 (5 + 3) + 3 = 11 against Guard 6, a hit." Cada acción aceptada se escribe en la ficha al ocurrir, por lo que recargar a mitad del combate es exacto y después se indica al Game Master que no vuelva a cambiar esos números.

Un combate con posiciones se dibuja en el tablero en lugar del escenario de retratos; véase Posiciones para saber qué hace el jugador con él. Toda distancia en él, en el menú y en el registro se expresa en TU unidad: "Juno moves to 4, 6 for 6 paces and has 2 paces left."

### Enfrentamientos: agarrar, empujar, liberarse

Algunos movimientos no son un ataque contra una defensa, sino un enfrentamiento: ambos bandos tiran y quien saca más se sale con la suya. Agarrar a alguien, derribarlo o apartarlo de un empujón y liberarse tienen la misma forma, así que un conjunto expresa cada uno como datos:

```json
"checks": [
  { "id": "brawn", "label": "Brawn", "value": { "abilityMod": "brawn" } },
  { "id": "wits", "label": "Wits", "value": { "abilityMod": "wits" } }
],
"contests": [
  {
    "id": "grab",
    "label": "Grab",
    "budget": "act",
    "attacker": { "checks": ["brawn"] },
    "defender": { "checks": ["brawn", "wits"] },
    "onWin": { "applies": [{ "condition": "held" }] }
  },
  {
    "id": "break_free",
    "label": "Break free",
    "budget": "act",
    "attacker": { "checks": ["brawn", "wits"] },
    "defender": { "checks": ["brawn"] },
    "from": { "holding": "held" },
    "onWin": { "ends": [{ "condition": "held", "on": "actor" }] }
  },
  {
    "id": "shove",
    "label": "Shove back",
    "budget": "act",
    "attacker": { "checks": ["brawn"] },
    "defender": { "checks": ["brawn"] },
    "ties": "attacker",
    "onWin": { "push": 4 }
  }
]
```

- **`checks`** son los números que lee un enfrentamiento, cada uno un valor de la ficha, leído una vez al empezar el combate igual que una defensa o una salvación. Una criatura escrita con números simples da los suyos (`"checks": { "brawn": 2 }`); una con ficha los lee de ella. Hasta doce.
- **La tirada.** Ambos bandos tiran tus `attackRoll.dice` y suman la mejor de las pruebas que pueden usar aquí (`attacker.checks`, `defender.checks`). Gana el total más alto; un empate favorece al defensor salvo que `ties` diga `"attacker"`. El registro indica ambos bandos: "Juno tries Grab on Ash-hound: 12 (6 + 6) + 3 = 15 with Brawn against 2 (1 + 1) + 2 = 4 with Brawn, and wins."
- **Ganar** hace lo que indica `onWin`, al menos una cosa:
  - `applies` aplica estados al perdedor con el ganador como origen, así que un estado que marques con `endsWhenSourceDown` termina cuando cae quien sujeta. `rounds` le da un reloj; sin él, dura hasta que algo lo termine.
  - `ends` quita estados al actor o al objetivo.
  - `push` aleja al perdedor del ganador en línea recta, esa distancia en tu unidad, deteniéndose antes de cualquier cosa sólida, de cualquiera que esté en pie, del borde del tablero y de una esquina demasiado estrecha para pasar. Es forzado: no gasta nada de su movimiento y nadie le golpea por ello. Un combate sin tablero no mueve a nadie.
    Perder no hace nada, que es lo que es un agarre fallido.
- **`from: { "holding": "held" }`** hace que un enfrentamiento solo apunte a quien aplicó ese estado al actor, y solo lo pone en el menú mientras dura: así se expresa liberarse.
- **`reach`**, en tu unidad de distancia, es hasta dónde llega; la casilla siguiente si lo omites. `reach` y `push` necesitan `combat.distance`.
- **`strike: true`** le permite ocupar el lugar de un ataque cuando una acción compra varios, igual que un ataque: si se elige primero, gasta el presupuesto y deja disponibles el resto de los ataques; si se elige con ataques disponibles, cuesta uno de ellos. La referencia de 5e hace así los agarres y los empujones.
- **En el menú**, un enfrentamiento tiene su propio grupo e indica su probabilidad de ganar. Un oponente que interpreta el motor lo evalúa como cualquier otra cosa, pero con moderación: un agarre o un empujón prepara algo, y liberarse es lo que más vale, solo mientras se está sujeto. Un Game Master que interpreta a un oponente elige uno del mismo menú.
- Las pruebas de un oponente que inventa un Game Master se limitan a su nivel, igual que su probabilidad de acertar.

Los enfrentamientos requieren Capability API 1.43 en un conjunto empaquetado.

<a id="windows-holding-the-fight-open"></a>

### Ventanas: mantener abierto el combate

Algunos momentos pertenecen a alguien distinto de quien está actuando. Engine mantiene el combate abierto para esa persona en lugar de decidir por ella, y esa pausa es una ventana.

Seis cosas abren una, y dos proceden de lo que ya declaraste:

- **Alguien se aleja.** Un movimiento que sale del alcance de un enemigo capaz de golpear se detiene en ese paso y le pregunta. Véase Golpes contra quien se aleja, más arriba.
- **Entre un turno y el siguiente.** Cuando termina un turno, se pregunta a cada oponente con `signaturePoints` que pueda pagar una de sus propias acciones `signature` si desea comprarla, antes de iniciar el siguiente turno. Es el único momento en el que se compran: una acción distintiva no aparece en el menú de turno de nadie, incluido el suyo.
- **Alguien usa algo.** Antes de resolverlo, se pregunta a todos los del OTRO bando que tengan una entrada a la espera de ese momento, sea quien sea el objetivo, siempre que esa entrada alcance a quien lo usa. Una acción estándar y un enfrentamiento no abren ese momento.
- **Algo apunta a alguien.** Antes de resolverlo, se pregunta a todos los objetivos del OTRO bando que tengan una entrada a la espera de ese momento. Un amigo que te cura no es una amenaza a la que responder, así que una acción de un amigo no abre ninguna ventana.
- **Una tirada de ataque impacta a alguien.** Antes de su daño, se pregunta a quien recibió el impacto, si tiene una entrada a la espera de ese momento. Lo que elija cuenta para este ataque: consulta El momento después de un impacto, más abajo.
- **Algo ha dañado a alguien.** Después de resolverlo, se pregunta a todos los que hayan recibido daño y tengan una entrada a la espera de ESE momento, sea quien sea el causante. Recibir daño es un hecho sobre ti; una entrada dirigida de vuelta a quien lo causó sigue sin poder apuntar a un amigo.

Las cuatro últimas son las que solicita una entrada de catálogo, o una acción propia de una criatura, al identificar el momento que espera. Cuando una acción abre tanto `used` como `aimed`, primero se pregunta por el uso; si nadie lo anula, a continuación se pregunta a aquellos a quienes apunta, y se resuelve una vez que ambos han respondido. Por recibir un impacto se pregunta a medida que impacta cada una de sus tiradas, y por recibir daño, una vez terminada toda la acción.

Qué hace una ventana, sea cual sea su causa:

- **Nada más se mueve mientras está abierta.** Ni el actor cuyo turno está en curso, ni el final de ese turno, ni otra ventana. El combate espera.
- **Pregunta de uno en uno**, en orden de turno, una sola vez a cada uno. Pasar siempre es una respuesta y no cuesta nada. Si alguien consultado no tiene ninguna opción que pueda ejecutar, se omite en lugar de preguntarle.
- **Continúa exactamente donde se detuvo.** Un movimiento termina por las casillas que le quedaban, pagando cada una de las que realmente cruzó.
- **Responde quien los interpreta.** La ventana de tu propio miembro del grupo es tuya, con la opción y **Pass** (pasar) a su lado en el menú; la de un oponente la responde quien lo interpreta, y a un jefe del Game Master se le pregunta mediante el Game Master, con dejar pasar el momento como una de las respuestas.
- **Se guarda con el combate.** Una partida cerrada a mitad de un movimiento vuelve con las mismas personas pendientes de responder y las mismas casillas por recorrer.

No declaras nada para las dos primeras: un conjunto de reglas con `opportunity.budget` obtiene una, un bestiario con `signaturePoints` obtiene la otra, y uno sin ambos nunca las ve.

**Indicar qué momento espera una entrada.** Escribe `mechanics.reaction` como un objeto en lugar de `true`:

```json
"reaction": { "on": "aimed", "at": "source", "cancels": true }
```

- `on` es `used`, `aimed`, `hit` o `harmed`, y es lo que coloca la entrada en el menú de esa ventana. Son los únicos cuatro momentos que vigila Engine. Una entrada que todavía indique `"reaction": true` solo dice que no se ejecuta en un turno; eso no basta para ofrecerla en ningún lugar, así que no aparece en ningún menú.
- `at` es `source` (el valor predeterminado) o `chosen`. `source` dirige lo ejecutado hacia quien causó el momento y completa el objetivo, de modo que no se pide elegir a nadie; `chosen` conserva los objetivos propios de la entrada y pregunta.
- `cancels` impide por completo que suceda lo que retenía la ventana. Solo una entrada `used` o `aimed` puede indicarlo: un momento que ya ocurrió no puede cancelarse, y un impacto ya se ha tirado.
- `against` restringe lo que responde la entrada a cosas usadas desde ciertos catálogos: `"against": { "catalogs": ["spells"] }` responde a un conjuro y a nada más. Una acción sin entrada detrás (un arma de una lista, una acción propia de una criatura) no procede de ningún catálogo, así que nunca abre el momento para una entrada que nombra catálogos. Omite `against` y la entrada responde a cualquier cosa.

Dale también un `budget`, o gastará el predeterminado de la lista. Una reacción casi siempre gasta un presupuesto propio, lo que impide que un turno contenga varias.

**Su coste se gasta antes de preguntar a nadie.** Una acción cancelada deja de suceder, pero no deja de haberse comprado: el presupuesto y las reservas ya se gastaron. Si tu sistema los devuelve, todavía no puede expresarlo.

Una contra se escribe así, y en un tablero solo responde a alguien dentro de su propio `range`:

```json
"mechanics": {
  "kind": "utility",
  "range": 8,
  "free": true,
  "cost": [{ "pool": "luck", "amount": 1 }],
  "reaction": { "on": "used", "cancels": true, "against": { "catalogs": ["knacks"] } }
}
```

**El momento después de un impacto.** Una entrada en `hit` es la forma de expresar Shield, una Parry y Uncanny Dodge. El ataque se retiene después de su tirada, y se pregunta a quien recibió el impacto antes de infligir ningún daño. Lo que elija cuenta para ese ataque: su tirada no se repite, sino que se vuelve a comprobar contra su defensa tal como esté ahora, así que un estado que suma a la defensa puede convertir el impacto en un fallo, y una cara natural que siempre impacta sigue impactando. Un estado que dura un ataque (`endsAfter: "attacked"`) aplicado como respuesta cubre este ataque, daño incluido, y se gasta con él, que es como se dice "reduce a la mitad el daño de ese ataque": `resist-all` durante un ataque.

```json
"mechanics": {
  "kind": "buff",
  "targets": "self",
  "budget": "reaction",
  "reaction": { "on": "hit" },
  "applies": [{ "condition": "shielded", "duration": { "rounds": 1, "at": "turn-start" } }]
}
```

El ataque continúa exactamente donde se retuvo: el resto de sus objetivos y, en una acción de criatura formada por otras acciones, el resto de sus partes. Cada tirada que impacta a alguien que tiene una respuesta así se retiene en su turno, y un combate guardado mientras la ventana está abierta vuelve igual. Un ataque hecho dentro de una ventana (un golpe contra alguien que se aleja, una acción distintiva, una respuesta) nunca se retiene, porque nada abierto dentro de una ventana abre otra. El registro muestra la tirada y contra qué se hizo cuando se abre la ventana y, si la respuesta cambió la defensa, si ahora falla. Los oponentes que interpreta Engine solo se protegen cuando eso convierte el impacto en un fallo.

Un paquete que identifique un momento necesita Capability API 1.33, y uno que use `used` o `against` necesita 1.44. Uno que espera `hit` necesita 1.46.

### Todavía no

Dicho claramente, porque un conjunto de reglas no debe prometer lo que Engine no hace:

- **Más allá del tablero modesto**: no hay cobertura de tres cuartos ni total, elevación, vuelo sobre obstáculos, paso por espacios estrechos, monturas, ocultación ni sorpresa, y nada mueve a nadie salvo su propio movimiento y el empujón de un enfrentamiento. Nadie arrastra a una criatura que sujeta.
- **Un enfrentamiento es simple.** No tiene límites de tamaño y no abre ninguna ventana (nadie puede responder a uno).
- **Una entrada solo puede esperar cuatro momentos**, `used`, `aimed`, `hit` y `harmed` (véase Ventanas, más arriba). Son los momentos que Engine detecta en nombre de una entrada; las otras dos ventanas, alguien que se aleja y la pausa entre turnos, las abre el propio combate y no son momentos que pueda solicitar una entrada. No existe un momento para una tirada de salvación, una muerte, el inicio de un turno o una caída.
- **Una contra detiene por completo lo que responde.** No puede distinguir una entrada de un catálogo de otra, y no hay ninguna prueba para detener algo más grande que ella.
- **No se encadenan.** El combate mantiene una ventana en lugar de una pila, así que nada abierto dentro de una ventana abre otra: una contra no puede recibir a su vez una contra, lo que inflige una reacción no abre otro momento, y un ataque hecho dentro de una ventana no se retiene cuando impacta.
- **Solo una tirada de ataque es un impacto.** Algo que alcanza sin ella (dardos que simplemente impactan, un área contra la que todos hacen una salvación) no abre ningún momento `hit`, y una respuesta no puede distinguir un golpe de un disparo, así que una Parry que las reglas limitan a las espadas también detiene flechas.
- **El daño se cambia reduciéndolo a la mitad, no en una cantidad.** Una respuesta puede reducir a la mitad el daño de un ataque (`resist-all` durante ese ataque); no puede restarle un número tirado.
- **No se devuelve nada.** Lo que costó una acción cancelada queda gastado.
- **Lo que cambia un estado es una lista cerrada.** La defensa, las tiradas de ataque, las salvaciones, las pruebas de enfrentamiento y la velocidad (y lo mismo para un nivel y para lo que hace un objeto en un combate), y nada más: ninguna bonificación al daño que inflige su portador, y ningún nivel que reduzca el máximo que puede tener un personaje o que lo saque del combate.
- **Una criatura escrita con números simples no tiene contador de heridas.** En un conjunto de reglas cuya salud sea un contador, esa criatura sigue perdiendo puntos; dale una `sheet` y sus golpes marcarán casillas, mitigados primero por sus propios `resist`, `vulnerable` e `immune`.
- **Un efecto adicional se activa solo.** `on` tiene un único valor, `hit`, así que lo aplica el primer impacto del período que cumpla los requisitos, sin ningún momento en el que se pregunte si deseas gastarlo.
- **Un número que mueven los ataques es simple.** Un golpe que gasta tira el número y nada más, así que ningún arma lo cambia, incluido su `floor` (un objetivo resistente absorbe un golpe que toma, y su dureza detiene uno que gasta, pero nada más reduce ninguno de los dos); un derrumbe se levanta tras un número fijo de turnos por muy profundo que fuera; y un ataque hecho en una ventana (un golpe contra alguien que se aleja, una reacción, un movimiento distintivo) se hace en el primer estilo.
- **Un oponente inventado no absorbe nada y no tiene dureza.** Una criatura que el Game Master inventa para un combate queda limitada a tu escala de amenaza, que no dice nada sobre absorción ni dureza, así que no tiene ninguna de las dos.

## Capas: variantes de tu propio conjunto de reglas

Una capa es una variante con nombre de tu conjunto de reglas que el jugador activa al crear una partida: magia escasa, invierno duro, una dificultad más cruda. Las capas residen en el archivo del conjunto de reglas, en una matriz opcional `layers`, por lo que viajan con él y nunca pueden faltar en una partida que las haya usado. El asistente las muestra como interruptores debajo de tu conjunto de reglas, y la elección queda fijada para toda la vida de esa partida, igual que el propio conjunto.

```json
"layers": [
  {
    "id": "hard_winter",
    "label": "Hard winter",
    "summary": "Cold, hunger and short days. Everything is harder.",
    "conflicts": ["mud_season"],
    "gm": {
      "guidance": "Hard winter is on. Let a failed check cost warmth, food or daylight as well as progress.",
      "worldGuidance": "Hard winter is on. Build a world of closed roads, thin stores and rationed settlements."
    },
    "fields": [{ "id": "calling", "removeValues": ["Sailor"], "default": "Hauler" }],
    "difficultyLadder": [{ "label": "Easy", "dc": 7 }],
    "catalogs": [{ "id": "knacks", "hide": { "filter": "grit", "above": 0 } }]
  },
  {
    "id": "mud_season",
    "label": "Mud season",
    "summary": "Thaw, flooded roads and slow going."
  }
]
```

**Qué puede hacer una capa.** La lista es cerrada, y cada efecto restringe algo o añade texto:

- `gm.guidance` se añade al final de tu `gm.checkGuidance`, después de tu propio texto y del de cualquier capa anterior. `gm.worldGuidance` se añade al final de `gm.worldGuidance` del mismo modo.
- `fields` retira valores de un campo **enumerado**. `removeValues` identifica valores que el campo ya contiene, debe sobrevivir al menos uno y, si el `default` del campo es uno de los retirados, la capa indica otro `default` que sobreviva.
- `difficultyLadder` sustituye tu escala por otra con la estructura de tu propio tipo de resolución: `{label, dc}` para `dice-sum` y `{label, successes, target?}` para `dice-pool`. Se somete exactamente a las mismas validaciones que tu propia escala. Cuando varias capas activas declaran una, gana la última.
- `catalogs` oculta entradas del selector del editor de fichas. Cada regla identifica uno de los `filters` declarados por ese catálogo y exactamente una comparación: `above` o `below` para un filtro `number`, `equals` o `notIn` para uno `text` o `tags`. Nunca se oculta una entrada que no establezca ese filtro.
- `currencies` retira monedas: `removeUnits` nombra monedas sueltas y `removeFamilies`, familias enteras. La moneda más pequeña de una familia solo se va con su familia, para que cada familia que queda pueda seguir pagando y dando cambio. Mientras la capa está activa, nadie gana, paga con, elige ni suelta una moneda retirada, y la línea del monedero la omite. Un precio expresado en ella se indica en la moneda más grande que queda y que lo paga exactamente, con el mismo valor: la larga noche de Gravewatch retira la crown, así que una pistola de guardia de 3 crowns cuesta 15 shillings. Un objeto cuya familia entera ha desaparecido no tiene precio.

**Qué no puede hacer una capa.** No puede añadir un valor enumerado, un campo, una habilidad, una reserva o un descanso, cambiar el tipo de resolución, tocar el estado en vivo o los números de combate ni añadir una llamada a un modelo. Un valor que una capa _añadiera_ sería desconocido para todos los demás lectores de la ficha, así que los valores solo se retiran. Todo lo que quede fuera de esta lista es un cambio del propio conjunto de reglas o un segundo conjunto.

**Conflictos.** `conflicts` identifica capas que no pueden estar activas juntas. Basta con indicar un lado del par. El asistente desactiva el otro interruptor y, si una elección guardada contiene ambos de algún modo, se descarta la capa declarada **después**, para que las mismas dos elecciones siempre produzcan las mismas reglas.

**Una ficha que ya contenga un valor retirado lo conserva.** Nada reescribe al personaje. El editor simplemente deja de ofrecer el valor, y un personaje que ya lo tenía lo muestra tal como es. Desactiva la capa en una nueva partida y el valor volverá a ofrecerse. Lo mismo ocurre con una entrada de catálogo oculta: queda fuera del selector y una fila que el jugador ya eligió permanece en la ficha. Las monedas que retira una capa y que alguien ya lleva se quedan en su bolsa.

**Límites.** 12 capas por conjunto de reglas y 4000 caracteres de instrucciones por capa, contando ambas cadenas juntas. Un conjunto de reglas empaquetado que declare `layers` o un `gm.worldGuidance` base necesita Capability API 1.25, y uno cuyas capas retiran monedas necesita 1.64. Uno que importes lo valida Engine al leerlo, así que no necesita nada.

**Las capas escritas por otra persona** (una capa de magia escasa para un conjunto de reglas que no escribiste, distribuida en su propio archivo) llegarán más adelante. Actualmente, una capa se distribuye dentro del conjunto de reglas al que pertenece.

<a id="trying-your-ruleset"></a>

## Probar tu conjunto de reglas

Los conjuntos de reglas comunitarios usan el mismo interruptor que los agentes importados. Abre **Settings** (ajustes) > **Advanced** (avanzado) > **Danger Zone** (zona de peligro) y comprueba que **Allow custom Agent imports** (permitir importaciones de agentes personalizados) esté activado. La importación también necesita acceso desde localhost o **Admin Access** (acceso de administrador) configurado.

1. Abre el panel **Agents** (agentes) y elige el botón **Import agents** (importar agentes), el icono de descarga de la fila de botones de la parte superior del panel.
2. Elige **Game Mode ruleset** (conjunto de reglas del modo de juego) y selecciona tu archivo JSON.
3. Lee la revisión. Muestra el nombre, la versión, la licencia, lo que cubre el conjunto de reglas y el texto del Game Master. Elige **Import** (importar).

Tu conjunto aparece en la sección **Rules** (reglas) del panel y en la elección **Rules** del asistente de configuración para nuevas partidas. Uno importado desde un archivo se clasifica como `local/<your id>`, de modo que nunca pueda confundirse con uno oficial ni con el de otra persona.

### Cambiar un conjunto de reglas que ya importaste

Una versión importada nunca se reescribe. Si cambias el archivo y lo vuelves a importar con la misma `version`, la importación se rechaza y te pide aumentar el número. Es intencional: una partida está ligada a la versión exacta con la que se creó, para que una campaña en curso nunca despierte con cálculos diferentes.

Por tanto, el ciclo durante la redacción es: editar, aumentar `version`, importar, iniciar una partida nueva. Las versiones antiguas permanecen instaladas junto a la nueva hasta que retires el conjunto de reglas de la sección **Rules**. Retirar uno que todavía usa una partida hace que esta indique que falta su conjunto de reglas hasta que vuelvas a importarlo.

Si cambias la estructura de la ficha (añades, retiras o renombras elementos), aumenta también `sheet.version`. Las fichas existentes se leen con tolerancia: se conservan los valores que la nueva ficha no conoce y los ausentes toman sus valores predeterminados.

## Compartir tu conjunto de reglas

**Como archivo.** Envía el archivo JSON a un amigo. Lo importa igual que tú.

**Desde un repositorio de GitHub.** Si guardas tu trabajo en un repositorio público de GitHub, coloca cada conjunto de reglas en una carpeta `rulesets` en la raíz del repositorio, un archivo por conjunto:

```text
your-repository/
  agents.json        (optional, only if you also share agents)
  rulesets/
    ember-roads.json
    another-system.json
```

Un usuario añade tu repositorio una sola vez mediante la lista de repositorios de agentes personalizados, revisa lo que contiene y puede sincronizarlo después para recibir nuevas versiones. La lista de repositorios personalizados es una función avanzada que quien ejecuta el servidor debe activar con `ENABLE_CUSTOM_AGENT_REPOS=true`. Los conjuntos de reglas de un repositorio se clasifican bajo el nombre de su propietario, como `alice/ember-roads`, para que dos autores puedan publicar uno llamado `v20` sin colisiones.

Se aplican dos límites. Un repositorio puede contener como máximo 32 archivos JSON directamente dentro de `rulesets`; si tiene más, se rechaza. Una cuenta llamada `local` no puede publicar conjuntos de reglas, porque `local/` se reserva para los importados desde un archivo.

**En el catálogo oficial.** Un sistema muy utilizado y con licencias claras puede ofrecerse a todo el mundo mediante **Download Agents** (descargar agentes). Eso requiere un pull request al repositorio [Marinara-Agents](https://github.com/Pasta-Devs/Marinara-Agents). Consulta allí el paquete `ruleset-5e-2014` para ver la estructura.

## Licencias

Publica solo texto de reglas que tengas derecho a compartir. Muchos sistemas publican un documento de referencia con una licencia abierta, y ese es el documento del que puedes copiar. Coloca el identificador de la licencia y el texto de atribución que exige bajo `license`. No copies texto de manuales sin licencia abierta. Un conjunto de reglas necesita principalmente nombres y números, y el texto del Game Master debe estar escrito con tus propias palabras.

## Solución de problemas

- **La importación dice que un nombre no existe.** Algo del archivo apunta a un identificador no declarado, como una habilidad que identifica una característica que retiraste. El mensaje indica la ruta hasta la línea.
- **La importación dice que una versión ya está instalada con otro contenido.** Aumenta `version` y vuelve a importar.
- **Mi conjunto de reglas no aparece en el asistente de configuración.** Comprueba que **Allow custom Agent imports** esté activado. Mientras esté desactivado, los conjuntos importados quedan fuera de las partidas nuevas. Las partidas que ya usan uno siguen funcionando.
- **Una partida dice que falta su conjunto de reglas.** No está instalada la versión exacta con la que se creó. Vuelve a importar esa versión del archivo.
