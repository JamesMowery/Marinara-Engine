# Escrever conjuntos de regras de Game Mode

Um conjunto de regras informa ao Game Mode como funciona um sistema de mesa: quais dados um teste rola, o que há na ficha, quais recursos são gastos e o que um descanso recupera. Este guia é para quem quer escrever seu próprio conjunto e compartilhá-lo. Para jogar com um criado por outra pessoa, comece por [Escolher as regras](../game/getting-started.md#choosing-rules).

Um conjunto é um arquivo JSON. São dados, não código. Nada nele é executado, então importá-lo não pode fazer nada no seu computador. Antes de importar um arquivo alheio, leia com atenção o texto do Game Master: ele é enviado ao modelo em toda partida que usa o conjunto.

## Leia primeiro: o que um conjunto pode e não pode fazer

Um conjunto só pode preencher os parâmetros de uma mecânica que o Engine já conhece. Hoje ele conhece duas formas de resolver um teste; o arquivo escolhe uma com `resolution.kind`:

- **`dice-sum`**: rolar dados, adicionar números da ficha e alcançar ou superar uma dificuldade. Isso cobre sistemas d20, sistemas de 2d6 mais atributo e muitos outros.
- **`dice-pool`**: rolar a quantidade de dados do personagem e contar os que atingem um alvo. Isso cobre sistemas em que uma pontuação representa um punhado de dados, não um bônus.

Ambos são descritos por completo em [Tipos de resolução](#resolution-kinds).

Uma mecânica que não se encaixa em nenhuma forma não pode ser escrita no arquivo. Por exemplo: usar o maior dado de uma reserva, testes percentuais abaixo de um valor, dados de símbolos e reservas opostas. Cada uma precisa de um novo tipo de resolução no Engine: uma contribuição de código com testes, não um JSON. Se seu sistema precisa disso, abra uma solicitação de recurso no repositório do Engine e descreva a mecânica com algumas rolagens resolvidas. Esses exemplos viram os testes.

Game Mode pode resolver uma luta com o combate próprio do Marinara ou com as regras do conjunto. O bloco opcional `battle` empresta ao combate do Marinara os números das fichas: veja [Batalhas](#battles-lending-the-sheet-to-marinaras-combat). O bloco opcional `combat` define como o conjunto resolve a luta: veja [Combate](#combat-a-fight-your-own-rules-resolve). O Engine já aplica essas regras. **Combat Preference** (Preferência de combate) escolhe a apresentação Classic ou, quando o conjunto define distância, um campo Tactical.

## Início rápido

1. Copie o exemplo que corresponde às rolagens do seu sistema. [`ember-roads.json`](https://github.com/Pasta-Devs/Marinara-Engine/blob/staging/docs/examples/rulesets/ember-roads.json) é um pequeno sistema de 2d6 com três atributos que mostra que o formato não pressupõe d20 nem seis atributos. [`gravewatch.json`](https://github.com/Pasta-Devs/Marinara-Engine/blob/staging/docs/examples/rulesets/gravewatch.json) é uma pequena reserva de dados de dez faces com três pontuações e seis ofícios. Como exemplo completo, veja o arquivo 5e (SRD 5.1), [`ruleset-5e-2014.example.json`](https://github.com/Pasta-Devs/Marinara-Engine/blob/staging/docs/development/ruleset-5e-2014.example.json).
2. Troque `id` pelo seu. Use letras minúsculas, dígitos e hífens simples, como `ember-roads`.
3. Edite a ficha, os descansos e o texto do Game Master.
4. Importe o arquivo; veja [Experimentar seu conjunto](#trying-your-ruleset). A importação verifica o arquivo inteiro e informa os erros linha por linha antes de salvar qualquer coisa.
5. Crie uma partida, escolha seu conjunto em **Rules** (Regras) e jogue alguns testes.

Para receber ajuda enquanto escreve, indique o JSON Schema ao editor adicionando esta primeira linha dentro das chaves externas do arquivo:

```json
"$schema": "https://raw.githubusercontent.com/Pasta-Devs/Marinara-Engine/staging/docs/extending/ruleset.schema.json",
```

O esquema detecta chaves mal escritas e tipos incorretos durante a digitação. Ele não verifica se os nomes apontam para elementos existentes, como uma perícia que nomeia um atributo. A importação faz isso.

Você pode adicionar uma linha `"$comment": "..."` a qualquer objeto para deixar uma nota. O Engine a ignora.

## As partes do arquivo

| Chave | Conteúdo |
| --- | --- |
| `schemaVersion` | Sempre `1`. |
| `id`, `version` | O nome do conjunto para o Engine e um inteiro que você aumenta a cada mudança publicada. |
| `name` | O que os jogadores veem no assistente. |
| `edition` | Opcional. Uma linha sobre a edição ou o rascunho. |
| `license` | Opcional. Um ID SPDX e a atribuição exigida pela fonte. |
| `coverage` | O que o conjunto cobre e o resumo de uma linha exibido no assistente. |
| `resolution` | Como um teste ou salvaguarda é rolado. |
| `sheet` | Tudo que há na ficha. |
| `rests` | O que cada descanso restaura e limpa. |
| `gm` | O texto fornecido ao modelo do Game Master e os valores de ficha que ele vê por personagem. |
| `catalogs` | Opcional. Entradas prontas oferecidas pelo editor para evitar listas longas digitadas à mão. |
| `items` | Opcional. O vocabulário em que seus itens são escritos: categorias, raridades, estatísticas, espaços, dinheiro. |
| `battle` | Opcional. O que uma batalha lê da ficha e grava de volta depois. |
| `combat` | Opcional. Como suas regras resolvem uma luta e o que a tela de batalha executa. |
| `layers` | Opcional. Variantes do conjunto ativadas na criação de uma partida. |

O arquivo pode ter até 256 KB. Texto que entra num prompt (nomes, rótulos, texto do Game Master) não pode conter quebras de linha, colchetes nem chaves duplas.

IDs dentro da ficha (atributos, perícias, campos, reservas etc.) usam letras minúsculas, dígitos e sublinhados, começando com uma letra, como `grit_max`.

<a id="resolution-kinds"></a>

### Tipos de resolução

`resolution.kind` escolhe como rolar um teste. Os dois tipos leem a mesma ficha e compartilham três chaves; as partes do arquivo depois de `resolution` não mudam ao alternar:

- `abilityModifier`: como uma pontuação da ficha vira um número. `identity` usa a própria pontuação; `floorHalfMinusTen` é a regra de 5e; `stepTable` permite definir seus limiares como `[[score, number], ...]`.
- `proficiencyTiers`: níveis de treinamento de uma perícia ou salvaguarda. O primeiro vale para uma perícia não listada. Um nível adiciona `flat`, ou `multiplier` vezes um bônus de proficiência, ou ambos. Se o sistema tem esse bônus, indique a origem com `"proficiency": { "bonus": { "derived": "proficiency_bonus" } }`.
- `proficiency`: opcional, necessário apenas para um nível que multiplica.

O significado do número depende do tipo: `dice-sum` o soma à rolagem; `dice-pool` rola essa quantidade de dados.

#### `dice-sum`: somar os dados

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

- `dice`: quantidade de dados e de faces. O total é comparado à dificuldade.
- `advantage`: se o GM pode pedir duas rolagens e manter uma.
- `naturals`: efeito das faces máxima e mínima de um dado único em testes e salvaguardas: `none`, `both`, `max-only` ou `min-only`. Omita para usar só aritmética. Exige um único dado; um sistema 2d6 precisa usar `none`.
- `difficultyLadder`: dificuldades entre as quais o GM deve escolher. `dc` é o número que o total precisa alcançar. O GM pode nomear um patamar em vez de escrever seu número, como `difficulty="Hard"`, e o Engine lê o número desse patamar.

#### `dice-pool`: rolar os dados e contá-los

O número da ficha é o **tamanho da reserva**, não um bônus adicional. Uma pontuação de 3 e um ofício que vale 2 rolam cinco dados. Essa é a ideia: sem novo vocabulário de ficha nem novo editor, um sistema cujas pontuações são punhados de dados usa as mesmas `abilities`, `skills` e `proficiencyTiers` de qualquer outro.

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

- `die`: faces de cada dado da reserva, de 2 a 100.
- `pool`: limites aplicados ao número da ficha antes de qualquer explosão. Um `min` de 0 permite que uma reserva vazia falhe sem rolar; `max` não pode passar de 100. `abilityPlusAbility: true` permite que um teste de atributo nomeie um segundo atributo com `with=` e role os dois, para sistemas que rolam dois atributos juntos; sem isso, `with=` não significa nada num teste de atributo.
- `target`: face que o dado precisa atingir para contar. Coloque `min` abaixo de `max` para permitir ao GM ajustá-la por teste com `threshold=`; iguale os três valores para fixá-la.
- `double`: opcional. Uma face igual ou superior a `from` conta duas vezes.
- `explode`: opcional. Uma face igual ou superior a `from` rola mais um dado, que também pode explodir. Dados extras são limitados a `pool.max` além da reserva inicial: um teste rola no máximo o dobro de `pool.max` e um `from` baixo não causa rolagens infinitas.
- Ambos podem ter um `min`: a face mais baixa até a qual um teste pode baixar a regra, com `explode="N"` ou `double="N"` na tag ou com uma entrada que o personagem usou. Com um `min`, você pode omitir `from`, e então a regra só dispara num teste que a peça: uma especialidade que faz os dez rolarem de novo numa perícia é `"explode": { "min": 10 }`. `from` não pode ficar abaixo de `min`.
- `cancel`: opcional. Uma face igual ou inferior a `upTo` retira um sucesso. A contagem nunca fica abaixo de zero.
- `botch`: opcional. Se **nenhum** dado teve sucesso e saiu uma face igual ou inferior a `upTo`, o teste é uma falha crítica. Uma reserva cujo único sucesso foi cancelado falhou, mas não é uma falha crítica. `"rule": "halfOrMore"` lê isso de outra forma: faces iguais ou inferiores a `upTo` em pelo menos metade dos dados rolados no início (sem contar explosões) são uma falha crítica quando nenhum dado teve sucesso e, caso contrário, mantêm o resultado e o marcam com `complication="true"`, algo que dá errado junto com ele. `"noSuccesses"` é o padrão.
- `exceptional`: opcional. Essa quantidade de sucessos líquidos ou mais, num teste bem-sucedido, produz um sucesso crítico.
- `situationalDice`: opcional. Limites de dados que o GM pode adicionar ou retirar com `bonus=` por acrobacias, ferimentos ou pouca luz.
- `reroll`: opcional. Novas rolagens que o sistema concede sem que ninguém pague por elas, até seis, cada uma com um `id` que o GM nomeia num teste com `reroll=`. Faces iguais ou inferiores a `upTo` são roladas de novo: com `"once"`, a nova face vale, seja qual for; com `"until"`, até mostrarem mais. `upTo` vai de 1 até uma face abaixo do número de faces do dado, já que rolar de novo todas as faces nunca pararia, e um `until` para de qualquer forma depois de cem novas rolagens num teste. Nenhuma é rolada a menos que o teste a nomeie. Um teste faz uma única nova rolagem: quando um gasto ou uma entrada usada pelo personagem também comprou uma, rola-se a que alcança mais faces e, se alcançarem as mesmas, `until` prevalece sobre `once`.
- `difficultyLadder`: `successes` informa quantos sucessos são necessários. Um patamar também pode nomear `target`, mas só se for ajustável e dentro dos limites. Esse alvo é o que um teste naquele patamar usa para contar: quando o GM nomeia o patamar com `difficulty=` ou escreve um `dc=` que exatamente um patamar exige. Quando vários patamares exigem o mesmo número de sucessos, `dc=` sozinho não diz qual foi o pretendido, então o alvo continua sendo o padrão; nomeie o patamar.

As faces de `cancel` e `botch` precisam ficar abaixo do alvo mínimo, e toda face nomeada deve existir no dado. Uma regra impossível de ativar é recusada na importação, em vez de ser descoberta durante a partida.

Um conjunto empacotado que usa reservas exige Capability API 1.24; um `min` em `explode` ou `double`, `pool.abilityPlusAbility` e `botch.rule` exigem 1.37; `reroll` exige 1.38. Um conjunto comunitário importado é validado pelo Engine que o lê, então não precisa de nada mais.

#### O que o Game Master pode escrever num teste de reserva

```
[skill_check: skill="Ward" dc="2" who="Bram the Quiet" threshold="8" bonus="-2" with="Sinew"]
[skill_check: skill="Ward" difficulty="Grim" explode="9"]
[skill_check: skill="Nerve" dc="1" with="Warmth"]
[skill_check: skill="Ward" dc="2" reroll="careful"]
```

- `dc` é a quantidade de **sucessos** necessária, não o alvo por dado. Vai de 1 ao máximo que uma rolagem poderia contar: máximo da reserva, dobrado se os dados explodem e dobrado novamente se as faces contam duas vezes.
- `threshold=` ajusta o alvo por dado e só é oferecido quando `target.min` é menor que `target.max`.
- `bonus=` adiciona ou retira dados e só é oferecido se `situationalDice` estiver declarado.
- `with=` rola uma perícia ou salvaguarda com outro atributo. Funciona nos dois tipos; 5e obtém "Strength (Intimidation)" pelo mesmo atributo. Num teste de atributo, ele adiciona um segundo atributo, quando `pool.abilityPlusAbility` está ativado.
- `difficulty=` nomeia um patamar da escala no lugar de `dc=`, nos dois tipos: seus sucessos (ou sua dificuldade numa soma) e, numa reserva, seu alvo. Um `dc=` escrito ao lado ainda prevalece para o número, e `threshold=` para o alvo. Um nome ao qual nenhum patamar responde, ou que dois patamares compartilham, é ignorado.
- `explode=` e `double=` movem as faces dessas regras neste teste e só são oferecidos quando a regra tem um `min`.
- `reroll=` nomeia uma das suas entradas `reroll` para este teste e só é oferecido quando você declarou alguma. Um nome ao qual nenhuma delas responde é ignorado.

Tudo respeita seu arquivo: valores fora dos limites são ajustados ao extremo mais próximo e um atributo não oferecido é ignorado sem recusar o teste. O registro salvo mostra o que a rolagem realmente usou: a dificuldade e o alvo como números, os dados extras depois dos limites, `with=` só quando o atributo foi trocado ou adicionado, uma face só quando o teste a moveu, `reroll=` só quando a nova rolagem que ele nomeia foi a rolada e rolou alguma coisa, e `complication="true"` quando uma falha crítica `halfOrMore` deu errado junto com um resultado. O Engine sempre rola os dados. Substitui resultados de reserva escritos pelo modelo, ignora `mode="advantage"` porque esse tipo não tem vantagem e não usa um dado que o jogador rolou antes do turno.

#### O que fica fora e por quê

Cada caso exige seu próprio tipo de resolução: nenhum pode ser expresso contando dados que atingem um alvo.

- **Usar o maior dado** (como em Blades in the Dark) exige um nível de sucesso parcial que o resultado de um teste não possui.
- **Reservas de postura comparadas a um atributo** (como em Lasers and Feelings) decidem a direção da comparação por teste.
- **Dados de símbolos** (como em Genesys) não produzem números.
- **Reservas opostas** resolvem dois personagens ao mesmo tempo; um teste tem um único rolador.
- **Percentuais abaixo do valor e percentuais abertos** comparam na outra direção.
- **Reservas somadas com um dado especial** (como em OpenD6) somam os dados e tratam um deles de forma diferente.

As duas coisas que esse tipo deixava de fora já estão modeladas; a seção sobre gastar para mudar uma rolagem, abaixo, explica como. Uma regra do próprio sistema, "spend a point for a success" ou "spend a point to throw the failures again", é `resolution.spend`. Uma nova rolagem que o sistema concede de graça quando a situação pede é `resolution.reroll`, acima. Uma que pertence a algo escolhido pelo personagem é `mechanics.check` numa entrada de catálogo, paga com o custo da própria entrada.

### Gastar para mudar uma rolagem

Alguns sistemas permitem pagar por uma rolagem prestes a acontecer: um ponto de vontade por um sucesso automático. `resolution.spend` define isso como regra permanente do sistema, não como algo comprado pelo personagem:

```json
"spend": [{ "pool": "resolve", "amount": 1, "successes": 1, "perCheck": 2 }]
```

- `pool` é uma das suas `live.pools`. Não pode começar vazia, pois não haveria nada para gastar no início da partida.
- `amount` é o custo de UMA compra. `successes`, `dice` e `reroll` informam o que ela compra; precisa comprar pelo menos um deles. Sucessos são adicionados depois da contagem dos dados e das anulações, porque ninguém os rolou. Dados são rolados com a reserva, dentro dos limites dela. Um `reroll` é `{ "upTo": 6, "mode": "once" }`, lido como `resolution.reroll`, e é comprado uma única vez, quantas compras o teste fizer: um dado rolado de novo duas vezes continua sendo uma nova rolagem.
- `perCheck` limita as compras de um teste: no máximo ele pode comprar o equivalente a `amount * perCheck` pontos. Esse teto impede uma reserva cheia de comprar uma rolagem impossível de perder. É um número de 1 a 10, um valor da ficha na mesma forma que um valor derivado lê (`{ "abilityScore": "nerve" }`, `{ "derived": "focus" }`) ou `"pool"`: tantas quantas forem os dados do teste, ou seja, o número da ficha para ele antes de ferimentos, dados extras ou modificadores. Um valor da ficha é lido para quem rola, arredondado para baixo e mantido entre 0 e 100, e um personagem cujo limite chega a 0 não compra nada nem paga nada.
- Só `dice-pool` permite isso: uma rolagem somada não tem sucessos nem reserva para receber acréscimos. Um conjunto `dice-sum` que declara `spend` é recusado na importação.
- Até quatro entradas, e duas não podem nomear a mesma reserva, pois o teste não conseguiria distingui-las.
- Um gasto que compra um `reroll`, um `perCheck` que não é um número e mais de duas entradas exigem Capability API 1.38 num conjunto empacotado.

**Isso vai no próprio teste.** O GM escreve `[skill_check: skill="Nerve" dc="2" spend="resolve:1"]`, não um comando `[sheet:]` separado: os dados são rolados antes de aplicar comandos de ficha e não restaria nada para mudar. Uma única resolução rola os dados e paga pelo que os modificou.

### Um amuleto que muda uma rolagem

`resolution.spend` é uma regra do sistema. Uma entrada que o personagem realmente ESCOLHEU também pode mudar um teste, com `mechanics.check` na entrada de catálogo:

```json
"mechanics": {
  "kind": "utility",
  "cost": [{ "pool": "blood", "amount": 1 }],
  "perCostStep": { "flat": 1 },
  "check": { "reroll": { "upTo": 1, "mode": "once" }, "successes": 1 }
}
```

- `reroll` rola novamente os dados iguais ou inferiores a `upTo`. `once` substitui cada um uma vez e mantém a face nova; `until` continua. `upTo` precisa ficar abaixo da face máxima ou rolaria a reserva inteira para sempre. O Engine limita quantos dados um teste pode rolar novamente, independentemente do arquivo.
- `dice` adiciona dados antes da rolagem; `successes` adiciona sucessos depois da contagem; `threshold` fixa o alvo por dado para aquela rolagem, dentro da faixa permitida por `target`.
- `explode` e `double` movem as faces dessas regras para aquela rolagem, e só quando sua regra tem um `min` e dentro dele: `"check": { "explode": 8 }` é uma entrada que faz os oitos rolarem de novo. Eles prevalecem sobre o que o GM escreveu na tag, como `threshold`. Capability API 1.37.
- É necessário pelo menos um destes; caso contrário, a entrada não diz nada e é recusada.
- Só `dice-pool` pode aplicar isso; um conjunto `dice-sum` com `mechanics.check` é recusado na importação.

**O custo é o `cost` da entrada**, pago pelo mesmo mecanismo de qualquer outro uso: a reserva e um uso de cada contador que essa entrada escreveu. `perCostStep` indica que ela ESCALA: se o declara, é comprada tantas vezes quantas seu preço foi pago; caso contrário, é comprada uma vez, independentemente do valor oferecido.

**O GM a nomeia no teste:** `[skill_check: skill="Brawl" dc="3" use="Potence" spend="blood:3"]`. Não usa um comando de ficha separado pelo mesmo motivo: os dados rolam antes da atualização dos registros.

**Tudo ou nada.** Se a reserva não cobre o custo, a compra não acontece e nada é descontado; a rolagem continua exatamente como seria. Pontos que não equivalem a um número inteiro de compras também não compram nada. Pedir mais que `perCheck` é limitado, não recusado; só o teto é pago. O Engine calcula tudo. O GM nomeia o que o jogador disse que gastaria e nunca mexe nos dados. O registro informa o que foi realmente pago, a entrada aplicada, os sucessos não rolados e quantos dados foram rolados novamente. Um amuleto não escolhido, ou cujo catálogo o Engine não consegue ler, não faz nada em vez de ser aplicado por confiança.

### A ficha

- `sections` agrupa elementos no editor. Campos, valores derivados e listas nomeiam a seção em que ficam com `section`, e atributos, perícias e salvaguardas também podem fazer isso: o editor, a ficha da partida e o bloco de ficha do GM os mostram sob esse título (uma ficha que não nomeia seção neles os mostra exatamente como antes). Uma seção também pode ter uma regra `untrained` (abaixo).
- `abilities` contém as pontuações principais. `skills` e `saves` podem nomear o atributo de suas rolagens, um `cap` e uma regra `untrained` (abaixo).
- `fields` contém valores individuais. Tipos: `number`, `text`, `longtext`, `boolean`, `enum` (lista fixa de escolhas) e `dice` (texto como `1d8`).
- Valores `derived` são calculados a partir de outros e não podem ser sobrescritos na digitação. Operações: `sum`, `min`, `max`, `scale` (multiplicar e arredondar), `stepTable` (consultar limiares, como quando um nível define proficiência) e `enumTable` (um número para cada valor de um campo enum ou de um estado vivo; veja Estados vivos abaixo).
- `lists` contém tabelas com colunas próprias, como equipamento, magias ou características. Uma lista com `pools` transforma cada linha num recurso com máximo próprio, para características de classe com usos limitados.
- `live` é o que muda durante a partida: `pools` (pontos de vida, espaços de magia, Grit), `tracks` (número numa escala, como exaustão, ou trilha de ferimentos com casas), `text` (notas curtas, como concentração), `conditions` e `states` (um valor de um conjunto fechado, como uma forma ou uma postura).

Tudo que lê um número o nomeia com uma referência de valor: um objeto com exatamente uma chave entre `const`, `field`, `derived`, `abilityScore`, `abilityMod`, `abilityModFromField`, `skillMod`, `saveMod`, `listSum`, `livePool`, `liveTrack` e `itemStat`. Por exemplo, uma reserva cujo máximo é derivado: `"max": { "derived": "grit_max" }`.

- `listSum` soma uma coluna numérica de uma lista: `{ "listSum": { "list": "gear", "column": "bulk", "onlyWhen": "packed" } }`. `onlyWhen` é opcional e nomeia uma coluna booleana; só contam as linhas em que ela está ativada. Uma célula vazia conta como o padrão da coluna, e uma lista ocultada por `hideWhen` não soma nada. Ember Roads calcula Burden assim, a partir do equipamento que o personagem leva na mochila, então ele nunca se desencontra da lista como um número digitado poderia. Uma coluna escalada de catálogo não pode ler uma soma de lista, nem por meio de um valor derivado: uma lista pode conter células escaladas, próprias ou que a leem de volta, e o recálculo nunca se estabilizaria.
- `livePool` lê o que resta numa das suas `live.pools` (a reserva de uma linha de lista não é uma delas). `liveTrack` lê uma das suas `live.tracks`, com um `read` opcional: `"value"`, onde ela está (o padrão); `"filled"`, quanto está acima do `min`; `"remaining"`, quanto falta para o `max`; ou, só numa trilha de ferimentos, `"penalty"`, a penalidade ativa. Uma reserva ou trilha oculta lê 0. O "Harm still to take" de Gravewatch é `{ "liveTrack": "harm", "read": "remaining" }`.
- Uma leitura viva pega o estado vivo como está quando um teste é rolado, uma luta começa ou o bloco de ficha do GM é escrito. Quando ainda não existe um (no editor de fichas, na revisão de uma importação), lê o estado em que a partida começa: uma reserva cheia ou vazia conforme seu `start`, uma trilha no seu `default`, uma trilha de ferimentos sem marcas.
- Nada calculado antes de existir um estado vivo pode ler um: o `max` de uma reserva ou de uma trilha, o bônus de proficiência ou a coluna escalada ou a escala de um catálogo. Isso vale também por meio de um valor derivado que lê um e por meio de uma perícia limitada por um valor vivo, e a importação nomeia o valor responsável.
- Esses três exigem Capability API 1.39 num conjunto empacotado.
- `itemStat` lê os itens que o personagem tem, como o valor de guard da armadura que veste, e segue a mesma regra de uma leitura viva. Veja [Itens na ficha](#items-on-the-sheet). Capability API 1.52.

**O que um teste faz sem treinamento.** Uma perícia ou salvaguarda em que o personagem não tem treinamento (seu nível é o primeiro) rola normalmente, a menos que o conjunto diga o contrário, com `untrained` na perícia ou salvaguarda ou na seção em que ela fica (a regra da própria perícia ou salvaguarda prevalece):

```json
"sections": [{ "id": "labour", "label": "Labour", "untrained": { "by": -1 } }],
"skills": [
  { "id": "dig", "label": "Dig", "ability": "sinew", "section": "labour", "untrained": "refuse" },
  { "id": "listen", "label": "Listen", "ability": "nerve", "untrained": "harder" }
]
```

- `"normal"`: normalmente (o padrão).
- `{ "by": -3 }`: somado ao número do teste, de -20 a 20: dados numa reserva, um valor fixo numa soma. Faz parte do número que a ficha mostra e que todo teste rola, antes de qualquer `cap`.
- `"harder"`: um passo mais difícil. Numa reserva, o alvo por dado sobe um (e fica dentro da faixa do alvo), então exige um conjunto de reserva cujo alvo possa se mover; em qualquer outro caso é recusado na importação, porque nunca poderia mudar uma rolagem.
- `"refuse"`: o teste não é rolado. O Engine devolve o pedido do GM com `reason="untrained"`, o que o encerra: nada o rola depois, o GM é informado de que o personagem não pôde tentar, e a narração diz isso. O endpoint de teste de perícia responde a esse teste com um 400 (`skill_check_untrained`), e uma ramificação feita a partir dele não mantém nenhuma das duas metades. O editor de fichas não mostra número para ele.
- Um personagem sem ficha não tem treinamento a ler, então nenhuma regra de falta de treinamento se aplica a ele.
- O lembrete do GM lista as regras de falta de treinamento, por seção ou por perícia, para que ele peça testes que o personagem possa fazer.

Seções em atributos, perícias e salvaguardas, e `untrained` em qualquer lugar, exigem Capability API 1.41 num conjunto empacotado.

Uma perícia ou salvaguarda pode ter um `cap`, uma referência de valor que é o máximo que seu teste pode alcançar: uma pontuação que ela não pode superar ou uma trilha que a segura. Gravewatch limita Soothe ao Resolve que resta a um guardião, `"cap": { "livePool": "resolve" }`. O número limitado é o que a ficha mostra e que todo teste rola, e uma troca com `with=` é calculada sobre o número antes do teto e depois limitada de novo. Um teto não pode ler um modificador de perícia ou de salvaguarda, e o valor derivado que ele lê, ou qualquer valor derivado declarado acima desse, também não pode, para que a ficha continue sendo calculada uma única vez, de cima para baixo. Capability API 1.39.

`hideWhen` oculta um campo, um valor derivado, uma lista, uma reserva ou uma trilha comum pelo valor de outro campo, de exatamente uma de três formas: `equals` um valor, `notEquals` um valor (oculto sempre que o campo contém qualquer outra coisa; é assim que algo fica restrito a um tipo de personagem) ou `in` uma lista de valores. Cada valor nomeado precisa ser um que o campo possa conter, ou a regra nunca coincidiria, e isso é verificado um valor por vez; para `notEquals` e `in`, isso inclui a faixa de um campo numérico e o comprimento de um campo de texto. A regra lê o valor que o editor de fichas mostra: um campo não definido é o valor com que uma ficha em branco começa, e um valor enum que o conjunto não oferece mais é o padrão do campo. O arquivo 5e usa `equals` para ocultar espaços de magia de um personagem que não lança magias; Gravewatch mantém o óleo de lanterna fora da ficha de todo guardião que não está na vigília noturna com `"hideWhen": { "field": "watch", "notEquals": "night" }`. Uma camada não pode remover um valor com o qual qualquer uma dessas regras compara. Uma trilha de ferimentos não pode ser ocultada, porque rolagens e lutas a leem seja o que for que a ficha mostre. `notEquals` e `in` exigem Capability API 1.39.

O `max` de uma trilha comum pode ser uma referência de valor em vez de um número, para que uma pontuação do personagem defina até onde ela vai: `"max": { "field": "willpower_rating" }`. Seus `min` e `default` continuam sendo números. Uma trilha de ferimentos com níveis nomeados tem um `max` igual ao seu número de níveis; uma trilha de casas numeradas tem o comprimento do próprio `max` (veja Casas numeradas abaixo). `alwaysShow: true` imprime uma trilha no bloco de ficha do GM mesmo no seu padrão, para uma pontuação que importa em todo turno; sem isso, uma trilha no padrão é omitida, porque três salvaguardas contra a morte em zero não dizem nada. Esses três exigem Capability API 1.37 num conjunto empacotado.

### Estados vivos: uma forma, uma postura, uma luz

Um campo enum é escolhido quando o personagem é criado e continua escolhido, e uma condição só está ativa ou não. Um **estado vivo** é um valor de um conjunto fechado que muda durante a partida: a forma de um metamorfo, uma postura de luta, quão acesa está uma lanterna.

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

- `values` são de dois a quarenta, cada um mostrado como sua entrada em `valueLabels` ou como ele mesmo. Seguem a regra de todo rótulo (uma linha, sem colchetes, sem chaves de macro) e nunca contêm aspas duplas, porque o comando de ficha os coloca entre aspas. Até doze estados.
- `default` é onde toda ficha começa, e o primeiro valor quando você o omite. Só um estado que saiu do padrão é salvo, e um valor salvo que o estado não oferece mais é lido como o padrão.
- `hideWhen` tira um estado da ficha pelo valor de um campo, como faz com uma reserva: ele não é mostrado, não é definido por comando e é lido por uma tabela (abaixo) como nenhum valor.
- O GM define um com `[sheet: who="Name" op="state" state="Stance" value="Reckless"]`, nomeando o estado pelo ID ou rótulo e o valor por ele mesmo ou por seu rótulo. Um estado ou valor que a ficha não tem é recusado (`unknown-state`, `unknown-value`). O Engine ensina o comando e lista cada valor só quando seu conjunto tem um estado, e todo bloco de ficha diz o valor de cada estado, inclusive o padrão, porque estar em forma humana é um fato de que a narração precisa a cada turno.
- A tela de ficha da partida mostra um seletor para cada estado.
- Um descanso restaura um estado com um passo de restauração que o nomeia: `{ "state": "stance", "to": "default" }`, ou `"to"` um dos seus valores.

**Números que acompanham.** Um valor derivado com `"op": "enumTable"` dá um número para cada valor de um campo enum ou de um estado vivo:

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

- `from` nomeia exatamente um entre `field` (um campo enum) e `liveState`. Cada linha de `table` é um valor que ele pode conter, de um a quarenta. Um valor que a tabela omite, e um estado que a ficha oculta, leem `default` (0 quando você o omite).
- Um campo é fixado quando o personagem é criado, então uma tabela baseada nele pode alimentar um máximo: Gravewatch dá um Resolve a mais a um guardião da vigília da aurora. Um estado vivo muda durante a partida, então uma tabela baseada nele é uma leitura viva, como `liveTrack`: nada calculado antes de existir um estado vivo pode lê-la, diretamente ou por meio de outro valor derivado. Por isso a chave é `liveState` e não `state`: toda leitura do estado vivo leva `live` no nome, e é assim que você e a importação sabem o que um máximo não pode usar.
- Para levar o número aos dados, aponte `resolution.adjust` para ele (veja Modificadores da ficha). Ember Roads soma sua Stance a toda rolagem de Brawn; Gravewatch aplica a toda rolagem de Nerve uma penalidade quando a lanterna está fechada ou apagada.
- Uma camada pode retirar de um campo um valor para o qual a tabela tem uma linha. Essa linha simplesmente nunca é lida enquanto a camada estiver ativa.

Estados vivos, `enumTable` e um passo de descanso que nomeia um `state` exigem Capability API 1.42 num conjunto empacotado.

### Trilhas de ferimentos: saúde marcada numa trilha

Muitos sistemas não contam pontos de vida. Têm uma coluna de casas, cada uma pior que a anterior, e você marca uma ao se ferir. Dê a uma entrada `live.tracks` os `kinds` e também `levels` nomeados ou `boxes` numeradas (abaixo) para que ela deixe de ser um número numa escala e vire uma dessas trilhas:

**Qual formato seu sistema precisa?** Uma reserva registra QUANTO dano entrou; uma trilha registra quanto E o tipo de cada parte. Se um ferimento é contundente, letal ou agravado e o tipo ainda importa após o golpe (porque o agravado cura mais devagar, não pode ser absorvido ou acaba matando), precisa continuar registrado após a rolagem. Só uma marca carrega isso. Uma reserva não pode: depois de subtrair o dano, só resta um número menor, sem lembrar quais pontos eram de cada tipo. Por isso `combat.damageKinds` é recusado quando a saúde é uma reserva, em vez de ignorado. Uma reserva ainda pode ter `damageTypes`, e oponentes podem resistir ou ser imunes a eles: isso trata de quanto dano entra, não de qual ferimento fica depois.

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

- `levels` tem 1–16 patamares, do melhor ao pior. Cada um tem `label` e `penalty` inteiro igual ou inferior a 0. Um número negativo grande expressa ficar fora de ação, então `-99` é válido.
- `kinds` tem 1–6 tipos de dano aceitos, cada um com `id`, `label` curto para a casa e `severity`. As gravidades precisam ser distintas; só a ordem importa, não os números em si, então distribua-os como quiser.
- Os dois andam juntos. `kinds` sem `levels` ou `boxes` é recusado porque não haveria onde marcar, e `levels` ou `boxes` sem `kinds` é recusado porque uma marca precisa ter um tipo. Uma trilha tem `levels` ou `boxes`, nunca os dois.
- **Separe os dois termos.** `kinds` define o que uma marca PODE SER. Uma MARCA é um desses tipos colocado na trilha durante a partida. A definição contém tipos; a ficha contém marcas.
- Uma trilha de ferimentos começa sem marcas, então seu `min` é 0. Uma trilha com níveis nomeados tem o comprimento dos seus níveis, então seu `max` é `levels.length`; qualquer outro valor é recusado em vez de corrigido, para que o arquivo nunca contenha dois comprimentos contraditórios. Uma trilha de casas tem o comprimento do próprio `max`, que pode ser um valor calculado pela ficha (veja Casas numeradas abaixo).

**As regras exatas**, porque uma leitura vaga produz a trilha errada:

- As marcas ficam ordenadas, **as mais graves primeiro**. Sete níveis comportam no máximo sete marcas.
- Uma marca é **inserida por ordem de gravidade**, nunca simplesmente no fim. Ocupa o nível mais alto que lhe cabe e empurra as leves para baixo.
- A penalidade ativa é a do **nível marcado mais baixo**, nunca a soma. Três marcas no exemplo dão `-3`, não `0 + -1 + -3`.
- `amount` é a quantidade de marcas de um tipo, **aplicadas uma por vez**. Uma trilha que enche no meio do processo segue a mesma regra de uma que já estava cheia.
- Marcar uma trilha **cheia** **aumenta um passo a gravidade da marca mais leve**, em vez de adicionar outra. Um passo na sua escala de tipos, qualquer que fosse o tipo da marca nova.
- Uma marca que passaria da gravidade máxima fica nela; a que não coube conta como **transbordamento**. Isso é salvo, para que recarregar não esqueça dano já recebido.
- **Curar usa o mesmo comando com quantidade negativa.** Limpa o transbordamento antes de qualquer marca. Uma cura que nomeia um dos tipos da trilha limpa só marcas desse tipo; assim, curar o dano contundente e deixar o letal é `kind="bashing"` com quantidade negativa. Uma cura que não nomeia nenhum tipo da trilha limpa primeiro as marcas mais leves.

**Marcar durante a partida.** O GM escreve `[sheet: op="damage" track="harm" kind="knock" amount="1"]` e cura com `amount` negativo. Numa trilha preenchida por casa, ele adiciona `box="3"` para indicar a casa que o golpe atinge. A forma de reserva de `damage`, que nomeia `pool=`, não muda. O comando comum `track` é recusado em trilhas de ferimentos: um número não identifica os tipos das novas marcas. O jogador também pode marcar e limpar casas manualmente na ficha, como esses sistemas esperam.

**Uma luta também pode marcá-la.** Aponte `combat.health` para a trilha em vez de uma reserva: um golpe acertado marca as casas indicadas por `combat.damageKinds.marks`, do tipo ao qual esse bloco mapeia seu tipo de dano. Um personagem com a trilha cheia está caído, o estado lido pela regra de morte. Numa trilha preenchida por casa, o dano `per-point` nomeia a casa que atinge e o `per-blow` mira a primeira, e um golpe que nenhuma casa pode receber derruba o personagem, que é o que ser posto fora de combate significa nos sistemas que usam esse tipo de trilha. Cura limpa uma marca, a mais leve, de qualquer tipo. Pontos temporários são recusados porque a trilha não tem um espaço para eles. O Engine lê a trilha como níveis RESTANTES; o restante da luta, cair, reviver, o registro e o resumo continuam iguais.

**Um descanso pode curar a trilha.** Uma restauração com `"to"` a reduz àquela quantidade de marcas, incluindo transbordamento; com um `"by"` NEGATIVO, limpa aquela quantidade, começando pelo transbordamento, então `"by": { "const": -2 }` limpa duas. Um passo que também nomeia um `kind` limpa só marcas desse tipo: o "Catch your breath" de Gravewatch é `{ "track": "harm", "kind": "knock", "to": "min" }`, que cura as pancadas e deixa os rasgos. Um passo que ADICIONARIA marcas não faz nada, porque um descanso não marca dano, e isso inclui um `"by"` positivo: ele é ignorado sem aviso, então escreva o sinal de menos.

#### Casas numeradas em vez de níveis nomeados

Quando uma trilha tem o comprimento da resistência do personagem, dê a ela `boxes` em vez de `levels`. Ela tem tantas casas quanto o próprio `max`, que pode ser um valor calculado pela ficha (nunca um que leia o estado vivo), até 64:

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

- A penalidade ativa é lida em `table` (uma tabela de patamares, como a de um valor derivado, com penalidades iguais ou inferiores a 0) no número de casas `filled` ou `remaining`, seja qual for esse número, então uma tabela pode custar algo sem nada marcado. Ela não depende de quais casas estão marcadas.
- As casas são numeradas: a ficha mostra "Box 3", e o bloco de ficha do GM diz quantas estão marcadas de quantas, em vez de nomear um nível.

#### Preencher por casa e trilhas que recusam

- `"fill": "indexed"` coloca uma marca na casa que o comando nomeia com `box=` (a primeira casa, quando não nomeia nenhuma) ou na próxima casa livre acima dela, e nunca move uma marca depois de colocada. Uma cura limpa a mais alta das marcas mais leves. A tela de ficha permite ao jogador clicar em qualquer casa livre.
- `"onFull": "refuse"` recusa uma marca que não tem casa livre, em vez de aumentar a gravidade da marca mais leve. O comando inteiro é recusado, então três marcas com duas casas livres não colocam nenhuma. Uma trilha indexada sempre recusa; declare isso com `"onFull": "refuse"` ao lado.

#### Níveis que uma lista adiciona

`"extra": { "list": "scars", "countColumn": "levels", "penaltyColumn": "penalty" }` numa trilha com níveis nomeados permite que uma lista da ficha a alongue por personagem. Cada linha, em ordem, adiciona sua quantidade de níveis (no máximo 16) com sua penalidade, depois do último nível cuja penalidade seja pelo menos tão boa, com o nome desse nível quando as penalidades coincidem e com o da penalidade caso contrário. As cicatrizes de Gravewatch funcionam assim. Uma lista que a ficha oculta não adiciona nada e, se uma linha for excluída enquanto seus níveis estão marcados, as marcas que não cabem mais ficam como transbordamento: editar a ficha não desfaz dano.

`boxes`, `fill`, `onFull`, `extra` e o `kind` de um passo de descanso exigem Capability API 1.40 num conjunto empacotado.

### A penalidade nas rolagens

`resolution.penaltyFrom` nomeia a trilha de ferimentos cuja penalidade afeta todo teste do conjunto. É declarada, não presumida: sem ela, as rolagens continuam como antes da existência das trilhas.

O EFEITO da penalidade depende do tipo de resolução, assim como o número da ficha:

- Em `dice-pool`, ela **retira dados da reserva**, sem baixar de `pool.min`. Um `pool.min` de 1 permite rolar um dado mesmo no último patamar; um `pool.min` de 0 permite não rolar nenhum e falhar sem lançamento.
- Em `dice-sum`, é um **modificador fixo na rolagem**, incorporado ao mesmo número ao qual atributo e treinamento já contribuem.

Ela precisa nomear uma trilha de ferimentos. Uma trilha comum não tem penalidade e é recusada na importação. O resultado indica a penalidade aplicada para explicar ao jogador seus dados a menos; o bloco de ficha do GM também mostra o patamar e seu custo.

### Modificadores da ficha

Alguns números da ficha acompanham toda rolagem que afetam: uma mochila pesada em toda escalada, uma penalidade de armadura em toda tentativa de furtividade, uma bênção em tudo. `resolution.adjust` declara isso, nos dois tipos:

```json
"adjust": [{ "value": { "derived": "burdened" }, "abilities": ["brawn"] }]
```

- `value` é um valor da ficha, na mesma forma que um valor derivado lê, incluindo o estado vivo. Um número negativo subtrai. Ember Roads soma o volume do equipamento que o personagem leva na mochila num Burden, e uma tabela de patamares transforma cada três pontos disso em um a menos em cada rolagem de Brawn. Uma tabela baseada num estado vivo é como uma forma ou uma postura chega aos dados (veja Estados vivos).
- `abilities` é opcional. Com ele, o modificador só se aplica a um teste rolado com um desses atributos: o próprio atributo, uma perícia ou salvaguarda que o usa, uma perícia rolada com ele por `with=` ou um par de atributos que o inclui. Sem ele, aplica-se a todo teste.
- É aplicado onde a penalidade de ferimentos é aplicada: dados numa reserva, sob o mesmo piso `pool.min`, e um número fixo numa soma, dentro do modificador que o registro soma. Até oito entradas, somadas em cada teste.
- Uma luta do conjunto o soma a toda rolagem que monta a partir de uma ficha, quando a luta começa: um ataque rolado com um atributo ou perícia, uma salvaguarda, um teste de disputa e a iniciativa lidos de um atributo, perícia ou salvaguarda recebem as entradas daquele atributo, e qualquer outra rolagem recebe só as entradas que valem para toda rolagem.
- Um personagem sem ficha não recebe nada disso, assim como não recebe modificador.

O resultado informa o que a ficha adicionou (`adjust="-2"` no registro e uma linha no cartão de dados), para que o jogador veja por que a rolagem saiu assim. Exige Capability API 1.38 num conjunto empacotado.

### Descansos

Um descanso lista restaurações e elementos a limpar. Cada passo nomeia um alvo (`pool`, `poolGroup`, `listPools`, `track` ou `state`) e o define (`"to": "max"`, `"to": "min"` ou um número) ou modifica (`"by": { "const": 1 }` ou `"by": { "fractionOfMax": 0.5 }`). Se nomeia uma trilha de ferimentos, só pode curá-la, então seu `by` é negativo; veja acima. Se nomeia um estado vivo, só pode defini-lo, como `"default"` ou um dos seus valores.

### Texto do Game Master

- `checkGuidance` substitui o parágrafo integrado que ensina a pedir um teste. Informe o sistema e quando rolar. O GM só nomeia perícia e dificuldade. O Engine rola e calcula a partir da ficha: não peça matemática ao modelo.
- `sheetGuidance` apresenta as fichas no prompt. Explique quais recursos importam e quando gastá-los.
- `worldGuidance` é opcional e lido uma vez ao gerar o mundo, para adequar a ambientação às regras: sem pólvora, magia rara, mortos que andam. Nunca chega a um turno.
- `sheetSummary` escolhe campos, valores derivados e linhas de listas que o GM vê por personagem. O Engine sempre mostra modificadores de atributo, perícias e salvaguardas treinadas e valores vivos. Mantenha o restante curto, pois é enviado a cada turno. O `nameColumn` de uma lista do resumo pode ser uma coluna de texto ou enum (um enum mostra o rótulo do seu valor), e `columns` nomeia até mais três colunas da própria linha para imprimir depois do nome, de modo que o bloco diga `Gear: Crowbar 1d6` em vez de mostrar só nomes. Uma coluna booleana imprime seu rótulo quando está ativada. Ambos exigem Capability API 1.37 num conjunto empacotado.
- Você não precisa ensinar o comando de ficha em `sheetGuidance`. O Engine ensina cada comando sozinho, com os nomes que seu arquivo declara, e, quando seu conjunto tem uma trilha de ferimentos, adiciona o comando `op="damage"` para marcá-la e lista seus níveis e tipos de dano. Quando ele tem estados vivos, adiciona o comando `op="state"` e lista os valores que cada um pode assumir.

## Catálogos: entradas prontas para as listas da ficha

Digitar uma lista de magias, tabela de equipamento ou página de características linha por linha é cansativo. Um catálogo é uma coleção nomeada de entradas prontas distribuída com o conjunto. O editor as oferece num seletor em cada lista alimentada pelo catálogo; escolher uma preenche a linha.

Catálogos são opcionais. Um conjunto pode ter até doze; o Engine não conhece o assunto deles: cada ID, coluna, filtro e palavra vem do arquivo.

### O cabeçalho

O cabeçalho fica em `catalogs`, no nível superior do arquivo, ao lado de `gm`.

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

- `id` e `label`: o ID segue as regras de IDs da ficha; o rótulo dá nome ao seletor.
- `holds`: `"rows"` (padrão, usado por todos os catálogos anteriores a esta versão), `"creatures"` ou `"items"`. Um catálogo de criaturas é um bestiário lido pela luta, e um catálogo de itens lista o que um grupo carrega. Nenhum dos dois escreve em fichas, declara `feeds` ou aparece no seletor. Veja [Criaturas](#creatures-a-bestiary-a-fight-reads) e [Itens](#items-what-a-party-carries) abaixo.
- `feeds`: de uma a oito listas da ficha onde as entradas podem escrever. Obrigatório num catálogo de linhas e recusado num catálogo de criaturas ou de itens. Uma entrada não pode escrever em outra lista nem colocar um valor que suas colunas não aceitam.
- `filters`: opcional, até oito. Define como filtrar o seletor: `number`, um valor `text` ou `tags` (várias palavras). `startFrom` nomeia um campo usado ao abrir: um personagem cujo Calling é Tinker vê primeiro entradas Tinker.
- `units`: opcional. Define o significado de alcance ou tamanho de área no bloco `mechanics` de uma entrada.

### Uma entrada

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

- `id`: letras minúsculas, dígitos e hífens simples, único no catálogo.
- `label` e `summary`: o que o seletor mostra. O resumo é opcional, numa linha e com até 300 caracteres.
- `filters`: valores dos filtros declarados no cabeçalho. `number` recebe um número; `text`, uma string; `tags`, uma lista de strings.
- `rows`: o que a escolha escreve, de uma a seis linhas. `list` pertence a `feeds`; as chaves de `values` são os IDs de colunas dessa lista.
- `creature`: um oponente em vez de linhas, num catálogo cujo `holds` indica criaturas. Cada entrada tem exatamente um entre `rows` e `creature`. Uma criatura não contém `mechanics`: informa seus efeitos nas próprias ações.

Cada valor é verificado contra as colunas de destino. Nome incorreto ou número fora da faixa é informado com sua entrada de origem. Entradas incluídas no arquivo de regras são verificadas ao carregá-lo, o que significa durante a importação de um arquivo importado. Um catálogo separado de pacote é verificado na primeira solicitação do seletor; se tiver erro, mostra os motivos em vez das entradas.

### Uma entrada, várias listas

Uma característica com usos limitados ocupa duas linhas: a característica e seu contador. Continua sendo uma única escolha.

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

### Valores mantidos atualizados pelo conjunto

Os números de uma linha pertencem ao jogador depois da escolha. Há uma exceção útil: máximos que acompanham o personagem, como usos iguais a um atributo ou um recurso de classe que cresce com o nível. Uma linha pode nomear até quatro colunas numéricas próprias num mapa `scaled`; o editor mantém essas células atualizadas.

```json
{
  "list": "tricks",
  "values": { "name": "Last Ember", "uses": 1, "recharge": "camp" },
  "scaled": { "uses": { "from": { "abilityScore": "heart" } } }
}
```

- A chave é uma das colunas `number` da lista.
- `from` é uma referência de valor comum, com o mesmo vocabulário fechado usado no restante. Para algo mais complexo, declare um valor `derived` e aponte `from` para ele (`"from": { "derived": "lay_on_hands_max" }`). Nenhuma aritmética nova é adicionada aqui.
- `table` é opcional. Consulta a referência numa tabela por patamares, como quando um nível define um número: `"scaled": { "max": { "from": { "field": "level" }, "table": [[1, 2], [3, 3], [6, 4]] } }`.
- `values` ainda precisa conter um número comum para a coluna; omiti-lo é recusado. Esse é o valor antes de conhecer a ficha e o que uma ficha sem a referência mantém.
- Uma linha com `scaled` deve ser a única da entrada para aquela lista; assim, uma linha marcada sempre corresponde a uma única especificação.

O valor é calculado ao editar, nunca ao ler: uma linha salva sempre contém o número que declara. Ele é ajustado à coluna: limitado entre `min` e `max` e arredondado para baixo quando a coluna usa inteiros. No exemplo, Heart 3 dá três usos; Heart 0 ou menos não dá nenhum. A linha permanece com 0 usos; como um contador com máximo 0 não é uma reserva, não há nada para gastar em jogo.

Colunas escaladas exigem Capability API 1.23 para conjuntos empacotados. Um conjunto comunitário importado é validado pelo Engine que o lê, sem precisar de mais nada.

### Linhas escolhidas são cópias

Cada linha escolhida é copiada para a ficha com uma chave extra, `_catalog`, contendo `<catalog id>/<entry id>`. IDs de colunas começam com letra, então essa chave nunca pode ser uma das suas.

A cópia pertence ao personagem. O jogador pode editá-la, a ficha continua funcionando sem o conjunto instalado e publicar uma versão nova nunca reescreve personagens. A marca permite ao seletor mostrar o que a ficha já tem e à atualização descrita abaixo reconhecer a origem.

### Atualizar a partir do conjunto

Graças à marca, o editor pode avisar quando seu texto novo difere da linha. Uma mensagem curta sob a lista informa quantas linhas têm texto novo; **Review** (Revisar) mostra o conteúdo da ficha ao lado do conjunto, com uma caixa por linha. Nada é escrito antes de **Update selected** (Atualizar selecionadas); só as colunas diferentes das linhas marcadas mudam. Todo o restante é preservado, inclusive a marca.

A comparação é deliberadamente restrita:

- Valores existentes só são comparados em colunas `text`, `longtext`, `dice` e `enum`. Valores `number` e `boolean` pertencem ao jogador e são preservados, inclusive 0 e false. Uma coluna ainda ausente pode ser oferecida com seu valor tipado, incluindo números e interruptores. Uma coluna escalada é excluída porque já acompanha a ficha.
- Só as colunas definidas pela entrada são comparadas. Uma coluna omitida nunca é tocada, qualquer que seja seu conteúdo.
- Um valor recusado pela própria coluna, como um `enum` retirado ou texto acima de `maxLength`, é ignorado em vez de escrito.
- A linha corresponde à de origem pela posição entre as linhas com a mesma marca naquela lista, enquanto a ficha mantiver tantas linhas quanto a entrada escreve. Caso contrário, só funciona se a entrada escrever uma única linha nessa lista. Se o jogador excluir uma de duas linhas, a entrada fica intacta em vez de ser adivinhada.
- Uma linha cuja entrada não existe mais no catálogo fica intacta, silenciosamente.

Assim, reformular ou renomear pode chegar aos personagens que já escolheram a entrada, se aceitarem. Mudar o significado de um número não pode e não vai: essa coluna pertence ao jogador desde que a linha é dele.

### `mechanics`: o que uma entrada faz em números

Uma entrada pode conter um bloco `mechanics` opcional para expressar efeitos numéricos: `kind` (`attack`, `heal`, `buff`, `debuff`, `utility`, `rider`), `range`, `area`, `targets`, `targetCount`, `friendlyFire`, `amount` (dados como `2d6` ou número fixo), `damageType`, `attackRoll`, `autoHit`, `save` (uma salvaguarda da ficha e o efeito do sucesso), `applies` (condições aplicadas), `temporary` (pontos temporários de saúde), `scales` (quantidade que cresce com a ficha), `cost` (reserva gasta), `perCostStep`, `budget` (parte da economia de ações gasta), `concentration`, `reaction`, `plus`, `free`, `gives`, `standard`, `rider` e `check`.

O seletor mostra o bloco numa linha. Quem lê o restante depende do bloco escolhido pelo conjunto:

- Com um [bloco `combat`](#combat-a-fight-your-own-rules-resolve), a luta lê seus efeitos de combate. `range`, `area` e `friendlyFire` valem em campos com posições; `reaction` marca uma entrada que responde a algo, em vez de ser usada num turno, e nomeia o momento que ela espera (veja [Janelas](#windows-holding-the-fight-open)); uma entrada marcada apenas com `true` não nomeia nenhum momento e não aparece em nenhum menu. `check` afeta testes de perícia, como explicado acima.
- Só com um [bloco `battle`](#battles-lending-the-sheet-to-marinaras-combat), a batalha lê `kind`, `range`, `area`, `friendlyFire`, `amount`, `damageType` e `cost`, pois são as partes que o combate próprio do Marinara consegue aplicar.

O vocabulário é fechado: uma chave ou valor fora da lista é recusado em vez de ignorado.

`cost` também é o preço pago pelo comando `use` do GM fora da batalha, descrito a seguir.

### O comando `use`: deixar o Game Master pagar um preço que você escreveu

Enquanto narra, o GM atualiza fichas com comandos `[sheet: ...]`: `spend`, `restore` (`heal` tem o mesmo sentido), `damage`, `temp`, `track`, `condition`, `note` e `rest`. Um conjunto com catálogos recebe mais um:

```
[sheet: who="Mira" op="use" name="Fireball"]
[sheet: who="Mira" op="use" name="Fireball" pool="3rd-level slots"]
```

`op="cast"` equivale a `op="use"` e `spell=` a `name=`; assim, a expressão escolhida pelo GM funciona sem o formato precisar conhecer a palavra "spell".

O nome é comparado sem distinguir maiúsculas às linhas do personagem vindas dos seus catálogos. Uma linha responde ao nome mostrado ao GM (coluna de nome de `sheetSummary`, depois `pools.nameColumn`, depois a primeira coluna de texto) e ao `label` da entrada de origem. Renomear uma linha não faz perder seu acesso. Nomes sem correspondência ou compartilhados por duas entradas diferentes são recusados.

O que é gasto:

- Cada termo de `mechanics.cost`. Um termo que nomeia uma reserva viva paga dela; um que nomeia um GRUPO paga da primeira reserva do grupo capaz de cobrir o custo, na ordem de declaração. Não há subida automática para uma reserva superior, porque um grupo nem sempre é uma escala.
- Mais um de cada reserva de linha escrita pela mesma entrada, como o contador de usos de uma característica: a segunda linha de `Last Ember`. Um contador com máximo 0 não tem usos; o comando é recusado em vez de executado de graça.

`pool=` é o lançamento em nível superior: o mesmo preço único, pago de outra reserva do grupo. Só é aceito se o custo tiver um único termo e a reserva nomeada compartilhar seu grupo. Qualquer outra situação é recusada sem reinterpretar.

É tudo ou nada. Se qualquer parte não puder ser paga, o comando inteiro é recusado, nada muda e o jogador é avisado. Uma entrada sem custo, como um truque ou característica passiva, é aceita sem mudar nada.

### Dentro do arquivo ou num arquivo próprio

Um catálogo pequeno fica dentro de `ruleset.json`, em `entries` do cabeçalho. Um longo fica num arquivo próprio, nomeado por `asset` no cabeçalho. Cada catálogo tem exatamente uma dessas opções.

```json
{ "id": "knacks", "label": "Knacks", "feeds": ["knacks"], "asset": "catalogs/knacks.json" }
```

O caminho sempre é `catalogs/<the catalog's id>.json`. O arquivo tem este formato:

```json
{ "schemaVersion": 1, "catalog": "knacks", "entries": [] }
```

Arquivos separados são para pacotes publicados pelo catálogo oficial: o pacote os lista em `contributions.assets.paths` junto a `ruleset.json` e precisa de Capability API 1.21. Um catálogo de criaturas, incluído ou separado, precisa de Capability API 1.27, e um catálogo de itens precisa de 1.49. **Um conjunto importado como arquivo único ou compartilhado pelo GitHub contém os catálogos dentro do arquivo**, então precisam caber no limite total de 256 KB. Isso comporta algumas centenas de entradas curtas.

Os limites são doze catálogos por conjunto, 2000 entradas por catálogo em ambos os formatos e 1 MB por arquivo de catálogo.

<a id="items-what-a-party-carries"></a>

## Itens: o que um grupo carrega

Armaduras, armas, poções, equipamentos, munição e dinheiro são itens. Um bloco opcional `items` declara o vocabulário em que todo item do seu conjunto é escrito, e um catálogo com `holds: "items"` lista os próprios itens. Ambos exigem Capability API 1.49; `rarityCaps` e `propose` do bloco, que regem os itens que o Game Master inventa, exigem 1.51; um valor que lê os itens de um personagem (`itemStat`) exige 1.52; o que um item faz com os testes enquanto vestido ou carregado, com os tetos de `bonus` do bloco, exige 1.53; o que um item exige de quem o veste e os atributos que ele altera exigem 1.54; o `attack` de uma arma exige 1.55; o que um item faz numa luta exige 1.56; o que uma arma dispara e mantém carregado exige 1.57; os modos de uma arma, o ataque com a mão inábil, o piso e as condições num acerto exigem 1.58; o que usar um item faz numa luta, com as cargas que ele contém, exige 1.59; um uso que restaura uma reserva exige 1.60; cargas recuperadas num descanso ou um item que quebra ao se esvaziar exigem 1.61; um uso que pede um teste antes (`gate`) exige 1.62; tabelas de loot e o loot de uma criatura exigem 1.63; uma linha de loot que deixa cair moedas ou uma camada que retira moedas exige 1.64; um mercado, o lugar `sold` de um item e um `service` exigem 1.65; um dia de mercado (um caixa, `buyBack`, o `stock` de um vendedor, uma contagem `limited` ou um `sold` que não nomeia lugar) exige 1.69; e um efeito adicional que dispara com armas empunhadas (`weapons`) exige 1.70.

### O bloco de itens

O bloco fica em `items`, no nível superior do arquivo. Este é o de Ember Roads, um pouco resumido:

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

- `categories`: de uma a 24. Todo item tem exatamente uma, como armadura, arma ou poção.
- `rarities`: opcional, até 12. Liste-as da mais comum à mais rara.
- `tags`: opcional, até 48. Propriedades que um item pode ter, como arremessável ou prateado.
- `stats`: opcional, até 24. Os números e palavras que um item carrega, declarados exatamente como uma coluna de lista: `number`, `text`, `boolean`, `enum` ou `dice`, com faixa, valores e padrão quando o tipo os tem. `promptVisible` (padrão `true`) diz se o Game Master vê a estatística ao lado do item. Defina como `false` para algo que só o jogador deve ver, como o lugar onde um item está escondido.
- `slots`: opcional, até 12. Onde um item é vestido ou empunhado, e quantos desse espaço um personagem tem, de 1 a 20.
- `binding`: opcional. Sintonia, investidura ou qualquer outra coisa que limite quantos itens um personagem pode ter vinculados ao mesmo tempo. `label` é como isso se chama, e `max` é um valor lido da ficha do personagem, como `{ "abilityScore": "nerve" }` ou um valor derivado. Não pode ler o estado vivo, porque a quantidade de itens que um personagem pode vincular não muda a cada golpe.
- `carry`: opcional. `stat` nomeia a estatística numérica que é o peso de um item, cujo `min` é 0 ou mais. `encumberedAbove` é quanto um personagem carrega antes de ficar sobrecarregado, e `limit` (opcional), o máximo que ele consegue carregar. Ambos são lidos da ficha como `binding.max`. Sem `carry`, o peso não significa nada e ninguém fica sobrecarregado.
- `currencies`: opcional, até seis famílias de uma a dez moedas cada.
  - As moedas de uma família se convertem umas nas outras pelo `value`, que é contado na menor moeda da família. Assim, a menor moeda vale 1, e duas moedas da mesma família nunca valem o mesmo.
  - Duas famílias nunca se convertem uma na outra. A moeda de uma segunda nação, ou os favores de um cenário, formam uma família própria.
  - O ID de uma moeda é único entre todas as famílias, porque o custo de um item nomeia só a moeda.
  - `perWeight` (opcional, e só ao lado de `carry`) é quantas moedas da família pesam uma unidade da estatística de peso.
  - As moedas são o dinheiro do grupo, levadas nas bolsas como qualquer item (veja Dinheiro, abaixo).
- `rarityCaps`: opcional, no máximo um por raridade. O máximo que um item inventado pelo Game Master pode conceder naquela raridade: o maior valor de cada estatística numérica nomeada em `stats`, dentro da própria faixa da estatística, e um número inteiro para uma estatística que só aceita inteiros, e `bonus` (Capability API 1.53), o maior bônus fixo que um dos seus modificadores de item vestido ou carregado pode adicionar. Numa raridade com `bonus`, um bônus em dados é omitido, já que dados não podem ser limitados a um número; uma penalidade nunca é limitada. Um item inventado fica sujeito a isso; os itens que seus catálogos listam são seus e nunca são limitados.
- `propose`: `true` por padrão. `false` proíbe o Game Master de inventar itens do seu conjunto.
- `native`: `true` por padrão. `false` desativa os itens sem tipo próprios do Game Mode nas partidas do seu conjunto (veja **O que lê os itens**). O Game Master ainda pode inventar itens, escritos no vocabulário do seu conjunto.
- `freeform`: o que vira um item que o jogador digita. `"plain"` (o padrão) o mantém como um item sem regras, como hoje. `"refuse"` permite só itens do seu conjunto.
- `lootTables` (Capability API 1.63): opcional, até 24. O que as criaturas de uma luta vencida e o `[loot:]` do Game Master deixam cair (veja Loot, abaixo).

### Um item

Um catálogo de itens não declara `feeds`:

```json
{ "id": "outfitter", "label": "Outfitter", "holds": "items", "entries": [] }
```

Cada entrada contém um `item` em vez de `rows` ou de uma `creature`:

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

- `id`, `label`, `summary` e `filters` funcionam como em qualquer entrada. O rótulo é o nome do item.
- `category` é obrigatório. `rarity` e `tags` são opcionais, e os três nomeiam IDs do próprio bloco. Um item só pode ter raridade quando o bloco declara alguma.
- `stats`: valores para as estatísticas do bloco. Cada um é um valor que sua estatística poderia conter.
- `slots`: quantos de cada espaço o item ocupa, nunca mais do que um personagem tem.
- `stack`: o máximo que uma pilha contém, de 1 a 999.999. Sem isso, uma pilha contém tanto quanto qualquer pilha do Game Mode. Um item que tem `charges` fica sozinho na pilha.
- `cost`: um `amount` inteiro de uma moeda, nomeada pelo ID em `unit`. O seletor o mostra, e o Game Master o vê ao lado do item (veja Dinheiro, abaixo).
- `sold` (Capability API 1.65): `{ "place": "city" }`, o menor dos lugares do seu mercado que vende o item, acima de qualquer regra que diga o contrário; com 1.69, ele pode, em vez disso ou além disso, dizer quantos um lugar mantém disponíveis num dia (`"limited": 2` ou `"limited": "1d3"`) (veja Mercados, abaixo).
- `service` (Capability API 1.65): `true` para hospedagem, passagem, uma bênção: comprado como um item e nunca carregado. Comprá-lo só paga; nada mais coloca um serviço numa bolsa (adicionar um é recusado como `service`), e o seletor deixa os serviços de fora. Um serviço tem um `cost` e nada relacionado a carregar ou usar (`slots`, `stack`, `binds`, `worn`, `carried`, `requires`, `attack`, `use`, `charges`).
- `binds`: o item precisa ser vinculado antes de fazer qualquer coisa enquanto vestido. `restriction` (opcional) diz em palavras quem pode vinculá-lo, e `cursed: true` marca um item que não se solta. Só um conjunto com `binding` pode ter itens que se vinculam.
- `worn` e `carried` (opcionais, Capability API 1.53): o que o item faz com os testes e as salvaguardas de quem o porta enquanto está vestido e enquanto está só carregado, (1.54) os atributos que ele define ou aumenta e (1.56) o que ele faz numa luta. Veja [Testes fora de uma luta](#checks-outside-a-fight) e [Armaduras e efeitos de itens vestidos numa luta](#armor-and-worn-effects-in-a-fight).
- `requires` (opcional, Capability API 1.54): o que o item exige de quem o veste e o que se aplica enquanto essa pessoa não atende à exigência. Veja [Testes fora de uma luta](#checks-outside-a-fight).
- `attack` (opcional, Capability API 1.55): o que o item faz como arma numa luta, enquanto vestido, (1.57) o que ele dispara e mantém carregado e (1.58) seus outros modos, um ataque com a mão inábil, um piso para o dano e as condições que impõe num acerto. Veja [Armas numa luta](#weapons-in-a-fight), [Munição e recarga](#ammunition-and-reloading) e [Modos, uma segunda arma, um piso e condições num acerto](#modes-a-second-weapon-a-floor-and-conditions-on-a-hit).
- `use` e `charges` (opcionais, Capability API 1.59): o que usar o item faz, numa luta e fora dela, as cargas que ele contém para esse uso gastar e (1.60) uma reserva que ele restaura. Veja [Usar itens numa luta](#using-items-in-a-fight) e [Usar itens fora de uma luta](#using-items-outside-a-fight).

Um item não tem `mechanics`: o que ele faz é escrito no seu bloco `item`.

### O que lê os itens

Tudo acima é verificado quando o conjunto é importado, e seus catálogos de itens são distribuídos com ele. Numa partida, o inventário do Game Mode os lê:

- **Seus itens ficam na bolsa.** Uma pilha pode ser um dos seus itens e continua sendo esse item, seja qual for o nome que o jogador lhe der. O botão **From the ruleset** (do conjunto de regras) do inventário abre um seletor com todos os itens que seus catálogos listam, com a busca e os filtros do catálogo, e uma camada que oculta uma entrada também a oculta ali.
- **Nomes encontram seus itens.** Um nome que o jogador digita, ou que o Game Master escreve em `[inventory: action="add"]`, e que seja o `label` de um dos seus itens, com qualquer combinação de maiúsculas e minúsculas, adiciona esse item. Quando dois itens compartilham um rótulo, o nome encontra o que seus catálogos listam primeiro.
- **`stack`** é respeitado: adicionar, definir, juntar ou dar além dele enche a pilha e começa uma nova.
- **`freeform: "refuse"`** deixa ao jogador só os seus itens: o seletor e os nomes que são seus itens.
- **`native: false`** deixa ao Game Master só os seus itens e os que ele inventa: um nome que não é nenhum dos dois é recusado (`not-ruleset-item`), enquanto mais unidades de algo que já está com o grupo ainda podem ser adicionadas, e as instruções dele dizem isso. Uma luta não pergunta mais a um modelo o que os itens do inventário fazem, então só os seus itens lutam: numa luta do conjunto, suas armas empunhadas e os usos dos seus itens (veja Armas numa luta, abaixo), e numa batalha Classic ou Tactical, seus itens com um `use` (veja Itens nas batalhas Classic e Tactical, abaixo). O que o jogador digita continua seguindo `freeform`, e o que o grupo carregava volta numa nova sessão de qualquer forma.
- **O que um item é** aparece na pilha selecionada: sua categoria, raridade e tags pelos rótulos, as estatísticas que concede, seu resumo e quantos uma pilha comporta. O seletor também mostra seu `cost`. O Game Master vê cada um dos seus itens que estão no inventário com categoria, raridade, tags e as estatísticas que você deixou `promptVisible`, como `Hand axe [Weapon, Common, Thrown; Damage 1d6, Reach close]`.

- **`slots`**: um item que ocupa espaços pode ser equipado por quem o carrega, enquanto tiver esses espaços livres, e um item de uma pilha maior é separado numa pilha própria para ser vestido. O inventário mostra cada espaço em uso por personagem.
- **`binding`**: um item com `binds` pode ser vinculado, até `binding.max` lido da própria ficha de quem o porta (um personagem sem ficha lê uma ficha em branco). Um item `cursed`, depois de vinculado, continua vinculado: o jogador não pode desvinculá-lo, tirá-lo, dá-lo nem removê-lo, e só o Game Master pode acabar com a maldição.
- **`carry`**: um item pesa seu valor de `carry.stat` (um item sem ele não pesa nada), e a carga de um personagem é o que sua bolsa pesa, comparada a `encumberedAbove` e `limit` lidos da própria ficha. Um item adicionado à visão compartilhada do inventário, pelo jogador ou pelo Game Master sem `who=`, vai para quem puder carregá-lo sem ficar sobrecarregado (primeiro o jogador, depois o grupo em ordem), dividido pelo espaço que resta a cada um quando ninguém consegue levar tudo. Nada passa do `limit` de ninguém: o que ninguém consegue carregar fica para trás, e o Game Master é avisado. Dê `"integer": false` a uma estatística de peso para pesos como um quarto de libra.
- **O Game Master inventa itens no seu vocabulário.** A menos que você defina `propose: false`, o `[inventory: action="add"]` dele pode descrever um item novo: `like=` um dos seus itens como ponto de partida e então qualquer um entre `category=`, `rarity=`, `tags=`, `stats=` (pares `id=value`), `slots=` (`id=count`), `binds=` (`yes`, `cursed` ou `no`), `worn=` e `carried=` (mudanças separadas por `;`, cada uma `+N`, `-N`, `advantage`, `disadvantage` ou `fails` para salvaguardas, em perícias ou salvaguardas pelo nome, ou em `checks` ou `saves` para todas elas, `+N` ou `-N` no nome de um atributo para aumentá-lo ou diminuí-lo e, onde há lutas, `+N`, `-N`, vantagem ou desvantagem em ataques e `+N` ou `-N` na defesa ou na sua palavra para ela: `worn="+1 Sneak; disadvantage on Sway checks"`, `carried="+1 Brawn"`, `worn="+1 Guard"` ou `none`) e `summary=`, cada parte pelo ID ou rótulo. Uma estatística de item que sua defesa já conta (o Guard de Ember Roads soma o `guard` do que está vestido) é a forma como uma armadura a aumenta; por isso, uma mudança vestida na defesa ao lado dessa estatística é omitida, e a resposta diz isso: um modelo pequeno escreve `guard=1` e `+1 Guard` para um único +1. Ele também lê o que um modelo pequeno costuma escrever no lugar: um número depois do nome (`Brawn +1`), "bonus" depois de ataques ou defesa (`+1 attack roll bonus`), `none` para uma lista vazia de tags, estatísticas ou espaços, e `worn=`, `carried=` ou `summary=` dentro de `stats=` quando você não tem estatística com esse nome. O Engine mantém só o que seu bloco tem: uma categoria, tag, estatística, espaço, perícia ou salvaguarda desconhecida é omitida, uma raridade que você não tem vira a mais baixa, um número é limitado à faixa da sua estatística e depois a `rarityCaps` para sua raridade, um bônus vestido ou carregado ao `bonus` da sua raridade (inclusive a parte herdada de `like`), e um nome que é um dos seus itens é simplesmente esse item. A resposta informa ao Game Master o que foi alterado (nunca sobre uma estatística que você não mostra a ele), e os detalhes do item mostram cada alteração ao jogador. A partida guarda o item, então o mesmo nome é esse item pelo resto da partida, e uma nova sessão o mantém enquanto alguém ainda o tiver.
- O Game Master pode usar `equip` e `unequip` nos seus itens com o comando de inventário quando você tem `slots`, e `bind` e `unbind` quando tem `binding`: ele só aprende as ações que seu conjunto tem. Ele vê a carga de cada personagem, os itens vinculados e os espaços, e o que está vestido ou vinculado.

- **O que um item faz enquanto vestido ou carregado** aparece na pilha selecionada e no seletor ("While worn: -1 on checks (Sneak)"), e o Game Master o vê ao lado do item (`worn: -1 on checks (Sneak)`). Os testes fora de uma luta o aplicam (veja Testes fora de uma luta, abaixo).
- **O que uma arma faz** aparece da mesma forma ("Attack (Action): Brawn to hit, 1d6 + Brawn cut damage"), e uma luta a oferece enquanto ela está empunhada (veja Armas numa luta, abaixo).
- **O que usar um item faz** aparece da mesma forma ("Use (Action): heals 1d4 + 1, range 0 paces, used up"), com as cargas que lhe restam. Uma luta oferece o uso, assim como o botão **Use** (usar) do inventário e o `[inventory: action="use"]` do Game Master (veja Usar itens numa luta e Usar itens fora de uma luta, abaixo).

Numa batalha Classic ou Tactical, cada um dos seus itens com um `use` faz ali o que diz, e um item sem ele não é oferecido (veja Itens nas batalhas Classic e Tactical, abaixo). Seja o que for que `native` diga, a ficha pode ler seus itens (abaixo), uma arma empunhada é um ataque numa luta do conjunto, disparando o que tira da bolsa, o que um item faz enquanto vestido ou carregado conta nessa luta (veja Armaduras e efeitos de itens vestidos numa luta, abaixo), um item com um `use` é usado pelas próprias regras, numa luta e fora dela (veja Usar itens numa luta e Usar itens fora de uma luta, abaixo), e suas `currencies` são o dinheiro do grupo (veja Dinheiro, abaixo).

<a id="items-on-the-sheet"></a>

### Itens na ficha

Uma referência de valor pode ler os itens que um personagem tem, com `itemStat` (Capability API 1.52). Ember Roads soma ao Guard de um viajante o guard da armadura que ele veste, que é o que um golpe precisa superar:

```json
{
  "id": "guard",
  "label": "Guard",
  "op": "sum",
  "of": [{ "const": 6 }, { "abilityMod": "wits" }, { "itemStat": { "stat": "guard", "from": "worn", "pick": "sum" } }]
}
```

- `from`: `"worn"` são os itens no personagem: um item que ocupa espaços enquanto está equipado, um que tem `binds` enquanto está vinculado e um que faz as duas coisas enquanto as duas valem. Um item que não faz nenhuma das duas nunca está vestido. `"carried"` é o restante do que ele leva, e `"all"` são os dois.
- `pick`: `"sum"` soma o valor de cada item vezes a quantidade, `"max"` e `"min"` leem o maior ou o menor valor individual, e `"count"` conta os itens, pela quantidade.
- `stat` nomeia uma das suas estatísticas de item. `sum`, `max` e `min` exigem uma estatística `number`. `count` pode omiti-la para contar todos os itens, ou nomear qualquer estatística para contar só os itens que a concedem.
- `slot`, `category` e `tag` (cada um opcional) mantêm só os itens que ocupam esse espaço, são dessa categoria ou têm essa tag.
- `default` (opcional, 0 quando omitido) é o que ele lê quando nenhum item é escolhido ou nenhum deles concede a estatística.

Os itens mudam durante a partida, então `itemStat` segue a mesma regra de uma leitura viva: o `max` de uma reserva ou de uma trilha, o bônus de proficiência, `binding.max`, os números de `carry` e a coluna escalada ou a escala de um catálogo não podem lê-lo, nem por meio de um valor derivado. Em qualquer outro lugar onde um número é lido, ele pode: um valor derivado, o `adjust` de um teste, o `cap` de uma perícia, a defesa, a absorção ou a iniciativa de uma luta.

Os itens de um personagem são os da própria bolsa. O card do jogador (o nomeado como quem o chat interpreta ou, se não houver, o primeiro) lê a bolsa do jogador, e cada outro card lê a sua. Só os itens do seu conjunto contam, já que um item comum não tem estatísticas para ler. A ficha na tela, um teste, o bloco de ficha do GM e uma luta, quando começa, leem o que cada personagem tem naquele momento. Fora de uma partida, no editor de fichas ou na revisão de uma importação, ninguém tem nada e `itemStat` lê seu `default`.

<a id="checks-outside-a-fight"></a>

### Testes fora de uma luta

Um teste ou salvaguarda que o Game Master pede fora de uma luta lê mais que a ficha: as condições ativas do próprio personagem, os níveis que ele atingiu, o que seus itens fazem enquanto vestidos ou carregados e o que a exigência de um item vestido custa enquanto ele não a atende. É o mesmo vocabulário que as condições de uma luta usam (Capability API 1.53 para as chaves de item e a restrição por perícia ou salvaguarda).

Um item diz o que faz com `worn`, que vale enquanto ele está vestido (e vinculado, quando se vincula), e `carried`, que vale enquanto ele está só carregado. O casaco de couro de Ember Roads range quando você se esgueira, e uma pedra-guia faz o povo das caravanas confiar em você:

```json
"worn": { "modifiers": [{ "to": "checks", "skills": ["sneak"], "flat": -1 }] }
```

```json
"carried": { "modifiers": [{ "to": "checks", "skills": ["sway"], "flat": 1 }] }
```

Cada um aceita as partes de uma condição que um teste lê:

- `effects`: `own-checks-advantage`, `own-checks-disadvantage`, `own-saves-advantage` e `own-saves-disadvantage`.
- `modifiers`: mudanças (`to`) em testes ou salvaguardas, com `flat`, `dice` (e `minus`), `mode` e suas próprias `skills` ou `saves`, exatamente como as de uma condição.
- `failsSaves`: salvaguardas que quem o porta falha sem rolar.
- `skills` e `saves`: a quais perícias ou salvaguardas se referem os efeitos e os modificadores que não nomeiam as suas próprias.
- `abilities` (Capability API 1.54): atributos que ele altera, por ID, cada um `{ "set": N }` (pelo menos N: uma pontuação maior permanece) ou `{ "add": N }` (um número negativo o diminui). Eles se aplicam antes de a ficha ser calculada, então tudo que lê o atributo lê o valor alterado: a ficha na tela, valores derivados, testes, o bloco de ficha do GM e uma luta, quando começa. As somas vêm primeiro, depois o maior `set`, e o `min` e o `max` do próprio atributo continuam valendo. O máximo de uma reserva ou de uma trilha e o bônus de proficiência são calculados só a partir da ficha, então um item não os altera.

As manoplas de couro de boi de Ember Roads dão a uma pegada fraca a força de um tropeiro:

```json
"worn": { "abilities": { "brawn": { "set": 2 } } }
```

O que um item faz numa luta (defesa, ataques, velocidade, os outros efeitos da luta e o dano e as condições que ele evita) está em [Armaduras e efeitos de itens vestidos numa luta](#armor-and-worn-effects-in-a-fight); um teste fora de uma luta lê só as partes acima. Um item se aplica uma vez, quantas pilhas dele alguém tenha.

Um item também pode exigir algo de quem o veste, com `requires`: um valor da ficha dessa pessoa (`value`, qualquer referência de valor, lida como tudo o mais durante a partida), o mínimo que ele pode ser (`atLeast`) e o que se aplica enquanto ela não atende à exigência (`otherwise`, no mesmo vocabulário, exceto que não pode alterar um atributo, já que o que é exigido pode lê-lo). Até quatro. A pá de coveiro de Gravewatch é trabalho pesado para um guardião fraco:

```json
"requires": [
  {
    "value": { "abilityScore": "sinew" },
    "atLeast": 3,
    "otherwise": { "modifiers": [{ "to": "checks", "skills": ["dig"], "flat": -1 }] }
  }
]
```

Uma exigência lê o atributo como os itens de quem veste o deixam, então manoplas que definem Sinew como 3 atendem à exigência da pá.

Num teste:

- Todo modificador que se refere a ele soma seu número, e seus dados são rolados: um número fixo num teste somado, dados numa reserva. Um teste de atributo, e um teste que o conjunto não sabe nomear, leem só o que não está restrito a perícias específicas, e uma salvaguarda lê só o que se refere a salvaguardas.
- Vantagem e desvantagem de todas as fontes, inclusive o `mode=` do Game Master, se anulam: havendo qualquer uma de cada, o teste é rolado uma vez. Só um conjunto que rola duas vezes (`resolution.advantage`) pende para um lado.
- Uma salvaguarda que uma condição ou um item faz falhar falha sem rolagem, e nada é gasto nela.
- O registro diz o que o alterou: `effects="-1"` para o número, `from="Leather coat"` para o que o alterou e `automatic="true"` para uma salvaguarda falhada sem rolagem. O cartão de dados mostra o mesmo, e o Game Master o lê no turno seguinte. Ele é informado de que o Engine aplica isso, para não somar de novo.

Gravewatch mostra o mesmo numa reserva: o sino da aurora vinculado adiciona um dado a Ward, e Rattled tira um de Soothe e de Barter.

<a id="weapons-in-a-fight"></a>

### Armas numa luta

Um item pode ser uma arma (Capability API 1.55). Seu `attack` tem a forma das [linhas de ataque](#combat-a-fight-your-own-rules-resolve) de um bloco de combate, com valores no lugar de colunas, e uma [luta do conjunto](#combat-a-fight-your-own-rules-resolve) o oferece como ataque enquanto o item está vestido: empunhado nas mãos que ocupa e vinculado, quando se vincula. Um item guardado ou só carregado não oferece nada, e as linhas de ataque e as ações próprias de uma criatura funcionam exatamente como antes. O machado de mão de Ember Roads:

```json
"attack": {
  "budget": "act",
  "toHit": { "abilities": { "stat": "swing" } },
  "damage": { "dice": { "stat": "damage" }, "abilities": { "stat": "swing" }, "type": "cut" },
  "reach": 2,
  "range": { "normal": 10, "long": 20 }
}
```

- `budget`: o orçamento que ele gasta, um de `combat.economy.budgets`.
- `toHit`: o que ele soma para acertar. `abilities` são um ou mais IDs de atributo, e conta o melhor deles (uma arma de acuidade lista dois). `skill` soma o que um teste daquela perícia soma, com o atributo do ataque no lugar do da perícia, como faz uma linha de ataque. `proficiency` é um valor da ficha de quem empunha: quando lê acima de 0, o bônus de proficiência do conjunto é somado (`{ "const": 1 }` para sempre). `bonus` é um número. `target` é o alvo por dado próprio de uma luta de reserva para o ataque desta arma (a pá de Gravewatch conta todo dado a partir de 6 em vez do 7 da reserva), e só um conjunto `dice-pool` cujo `target` pode se mover pode definir um; numa luta somada, o `bonus` diz o mesmo.
- `damage`: os `dice` que ela causa, o melhor dos seus `abilities` de dano somado, um `bonus` e um `type`. Uma luta somada exige `dice`; numa luta de reserva, um acerto causa seus sucessos e `dice` adiciona mais essa quantidade de dados do próprio tipo de dado da reserva.
- `reach` e `range` (`normal`, e `long` para o que ela ainda alcança além disso) ficam na sua unidade de `combat.distance` e precisam dela. Uma arma com os dois é de arremesso, como uma linha de ataque: um golpe de perto, um arremesso além. Sem nenhum dos dois, ela alcança uma célula.
- `versatile`: os `dice` que ela causa no lugar enquanto cada espaço que ocupa tiver lugar para outro tanto entre o que quem a empunha veste: uma lança numa mão com a outra livre.
- `strikes`: quantos ataques um gasto compra, lido da ficha de quem empunha, como o `strikes` de uma linha de ataque. Uma arma sem isso dá um ataque por gasto, quantos ataques seu portador tenha.

Qualquer um desses números ou palavras pode ler a própria estatística do item: `{ "stat": "damage" }` é o `damage` do item, e uma estatística à qual o item não dá valor é como se o valor não estivesse escrito. `abilities` e `skill` leem uma estatística enum cujas palavras são IDs de atributo ou de perícia, e `type`, uma estatística de texto ou enum. Assim, um item que o Game Master inventa `like=` uma arma também é uma arma, que luta com as próprias estatísticas, limitada por `rarityCaps` como qualquer estatística (um valor que você escreve diretamente é copiado como está). Um item inventado sem espaço e sem vínculo nunca é vestido, então é criado sem o ataque. Um modelo pequeno tende a descrever uma arma e omitir `like=`; por isso, um item inventado numa categoria das suas armas, sem nada de onde partir, luta como a arma dessa categoria com o nome mais parecido (uma palavra contida no outro nome, em qualquer direção, então Crossbow se parece com Bow) ou como a primeira delas, herdando qualquer estatística que o ataque leia e que a proposta omitiu, e a resposta diz isso ("It fights as Hand axe does."). O formulário de proposta do Game Master também lhe diz que uma arma criada como outra luta como ela.

As tags de uma arma são o que seus golpes carregam. A entrada `resist` ou `immune` de uma criatura pode dizer o que a atravessa: `{ "type": "tearing", "except": ["silver"] }` é recebido por inteiro de uma arma com a tag `silver` e resistido de qualquer outra coisa. O grave wight de Gravewatch é escrito assim, e o prego de caixão de prata do conjunto o atravessa.

Os detalhes da arma e a linha do Game Master dizem o que ela faz: `attack (Action): Brawn to hit, 1d6 + Brawn cut, reach 2 paces, range 10 to 20 paces`, com `1d8 with a hand free` para a lança de javali de Ember Roads e `at 6` ao lado da pá de Gravewatch.

<a id="ammunition-and-reloading"></a>

#### Munição e recarga

Uma arma pode disparar algo e manter uma contagem carregada própria (Capability API 1.57). O arco de caça de Ember Roads dispara flechas, e a pistola de vigia de Gravewatch comporta uma bala, carregada a partir do chumbo e da pólvora do guardião:

```json
"ammo": { "tag": "arrow", "recover": 0.5 }
```

```json
"ammo": { "tag": "shot" },
"clip": { "max": 1, "reload": "act" }
```

- `ammo`: o que a arma dispara, por uma das suas `tags` de item. Cada ataque consome `perAttack` (1 quando não diz nada) dos itens com essa tag que quem a empunha carrega, vestidos ou não, da primeira pilha desse tipo na bolsa e depois da seguinte. A arma só é oferecida enquanto ele carrega o suficiente, e o menu da luta diz quantos restam. Depois de uma luta que o grupo vence, `recover` (de 0 a 1) do que cada um disparou volta para a pilha de onde saiu: a fração é somada na luta inteira e arredondada para baixo uma vez por pilha, então metade de cinco flechas disparadas de uma aljava dá duas. Uma luta perdida ou abandonada em fuga não devolve nada.
- `clip`: uma contagem carregada que a arma guarda em si. `max` é quanto ela comporta, escrito diretamente ou lido de uma estatística numérica do item, e `reload` é o orçamento que uma recarga gasta. Um ataque gasta o que está carregado em vez de tirar da bolsa, e a arma só é oferecida enquanto tiver carregado o suficiente para um ataque. Uma opção **Reload** (recarregar) no menu da luta gasta `reload` e a enche: a partir da bolsa quando ela tem `ammo` (tanto quanto a bolsa ainda tiver, quando for menos) e por completo quando não tem. A contagem fica na pilha da arma no inventário, então uma pistola esvaziada numa luta continua vazia na próxima. Uma arma que ninguém disparou ainda está carregada, assim como uma que o jogador junta numa pilha de várias iguais, já que uma contagem carregada pertence a uma única arma. A munição carregada não é recolhida depois de uma luta, então `recover` vale para armas que não têm essa contagem carregada.

O que uma luta dispara, carrega e recolhe é gravado no inventário a cada passo, como a saúde do grupo, e o diário diz o que foi usado. Um passo cuja pilha sumiu, ou contém menos do que a luta contava, é recusado, assim como um item gasto que a luta não encontra. O registro da luta diz cada disparo, recarga e recolhimento ("Hunting bow: 2 left to shoot.", "Ada loads 1 into Watch pistol: 1 of 1 loaded."), e os detalhes do item e a linha do Game Master dizem o que a arma dispara e comporta: `ammunition Arrow (1 an attack, 50% picked up after a won fight)` e `holds 1, reload (Act)`.

<a id="modes-a-second-weapon-a-floor-and-conditions-on-a-hit"></a>

#### Modos, uma segunda arma, um piso e condições num acerto

Uma arma pode ter outras formas de fazer seu ataque, uma parceira na mão inábil, um dano mínimo e condições que impõe ao que acerta (Capability API 1.58). O arco de caça de Ember Roads pode disparar uma saraivada, e o prego de caixão de prata de Gravewatch pode ser empunhado em cada mão e marca o que fere fundo:

```json
"modes": [{ "id": "volley", "label": "Volley", "ammo": 2, "toHit": -2, "targets": 2 }]
```

```json
"offHand": true,
"onHit": [{ "condition": "marked", "atLeast": 2, "rounds": 2 }]
```

- `modes`: até seis outras formas de fazer o ataque, cada uma com um `id`, um `label` e o que ela muda. `ammo` é quantos disparos um ataque nesse modo consome, então a arma tem `ammo` ou um `clip` de onde disparar. `toHit` é o que ele soma para acertar: dados numa luta de reserva e um número numa luta somada. `target` move nessa quantidade o alvo por dado de uma luta de reserva (a partir do da própria arma, senão do da reserva; `+1` é mais difícil), quando o alvo da reserva pode se mover. `targets` é em quantos alvos ele pode ser mirado, cada um com sua própria rolagem de ataque. O menu da luta pergunta de que forma assim que o ataque é escolhido (depois do seu estilo de iniciativa, quando há estilos), cada uma com o que se espera que faça, e oferece só os modos para os quais quem empunha tem disparos. Um membro do grupo jogado pelo Engine avalia cada modo mirado num único alvo e deixa a um jogador os modos para vários alvos. Numa janela, um ataque é feito como está.
- `offHand`: uma arma para a mão inábil. O bloco de combate nomeia o orçamento que um segundo ataque gasta e se o dano dele mantém um atributo positivo:

  ```json
  "offHand": { "budget": "quick", "ability": "full" }
  ```

  Depois que quem a empunha atacou neste turno com uma arma marcada `offHand`, cada outra arma desse tipo que ele veste oferece um segundo ataque com aquele orçamento, chamado "Silver coffin nail, off hand" no menu. `ability` é `full` (o padrão) ou `penalty-only`, que soma o atributo de dano só quando ele tira algo, como faz o combate com duas armas de 5e. Um ataque com a mão inábil é um único golpe, quantos ataques o ataque principal compre, e nunca um golpe contra quem se afasta. Uma arma marcada `offHand` exige o `offHand` do bloco de combate.
- `floor`: o mínimo que um acerto causa, escrito diretamente ou lido de uma estatística numérica do item. O dano de uma luta de reserva depois da absorção, ou o dano de uma luta somada, é elevado a ele antes que uma resistência o reduza à metade, e o registro diz isso. Um golpe que gasta rola o número de quem o faz e nunca é elevado.
- `onHit`: até quatro das suas condições que o alvo recebe quando o dano que o golpe causou chegou a `atLeast`, por `rounds` dos turnos dele ou, sem isso, até que algo as remova. O dano é o que o atingiu depois da absorção e das resistências; um golpe que toma retira iniciativa em vez de causar dano, então não impõe nenhuma condição.

Os detalhes da arma e a linha do Game Master dizem tudo isso: `modes Volley (2 shots, -2 to hit, up to 2 targets)`, `off hand (Quick)`, `at least 1 on a hit before resistance` e `Marked for 2 rounds when a hit deals 2 or more`.

<a id="armor-and-worn-effects-in-a-fight"></a>

### Armaduras e efeitos de itens vestidos numa luta

O que um item faz enquanto vestido ou carregado também pode mudar uma luta (Capability API 1.56), para quem o porta, junto com suas condições e níveis. Cada um aceita as partes de uma condição que uma luta lê:

- `effects`: qualquer um dos de uma condição, exceto os quatro que um nível não pode ter (`half-move-to-stand`, `ends-on-damage`, `cannot-target-source`, `cannot-approach-source`), já que ninguém colocou um item em quem o porta e ele nunca termina sozinho: `own-attacks-advantage`, `attacks-against-disadvantage`, `resist-all`, `speed-zero` e os demais.
- `modifiers`: mudanças (`to`) em defesa, ataques e velocidade, além de testes e salvaguardas, exatamente como as de uma condição; numa luta `dice-pool`, uma mudança em ataques é um número fixo de dados, como numa condição.
- `resist`, `vulnerable` e `immune`: tipos de dano dos quais quem o porta recebe metade, o dobro ou nada, como os de uma criatura (e validados contra `combat.damageTypes` quando você declara algum). `conditionImmunities`: suas próprias condições que nunca são impostas a quem o porta.

O anel da viúva amaldiçoado de Gravewatch custa um dado em todo ataque, o sino da aurora vinculado do mesmo conjunto torna quem o porta imune a Rattled, e uma pedra-guia de Ember Roads, carregada, afasta o calor:

```json
"worn": { "modifiers": [{ "to": "attacks", "flat": -1 }] }
```

```json
"worn": { "modifiers": [{ "to": "checks", "skills": ["ward"], "flat": 1 }], "conditionImmunities": ["rattled"] }
```

```json
"carried": { "modifiers": [{ "to": "checks", "skills": ["sway"], "flat": 1 }], "resist": ["burn"] }
```

Uma luta lê cada item uma vez, com o nome da pilha ("Widow's ring" ao lado de uma rolagem no registro, e numa defesa), como faz um teste; o `otherwise` de uma exigência não atendida também conta numa luta, então uma armadura pesada demais para quem a veste pode custar velocidade. Os detalhes de um item e a linha do Game Master dizem tudo isso ("While worn: -1 on attacks", "worn: +1 on checks (Ward), immune to Rattled"). Uma armadura que simplesmente soma a uma defesa ou a uma absorção é uma estatística de item lida com `itemStat` (veja Itens na ficha), como o Guard de Ember Roads lê o casaco vestido.

**Dureza.** Quando a iniciativa é um número que os ataques movem, `combat.pool.hardness` é a dureza de um combatente, lida da ficha dele quando a luta começa (a da armadura, por meio de `itemStat`), e uma criatura define a própria `hardness`. Um golpe que gasta cujos dados ficam abaixo dela acerta e não faz nada: o número de quem o fez ainda volta à base, o registro diz isso ("Ada's Grave spade lands on the warden with 5 dice, below a hardness of 6, and does nothing."), e o menu não prevê dano para ele. Só um conjunto com um estilo que gasta pode tê-la, e uma criatura escrita como ficha a obtém da ficha.

<a id="using-items-in-a-fight"></a>

### Usar itens numa luta

Um item pode ser usado numa luta (Capability API 1.59): um emplastro pressionado num corte, um tônico engolido, um sino tocado. Seu `use` diz o que usá-lo faz nas mesmas palavras que o `mechanics` de uma entrada de catálogo usa (veja [O que um combate lê de `mechanics`](#what-a-fight-reads-from-mechanics)): `kind` (`heal`, `attack`, `buff` ou `debuff`), `amount`, `damageType`, `plus`, `attackRoll`, `save`, `applies`, `temporary`, `range`, `area`, `targets`, `targetCount` e `friendlyFire`, com o `budget` que gasta, ou `free`. O emplastro de Ember Roads e o tônico de aquecimento e o sino da aurora de Gravewatch:

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

- Um uso fica no menu da luta em **Items** (itens), com o nome do item ("Poultice"), seja qual for o nome da pilha. Um item que ocupa espaços ou se vincula é usado enquanto está vestido (e vinculado, quando se vincula), como o sino; qualquer outro, enquanto é carregado.
- Ele gasta um orçamento, ou é `free`, e paga com ele mesmo: `cost`, `perCostStep`, `check`, `concentration`, `reaction`, `scales`, `gives`, `standard` ou `rider` são para uma entrada de catálogo que uma ficha contém, e são recusados num uso.
- `toHit`, num uso com `attackRoll`, é o que ele soma para acertar, como o `toHit` de uma arma o define (veja Armas numa luta): atributos, uma perícia, um bônus e, numa luta de reserva, um alvo por dado. Sem ele, a rolagem não soma nada.
- `saveDifficulty`: o número contra o qual é rolada uma salvaguarda que ele pede (a sua própria, uma que termina uma condição que ele aplica ou a de uma cláusula sem `difficulty`), escrito diretamente ou lido de uma estatística numérica do item. Uma entrada numa ficha lê esse número da fonte do seu catálogo; um item não tem fonte, então diz o próprio número.
- `consumes: true` tira um da pilha do item a cada uso, e o menu diz quantos restam. O último usado leva a pilha junto.
- `restore` (Capability API 1.60) devolve parte de uma reserva de cada ficha que atinge, como uma bolsa de sangue devolve sangue: `{ "pool": "resolve", "amount": { "flat": 1 } }`, com `dice` e `flat` como num `amount`. Ele nomeia uma das suas reservas vivas, nunca a reserva de saúde (a saúde volta com uma cura), e fica num uso que ajuda, um `heal` ou um `buff`. Uma criatura não tem reservas, então não recebe nada. O registro diz isso ("Ada gets back 1 Resolve, and is on 3 of 4."), e um membro do grupo jogado pelo Engine deixa em paz uma reserva cheia. O tônico de aquecimento de Gravewatch restaura um ponto de Resolve além da cura.
- `charges` gasta essa quantidade das `charges` do próprio item, cujo `max` é quantas ele comporta, de 1 a 100, escrito diretamente ou lido de uma estatística numérica do item (um item que dá a essa estatística menos de 1 é recusado, e um acima de 100 comporta 100). A contagem fica na pilha do item no inventário, então um sino tocado duas vezes numa luta tem uma carga restante na próxima, e um que ninguém tocou está cheio. Cargas pertencem a um único item, então um item que as tem tem `stack` 1. Um uso consome o item ou gasta cargas, nunca os dois, e as cargas de um item são sempre gastas pelo seu uso. Há mais duas chaves em `charges` (Capability API 1.61), como as do sino da aurora de Gravewatch:

  ```json
  "charges": { "max": 3, "recharge": { "rests": ["vigil"], "amount": "max" }, "breaksOn": { "die": 20, "atMost": 1 } }
  ```

  - `recharge`: quais dos seus `rests` trazem as cargas de volta, e quantas: `"max"` enche o item, e uma quantidade (`{ "dice": "1d6", "flat": 1 }`) devolve essa quantidade, nunca além de `max`. Um descanso traz de volta as cargas de todo item que o personagem carrega e que nomeia esse descanso, seja pelo botão Rest da ficha, seja pelo `[sheet: op="rest"]` do Game Master; a ficha e a bolsa são gravadas juntas, e uma resposta regenerada recarrega só uma vez. A ficha diz o que voltou depois do descanso ("Dawn bell 3/3 charges"), e o Game Master vê no inventário, ao lado de cada item, as cargas restantes ("2 of 3 charges left").
  - `breaksOn`: quando um uso gasta a última carga, o Engine rola um d`die` e, com resultado igual ou inferior a `atMost`, o item quebra e some da bolsa, numa luta e fora dela. O registro da luta diz isso ("Dawn bell breaks (a 1 on its die)."), o relatório do botão Use também ("Its last charge spent, it breaks."), e o diário o conta como perdido.
- `gate` (Capability API 1.62) é um teste em que quem usa precisa passar antes de o item funcionar, como o pergaminho de uma magia acima do nível de quem o lê exige. `check` nomeia o que é rolado: `{ "skill": "ward" }`, `{ "ability": "wits" }` ou um valor da ficha (`{ "value": { "derived": "spell_mod" } }`, para um número que muda de um personagem para outro, como o modificador de conjuração de um conjurador de 5e). `difficulty` vai de 1 a 100, escrito diretamente ou lido de uma estatística numérica do item (um item que não a fornece fica fora do menu, e seu botão Use diz isso claramente). `unless` pula o teste quando um valor na ficha de quem usa é pelo menos `atLeast`. Um teste falhado consome o item (ou gasta suas cargas) por nada. Numa luta, o teste é rolado quando o item é gasto, com os próprios dados da luta (uma reserva numa luta de reserva, em que `difficulty` é o número de sucessos necessário) e o número da ficha lido quando a luta começou, e conta o que as condições e os itens vestidos de quem usa somam a esse teste, inclusive o da própria perícia. O registro diz isso ("Ada rolls Ward to use Page of the vigil litany: 2 successes from 2 dice at 7 or more (8, 9), needing 2 successes, a success."). A página da litania da vigília de Gravewatch tem um:

  ```json
  "gate": { "check": { "skill": "ward" }, "difficulty": 2, "unless": { "value": { "abilityScore": "nerve" }, "atLeast": 3 } }
  ```

O que uma luta consome e as cargas que deixa são gravados no inventário a cada passo, como os disparos de uma arma, e o diário diz o que foi usado. O registro da luta diz cada uso ("Poultice: 1 of 2 left.", "Dawn bell: 2 of 3 left."). Um membro do grupo jogado pelo Engine só usa uma cura em alguém ferido, e ninguém usa um item para golpear quem se afasta. Os detalhes do item e a linha do Game Master dizem o que usá-lo faz: "Use (Action): heals 1d4 + 1, range 0 paces, used up" e `use (Act): Steel 7 save negates it, Rattled, 1 of 3 charges`, e os detalhes mostram as cargas restantes. Um item que o Game Master inventa `like=` um dos seus é usado como aquele, com cargas e tudo.

### Itens nas batalhas Classic e Tactical

Um conjunto que não resolve as próprias lutas as deixa para as batalhas Classic e Tactical do próprio Game Mode, em que um modelo é consultado sobre o que os itens do inventário fazem quando uma luta começa. Seus itens não são adivinhados: cada um com um `use` faz o que ele diz, com os próprios números do Engine, e o palpite do Game Master só é pedido para os itens que não são seus (e para nenhum, com `native: false`).

- Um `heal` cura e um `attack` fere por uma fração da saúde máxima do alvo, a partir da média do seu `amount`: uma média de 7, o `1d8+3` de uma arma básica, é um pouco mais de um quinto dela, nunca menos de um vigésimo e no máximo tudo. Essa é a escala em que a ponte de combate lê os números de uma entrada de catálogo. O `damageType` é o elemento.
- A primeira condição em `applies` entra como um status com o próprio nome, pelos seus `rounds` (2 sem eles), e um `buff` ou `debuff` é só esse status, que aumenta ou reduz a defesa como os próprios status do Engine fazem.
- `targets` diz em quem ele é usado. Sem isso, uma cura ou uma melhoria vai para um aliado, e um ataque ou um enfraquecimento para um inimigo.
- `consumes: true` tira um da pilha, e um uso sem isso deixa o item onde está.
- Uma rolagem para acertar, uma salvaguarda, uma área, pontos temporários e uma reserva restaurada não têm lugar nessas batalhas, como acontece ali com o `mechanics` de uma entrada de catálogo: a descrição do item no menu ainda os menciona.
- Um item que ocupa espaços só é usado enquanto está vestido, e um que se vincula só enquanto está vinculado, como numa luta do conjunto. Um item consumido é tirado do que está vestido, nunca de um sobressalente na bolsa.
- Um item cujo uso gasta `charges` é oferecido enquanto resta um uso, e o menu Items conta seus usos. Cada uso gasta as cargas da pilha do item, primeiro a do próprio jogador, e o último gasto rola seu `breaksOn`: quebrado, o item some da bolsa, e o diário o conta como perdido.
- Um item que pede um teste antes (`gate`) tem esse teste rolado no servidor quando um membro do grupo o usa, com a ficha desse membro e o que ele veste (um companheiro sem card próprio rola numa ficha em branco, e só a unidade do próprio jogador recorre ao card do jogador), como o botão Use o rola, e pulado quando `unless` vale. Se falhar, o item é gasto (ou suas cargas são) e não faz nada, e o registro diz isso: "Ada rolls Ward to use Page of the vigil litany: 0 against 2, failed, and it is used up for nothing."
- Um item sem `use` não é oferecido nessas batalhas, nem uma cura sem `amount`.

O Engine calcula sozinho o efeito de cada item, então o que a tela envia nunca decide o que um dos seus itens faz.

### Loot

Uma luta vencida deixa loot em toda partida do Game Mode, uma vez, nas bolsas do grupo. Sem conjunto de regras, ele vem das tabelas próprias do Game Mode (mais itens, e mais raros, numa partida mais difícil), e o mesmo acontece num conjunto que não declara tabelas de loot e deixa `native` ativado. Um conjunto que declara tabelas de loot deixa cair os próprios itens no lugar, e um com `native: false` e sem tabelas não deixa cair nada.

`lootTables` no bloco `items` (Capability API 1.63) as lista, como fazem os bens funerários de Gravewatch:

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

- `rolls` é quantas escolhas a tabela faz: um número de 0 a 20 ou dados (`"1d2"`), 1 por padrão.
- Cada escolha sorteia uma linha por `weight` (1 por padrão) contra as outras. Uma linha nomeia um dos seus itens como `<catalog>/<entry>`, um `filter` por `rarity`, `category` e `tag`, que escolhe por igual entre todos os itens que coincidem, ou (Capability API 1.64) `coins`, uma das suas moedas pelo ID. `count` é quantos caem, um número ou dados, 1 por padrão. Um item ou uma moeda que uma camada retira não deixa cair nada, nem um filtro que não encontra nenhum item.
- Uma criatura de bestiário nomeia a tabela que carrega com `loot` (veja Criaturas, abaixo). Uma luta do conjunto vencida rola a tabela de cada criatura derrotada, e uma luta que o Engine não resolve pelas suas regras não tem criaturas de bestiário, então não deixa cair nada.
- O que cai vai para a visão compartilhada como uma adição, começando pela bolsa do jogador, e o que ninguém consegue carregar fica para trás. O resumo que o Game Master recebe diz o que caiu e que isso já está nas bolsas, o diário registra, e uma notificação mostra.
- O Game Master rola uma tabela na história com `[loot: table="grave_goods" who="Ada"]` (`who=` para a bolsa de um personagem), medida a partir de onde o turno começou, como suas tags de inventário, então uma resposta regenerada deixa cair só uma vez. Cada item que cai é respondido como uma adição, e as instruções dele listam suas tabelas.

### Dinheiro

Suas `currencies` são o dinheiro do grupo. Uma moeda é um item nas bolsas, uma pilha de cada moeda como qualquer outra, nomeada pelo seu `label`:

- Ela pesa uma unidade da sua estatística de peso a cada `perWeight` moedas da família, conta para a carga de quem a porta e é colocada pela regra de carga quando entra. Uma família sem `perWeight` não pesa nada.
- O jogador adiciona moedas pela lista **Coins** (moedas) do seletor e as divide, dá e junta como qualquer pilha.
- Uma linha acima das pilhas do inventário mostra as moedas de cada família à vista e seu valor na menor moeda da família. O Game Master vê esse valor ao lado de cada bolsa (`Coin worth 432 pennies`, ou `no coins` para uma bolsa vazia, com a bolsa do jogador listada mesmo quando não tem nada), e o `cost` de um item ao lado do item (`costs 3 shillings`).
- O Game Master paga com `[inventory: action="pay" amount="3 shillings" who="Ada"]` e recebe com `action="earn"`. `amount` é uma quantidade e uma moeda, pelo ID ou rótulo, no singular ou no plural, com qualquer combinação de maiúsculas e minúsculas (`1 penny`, `12 pennies`). Um pagamento sai de uma única bolsa (a do jogador, sem `who=`), só na família da própria moeda: as maiores moedas que cabem vão primeiro, depois a menor moeda restante que cobre o que ainda se deve é trocada, e o troco volta nas moedas menores da família, das maiores para as menores. A resposta diz o que foi pago e o que voltou: 3 shillings de uma bolsa com uma crown dá `Paid with crowns ×1; shillings ×2 back.` Um pagamento que a bolsa não consegue cobrir é recusado (`cannot-afford`), assim como uma moeda que você não tem (`unknown-coin`) e qualquer pagamento num conjunto sem moedas (`no-currencies`). Um recebimento é uma adição dessa moeda, na bolsa de `who` ou dividida pela regra de carga. O diário lista o que foi gasto e recebido, e uma notificação mostra. As instruções dele listam suas moedas, e só um conjunto que tem moedas recebe essas instruções.
- Comprar é um pagamento seguido de uma adição ou, num conjunto com mercado, um `buy` (abaixo). Não há tela de loja.

### Mercados

Um `market` no bloco `items` (Capability API 1.65) diz o que um lugar vende e a que preço, em vez de toda loja vender tudo pelo preço de tabela. O de Gravewatch, um pouco resumido:

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

- `prices`: de um a oito níveis, cada um um multiplicador (`times`, acima de 0) sobre o `cost` de um item, e um deles o `default`. Um preço é o custo na menor moeda da família vezes o nível, arredondado nela (nunca abaixo de uma moeda para algo que custa alguma coisa), e expresso na moeda em que seu custo está escrito quando essa moeda o paga exatamente, senão na maior moeda que suas camadas deixam e que o paga: em `dear`, um casaco que custa 8 shillings sai por 12 shillings, e uma poção de 5e que custa 50 gp sai por 50 gold pieces, nunca 5 platinum. Pechinchar ou o humor de um vendedor mudam o nível, nunca o número.
- `places`: de um a doze tamanhos de lugar, do menor ao maior, nas suas próprias palavras.
- `sold`: regras para o menor lugar que vende o que um filtro escolhe. Um filtro nomeia `rarity`, `category` e `tag`, quaisquer deles, e coincide com um item que tem todos os que nomeia; a primeira regra que coincide com um item decide. O `sold` do próprio item prevalece sobre as regras, e um item que nada nomeia é vendido em qualquer lugar.
- `sellers`: tipos de vendedor, cada um com o que vende em `sells` (filtros, qualquer um dos quais escolhe um item), o menor `place` que tem um (qualquer lugar, sem isso) e, opcionalmente, a quem ele vende com `only`: um valor da ficha do comprador, com seu estado vivo e o que ele carrega, de pelo menos `atLeast`, informado ao Game Master como `label`. Sem `sellers`, qualquer pessoa num lugar vende o que aquele lugar vende.
- Um item sem `cost` não está à venda, nem um que uma camada oculta ou cujas moedas uma camada retirou.

**Onde o grupo está.** O Game Master diz em que lugar uma cena está e o tamanho dele com `[place: name="Millbrook" size="market town"]`, o tamanho pelo ID ou rótulo, e omite o tamanho para um lugar sem mercado (uma estrada, a natureza selvagem). O Engine responde a cada tag no lugar, recusando um tamanho que você não tem, e o último lugar respondido nas respostas do Game Master que o jogador vê, a partir do início mais recente da conversa, é o lugar em vigor até que outro seja dito. Uma tag de lugar que o jogador digita nunca conta. Uma resposta regenerada lê só o que veio antes da resposta que ela substitui.

**Comprar.** O Game Master compra com `[inventory: action="buy" item="Hand axe" count="1" level="dear" seller="smith" who="Ada"]`, sendo `level` (o padrão, sem ele) e `seller` (qualquer vendedor ali que venda o item, sem ele) opcionais. O Engine define o preço no lugar em vigor quando a resposta termina, tira o preço da bolsa do comprador (a do jogador, sem `who=`) como num pagamento, com troco, e coloca o item na bolsa dele, ou só paga, no caso de um serviço. Ou as duas coisas acontecem, ou nenhuma. A resposta diz quanto custou (`price="12 shillings"`) e o que foi pago e devolvido, e uma notificação mostra. Uma compra é recusada, sem mudar nada, quando não há lugar dito (`no-place`), quando um nível, item ou vendedor não existe no seu conjunto (`unknown-level`, `unknown-item`), quando não há preço (`not-for-sale`), quando o lugar é pequeno demais para o item (`not-here`), quando não há vendedor ali que o venda (`no-seller`), quando o comprador não atende ao `only` do vendedor (`not-to-you`), quando a bolsa não consegue pagar (`cannot-afford`) ou quando a bolsa não consegue carregar o item.

**O que o Game Master vê.** Enquanto uma cena está num lugar com mercado, um bloco MARKET nomeia o lugar e seu tamanho, os níveis de preço e cada vendedor ali com uma dúzia do que vende no nível padrão, do mais barato ao mais caro; qualquer coisa que vendam pode ser comprada pelo nome. As instruções dele ensinam `[place:]` e `buy`, e só um conjunto com mercado as tem.

#### Um dia de mercado

Um dia de mercado (Capability API 1.69) faz as prateleiras de um lugar mudarem e se esgotarem, e permite que o grupo venda de volta. O de Gravewatch, um pouco resumido:

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

- **O dia.** Um dia num lugar dura até o Game Master nomear outro lugar; ele começa um novo dia no mesmo lugar dizendo-o de novo com `day="new"` (`[place: name="Millbrook" size="market town" day="new"]`). Tudo abaixo é calculado por dia a partir das respostas do Game Master que o jogador vê, então um swipe, uma resposta regenerada ou uma ramificação não precisam de nada salvo, e um dia sempre rola a mesma prateleira.
- **`limited`** numa regra `sold` ou no `sold` do próprio item: quantos um lugar mantém disponíveis num dia, um número ou dados rolados para o dia. Uma regra pode nomear um limite sem lugar (vendido em qualquer lugar, poucos de cada vez); o lugar de um item vem da primeira regra que nomeia um lugar, e seu limite, da primeira que nomeia um limite. Depois que se esgotam, uma compra é recusada (`sold-out`) até um novo dia.
- **`stock`** num vendedor: uma das suas tabelas de loot, rolada uma vez por dia para o que aquele vendedor tem em estoque além do que vende. O que ela rola (só itens, nunca moedas) é vendido só por aquele vendedor, na quantidade rolada, seja o que for que `sells` e `sold` digam; o `only` de um vendedor continua valendo.
- **`buyBack`**: a fração (acima de 0, no máximo 1) do preço padrão de um item que um vendedor paga por ele. Um vendedor ali que vende esse tipo de coisa o compra (qualquer pessoa, sem vendedores). Sem `buyBack`, ninguém compra nada de volta.
- **`purse`** num tamanho de lugar: o caixa, isto é, o máximo que seus vendedores pagam num dia, todos juntos, numa das suas moedas. Vendas pagas na família dessa moeda contam contra ele; sem caixa, eles compram sem limite.
- **Vender.** O Game Master vende com `[inventory: action="sell" item="Silver coffin nail" count="1" seller="smith" who="Ada"]`. O Engine define o preço, tira o item da bolsa de quem vende (a do jogador, sem `who=`; nunca um item amaldiçoado vinculado) e coloca as moedas, das maiores para as menores, ou recusa e não muda nada: nenhum lugar dito, ninguém ali compra o item (`not-bought`), sem preço (`not-for-sale`), caixa do dia esgotado (`purse-spent`), nenhum em mãos, ou moedas que a bolsa não consegue carregar. A resposta diz quanto rendeu (`price="4 shillings"`), o diário o lista como vendido, e uma notificação mostra.
- O bloco MARKET adiciona o que cada vendedor tem em estoque hoje, quantos restam de um item limitado, o que os vendedores ali pagam e quanto resta do caixa do dia; as instruções dele ensinam `sell` e `day="new"`.

<a id="using-items-outside-a-fight"></a>

### Usar itens fora de uma luta

O mesmo `use` funciona fora de uma luta (Capability API 1.60 para `restore`; o resto não precisa de nada novo). O botão **Use** do inventário, num dos seus itens com um `use`, e o `[inventory: action="use" item="Poultice" who="Juno"]` do Game Master fazem o que o item faz em quem o carrega, com os dados do Engine:

- Uma cura devolve saúde: a quantidade numa reserva de saúde, ou uma marca limpa numa trilha de ferimentos, como faz uma cura numa luta. Pontos `temporary` vão para uma reserva de saúde, `restore` devolve sua reserva, e cada condição que ele aplica (`applies`) é imposta até que algo a remova.
- Um uso que fere, ou que mira o outro lado (`kind` `attack` ou `debuff`, ou `targets` `enemy`), não atinge ninguém: fora de uma luta, não há ninguém no tabuleiro para atingir. O item ainda é usado, e o Game Master é informado do que ele faz, para narrar.
- Ele tira um da pilha ou gasta suas cargas exatamente como numa luta, e um item sem nada restante, ou um que ocupa um espaço ou se vincula e não está vestido, é recusado e não muda nada. A ficha e a bolsa são gravadas juntas.
- Um `gate` é rolado primeiro, como um teste: os dados do próprio conjunto com o número da ficha, a penalidade de ferimentos de quem usa e o que suas condições e itens vestidos ou carregados fazem com esse teste. Um teste falhado consome o item por nada, e a linha diz isso ("Ada uses Page of the vigil litany: Ward check 1 against 2, failed; it is used up for nothing. 1 left.").

O que aconteceu é dito numa linha: "Juno uses Poultice: heals 4 (Grit 6/11). 1 left." O botão Use envia isso ao Game Master num bloco `[item_used]` depois de "I use my Poultice.", que o chat mostra como um selo; a tag do próprio Game Master é respondida com ele, medida a partir de onde o turno começou como suas outras tags, então uma resposta regenerada nunca usa um item duas vezes. Um item sem `use`, ou numa partida sem conjunto de regras, continua sendo simplesmente dito: "I use my rope."

<a id="battles-lending-the-sheet-to-marinaras-combat"></a>

## Batalhas: emprestar a ficha ao combate do Marinara

Por padrão, uma batalha não conhece a ficha. Constrói os combatentes como sempre, e um personagem pode sair da luta sem mudar os pontos de vida na ficha.

O bloco opcional `battle` muda isso numa única direção: empresta os números à luta e grava seu resultado de volta. **Ele não faz o combate seguir suas regras.** A matemática, quem acerta e quanto dano causa continuam sendo do Marinara. Por isso a saúde é transferida como proporção do máximo, não como seu número: meia saúde na ficha começa com meia barra daquela criada pelo Marinara. Sua reserva de 9 pontos nunca entra diretamente numa luta em que um golpe causa 12.

```json
"battle": {
  "health": { "pool": "grit" },
  "energy": { "pool": "luck" },
  "skills": [{ "list": "knacks" }]
}
```

- `health`: obrigatório. Reserva viva que representa pontos de vida na luta. Precisa pertencer a `sheet.live.pools`, não ser uma lista cujas linhas são reservas.
- `energy`: opcional. Reserva que a luta pode gastar e que vira a barra MP. Precisa ser diferente de `health`, pois a luta não pode gastar saúde como energia.
- `slots`: opcional. Reservas gastas uma unidade por vez, cada uma com `level` de 1 a 9: `[{ "pool": "slots_1", "level": 1 }]`. Cada nível e reserva são usados uma única vez.
- `skills`: opcional, até oito. Listas cujas linhas viram habilidades de combate. Só contam linhas dos seus catálogos cuja entrada tenha `mechanics`: uma linha digitada à mão não expressa efeitos numéricos. `onlyWhen` nomeia uma coluna booleana que deve estar ativa, como uma magia preparada. `alwaysWhen` nomeia uma coluna e um valor que permitem entrar mesmo assim, como magias sem preparação. É a exceção a `onlyWhen` e é recusada sem ele.

### O que entra e o que volta

**Na entrada**, para cada membro cuja ficha a partida possui: a proporção de saúde determina o início na barra do Marinara; energia vira MP; cada reserva de espaços vira espaços daquele nível; linhas marcadas viram habilidades. Saúde máxima, ataque, defesa, velocidade e nível continuam sendo números do Marinara. Saúde zero faz começar caído; acima de zero, o personagem nunca começa com menos de um ponto, para que uma proporção pequena não o derrube por arredondamento.

**Na saída**, depois de terminar a luta: a proporção final da barra é convertida para a escala da reserva, e a diferença desde o início é aplicada como dano ou cura. Energia e espaços são quantidades, não proporções, e voltam como estão. Tudo segue as regras dos botões da ficha; uma mudança recusada é ignorada e informada, não forçada. Uma luta que não mudou a saúde não escreve mudança nenhuma, para que as duas conversões não alterem a ficha sozinhas.

**Em nenhuma direção**: rolagens de ataque, salvaguardas, concentração e efeitos de um custo maior. Isso fica em `mechanics` para um sistema completo de combate ler um dia; esta ponte não aplica esses efeitos e o conjunto não deve afirmar que aplica.

Uma batalha abandonada não grava nada de volta. Se você excluir a mensagem que a iniciou ou ela nunca terminar, a ficha continua igual: a luta não aconteceu.

### Como uma entrada vira habilidade

O bloco `mechanics` é lido assim:

- `kind` vira o tipo da habilidade. Entradas `utility` e as marcadas `reaction` são omitidas, pois o combate do Marinara não tem onde aplicá-las.
- `amount` define a força como multiplicador do ataque do combatente, não como número de dano. Dados maiores nunca acertam mais fraco; o multiplicador fica na faixa das habilidades geradas.
- `range` e `area.size` são divididos por `units.distance.perCell` do catálogo para obter casas, nunca arredondando para zero. Uma explosão usa o raio, um cone a metade e uma linha uma casa. Áreas atingem todos os inimigos cobertos e respeitam `friendlyFire`.
- `damageType` vira o elemento. `targets` não é transferido: Marinara decide os alvos de curas, melhorias e ataques pelo tipo da habilidade.
- Um `cost` de energia vira custo MP; vários são somados. Um `cost` de exatamente um espaço gasta um daquele nível. Marinara cobra uma quantidade de energia ou um espaço, nunca ambos: omite entradas com dois espaços, espaços de dois níveis ou espaço mais energia. Também omite custos em outras reservas, como saúde ou recurso de classe, para não concedê-los de graça.
- `buff` e `debuff` viram os efeitos próprios do Marinara. Outras promessas do texto, como remover uma condição da ficha, não são aplicadas. Omita `mechanics` se o efeito só fizer sentido fora da batalha.

`coverage.combat` continua separado e mantém seu significado: ative apenas quando as batalhas realmente seguem as regras do seu sistema.

<a id="combat-a-fight-your-own-rules-resolve"></a>

## Combate: uma luta resolvida pelas suas próprias regras

O bloco `battle` acima empresta à luta os números da ficha, enquanto a aritmética continua sendo do Marinara. O bloco opcional `combat` é a outra coisa: define como suas regras RESOLVEM a luta. Ele parametriza um tipo de combate do Engine, assim como `resolution` parametriza um tipo de teste, e todos os nomes nele são seus. Há dois tipos: `attack-vs-defense`, em que os dados são somados e comparados com um número, e `dice-pool`, em que a luta rola as próprias reservas do seu conjunto `dice-pool` e conta sucessos (veja Uma luta em reservas de dados, abaixo).

**Uma partida cujo conjunto declara `combat` luta conforme esse bloco.** Os números do grupo vêm das fichas, os oponentes do bestiário ou da escala de ameaça, e cada turno é resolvido pelos seus dados. Tudo que um personagem gasta ou perde é gravado na ficha imediatamente: fechar a aba no meio não perde nada. A tela usa seus termos: ataques e habilidades no menu, orçamentos, condições e registro com os cálculos reais. As limitações estão na seção sobre o que ainda falta.

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

Esse é o bloco inteiro de Ember Roads, e suas partidas o usam para lutar. O rascunho 5e usa as mesmas chaves para d20:

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

### Todas as chaves

- `kind`: `"attack-vs-defense"` (um lado rola dados contra a defesa do outro; acertar causa dano) ou `"dice-pool"` (veja Uma luta em reservas de dados). Cada chave abaixo significa o mesmo nos dois, lida do jeito que aquele tipo lê números.
- `health`: obrigatório. O que a luta retira: `{ "pool": "grit" }`, uma reserva que diminui e cujo buffer temporário absorve o dano primeiro, se existir; ou `{ "track": "harm" }`, uma trilha de ferimentos que recebe MARCAS. A trilha exige `damageKinds` e não concede pontos temporários.
- `defense`: obrigatório, referência de valor. Um campo digitado pelo jogador ou um valor derivado calculado.
- `initiative`: obrigatório. Dados rolados uma vez no início e referência opcional de modificador. Empates favorecem o maior modificador e depois a ordem de preparação da luta. `"each": "round"` rola de novo a iniciativa de todos no início de cada nova rodada, com o modificador como estiver naquele momento (então um modificador que lê uma trilha de ferimentos fica mais lento depois de ferido), e a rodada começa por quem estiver primeiro na nova ordem. Capability API 1.47. Uma luta `dice-pool` pode rolá-la como reserva: `pool` (uma referência de valor) é quantos dados, e os sucessos deles mais `plus` são o número, sem `dice` nem `modifier` ao lado. `resource`, que exige `pool`, mantém esse número e deixa os ataques movê-lo (veja "Iniciativa que os ataques movem", abaixo). Capability API 1.48.
- `attackRoll`: obrigatório em `attack-vs-defense` e recusado em `dice-pool`, que rola as próprias reservas da sua resolução. Define dados, se rola duas vezes e mantém uma (`advantage`), efeitos das faces extremas de um dado (`naturals.max`: `critical`, `hit` ou `none`; `naturals.min`: `miss` ou `none`) e efeito do crítico (`critical`: `double-dice` rola novamente os dados de dano; `max-dice` adiciona uma vez suas faces máximas; `none` é um acerto comum). Faces especiais exigem um único dado, como nos testes. Salvaguardas durante a luta usam esses mesmos dados.
- `economy`: obrigatório. `budgets` define o que cabe num turno: ID, rótulo, `per` (`turn` recarrega no início do turno do portador; `round`, no início da rodada) e `count`. O PRIMEIRO orçamento é o principal, gasto por ações padrão. `movement` é uma referência opcional de distância percorrida por turno, na sua unidade. Lutas em tabuleiro a leem; veja Posições.
- `attacks`: opcional. Listas da ficha cujas linhas são armas. `name` é a coluna de nome; `damage.dice`, a de dados; `toHit.ability`, `toHit.proficiency`, `toHit.bonus`, `damage.ability`, `damage.bonus` e `damage.type` nomeiam colunas da mesma lista. Uma coluna `ability` é um `enum` com um ID de atributo; outros valores não acrescentam nada. `proficiency` é `boolean`: ativar adiciona o bônus de proficiência. Sem dados legíveis, a linha não é um ataque; uma corda continua sendo corda.
  `toHit.skill` nomeia uma coluna `enum` com um dos seus IDs de perícia: a linha soma o que um teste daquela perícia soma, com o `ability` da própria linha no lugar do da perícia quando ela nomeia um, exatamente como o `with=` de um teste faz a troca (Capability API 1.47).
  `strikes` referencia quantos ataques UM gasto do orçamento compra. Escolher uma linha sem ataques disponíveis gasta o orçamento e deixa os restantes disponíveis. Enquanto houver algum, todas as linhas que declaram `strikes` são gratuitas: trocar de arma, alvo ou andar entre ataques decorre do próprio menu. `strikesCappedBy` nomeia uma coluna booleana que limita SUA linha a um único ataque, independentemente de quantos a lista compra: serve para armas que disparam uma vez por turno, como a propriedade Loading do SRD 5.1. Não faz sentido, e é recusado, numa lista que já compra um ataque por gasto. Os ataques disponíveis pertencem ao COMBATENTE, não a uma lista: duas listas com `strikes` compartilham a mesma quantidade, qualquer que seja a linha usada. São removidos ao terminar o turno que os comprou. Sem declaração, cada gasto compra um ataque, como antes.

  ```json
  {
    "list": "attacks",
    "budget": "action",
    "name": "name",
    "strikes": { "field": "attacks_per_action" },
    "damage": { "dice": { "column": "damage" } }
  }
  ```

- `abilities`: opcional. Listas cujas linhas marcadas por catálogo são habilidades, filtradas como `battle.skills` com `onlyWhen` e `alwaysWhen`. Os efeitos são o `mechanics` da entrada; o bloco define o `budget` padrão, o `toHit` adicionado quando rola para acertar e a `saveDifficulty` das salvaguardas. Uma entrada que pede salvaguarda, própria ou para terminar uma condição aplicada, é recusada se a lista não tiver `saveDifficulty`: uma salvaguarda contra nada sempre passaria.
- `standard`: opcional, lista fechada `dash`, `disengage`, `dodge`, `help`, `hide`, `ready`. `dodge` (ataques recebidos rolam duas vezes e mantêm o pior) e `help` (próximo ataque do aliado rola duas vezes e mantém o melhor) sempre são resolvidos. `dash` (outra cota completa de movimento) e `disengage` (ninguém ataca por você se afastar neste turno) são resolvidos no tabuleiro e apenas registrados fora dele. `hide` e `ready` são aceitos, mas ainda não fazem nada.
- `standardEffects`: opcional, efeitos de uma ação padrão ausentes de seu indicador. Hoje só `dodge` tem isso: `{ "dodge": { "saves": ["dex_save"] } }` informa quais salvaguardas rolam duas vezes e mantêm a melhor enquanto durar. Nomeie só salvaguardas declaradas e apenas se `standard` incluir `dodge`. Sem isso, esquivar continua dificultando acertos e nada mais.
- `conditions`: opcional. Mapeia SEUS IDs de condições para seus efeitos, unindo ficha e luta num registro: um personagem envenenado continua assim depois. Lista fechada: `own-attacks-advantage`, `own-attacks-disadvantage`, `attacks-against-advantage`, `attacks-against-disadvantage`, `attacks-against-adjacent-advantage`, `attacks-against-far-disadvantage`, `attacks-from-adjacent-critical`, `cannot-act`, `cannot-react`, `speed-zero`, `half-move-to-stand`, `ends-on-damage`, `own-saves-advantage`, `own-saves-disadvantage`, `own-checks-advantage`, `own-checks-disadvantage`, `resist-all`, `cannot-target-source` e `cannot-approach-source`. `failsSaves` nomeia salvaguardas que falham sem rolar. Os seis efeitos que precisam de distância ou movimento (`attacks-against-adjacent-advantage`, `attacks-against-far-disadvantage`, `attacks-from-adjacent-critical`, `speed-zero`, `half-move-to-stand`, `cannot-approach-source`) só atuam no tabuleiro; veja Posições. `cannot-react` exclui o portador das janelas abertas por movimento, então ele nunca é consultado. Há mais quatro chaves ao lado dos efeitos:
  - `modifiers`: os números que a condição altera enquanto vale. Veja Números que uma condição altera, abaixo.
  - `saves`: salvaguardas a que se referem os dois efeitos de salvaguarda e qualquer modificador de salvaguardas. Sem ela, todas; declará-la sem nenhum dos dois é recusado.
  - `skills`: perícias a que se referem os dois efeitos de teste e qualquer modificador de testes que não nomeie perícias próprias. Sem ela, todas; declará-la sem nenhum dos dois é recusado. As disputas de uma luta rolam os próprios testes da luta, não perícias, então algo restrito a perícias nunca chega a uma disputa; um teste fora de uma luta com uma dessas perícias o lê (veja Testes fora de uma luta, abaixo). Capability API 1.53.
  - `whileSourceInSight`: o que só vale enquanto quem aplicou a condição estiver visível para o portador. `true` condiciona a condição inteira; uma lista de efeitos condiciona só esses e mantém os demais, como medo que impede aproximação mesmo sem ver a origem. Efeitos ausentes da condição são recusados. Sem tabuleiro, não há linha de visão para romper, então tudo vale igualmente.
  - `endsWhenSourceDown`: a condição termina quando quem a aplicou cai.

  `own-saves-advantage` e seu oposto rolam duas vezes e mantêm uma, como ataques; eles se anulam. `own-checks-advantage` e seu oposto fazem o mesmo com o lado de quem a porta numa disputa, que é onde uma luta faz seus testes de atributo, e com os testes dessa pessoa fora de uma luta. `resist-all` reduz todo dano pela metade além das defesas próprias do alvo e se anula com uma vulnerabilidade da mesma forma. `cannot-target-source` impede mirar qualquer coisa em quem aplicou a condição. `cannot-approach-source` impede chegar mais perto que a casa atual, incluindo o caminho: contornar até uma casa igualmente distante é permitido, mas passar perto e sair do outro lado não.

  ```json
  { "condition": "restrained", "effects": ["own-saves-disadvantage"], "saves": ["dex_save"] }
  ```

  **Números que uma condição altera.** Cada entrada em `modifiers` nomeia o que altera, `to`, e em quanto:
  - `defense`: o número que um ataque contra quem a porta precisa alcançar. Só um número fixo.
  - `attacks`, `saves`, `checks`: as próprias rolagens de ataque, salvaguardas e testes de disputa de quem a porta. Um número fixo, ou `dice` rolados toda vez que a rolagem é feita (`"1d4"`); `minus: true` subtrai os dados em vez de somá-los.
  - `speed`: quanto quem a porta anda, na sua própria unidade de distância (`flat`), ou `times` 0.5 ou 2 para metade ou o dobro, aplicado depois de toda mudança fixa. Lido só num tabuleiro, como `speed-zero`.

  Um modificador de `checks` pode nomear as próprias `skills`, e um de `saves` as próprias `saves`, que o restringem no lugar das da condição. Qualquer um deles pode rolar duas vezes com `mode: "advantage"` ou `"disadvantage"`, contado junto com os efeitos de mesmo nome e se anulando como eles, com ou sem um número ao lado; assim, uma condição pode tornar Sneak mais difícil e Climb mais fácil ao mesmo tempo. Essas três chaves exigem Capability API 1.53.

  ```json
  { "condition": "muddy", "modifiers": [{ "to": "checks", "skills": ["sneak"], "mode": "disadvantage" }] }
  ```

  Um número fixo carrega o próprio sinal, e 0 é recusado. Todo modificador aparece no registro ao lado da rolagem que alterou, com o nome da condição ("12 + 5 + 3 (Blessed) = 20"), e a chance de acertar e a chance de vencer do menu o levam em conta. Condições se acumulam umas com as outras; a mesma condição duas vezes continua sendo uma condição.

  ```json
  "conditions": [
    { "condition": "blessed", "modifiers": [{ "to": "attacks", "dice": "1d4" }, { "to": "saves", "dice": "1d4" }] },
    { "condition": "shielded", "modifiers": [{ "to": "defense", "flat": 5 }] },
    { "condition": "slowed", "modifiers": [{ "to": "speed", "times": 0.5 }, { "to": "defense", "flat": -2 }] }
  ]
  ```

- `levels`: opcional. Níveis de uma trilha viva que contam como condições enquanto a trilha estiver alta o bastante, que é como se expressa uma condição que piora em etapas, como a exaustão. Cada entrada nomeia uma `track` comum (não uma trilha de ferimentos), o nível `at` em que começa e o que faz, com os mesmos `effects`, `modifiers`, `failsSaves`, `saves` e `skills` de uma condição. Todo nível que a trilha atingiu conta, então eles se somam conforme ela sobe. Um nível não tem origem e só termina quando o que ele lê diminui, então `half-move-to-stand`, `ends-on-damage`, `cannot-target-source` e `cannot-approach-source` são recusados num nível. Só um combatente com ficha tem trilhas. O registro nomeia um nível pela trilha e pelo número ("- 1 (Heat 3)").

  No lugar de `track`, um nível pode ler um valor `derived` (Capability API 1.54), calculado com o estado vivo e os itens do personagem, para que ninguém precise marcar nada: Ember Roads deixa mais lento qualquer um que carregue 10 de volume ou mais.

  ```json
  { "derived": "bulk_carried", "at": 10, "modifiers": [{ "to": "speed", "flat": -2 }, { "to": "checks", "skills": ["sneak"], "flat": -1 }] }
  ```

  Um limiar que muda de um personagem para outro (mais de cinco vezes a Força, por exemplo) é um valor derivado que o subtrai: `scale` por -1 dá a parte a tirar, e uma `sum` das duas é o que o nível lê.

  ```json
  "levels": [
    { "track": "exhaustion", "at": 1, "effects": ["own-checks-disadvantage"] },
    { "track": "exhaustion", "at": 2, "modifiers": [{ "to": "speed", "times": 0.5 }] },
    { "track": "exhaustion", "at": 3, "effects": ["own-attacks-disadvantage", "own-saves-disadvantage"] }
  ]
  ```

  Modificadores, níveis e os efeitos de teste exigem Capability API 1.45.

- `checks` e `contests`: opcionais. O que uma disputa lê e as disputas que qualquer um numa luta pode iniciar: agarrar, empurrar, se soltar. Veja Disputas, abaixo. Capability API 1.43.
- `concentration`: opcional. Campo vivo `text` que registra o efeito mantido, `save` exigida pelo dano, `floor` mínimo da dificuldade e `fromDamage`, proporção do dano que a define quando maior. Iniciar outra habilidade de concentração termina a primeira; falhar a salvaguarda a encerra e remove as condições mantidas por ela.
- `dying`: opcional, `kind: "saves"`. Duas trilhas que contam as rolagens (a quantidade necessária é o máximo de cada uma), `dice`, `succeedAt`, efeitos das faces extremas (`naturals.max`: `revive-1` ou `success`; `naturals.min`: `one-failure` ou `two-failures`), custo de dano enquanto caído (`damageWhileDown`, `criticalWhileDown`) e `condition` do personagem caído. Sem o bloco, saúde zero só o deixa caído e cura o traz de volta.
- `damageTypes`: opcional. Tipos do sistema, comparados sem distinguir maiúsculas.
- `damageKinds`: obrigatório se `health` nomeia uma trilha de ferimentos e recusado para uma reserva, pois só uma marca mantém um tipo. Define quais `kinds` o golpe marca e quantas casas. `default` recebe o que não foi mapeado, inclusive dano sem tipo; `byType` mapeia `damageTypes` para tipos de marca, ignorando maiúsculas: `"Fire"` e `"fire"` são uma chave e declarar ambas é recusado. `marks` não tem padrão porque as respostas são opostas: `"per-point"` conta níveis de saúde pelo dano, então três de dano marcam três casas e reduzi-lo ajuda; `"per-blow"` marca uma casa se o golpe acerta, qualquer que seja a força. Um golpe com várias cláusulas ainda marca uma casa, usando o tipo mais grave que entrou. Informe qual seu sistema usa: `{ "default": "bashing", "byType": { "fire": "aggravated" }, "marks": "per-point" }`.
- `pool`: obrigatório em `dice-pool` e recusado em `attack-vs-defense`. Veja Uma luta em reservas de dados.
- `spendLimits`: opcional. Quanto de uma reserva viva um combatente pode gastar por `turn` ou por `round` numa luta, seja qual for o tipo: `[{ "pool": "blood", "max": { "derived": "blood_per_turn" }, "per": "turn" }]`. `max` é lido uma vez quando a luta começa. Um custo além do que resta do limite não pode ser pago, então fica fora do menu, por mais que haja na reserva. Um limite por turno recomeça no início do próprio turno de quem o tem, e um por rodada quando a rodada começa. Um oponente escrito com números simples não paga nada de uma ficha, então um limite nunca o restringe; um oponente escrito como ficha paga das próprias reservas e fica sujeito ao limite como qualquer outro. Capability API 1.47.
- `threat`: opcional e necessário para bestiários. `tiers` é a escala de oponentes: ID, rótulo, faixa `health`, `defense`, `toHit`, faixa `damagePerRound` e `saveDifficulty`. Toda criatura nomeia um patamar; um oponente não escrito é ajustado ao que o GM pediu, sem sair da escala. `damagePerRound` expressa o que causa a UM alvo por rodada, incluindo toda a sequência.

### Uma luta em reservas de dados: `dice-pool`

Um conjunto `dice-pool` conta sucessos em vez de somar dados, e suas lutas também podem fazer isso. `"kind": "dice-pool"` exige que `resolution.kind` seja `"dice-pool"` e lê a ficha como seus testes de reserva já leem: **todo número que uma rolagem soma é uma quantidade de dados, e todo número que ela enfrenta é uma contagem de sucessos.** Então nada é renomeado:

- Um número para acertar (o de uma linha de ataque, o `toHit` de uma lista de habilidades, o de uma ação de criatura) é a reserva que um ataque rola. O dado, o alvo que um dado precisa atingir e as faces em que ele dobra, explode, anula e falha criticamente são os da sua `resolution`. A trilha de ferimentos que seu `resolution.penaltyFrom` nomeia tira sua penalidade da reserva, como faz num teste.
- `defense` é quantos sucessos um ataque precisa, e nunca menos de um. O modificador de `defense` de uma condição, e `cover.bonus`, somam a ele.
- Cada sucesso além dos necessários adiciona um dado de dano. Uma falha crítica erra, seja o que for que tenha contado. Não há críticos: um acerto forte vale pelos dados extras.
- O número de uma salvaguarda é sua reserva, e sua dificuldade, os sucessos de que precisa. Uma disputa rola o teste de cada lado como reserva, e o lado com mais sucessos vence.
- O modificador de uma condição para ataques, salvaguardas ou testes é uma quantidade de dados somada ou retirada, então é um número `flat`: um modificador rolado (`dice`) é recusado.
- O dano é medido em dados do seu dado: o `dice` de uma quantidade (de uma criatura, de uma entrada) diz quantos, e precisa ser do seu dado, e sua parte `flat` são sucessos automáticos, nunca rolados. A coluna de dados de dano de uma linha de ataque é lida pela contagem, `damage.ability` soma a pontuação daquele atributo em dados, e `damage.bonus` soma sucessos automáticos. `perCostStep` e `scales` somam dados, como sempre. Cura e pontos temporários são quantidades e se somam como antes.
- A iniciativa é uma soma (`initiative.dice` e `modifier`) ou uma reserva, quando você pede uma (`initiative.pool`, abaixo), e uma regra de morte mantém os próprios dados.

O que uma luta de reserva rola além disso é definido pelo bloco `pool`:

```json
"pool": {
  "advantage": true,
  "damageTarget": 6,
  "soak": { "roll": true, "byKind": { "knock": { "abilityMod": "sinew" } } }
}
```

- `advantage`: se uma rolagem que pende para um lado é rolada duas vezes, mantendo a que teve mais sucessos (uma falha crítica conta como menos que qualquer rolagem sem falha crítica). Sem isso, uma luta rola uma vez, diga o que disser uma condição.
- `damageTarget`: o alvo por dado contra o qual são rolados os dados de dano e de absorção. É o alvo padrão da sua resolução quando você o omite. Esses dados contam uma vez cada face igual ou superior a ele e nada mais: nada dobra, explode, anula ou falha criticamente neles.
- `soak`: o que um alvo tira do dano de um acerto, por tipo de dano. `all` é uma referência de valor para qualquer tipo, e `byKind`, uma por tipo da sua trilha de saúde, que prevalece sobre `all` para o seu tipo; um tipo que nenhum dos dois nomeia não é absorvido. Com `"roll": true`, o alvo rola essa quantidade de dados contra o alvo de dano, e cada sucesso tira um. Com `"roll": false`, o número sai dos dados de dano antes de serem rolados, então nunca afeta sucessos automáticos, que a absorção rolada tira como qualquer outro. Nada fica abaixo de zero, e nada é rolado para absorver um golpe que não contou nada. Uma criatura define os próprios números como `soak: { "all": 1, "byKind": { "knock": 3 } }`, o que exige que o `soak` do bloco diga como a absorção é feita. Resistência, vulnerabilidade, imunidade e `resist-all` se aplicam depois da absorção, ao que restar.
- `hardness`: uma referência de valor para a dureza de um combatente, quando a iniciativa é um número que os ataques movem. Veja [Armaduras e efeitos de itens vestidos numa luta](#armor-and-worn-effects-in-a-fight).

O dano de cada alvo é rolado para aquele alvo, porque o que um acerto ganhou além dos sucessos necessários, e o que o alvo absorve, são dele. A segunda cláusula de um golpe, e um efeito adicional, são cada um uma reserva de dano própria, do seu tipo, sem dados extras do acerto. O menu prevê uma reserva com exatidão: a chance de alcançar os sucessos necessários e quanto valem os dados de dano depois do que o primeiro alvo absorve.

Gravewatch é escrito assim: seus guardiões rolam uma pontuação e um ofício, absorvem pancadas com Sinew e não absorvem rasgos, podem gastar um ponto de Resolve por turno e rolam a iniciativa de novo a cada rodada.

**Declarar em ordem inversa não está implementado.** Alguns sistemas fazem todos declararem sua ação antes de qualquer um agir, do mais lento ao mais rápido. Numa luta em que cada combatente escolhe uma ação quando chega seu turno, declarar primeiro não muda nada que alguma regra leia, então não há nada que uma chave possa dizer.

### Iniciativa que os ataques movem

Alguns sistemas de reserva mantêm a iniciativa como um número durante a luta inteira e deixam os ataques movê-lo: uma forma de atacar a tira do alvo, outra gasta a do próprio atacante como dano, e quem cai até uma linha definida entra em colapso. Uma luta `dice-pool` pode declarar isso (Capability API 1.48):

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

- **A abertura.** Todos rolam `pool` dados quando a luta começa, e os sucessos mais `plus` são o seu número; por isso `resource` exige `pool`: dados somados são uma ordem, não uma quantidade de dados. O `initiativeModifier` de uma criatura é sua reserva, já que todo número que uma luta de reserva soma é em dados. O número é mantido: nunca é rolado de novo (`each` é recusado ao lado de `resource`), e no início de cada rodada a ordem é organizada pelos números como estiverem.
- **Estilos.** Todo ataque (uma linha de ataque, uma habilidade ou entrada de catálogo que rola para acertar e causa dano, uma ação de criatura que faz isso e uma ação feita de outras ações) é oferecido uma vez por estilo, e o jogador escolhe o estilo antes do alvo. Até quatro, cada um toma ou gasta, e pelo menos um toma. Uma escolha que não nomeia estilo é feita no primeiro; uma num estilo em que o ataque não é oferecido é recusada. Um ataque mantém o estilo em que foi feito até terminar, mesmo quando uma resposta muda o número de quem o fez antes que ele acerte.
- **Um estilo que toma.** Num acerto, o dano é rolado como de costume (os dados extras do acerto, os dados da arma, a absorção por tipo, os sucessos automáticos), e o que ele conta sai do número do alvo em vez da saúde dele. O atacante ganha tudo isso mais `gain`.
- **Um estilo que gasta.** Oferecido só enquanto o número do atacante está acima da linha de colapso (0 quando você não define `crash`), e nunca para uma ação feita de outras ações, já que um número é gasto num único golpe. Num acerto, o dano é o número do atacante no momento do ataque, em dados contra o alvo de dano, e nada mais: sem dados da arma, sem dados extras do acerto, sem sucessos automáticos, sem absorção. Ele marca a saúde pelo próprio tipo de dano do ataque. Quando o ataque termina, um número que acertou alguém volta para `base`, e um que não acertou ninguém perde o que `onMiss` diz para o número com que foi feito (uma tabela de patamares: pares `[at least, lose]`, em ordem crescente; nada abaixo do primeiro patamar).
- **Colapso.** Um número que cai até `crash.at` ou abaixo entra em colapso: a `condition` (uma das suas, e opcional) é imposta a quem o tem, e quem o levou até ali ganha `bonus`. Um erro que leva o próprio autor ao colapso não dá bônus a ninguém. Um combatente em colapso não pode gastar. O colapso termina, com condição e tudo, quando o número dele volta a ficar acima da linha, depois de `recoverAfter` dos próprios turnos (o número volta para `base` no início desse turno) e quando a luta termina, seja como for. Quem tem a rolagem de abertura na linha ou abaixo começa a luta em colapso. `base` precisa ficar acima de `crash.at`, ou voltar a ela causaria um novo colapso.

O menu mostra a previsão de cada estilo: a chance de acertar e o que um estilo que toma tomaria ou o dano que um que gasta causaria. O painel de status mostra o número de todos, e o registro diz cada mudança e o motivo: "Ada gains 5 initiative for crashing Rats, and is on 14." O seletor do Engine avalia um golpe que toma com um turno de antecedência: tomar e depois gastar, contra gastar agora e de novo a partir da base.

<a id="what-a-fight-reads-from-mechanics"></a>

### O que um combate lê de `mechanics`

`kind` determina se `amount` é dano ou cura; qualquer entrada marcada como `reaction` fica fora do menu, assim como uma entrada `utility`, a menos que ela mude o que o próprio turno pode conter (veja abaixo). `attackRoll` faz uma rolagem contra a defesa do alvo com o `toHit` da lista; `autoHit` pula isso inteiramente. `save` rola a salvaguarda do próprio alvo contra a `saveDifficulty` da lista, e `onSuccess` determina se um sucesso recebe metade ou nada. `targetCount` indica quantos alvos ela pode atingir. Uma habilidade que não rola ataque (uma área contra a qual todos fazem salvaguardas, algo que simplesmente acerta) rola seus dados UMA VEZ para todos eles; uma que rola para acertar cada alvo rola seus dados novamente a cada acerto. `applies` impõe condições ao que afeta, cada uma com uma `duration` de `instant` (sem relógio próprio: permanece até que algo a remova), `until-save` (que precisa de `saveEnds` ao lado) ou `{ "rounds": n }`, e um `saveEnds` opcional que nomeia a salvaguarda e indica se ela se repete em `turn-end` ou `turn-start`. As rodadas contam os próprios turnos de quem a porta, diminuindo ao fim de cada um; `{ "rounds": 1, "at": "turn-start" }` as conta no início de cada um, que é como se diz até o início do seu próximo turno. `endsAfter` remove a condição depois do primeiro `own-attack` (a próxima rolagem de ataque de quem a porta), `attacked` (a próxima rolagem de ataque feita contra quem a porta) ou `own-save` (a próxima salvaguarda de quem a porta), diga o que disser o relógio dela; assim, `{ "duration": { "rounds": 1 }, "endsAfter": "own-attack" }` significa no próximo ataque, antes do fim do próximo turno. Uma condição que dura um ataque dura o ataque inteiro, inclusive o dano, e uma nova que o mesmo golpe impõe permanece. Ambos exigem Capability API 1.45. `temporary` concede pontos temporários à reserva de saúde, e eles nunca se acumulam: permanece a maior proteção. `scales` aumenta a quantidade pelos DADOS extras que sua tabela indica para o valor lido. `cost` é pago pelo comando `use` da própria ficha, e `budget` substitui qual parte da economia é gasta.

`plus` contém até três quantidades A MAIS no mesmo golpe, ao lado de `amount`, cada uma com sua própria rolagem e tipo ("and 2d6 fire"). Uma cláusula é `{ "dice": "2d6", "flat": 1, "type": "fire" }` e pode carregar sua própria `save`, `{ "save": "con_save", "difficulty": 13, "onSuccess": "none" | "half" }`, que o ALVO rola independentemente do que a ação já lhe pediu: `none` elimina toda essa cláusula em um sucesso e `half` deixa metade, enquanto o restante do golpe permanece intacto nos dois casos. Sem `difficulty`, ela recorre ao número usado pela salvaguarda da própria ação e, depois, à `saveDifficulty` da lista. Um crítico dobra os dados de cada cláusula pela mesma regra que dobra os da primeira quantidade; uma cláusula sem `type` usa o tipo de dano do próprio golpe, e o golpe inteiro continua sendo UM teste de concentração, com o dano somado, e um teste para cair. Uma cláusula precisa de um `amount` para acompanhar, e um `heal` não carrega nenhuma.

```json
{
  "kind": "attack",
  "attackRoll": true,
  "amount": { "dice": "1d8" },
  "damageType": "piercing",
  "plus": [{ "dice": "2d6", "type": "fire" }]
}
```

Três chaves dizem o que uma entrada faz com a economia do próprio turno, e uma entrada `utility` que declara qualquer uma delas é oferecida em vez de descartada:

- `free`: não custa orçamento algum. Ainda paga o `cost` que indicar e não pode também indicar um `budget`.
- `gives`: `[{ "budget": "action", "count": 1 }]`, até quatro. Usá-la adiciona a esses orçamentos imediatamente, com limite igual ao que um turno comporta mais a concessão, para que nada possa ser guardado para um turno posterior.
- `standard`: `{ "actions": ["dash", "disengage", "hide"], "budget": "bonus" }`. Seu portador pode executar essas ações padrão com ESSE orçamento. Elas são oferecidas ao lado das comuns como `standard:<id>@<budget>`, e a própria entrada fica fora do menu quando é apenas essa permissão, pois uma permissão não é algo que alguém execute.

Uma entrada do novo `kind: "rider"` é PASSIVA: ninguém a executa, ela nunca aparece no menu e adiciona automaticamente mais uma cláusula de dano ao primeiro acerto qualificado de um período. Ela carrega `rider` e nada mais que possa ser executado:

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

`sources` nomeia as listas de ataques de onde ela vem e `requires` uma coluna verdadeira de suas linhas, de modo que um efeito adicional que só dispara com certas armas pode especificá-las sem que o Engine saiba o que é uma arma; não nomear nenhum significa qualquer acerto de seu portador. Uma arma empunhada tirada da bolsa não é uma linha, então `weapons` (Capability API 1.70) também nomeia as tags de item com que ele dispara: `"weapons": { "tags": ["finesse",
"ranged"] }` dispara num acerto com uma arma empunhada que tenha qualquer uma das tags, inclusive o golpe com a mão inábil. Nomear só `weapons` não dispara com nenhuma linha, e nomear só linhas não dispara com nenhuma arma empunhada. `when` aceita QUALQUER das opções: `advantage` é a inclinação final da rolagem de ataque, e `ally-adjacent` é um aliado do atacante que esteja de pé e possa agir, a uma célula do alvo em um tabuleiro ou em qualquer lugar sem tabuleiro. `oncePer` é `turn` (renovado no começo de todos os turnos, de modo que um golpe desferido enquanto outra pessoa age ainda pode carregar um) ou `round`. `amount` cresce com o `scales` da própria entrada, e `type` é o tipo de dano, por padrão o do próprio golpe.

<a id="creatures-a-bestiary-a-fight-reads"></a>

### Criaturas: um bestiário que o combate lê

Um catálogo que declara `"holds": "creatures"` contém oponentes em vez de linhas de ficha. Ele não alimenta nenhuma lista, o seletor do editor de fichas nunca o oferece e todos os seus números usam as chaves que seu bloco `combat` já declara. Ele precisa de um bloco `combat` e de uma escala `threat`, pois uma criatura é classificada em um dos seus próprios níveis.

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

Os números abaixo são a maneira direta de escrever uma criatura. Uma escrita nos termos do seu próprio conjunto de regras, como `sheet`, obtém `health`, `defense`, `initiativeModifier`, `speed`, `abilities`, `saves`, `checks`, `soak` e `hardness` dessa ficha (veja "Uma criatura escrita nos termos do seu conjunto de regras", abaixo).

- `health`: um número, ou `{ "dice": "3d6", "flat": 2 }`, rolado uma vez quando o combate é criado. Uma previsão lê a média, para que um menu nunca prometa um dado que ninguém rolou.
- `defense`, `initiativeModifier`, `speed`: contra o que um ataque é rolado, o que é somado à iniciativa (ou, quando a iniciativa é rolada como reserva, quantos dados ela rola) e quanto ela anda em um turno, na sua própria unidade de distância.
- `abilities` e `saves`: usam como chaves os identificadores de atributos e salvaguardas que sua ficha declara. Uma salvaguarda ausente é lida como zero.
- `checks`: o que ela soma numa disputa, com os IDs de `combat.checks` como chaves. Um que ela não nomeia é lido como zero. Capability API 1.43.
- `soak`: só numa luta `dice-pool`, o que ela absorve: `all` para qualquer dano, `byKind` para um tipo da sua trilha de saúde. Capability API 1.47.
- `hardness`: quando a iniciativa é um número que os ataques movem, um golpe que gasta cujos dados ficam abaixo dela não faz nada. Capability API 1.56.
- `resist`, `vulnerable`, `immune`: tipos de dano, comparados sem distinguir maiúsculas, e validados contra `combat.damageTypes` quando você declara algum. Uma entrada `resist` ou `immune` também pode dizer o que a atravessa, `{ "type": "tearing", "except": ["silver"] }`: um golpe de uma arma que tenha uma dessas tags de item é recebido por inteiro (Capability API 1.55, veja [Armas numa luta](#weapons-in-a-fight)). `conditionImmunities` nomeia suas próprias condições.
- `tier`: a qual degrau de `combat.threat` ela pertence.
- `loot` (Capability API 1.63): o ID da tabela de loot que uma luta vencida rola para ela (veja Loot, acima).
- `traits`: pares curtos de nome e texto mostrados ao Game Master. Nunca são resolvidos; portanto, qualquer coisa com números pertence a uma ação.
- `signaturePoints`: pontos devolvidos no início do próprio turno, gastos em ações `signature`.
- `riders`: até quatro, iguais ao `rider` de uma entrada de catálogo, escritos no bloco. Cada um é `{ "id": "pack", "name": "Pack", "on": "hit", "oncePer": "turn" | "round", "amount": { "dice": "1d6" } }`, com um `type` opcional e um `actions` opcional que nomeia em quais ações do próprio bloco ele dispara. O efeito adicional de um bloco não lê listas de ficha; por isso, `sources` e `requires` são as duas chaves que ele não possui. Uma criatura escrita como ficha recebe os efeitos adicionais de suas listas, exatamente como um personagem.
- `actions`: até doze, cada uma com seu próprio `id`. Uma ação carrega o que um bloco de estatísticas escrito à mão carrega (`toHit`, `autoHit`, `damage`, `save`, `applies`, `targetCount`, `reach`, `range`, `area`), mais quatro coisas exclusivas das criaturas. `reach` indica até onde ela golpeia, `range` até onde é arremessada ou disparada e `area` a forma em que atinge, tudo na sua própria unidade de distância; `range` pode ser um número simples ou `{ "normal": 30, "long": 120 }` quando ainda alcança mais longe com uma penalidade, e `area` é `{ "shape": "burst" | "cone" | "line", "size": n, "friendlyFire": false }` (veja Posições):
  - `uses`: `{ "per": "encounter" | "day", "count": n }`. Quando se esgotam, a ação sai do menu.
  - `recharge`: `{ "dice": { "count": 1, "sides": 6 }, "from": 5 }`. Ela começa o combate disponível, é gasta ao ser usada e rola no início do próprio turno da criatura: `from` ou mais a recupera. O registro contém os dados nos dois casos.
  - `sequence`: outras ações do mesmo bloco, em ordem, cada uma com seu próprio alvo. **É assim que se escreve uma criatura que golpeia duas vezes em uma ação.** Um orçamento paga a sequência inteira. Uma sequência não carrega nada próprio e nunca pode nomear outra sequência.
  - `signature`: `{ "cost": n }`, comprada com os pontos da própria criatura em vez de um orçamento e apenas enquanto outra pessoa age: o combate a oferece na janela entre um turno e o seguinte (veja Janelas).
  - `reaction`: o mesmo objeto que o `mechanics.reaction` de uma entrada de catálogo (veja Janelas): a ação não aparece no menu de nenhum turno e é oferecida no momento que ela nomeia. Uma reação não é também uma ação característica, e nenhuma sequência pode incluir uma.
  - `self: true`: ela atinge a própria criatura em vez de outra pessoa, e por isso não aceita `targetCount` nem `area`. Ambos exigem Capability API 1.46. Um Parry é as duas coisas, com uma condição `parrying` que soma 2 à defesa:

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
- Uma salvaguarda precisa de uma dificuldade na própria ação: `save.difficulty` para uma salvaguarda imposta pela ação, ou `saveDifficulty` para uma condição que termina com uma salvaguarda quando a ação não tem a sua própria. Uma ação de bloco é escrita com números simples mesmo em uma criatura com ficha; portanto, esse número fica na ação. A salvaguarda de uma cláusula pode omitir sua `difficulty` e recorrer ao mesmo número.
- `damage.plus` é a mesma lista de cláusulas que o `plus` de uma entrada de catálogo e é lida exatamente da mesma forma: `"damage": { "dice": "1d6", "flat": 2, "type": "piercing", "plus": [{ "dice": "1d4", "type": "fire" }] }` é uma mordida cujo calor tem sua própria quantidade, resistida e dobrada separadamente.

O bestiário do rascunho de 5e contém cinco criaturas escritas à mão em `docs/development/ruleset-5e-2014.example.json`, cobrindo uma sequência, uma recarga, uma salvaguarda com condição, resistências e imunidades, usos limitados, pontos de ações características e uma criatura escrita como ficha.

#### Uma criatura escrita nos termos do seu conjunto de regras

Uma criatura não precisa ser escrita com números simples. Dê a ela uma `sheet`, com exatamente a estrutura de uma ficha de personagem, e o combate a constrói como um membro do grupo: sua saúde, defesa, salvaguardas, iniciativa, velocidade e todos os ataques e habilidades das listas são o que suas próprias fórmulas de ficha determinarem. É assim que um conjunto de regras indica que seus oponentes têm os mesmos atributos, perícias e listas que seus personagens, sejam quais forem. O Toll Warden de Ember Roads:

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

- **Todas as partes são opcionais**: `abilities`, `skills`, `saves`, `bonuses`, `fields` e `lists`, com os identificadores declarados pela ficha como chaves. Tudo que for omitido usa o padrão da própria ficha, exatamente como em um personagem em branco. O Grit do guardião é 9 porque seu `grit_max` soma 4, a Toughness e o Brawn dele, e sua Guard é 7 por um motivo semelhante.
- **Cada número tem uma única origem.** Uma criatura com ficha não fornece também `health`, `defense`, `initiativeModifier`, `speed`, `abilities`, `saves` ou `checks`, e o Engine recusa o arquivo se ela fizer isso. Ela pode não ter `actions` próprias, pois suas listas definem o que faz. Uma criatura sem ficha ainda fornece os três primeiros e pelo menos uma ação.
- **Ela é validada como os dados de autoria que é.** Cada identificador deve ser declarado pela ficha; uma perícia ou salvaguarda recebe um dos níveis de proficiência oferecidos para ela; um campo, pontuação, bônus ou coluna contém o que declara aceitar (um inteiro dentro do intervalo, um de seus valores e assim por diante), e uma lista não contém mais linhas do que permite. Não há parte `live`, pois o combate guarda o que a criatura gastou.
- **Uma linha pode vir de um catálogo.** `_catalog: "<catalog>/<entry>"` nomeia a entrada de onde uma linha foi escolhida, como na ficha de um personagem, e é nessa entrada que o combate lê o que a linha custa e faz. O catálogo precisa alimentar essa lista. Quando o catálogo está escrito no próprio arquivo, a entrada deve estar nele; quando tem seu próprio arquivo, uma linha que nomeia uma entrada ausente do arquivo simplesmente não dá nada à criatura. Os catálogos nomeados pelas fichas de um bestiário são carregados para o combate junto com o bestiário.
- **O que a entrada declara ao lado da ficha ainda conta**: `tier`, `traits`, `actions`, `signaturePoints`, `riders`, `resist`, `vulnerable`, `immune` e `conditionImmunities`.
- **Ela paga com suas próprias reservas.** Elas começam cheias, são gastas no que suas listas fornecem, e as formas maiores de pagamento (uma magia com um espaço superior) são oferecidas exatamente como para um membro do grupo, seja o Engine ou o Game Master quem decide por ela. Hold the Line custa Luck ao guardião.
- **Com uma trilha de ferimentos, a saúde é a trilha.** Um golpe marca a própria trilha da criatura segundo seus `damageKinds`, depois da aplicação de `resist`, `vulnerable` e `immune`, de modo que uma criatura imune a um tipo de dano não recebe nenhuma marca dele.
- **Ela continua sendo um oponente.** A zero, fica fora de combate em vez de morrer aos poucos; nunca rola contra a morte; a tela mostra o que sempre mostrou de um oponente e nada da ficha, e nada do que ela gastou é gravado de volta em outro lugar, mesmo quando um personagem compartilha seu nome.
- **Uma ficha cuja saúde total seja zero** fica fora do combate, e o registro inicial explica por quê, em vez de introduzir algo que ninguém pode ferir.
- Uma camada que remova um valor de um dos campos enumerados nunca elimina uma criatura que o use: enquanto a camada estiver ativa, esse campo usa seu padrão para a criatura, exatamente como para um personagem, e a criatura não é recusada por isso.
- Um Game Master também pode inventar uma, que fica limitada ao seu nível (veja Oponentes que ninguém escreveu).
- Um pacote que inclua uma declara Capability API 1.34.

O Toll Sergeant do rascunho de 5e é a mesma coisa em uma ficha d20: sua Armor Class, pontos de vida, salvaguardas e dois golpes por ação vêm de seus próprios campos e de sua lista de ataques.

#### Oponentes que ninguém escreveu

Quando um Game Master inventa um oponente, o Engine ajusta a proposta à sua escala `threat` antes de qualquer rolagem: a saúde fica na faixa do nível; defesa, ataque e dificuldades de salvaguarda ficam no máximo dois acima dos valores do nível; e o dano é reduzido até que a melhor rodada da criatura (sua sequência mais forte ou ação individual mais forte, medida contra um alvo) caiba no `damagePerRound` do nível. Primeiro reduz a quantidade de dados, depois a parte fixa, depois um golpe de uma sequência e só então o tamanho do dado, sem nunca reduzir nada a zero. Nomes ausentes do seu conjunto de regras são descartados: tipos de dano, condições e salvaguardas desconhecidos, e qualquer ação além das seis primeiras. Um nível que você nunca declarou recorre à base da escala. Cada alteração retorna como uma frase simples, para que o registro possa dizer o que foi feito.

Uma invenção também pode ser escrita como `sheet`, da mesma forma que uma criatura de bestiário; é assim que um mago inventado recebe espaços e magias. O Game Master vê os identificadores da ficha e o que cada um pode conter, as listas lidas pelo combate e os nomes que seus catálogos oferecem para elas, para que uma magia seja nomeada em vez de descrita: uma linha que nomeia uma entrada de catálogo, com qualquer combinação de maiúsculas, torna-se essa entrada, e os valores próprios do Game Master (como uma magia estar preparada) são aplicados por cima. A ficha é lida com tolerância, pois foi escrita por um modelo: um nome ausente do seu conjunto de regras é descartado, um valor é ajustado ao seu campo ou coluna e os números escritos ao lado da ficha não são usados.

Uma criatura inventada que não seja um chefe fica limitada ao que seu conjunto de regras lhe abre. Um filtro de catálogo com `startFrom` nomeia o campo de ficha pelo qual suas entradas são organizadas (a lista de magias do pacote 5e por `class`), e uma criatura inventada mantém apenas as entradas cujo filtro corresponde ao seu próprio valor desse campo, comparado como o seletor faz ao abrir. Assim, um Sorcerer nunca tem a lista inteira de magias, e quem não nomeia uma classe não recebe nada de um catálogo organizado por classe. As escolhas que ficaram abertas são preenchidas sem perguntar novamente ao Game Master: para cada lista cujas linhas só contam depois de escolhidas (`onlyWhen` em uma fonte de habilidades de combate), entre as entradas disponíveis que pode pagar com suas próprias reservas, cada reserva recebe um pequeno número de entradas (mais para uma criatura mais competente), assim como aquilo que pode usar à vontade. O que ela recebe depende do temperamento e da competência, os mesmos com que luta: uma criatura protetora ou de apoio busca o que sustenta seu lado; uma imprudente, dano; uma metódica ou paciente, o que detém o inimigo; e quanto mais competente for, maior a chance de carregar uma reação, uma contra-ação ou qualquer outra coisa que altere o turno. A seleção usa a semente do próprio combate, de modo que o mesmo combate sempre é preenchido da mesma maneira. Uma linha nomeada pelo Game Master de uma lista assim conta como escolhida.

Um chefe fica inteiramente a cargo do Game Master, como a exceção que pode ser: nada é retirado e nada é preenchido.

Depois, ambos ficam limitados ao seu nível:

- A saúde entra na faixa do nível pelo único campo do qual ela é lida: o máximo da reserva é esse campo ou uma `sum` com exatamente um campo (o máximo de pontos de vida de 5e, a Toughness de Ember Roads). Uma fórmula de saúde sem um campo único fica como foi escrita, e o registro informa isso. O comprimento de uma trilha de ferimentos é seu e nunca é alterado.
- Depois que a criatura é construída, defesa, ataque e dificuldades de salvaguarda ficam limitados a dois acima dos valores do nível, e o dano é reduzido até que sua melhor rodada caiba no `damagePerRound` do nível, contando o maior pagamento que ela pode fazer. Primeiro é reduzido o que um pagamento maior compra, depois os dados, a parte fixa, um golpe e só então o tamanho do dado.

Seu próprio bestiário nunca é limitado. São dados que você escreveu, portanto o Engine os aceita como estão.

### Posições: um combate no tabuleiro

Um combate acontece no teatro da mente até que seu bloco diga quanto vale uma célula do tabuleiro. Declare `distance` e ele poderá acontecer em uma grade; então movimento, alcance, distâncias, áreas, linha de visão, cobertura e golpes contra quem se afasta passam a ter significado. Cada um é um número que você escreveu; o Engine fornece o tabuleiro e nada mais.

```json
"distance": { "label": "ft", "perCell": 5 },
"ranged": { "long": "disadvantage", "adjacentFoe": "disadvantage" },
"cover": { "bonus": 2 },
"opportunity": { "budget": "reaction" }
```

Ember Roads declara uma linha disso e nada mais; esse é o objetivo: o restante não é obrigatório.

```json
"distance": { "label": "paces", "perCell": 2 }
```

**A célula.** `distance.perCell` indica quanto da SUA unidade vale uma célula, e `label` é o nome dessa unidade. Todas as distâncias do mundo do bloco a usam: `economy.movement`, a `speed` de uma criatura, o `reach` e o `range` de uma arma e o `reach` e o `range` de uma ação de criatura. Um catálogo que declara seu próprio `units.distance` converte seus `mechanics.range` e `area.size` com seu próprio `perCell`; um que não declara usa este. Uma distância acima de zero é arredondada para a célula mais próxima, nunca para nenhuma; assim, tudo a que você deu um número alcança pelo menos uma. Zero não é uma distância curta e mantém seu próprio significado: um `mechanics.range` de 0 significa si mesmo ou toque (e tocar outra pessoa alcança a célula vizinha), e uma coluna de arma `reach` ou `range` com valor 0 em uma linha significa que essa linha não tem essa distância.

**Se o combate usa um tabuleiro.** Duas coisas precisam coincidir: seu bloco declara `distance` e a partida do jogador usa o estilo de combate Tactical. Com o estilo Classic, ou um conjunto de regras sem `distance`, o combate continua no teatro da mente: qualquer um pode mirar em qualquer um, e nada abaixo é lido.

**O que o jogador vê.** O tabuleiro é desenhado com o terreno do próprio estilo tático. Cada quadrado é um botão, acessível pelo ponteiro ou pelas teclas de direção, e informa o que é, quem está nele e o que a escolha parcialmente feita significa para ele. Caminhar ilumina os quadrados oferecidos pelo menu, cada um com seu custo NA SUA UNIDADE, desenha o caminho e marca em âmbar qualquer quadrado cujo caminho provocaria um golpe de alguém, nomeando essa pessoa abaixo do tabuleiro. Uma opção que recebe um alvo ilumina quem pode ser escolhido, no tabuleiro e na lista ao mesmo tempo. Uma opção com `area` mira em um quadrado, e o quadrado sob o ponteiro informa quem seria atingido, incluindo aliados. O restante do deslocamento permitido aparece ao lado dos orçamentos, também na sua unidade. A tela não mede nada: cada quadrado, custo, caminho, alvo e ponto de mira é enviado pelo servidor.

**Movimento.** O deslocamento permitido de um turno é `economy.movement` para um membro do grupo, ou a `speed` da própria criatura, dividido por `perCell` e arredondado PARA BAIXO, nunca menos de uma célula enquanto ela puder se mover. Ele é reposto no início do próprio turno de seu portador e pode ser gasto antes, entre e depois das ações: caminhar, golpear, caminhar de novo. Entrar em uma célula custa um, ou mais em terreno difícil. Oito direções, todas com o mesmo custo, pois é assim que se jogam as grades de RPG de mesa a que isso se destina. É possível passar por um aliado, mas não parar sobre ninguém; um oponente é uma parede; não se pode entrar em nada sólido nem cortar um canto entre duas células sólidas.

**Alcance e distância.** Uma linha de arma os obtém de `combat.attacks[].reach` e `.range`, cada um sendo uma coluna da mesma lista ou o mesmo número em todas as linhas:

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

Uma coluna com valor 0 em uma linha indica que essa linha não possui tal distância; assim, uma espada comum pode estar na mesma lista que um machado de arremesso. Uma linha sem alcance chega a uma célula. Uma ação de criatura usa seu próprio `reach` ou `range`, e uma habilidade de catálogo usa `mechanics.range` (0 significa si mesmo ou toque, equivalente a uma célula quando mira em outra pessoa).

Uma linha com AMBOS é uma arma de arremesso: dentro do alcance, é um golpe; além dele, um disparo. Portanto, as regras abaixo para disparos não a afetam na mão de alguém, e ela serve para golpear quem passa, algo que um arco não faz.

Uma ação de criatura também pode carregar a `area` em que atinge, na sua própria unidade: `{ "shape": "cone",
"size": 15 }`, com `"friendlyFire": false` para poupar seu lado. É assim que uma arma de sopro se torna um cone real no tabuleiro em vez de um número de alvos. Uma sequência não tem forma própria; as ações que nomeia têm as suas. Um combate sem tabuleiro ignora a forma e usa `targetCount`, de modo que uma entrada de criatura pode carregar ambos e ser fiel nos dois casos.

**Até onde uma forma pode ser enviada.** `range` informa isso: uma bola lançada a cem pés possui um. Sem distância, uma explosão ocorre onde é colocada, na célula do próprio ator, e um cone ou linha pode mirar em qualquer lugar dentro do comprimento que desenha, pois ali a célula só indica a direção. Isso vale tanto para o `mechanics.area` de uma entrada de catálogo quanto para a área de uma criatura.

`ranged` indica o custo de um disparo feito além de sua distância comum `normal`, ou com alguém do outro lado na célula vizinha. Cada um é `"disadvantage"` ou `"normal"`; omita o bloco e nenhum custa nada. Um golpe nunca é um disparo, então nenhuma regra o afeta, nem uma arma de arremesso usada dentro do próprio alcance.

**Áreas.** O `mechanics.area` de uma entrada se torna uma forma real no tabuleiro, mirando em uma célula em vez de alguém, e `targetCount` não determina nada: a forma decide quantos alcança. Todos que estiverem nas células são atingidos, amigos e inimigos, a menos que a entrada diga `"friendlyFire": false`.

```
burst, size 2, aimed at X        cone, size 3, aimed right      line, size 3, aimed right
. . . . .                        . . . .                        . . . .
. # # # .                        . . # .                        A # # #
. # X # .                        A # # #                        . . . .
. # # # .                        . . # .
. . . . .                        . . . .
```

Uma explosão abrange todas as células dentro do seu tamanho ao redor da célula visada. Um cone parte do ator em direção a essa célula, com largura em cada passo igual à distância percorrida. Uma linha segue da mesma forma, com uma célula de largura. Os três param em qualquer coisa sólida.

**Linha de visão e cobertura.** Uma linha reta de células entre os dois: qualquer coisa sólida nela bloqueia um disparo e impede que uma área se espalhe além, e o alvo simplesmente não aparece no menu. O terreno que oferece cobertura adiciona `cover.bonus` à defesa contra a qual o ataque é rolado, e o registro informa isso. Não há cobertura de três quartos, cobertura total nem elevação.

**Golpes contra quem se afasta.** Declare `opportunity.budget` e, quando um combatente caminhar para fora do alcance de um inimigo de pé que possa agir, tenha esse orçamento e tenha algo corpo a corpo com que golpear, a caminhada PARA onde está e esse inimigo é consultado sobre golpear. Aceitar gasta o orçamento e resolve exatamente como o mesmo ataque no próprio turno; deixar passar não custa nada. Nos dois casos, a caminhada continua de onde foi interrompida, pagando cada célula realmente cruzada, e um golpe que derruba quem se move encerra a caminhada onde caiu. Uma oportunidade para cada inimigo na caminhada inteira, independentemente de quantas vezes o caminho saia do mesmo alcance. `disengage` impede isso pelo restante do turno, e um conjunto de regras sem `opportunity` não tem nada disso.

A pergunta é uma JANELA e segura o combate inteiro: nada mais se move até que todos os consultados respondam. O jogador responde à janela de um membro do grupo, com o golpe ou **Pass** (passar) ao lado; as demais são respondidas por quem interpreta o combatente, e um chefe do Game Master pela decisão do próprio Game Master. Veja Janelas abaixo.

**O que um oponente faz com o tabuleiro.** Um oponente que ninguém interpreta avalia cada célula alcançável contra cada opção que poderia executar dali, desconta cada golpe que a caminhada provocaria e prefere não se mover quando já pode fazer o melhor de onde está. Sem nada ao alcance, aproxima-se e primeiro corre se sua lista `standard` tiver `dash`.

**Recusas que você pode ver.** `out-of-reach` (mais longe do que alcança), `no-line-of-sight` (algo sólido no caminho), `unreachable` (uma célula cujo custo a caminhada não pode pagar ou onde não pode terminar) e `bad-cell` (uma forma mirando em um lugar proibido).

### O que o combate faz com seu bloco no servidor

Uma partida cujo conjunto de regras declara `combat` recebe um combate resolvido por ele, na mesma batalha salva que o Engine sempre usou:

- **Quem participa.** O Game Master diz quem está lutando; o Engine lê os números de cada membro do grupo em sua própria ficha. Um membro sem ficha para seu conjunto de regras é recusado pelo nome, em vez de receber números que você não escreveu.
- **De onde vêm os números de um oponente**, nesta ordem: a criatura nomeada pelo Game Master no bestiário; depois uma cujo rótulo corresponda ao nome do oponente; depois um bloco de estatísticas proposto pelo Game Master para esse combate, ajustado à sua escala de ameaça; por último, uma criatura simples construída com os números do próprio degrau. Cada alternativa e cada ajuste são registrados em palavras simples para que o combate possa explicar o que fez. Um conjunto de regras sem entrada de bestiário, proposta ou escala de ameaça recusa o combate em vez de inventar um.
- **Suas fichas são o registro.** Saúde, reservas, condições, concentração e contadores da sua regra de morte são gravados pelas regras da própria ficha após cada ação aceita; assim, recarregar no meio do combate mostra exatamente o que ele deixou, sem uma contagem ao final que possa discordar.
- **Seu menu é a única fonte de legalidade.** Todo aquele que age, jogador ou oponente, escolhe um identificador no mesmo menu produzido pelo seu bloco. Um oponente controlado pelo Engine escolhe nele com as táticas do próprio Engine; um controlado pelo Game Master é instruído a escolher um identificador desse mesmo menu, vendo seus números sem jamais saber o que os dados farão.
- **Seus dados.** Um combate carrega sua própria semente e um cursor; portanto, um combate lido novamente do disco continua com os dados que teria rolado.

### Na tela

O combate acontece na tela de batalha com suas palavras. O menu contém seus ataques, suas habilidades, suas disputas e as ações padrão que você listou, cada uma informando o que gasta dos seus orçamentos e reservas. São mostrados a ordem dos turnos, a rodada, cada condição nomeada com suas rodadas restantes, os pontos temporários, a concentração e os dois contadores da sua regra de morte. O registro imprime a aritmética real nos seus termos: "Juno attacks Rust jackal with Road axe: 8 (5 + 3) + 3 = 11 against Guard 6, a hit." Cada ação aceita é gravada na ficha quando acontece; assim, recarregar no meio do combate é exato, e depois o Game Master é instruído a não alterar esses números novamente.

Um combate com posições é desenhado no tabuleiro em vez do palco de retratos; veja Posições para saber o que o jogador faz com ele. Toda distância nele, no menu e no registro, é expressa na SUA unidade: "Juno moves to 4, 6 for 6 paces and has 2 paces left."

### Disputas: agarrar, empurrar, se soltar

Alguns movimentos não são um ataque contra uma defesa, mas uma disputa: os dois lados rolam, e quem se sai melhor consegue o que quer. Agarrar alguém, derrubá-lo ou empurrá-lo para longe e se soltar têm todos a mesma forma, então um conjunto de regras declara cada um deles como dados:

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

- **`checks`** são os números que uma disputa lê, cada um um valor da ficha, lido uma vez quando a luta começa, como uma defesa ou uma salvaguarda. Uma criatura escrita com números simples define os próprios (`"checks": { "brawn": 2 }`); uma com ficha os lê dela. Até doze.
- **A rolagem.** Os dois lados rolam seus `attackRoll.dice` e somam o melhor dos testes que podem usar ali (`attacker.checks`, `defender.checks`). O maior total vence; um empate fica com o defensor, a menos que `ties` diga `"attacker"`. O registro mostra os dois lados: "Juno tries Grab on Ash-hound: 12 (6 + 6) + 3 = 15 with Brawn against 2 (1 + 1) + 2 = 4 with Brawn, and wins."
- **Vencer** faz o que `onWin` diz, pelo menos uma coisa:
  - `applies` impõe condições a quem perde, com o vencedor como origem, então uma condição que você marca com `endsWhenSourceDown` termina quando quem está segurando cai. `rounds` dá um relógio à condição; sem isso, ela dura até que algo a encerre.
  - `ends` remove condições do ator ou do alvo.
  - `push` move quem perde diretamente para longe do vencedor, essa distância na sua unidade, parando antes de qualquer coisa sólida, de alguém de pé, da borda do tabuleiro e de um canto apertado demais para passar. É forçado: não gasta movimento de quem é empurrado, e ninguém o golpeia por isso. Uma luta sem tabuleiro não move ninguém. Perder não faz nada, que é o que um agarrão falhado é.
- **`from: { "holding": "held" }`** faz uma disputa mirar só em quem impôs essa condição ao ator e a coloca no menu só enquanto ela vale: é assim que se expressa se soltar.
- **`reach`**, na sua unidade de distância, é até onde ela alcança; a célula vizinha, quando você o omite. `reach` e `push` exigem `combat.distance`.
- **`strike: true`** permite que ela ocupe o lugar de um ataque quando uma ação compra vários, como faz um ataque: escolhida primeiro, gasta o orçamento e deixa o restante dos ataques disponíveis; escolhida com ataques disponíveis, custa um deles. A referência 5e agarra e empurra assim.
- **No menu**, uma disputa tem seu próprio grupo e informa sua chance de vencer. Um oponente jogado pelo Engine a avalia como qualquer outra coisa, mas com moderação: um agarrão ou um empurrão prepara algo, e se soltar vale mais, só enquanto estiver agarrado. Um Game Master que interpreta um oponente escolhe uma no mesmo menu.
- Um oponente que um Game Master inventa tem seus testes limitados ao seu nível, como sua chance de acertar.

Disputas exigem Capability API 1.43 num conjunto empacotado.

<a id="windows-holding-the-fight-open"></a>

### Janelas: manter o combate aberto

Alguns momentos pertencem a alguém que não é quem está agindo. O Engine mantém o combate aberto para essa pessoa em vez de decidir por ela, e essa pausa é uma janela.

Seis coisas abrem uma, e duas vêm do que você já declarou:

- **Alguém se afasta.** Uma caminhada que sai do alcance de um inimigo capaz de golpear para nesse passo e pergunta a ele. Veja Golpes contra quem se afasta, acima.
- **Entre um turno e o próximo.** Quando um turno termina, cada oponente com `signaturePoints` que pode pagar uma de suas próprias ações `signature` é consultado sobre comprá-la, antes do próximo turno começar. Esse é o único momento em que elas são compradas: uma ação característica não aparece no menu de turno de ninguém, nem no próprio.
- **Alguém usa algo.** Antes de resolver, todos do OUTRO lado que possuem uma entrada esperando esse momento são consultados, seja quem for o alvo, desde que essa entrada alcance quem está usando. Uma ação padrão e uma disputa não abrem esse momento.
- **Algo mira em alguém.** Antes de resolver, todos os alvos do OUTRO lado que possuem uma entrada esperando esse momento são consultados. Um amigo curando você não é uma ameaça à qual responder, então a ação de um amigo não abre uma janela.
- **Uma rolagem de ataque acerta alguém.** Antes do dano, quem foi atingido é consultado, quando tem uma entrada esperando esse momento. O que essa pessoa faz em resposta conta para este ataque: veja O momento depois de um acerto, abaixo.
- **Algo feriu alguém.** Depois de resolver, todos que receberam dano e possuem uma entrada esperando ESSE momento são consultados, independentemente de quem causou. Ser ferido é um fato sobre você; uma entrada direcionada de volta a quem causou isso ainda não pode mirar em um amigo.

As quatro últimas são as que uma entrada de catálogo, ou a própria ação de uma criatura, solicita ao nomear o momento que espera. Quando uma ação abre tanto `used` quanto `aimed`, o uso é consultado primeiro; se ninguém o cancelar, os alvos são consultados em seguida, e ela se resolve depois que os dois forem respondidos. Ser atingido é consultado conforme cada uma das suas rolagens acerta, e ser ferido depois que a ação inteira termina.

O que uma janela faz, seja qual for sua origem:

- **Nada mais se move enquanto ela está aberta.** Nem o ator de quem é o turno, nem o fim desse turno, nem outra janela. O combate espera.
- **Ela pergunta a um de cada vez**, na ordem dos turnos, uma vez a cada um. Passar sempre é uma resposta e não custa nada. Quem seria consultado e não tem nenhuma opção que possa executar é pulado em vez de consultado.
- **Ela continua exatamente de onde parou.** Uma caminhada termina pelas células restantes, pagando cada uma que realmente cruzou.
- **Quem responde é quem interpreta.** A janela do seu membro do grupo é sua, com a opção e **Pass** (passar) ao lado no menu; a de um oponente é respondida por quem o interpreta, e um chefe do Game Master é consultado pelo Game Master, com deixar o momento passar entre as respostas.
- **Ela é salva com o combate.** Uma partida fechada no meio de uma caminhada volta com as mesmas pessoas ainda a consultar e as mesmas células ainda a percorrer.

Você não declara nada para as duas primeiras: um conjunto de regras com `opportunity.budget` recebe uma, um bestiário com `signaturePoints` recebe a outra, e um conjunto de regras sem ambos nunca as vê.

**Indicar qual momento uma entrada espera.** Escreva `mechanics.reaction` como um objeto em vez de `true`:

```json
"reaction": { "on": "aimed", "at": "source", "cancels": true }
```

- `on` é `used`, `aimed`, `hit` ou `harmed`, e é o que coloca a entrada no menu dessa janela. Esses quatro são os únicos momentos que o Engine observa. Uma entrada que ainda diz `"reaction": true` apenas informa que não é executada em um turno; isso não basta para oferecê-la em algum lugar, então ela não aparece em nenhum menu.
- `at` é `source` (o padrão) ou `chosen`. `source` direciona o que é executado a quem causou o momento e preenche o alvo, para que ninguém precise escolher; `chosen` mantém os alvos da própria entrada e pergunta.
- `cancels` impede completamente que aconteça o que a janela estava segurando. Só uma entrada `used` ou `aimed` pode declará-lo: um momento que já aconteceu não pode ser cancelado, e um acerto já foi rolado.
- `against` restringe aquilo a que a entrada responde a coisas usadas de certos catálogos: `"against": { "catalogs": ["spells"] }` responde a uma magia e a nada mais. Uma ação sem entrada por trás (uma arma numa lista, a própria ação de uma criatura) não vem de nenhum catálogo, então nunca abre o momento para uma entrada que nomeia catálogos. Omita `against` e a entrada responde a qualquer coisa.

Dê a ela também um `budget`, ou ela gastará o padrão da lista. Uma reação quase sempre gasta um orçamento próprio, o que impede um turno de conter várias.

**Seu custo é gasto antes de consultar qualquer pessoa.** Uma ação cancelada é impedida de acontecer, mas não deixa de ter sido comprada: o orçamento e as reservas já foram gastos. Se seu sistema os reembolsa, ainda não pode expressar isso.

Uma contra-ação se escreve assim e, num tabuleiro, só responde a alguém dentro do próprio `range`:

```json
"mechanics": {
  "kind": "utility",
  "range": 8,
  "free": true,
  "cost": [{ "pool": "luck", "amount": 1 }],
  "reaction": { "on": "used", "cancels": true, "against": { "catalogs": ["knacks"] } }
}
```

**O momento depois de um acerto.** Uma entrada em `hit` é como se expressam Shield, um Parry e Uncanny Dodge. O ataque fica retido depois da rolagem, e quem foi atingido é consultado antes de qualquer dano. O que essa pessoa faz conta para aquele ataque: a rolagem não é feita de novo, mas verificada de novo, contra a defesa dela como estiver agora, então uma condição que soma à defesa pode transformar o acerto em erro, e uma face natural que sempre acerta continua acertando. Uma condição que dura um ataque (`endsAfter: "attacked"`) imposta como resposta cobre este ataque, inclusive o dano, e é consumida por ele, que é como se expressa reduzir à metade o dano daquele ataque: `resist-all` para um ataque.

```json
"mechanics": {
  "kind": "buff",
  "targets": "self",
  "budget": "reaction",
  "reaction": { "on": "hit" },
  "applies": [{ "condition": "shielded", "duration": { "rounds": 1, "at": "turn-start" } }]
}
```

O ataque continua exatamente de onde ficou retido: o restante dos alvos e, para uma ação de criatura feita de outras ações, o restante das partes. Cada rolagem que acerta alguém com essa resposta fica retida na sua vez, e uma luta salva enquanto a janela está aberta volta da mesma forma. Um ataque feito dentro de uma janela (um golpe contra quem se afasta, uma ação característica, uma resposta) nunca fica retido, porque nada aberto dentro de uma janela abre outra. O registro mostra a rolagem e contra o que ela foi feita quando a janela abre e, quando a resposta mudou a defesa, se agora ela erra. Oponentes jogados pelo Engine só usam uma proteção quando ela transforma o acerto em erro.

Um pacote que nomeia um momento precisa da Capability API 1.33, e um que usa `used` ou `against` precisa da 1.44. Um que espera `hit` precisa da 1.46.

### Ainda não

Dito claramente, pois um conjunto de regras não deve prometer o que o Engine não faz:

- **Além do tabuleiro modesto**: não há cobertura de três quartos ou total, elevação, voo sobre obstáculos, passagem apertada, montarias, ocultação ou surpresa, e nada move ninguém além da própria caminhada e do empurrão de uma disputa. Ninguém arrasta uma criatura que está segurando.
- **Uma disputa é simples.** Não tem limites de tamanho e não abre janela (ninguém pode responder a uma).
- **Uma entrada só pode esperar quatro momentos**, `used`, `aimed`, `hit` e `harmed` (veja Janelas, acima). São os momentos que o Engine percebe em nome de uma entrada; as outras duas janelas, alguém se afastando e a pausa entre dois turnos, são abertas pelo próprio combate e não são momentos que uma entrada pode solicitar. Não há momento para uma salvaguarda ser rolada, uma morte, um turno começar ou algo cair.
- **Uma contra-ação impede por completo o que responde.** Ela não distingue uma entrada de um catálogo de outra, e não há teste para impedir algo maior que ela mesma.
- **Sem encadeamento.** O combate mantém uma janela em vez de uma pilha, então nada aberto dentro de uma janela abre outra: uma contra-ação não pode ser anulada por outra, o que uma reação causa não abre outro momento, e um ataque feito dentro de uma janela não fica retido quando acerta.
- **Só uma rolagem de ataque é um acerto.** Algo que atinge sem rolagem (dardos que simplesmente acertam, uma área contra a qual todos fazem salvaguardas) não abre o momento `hit`, e uma resposta não distingue um golpe de um disparo, então um Parry que as regras restringem a espadas também apara flechas.
- **O dano é alterado pela metade, não por uma quantidade.** Uma resposta pode reduzir à metade o dano de um ataque (`resist-all` para aquele ataque); não pode tirar dele um número rolado.
- **Nada é reembolsado.** O custo de uma ação cancelada permanece gasto.
- **O que uma condição altera é uma lista fechada.** Defesa, rolagens de ataque, salvaguardas, testes de disputa e velocidade (e o mesmo vale para um nível e para o que um item faz numa luta), e nada mais: nenhum bônus ao dano que quem a porta causa, e nenhum nível que reduza o máximo que um personagem pode ter ou o tire da luta.
- **Uma criatura escrita com números simples não tem trilha de ferimentos.** Em um conjunto de regras cuja saúde é uma trilha, essa criatura ainda perde pontos; dê a ela uma `sheet` e os golpes recebidos marcarão casas, mitigados primeiro pelos próprios `resist`, `vulnerable` e `immune`.
- **Um efeito adicional dispara sozinho.** `on` tem um valor, `hit`, então o primeiro acerto qualificado do período o aplica, sem um momento em que você seja consultado sobre gastá-lo.
- **Um número que os ataques movem é simples.** Um golpe que gasta rola o número e nada mais, então nenhuma arma o altera, inclusive seu `floor` (um alvo resistente absorve um golpe que toma, e sua dureza detém um que gasta, mas nada mais reduz nenhum dos dois); um colapso termina depois de um número fixo de turnos, por mais fundo que tenha ido; e um ataque feito numa janela (um golpe contra quem se afasta, uma reação, uma ação característica) é feito no primeiro estilo.
- **Um oponente inventado não absorve nada e não tem dureza.** Uma criatura que o Game Master cria para uma luta fica limitada à sua escala de ameaça, que não diz nada sobre absorção ou dureza, então ela não tem nenhuma das duas.

## Camadas: variantes do seu próprio conjunto de regras

Uma camada é uma variante nomeada do seu conjunto de regras que o jogador ativa ao criar uma partida: pouca magia, inverno rigoroso, uma dificuldade mais dura. As camadas ficam no arquivo do conjunto de regras, em uma matriz opcional `layers`; assim, viajam com ele e nunca podem faltar em uma partida que as usou. O assistente as mostra como interruptores abaixo do conjunto de regras, e a escolha fica fixa durante toda a vida da partida, exatamente como o próprio conjunto.

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

**O que uma camada pode fazer.** A lista é fechada, e cada efeito restringe algo ou adiciona texto:

- `gm.guidance` é acrescentado ao fim do seu `gm.checkGuidance`, depois do seu texto e do de qualquer camada anterior. `gm.worldGuidance` é acrescentado a `gm.worldGuidance` da mesma forma.
- `fields` remove valores de um campo **enumerado**. `removeValues` nomeia valores que o campo já possui; pelo menos um deve permanecer e, se o `default` do campo for removido, a camada nomeia outro `default` que permaneça.
- `difficultyLadder` substitui sua escala por outra, na estrutura do seu próprio tipo de resolução: `{label, dc}` para `dice-sum` e `{label, successes, target?}` para `dice-pool`. Ela passa exatamente pelas mesmas verificações da sua escala. Quando várias camadas ativas declaram uma, a última vence.
- `catalogs` oculta entradas do seletor do editor de fichas. Cada regra nomeia um dos `filters` declarados pelo catálogo e exatamente uma comparação: `above` ou `below` para um filtro `number`, `equals` ou `notIn` para um filtro `text` ou `tags`. Uma entrada que não define esse filtro nunca é ocultada.
- `currencies` retira moedas: `removeUnits` nomeia moedas individuais, e `removeFamilies`, famílias inteiras. A menor moeda de uma família só sai junto com a família, para que toda família restante ainda possa pagar e dar troco. Enquanto a camada estiver ativa, ninguém recebe, paga com, escolhe ou deixa cair uma moeda retirada, e a linha de moedas a omite. Um preço expresso nela é dito na maior moeda restante que o paga exatamente, pelo mesmo valor: a camada da longa noite de Gravewatch retira a crown, então uma pistola de vigia de 3 crowns custa 15 shillings. Um item cuja família inteira sumiu não tem preço.

**O que uma camada não pode fazer.** Ela não pode adicionar um valor enumerado, campo, perícia, reserva ou descanso, mudar o tipo de resolução, tocar no estado atual ou nos números de combate, nem adicionar uma chamada de modelo. Um valor que uma camada _adicionasse_ seria desconhecido de todos os outros leitores da ficha, então valores só podem ser removidos. Tudo além dessa lista é uma alteração no próprio conjunto de regras ou um segundo conjunto.

**Conflitos.** `conflicts` nomeia camadas que não podem ficar ativas juntas. Basta nomear um lado do par. O assistente desativa o outro interruptor e, se uma escolha salva tiver ambos de alguma forma, a camada declarada **depois** é descartada; assim, as mesmas duas escolhas sempre produzem as mesmas regras.

**Uma ficha que já contém um valor removido o mantém.** Nada reescreve um personagem. O editor simplesmente deixa de oferecer o valor, e um personagem que já o tinha o mostra como é. Desative a camada em uma nova partida e o valor será oferecido de novo. O mesmo vale para uma entrada de catálogo oculta: ela sai do seletor, e uma linha que o jogador já escolheu permanece na ficha. Moedas que uma camada retira e que alguém já carrega continuam na bolsa dessa pessoa.

**Limites.** 12 camadas por conjunto de regras e 4000 caracteres de orientações por camada, contando as duas strings juntas. Um conjunto de regras empacotado que declara `layers`, ou um `gm.worldGuidance` de base, precisa da Capability API 1.25, e um cujas camadas retiram moedas precisa da 1.64. Um conjunto importado é validado pelo Engine que o lê, então não precisa de nada.

**Camadas escritas por outra pessoa** (uma camada de pouca magia para um conjunto de regras que você não escreveu, distribuída no próprio arquivo) virão depois. Hoje, uma camada é distribuída dentro do conjunto de regras ao qual pertence.

<a id="trying-your-ruleset"></a>

## Testar seu conjunto de regras

Conjuntos de regras da comunidade usam o mesmo interruptor dos agentes importados. Abra **Settings** (configurações) > **Advanced** (avançado) > **Danger Zone** (zona de perigo) e confirme que **Allow custom Agent imports** (permitir importações de agentes personalizados) esteja ativado. A importação também precisa de acesso por localhost ou de **Admin Access** (acesso de administrador) configurado.

1. Abra o painel **Agents** (agentes) e escolha o botão **Import agents** (importar agentes), o ícone de download na fileira de botões no topo do painel.
2. Escolha **Game Mode ruleset** (conjunto de regras do modo de jogo) e selecione seu arquivo JSON.
3. Leia a revisão. Ela mostra nome, versão, licença, o que o conjunto de regras cobre e o texto do Game Master. Escolha **Import** (importar).

Seu conjunto aparece na seção **Rules** (regras) do painel e na escolha **Rules** do assistente de configuração para novas partidas. Um conjunto importado de um arquivo é classificado como `local/<your id>`, para que nunca seja confundido com um conjunto oficial ou de outra pessoa.

### Alterar um conjunto de regras já importado

Uma versão importada nunca é reescrita. Se você alterar o arquivo e importá-lo novamente com a mesma `version`, a importação será recusada e pedirá que aumente o número. Isso é intencional: uma partida fica vinculada à versão exata em que foi criada, para que uma campanha em andamento nunca acorde com cálculos diferentes.

Portanto, o ciclo durante a elaboração é: editar, aumentar `version`, importar, iniciar uma nova partida. As versões antigas continuam instaladas ao lado da nova até você remover o conjunto de regras da seção **Rules**. Remover um conjunto ainda usado por uma partida faz com que ela informe que o conjunto está ausente até você importá-lo novamente.

Se mudar a estrutura da ficha (adicionar, remover ou renomear elementos), aumente também `sheet.version`. As fichas existentes são lidas com tolerância: valores que a nova ficha desconhece são mantidos, e os ausentes recebem seus padrões.

## Compartilhar seu conjunto de regras

**Como arquivo.** Envie o arquivo JSON a um amigo. Ele o importa como você fez.

**De um repositório do GitHub.** Se você mantém seu trabalho em um repositório público do GitHub, coloque cada conjunto de regras em uma pasta `rulesets` na raiz do repositório, um arquivo por conjunto:

```text
your-repository/
  agents.json        (optional, only if you also share agents)
  rulesets/
    ember-roads.json
    another-system.json
```

Um usuário adiciona seu repositório uma vez pela lista de repositórios de agentes personalizados, revisa o conteúdo e pode sincronizar depois para receber novas versões. A lista de repositórios personalizados é um recurso avançado que a pessoa responsável pelo servidor precisa ativar com `ENABLE_CUSTOM_AGENT_REPOS=true`. Conjuntos de regras de um repositório são classificados sob o nome do proprietário, como `alice/ember-roads`, para que dois autores possam publicar um conjunto chamado `v20` sem conflitos.

Dois limites se aplicam. Um repositório pode conter no máximo 32 arquivos JSON diretamente em `rulesets`; se tiver mais, será recusado. Uma conta chamada `local` não pode publicar conjuntos de regras, pois `local/` é reservado aos importados de arquivos.

**No catálogo oficial.** Um sistema muito jogado e com licenciamento claro pode ser oferecido a todos por **Download Agents** (baixar agentes). Isso exige um pull request para o repositório [Marinara-Agents](https://github.com/Pasta-Devs/Marinara-Agents). Veja o pacote `ruleset-5e-2014` para conferir a estrutura.

## Licenciamento

Publique apenas textos de regras que você tem o direito de compartilhar. Muitos sistemas publicam um documento de referência com licença aberta, e é desse documento que você pode copiar. Coloque o identificador da licença e o texto de atribuição exigido em `license`. Não copie textos de livros de regras sem licença aberta. Um conjunto de regras precisa principalmente de nomes e números, e o texto do Game Master deve ser escrito com suas próprias palavras.

## Solução de problemas

- **A importação diz que um nome não existe.** Algo no arquivo aponta para um identificador não declarado, como uma perícia que nomeia um atributo removido. A mensagem informa o caminho até a linha.
- **A importação diz que uma versão já está instalada com conteúdo diferente.** Aumente `version` e importe novamente.
- **Meu conjunto de regras não aparece no assistente de configuração.** Confira se **Allow custom Agent imports** está ativado. Enquanto estiver desativado, conjuntos importados ficam fora de novas partidas. Partidas que já usam um continuam funcionando.
- **Uma partida diz que seu conjunto de regras está ausente.** A versão exata em que a partida foi criada não está instalada. Importe novamente essa versão do arquivo.
