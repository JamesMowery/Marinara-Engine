# 编写 Game Mode 规则集

规则集告诉 Game Mode 桌面系统如何运作：检定掷什么骰子、人物卡有哪些内容、消耗哪些资源，以及休息恢复什么。本指南面向希望自行编写并分享规则集的人。若要游玩别人制作的规则集，请先阅读[选择规则](../game/getting-started.md#choosing-rules)。

规则集是一个 JSON 文件，是数据而非代码，其中没有任何内容会运行，因此导入不会在你的电脑上执行任何操作。导入他人的文件前，值得仔细阅读的部分是 Game Master 文本，因为每场使用该规则集的游戏都会将它发送给模型。

## 先读这里：规则集能做什么、不能做什么

规则集只能填写 Engine 已理解机制中的空项。当前 Engine 有两种检定结算方式，文件通过 `resolution.kind` 选择：

- **`dice-sum`**：掷骰，加上人物卡数值，达到或超过难度。适用于 d20、2d6 加属性等许多系统。
- **`dice-pool`**：掷出角色数值对应数量的骰子，数出达到目标的骰子。适用于评级代表一把骰子而非加值的系统。

两者都在[结算类型](#resolution-kinds)中完整说明。

不符合两种形式的机制无法写进规则集文件，例如取骰池最高骰、掷低于目标的百分骰检定、符号骰或对抗骰池。每种都需要 Engine 内新增结算类型，也就是带测试的代码贡献，而非 JSON 文件。若系统需要，请在 Engine 仓库提出功能请求，用几个演算完整的掷骰例子说明机制。这些例子会成为测试。

Game Mode 可用 Marinara 自身战斗或规则集规则结算战斗。可选的 `battle` 块将人物卡数值提供给 Marinara 战斗，见[战斗桥接](#battles-lending-the-sheet-to-marinaras-combat)。可选的 `combat` 块则定义规则集如何自行结算，见[独立规则战斗](#combat-a-fight-your-own-rules-resolve)。Engine 目前已能执行这些规则。游戏的 Combat Preference 选择 Classic 呈现，或在规则集定义距离时选择 Tactical 战场。

## 快速开始

1. 复制符合系统掷骰方式的示例文件。[`ember-roads.json`](https://github.com/Pasta-Devs/Marinara-Engine/blob/staging/docs/examples/rulesets/ember-roads.json) 是三个属性的小型 2d6 系统，用来说明格式不预设 d20 或六项属性。[`gravewatch.json`](https://github.com/Pasta-Devs/Marinara-Engine/blob/staging/docs/examples/rulesets/gravewatch.json) 是三个评级、六项专业技能的小型十面骰池。完整规模的例子见 [`ruleset-5e-2014.example.json`](https://github.com/Pasta-Devs/Marinara-Engine/blob/staging/docs/development/ruleset-5e-2014.example.json) 中的 5e (SRD 5.1)。
2. 将 `id` 改为自己的值。ID 使用小写字母、数字和单个连字符，例如 `ember-roads`。
3. 编辑人物卡、休息和 Game Master 文本。
4. 导入，参见[试用规则集](#trying-your-ruleset)。导入会在保存前检查整个文件，逐行说明问题。
5. 创建新游戏，在 **Rules**(规则) 中选中规则集，实际进行几次检定。

要获得输入辅助，请在文件最外层花括号内的第一行添加以下内容，为编辑器指定 JSON Schema：

```json
"$schema": "https://raw.githubusercontent.com/Pasta-Devs/Marinara-Engine/staging/docs/extending/ruleset.schema.json",
```

模式会在输入时发现键名拼写和类型错误，但无法检查名称是否指向真实存在的对象，例如技能引用的属性。这由导入完成。

你可以在文件的任何对象中添加 `"$comment": "..."` 一行作为备注。Engine 会忽略它。

## 文件组成

| 键 | 内容 |
| --- | --- |
| `schemaVersion` | 始终为 `1`。 |
| `id`, `version` | Engine 使用的规则集名称，以及每次发布修改时递增的整数。 |
| `name` | 玩家在设置向导中看到的名称。 |
| `edition` | 可选，用一行说明版本或草案。 |
| `license` | 可选，SPDX ID 和来源要求的署名文本。 |
| `coverage` | 规则集覆盖范围，以及设置向导展示的一行摘要。 |
| `resolution` | 如何掷检定或豁免。 |
| `sheet` | 人物卡中的所有内容。 |
| `rests` | 各种休息恢复和清除什么。 |
| `gm` | 给 Game Master 模型的文本，以及它能看到的各角色人物卡数值。 |
| `catalogs` | 可选，人物卡编辑器提供的现成条目，省去手动输入长列表。 |
| `items` | 可选，书写物品所用的词汇：类别、稀有度、数值、槽位和货币。 |
| `battle` | 可选，战斗可从人物卡读取什么，以及结束后写回什么。 |
| `combat` | 可选，如何按自身规则结算战斗，以及战斗界面执行什么。 |
| `layers` | 可选，玩家创建游戏时启用的规则集变体。 |

文件最大 256 KB。会进入提示词的文本(名称、标签、Game Master 文本)不能包含换行、方括号或双花括号。

人物卡内的 ID(属性、技能、字段、池等)以字母开头，使用小写字母、数字和下划线，例如 `grit_max`。

<a id="resolution-kinds"></a>

### 结算类型

`resolution.kind` 选择检定如何掷骰。两种类型读取相同人物卡，共享三个键，因此切换时 `resolution` 以下的文件部分不用改变：

- `abilityModifier`：人物卡分值如何变成一个数字。`identity` 表示直接使用分值；`floorHalfMinusTen` 是 5e 规则；`stepTable` 允许用 `[[score, number], ...]` 列出自己的阈值。
- `proficiencyTiers`：技能或豁免的训练等级。未列出的技能使用第一档。每档增加 `flat`、熟练加值的 `multiplier` 倍或两者都有。如果系统有熟练加值，用 `"proficiency": { "bonus": { "derived": "proficiency_bonus" } }` 指定来源。
- `proficiency`：可选，仅乘算档位需要。

结果数字的含义由类型决定：`dice-sum` 把它加到掷骰上，`dice-pool` 掷出相应数量的骰子。

#### `dice-sum`：将骰子求和

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

- `dice`：骰子数量和面数。总和用于与难度比较。
- `advantage`：是否允许 Game Master 要求掷两次并保留其中一次。
- `naturals`：单颗骰子的最高和最低点数对检定及豁免的影响，取 `none`、`both`、`max-only` 或 `min-only`。纯算术可省略。它要求单颗骰子，所以 2d6 系统必须使用 `none`。
- `difficultyLadder`：向 Game Master 提供的难度候选。`dc` 是总和必须达到的数字。Game Master 也可以不写数字，而是直接点名某一档，如 `difficulty="Hard"`，Engine 会从该档读出数字。

#### `dice-pool`：掷骰并计数

人物卡数字代表**骰池大小**，而不是额外加值。评级 3 加专业技能 2 就掷五颗骰子。原理仅此而已：无需新人物卡词汇或编辑器，评级代表一把骰子的系统同样使用 `abilities`、`skills` 和 `proficiencyTiers` 编写。

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

- `die`：骰池中每颗骰子的面数，2 到 100。
- `pool`：任何爆骰发生前，人物卡数值被限制到的范围。`min` 为 0 时，空池可以完全不掷骰就失败；`max` 最高为 100。`abilityPlusAbility: true` 允许属性检定用 `with=` 指定第二个属性并把两者一起掷出，适用于同时掷两个属性的系统；没有它时，`with=` 对属性检定没有意义。
- `target`：骰子计为成功需要达到的点数。将 `min` 设为低于 `max`，允许 Game Master 用 `threshold=` 逐次调整；三个值相同则固定。
- `double`：可选，点数达到或超过 `from` 时计两次。
- `explode`：可选，达到或超过 `from` 时多掷一颗，新增骰也可继续爆骰。额外骰在原骰池之外最多 `pool.max` 颗，因此单次检定最多掷两倍 `pool.max`，较低的 `from` 也不会无限掷下去。
- 两者都可以带 `min`：检定能把该规则下调到的最低点数，通过标签上的 `explode="N"` 或 `double="N"`，或角色使用的条目来调整。有 `min` 时可以省略 `from`，此时只有提出要求的检定才会触发该规则：只让某一项技能的十点再掷一次的专长写作 `"explode": { "min": 10 }`。`from` 不能低于 `min`。
- `cancel`：可选，点数不高于 `upTo` 时减去一次成功，成功数不低于零。
- `botch`：可选，**没有任何**骰子成功，且出现不高于 `upTo` 的点数时为大失败。唯一一次成功被抵消的骰池是普通失败，不是大失败。`"rule": "halfOrMore"` 换一种读法：首次掷出的骰子（不计爆骰）中至少一半点数不高于 `upTo` 时，若没有骰子成功就是大失败；否则结果照常成立，并标记 `complication="true"`，表示同时出了岔子。默认值是 `"noSuccesses"`。
- `exceptional`：可选，成功的检定净成功数达到此值或更多时为大成功。
- `situationalDice`：可选，Game Master 为特技、伤势或昏暗照明等情境，用 `bonus=` 对单次检定增减骰子的范围。
- `reroll`：可选，系统免费给予的重掷，最多六项，每项有一个 `id`，Game Master 在检定中用 `reroll=` 点名。点数不高于 `upTo` 的骰子会再掷一次：`"once"` 表示新点数无论是多少都保留，`"until"` 则一直重掷到点数更高为止。`upTo` 取 1 到骰子面数减一，因为重掷所有点数永远不会停下；`until` 在单次检定重掷一百次后也会停止。检定不点名就不会重掷。一次检定只进行一次重掷：如果角色的消耗或使用的条目也买了一次，就采用覆盖点数更多的那一次，覆盖相同时 `until` 优先于 `once`。
- `difficultyLadder`：`successes` 是要求的成功次数。某一档也可指定 `target`，但仅当目标可调且在范围之内时有效。该档的检定就按这个目标计数：Game Master 用 `difficulty=` 点名该档时，或者写下恰好只有一档需要的 `dc=` 时。如果几档需要相同的成功次数，单靠 `dc=` 无法说明指的是哪一档，目标就保持默认；请点名该档。

`cancel` 和 `botch` 的点数必须低于最低目标，所有规则指定的点数必须实际存在于骰子上。永远无法触发的规则会在导入时被拒绝，而不是等游玩时才发现。

打包形式的骰池规则集要求 Capability API 1.24；`explode` 或 `double` 上的 `min`、`pool.abilityPlusAbility` 和 `botch.rule` 要求 1.37；`reroll` 要求 1.38。导入的社区规则集由读取它的 Engine 验证，无须另行声明。

#### Game Master 可以在骰池检定中写什么

```
[skill_check: skill="Ward" dc="2" who="Bram the Quiet" threshold="8" bonus="-2" with="Sinew"]
[skill_check: skill="Ward" difficulty="Grim" explode="9"]
[skill_check: skill="Nerve" dc="1" with="Warmth"]
[skill_check: skill="Ward" dc="2" reroll="careful"]
```

- `dc` 是所需**成功次数**，不是目标点数。范围为 1 到一次掷骰理论上最多可计的成功数：骰池最大值，允许爆骰时翻倍，点数可计两次时再翻倍。
- `threshold=` 调整单骰目标，仅在 `target.min` 小于 `target.max` 时提供。
- `bonus=` 增减骰子，仅在声明 `situationalDice` 时提供。
- `with=` 用其他属性替代技能或豁免原有属性。两种类型都支持，因此 5e 的 "Strength (Intimidation)" 也使用同一属性表达。在 `pool.abilityPlusAbility` 开启时，它在属性检定上改为追加第二个属性。
- `difficulty=` 在两种类型上都可以代替 `dc=` 点名难度梯度的某一档：取该档的成功次数（求和检定则取其难度），骰池检定还会取该档的目标。同时写下的 `dc=` 仍决定数字，`threshold=` 仍决定目标。没有任何一档对应的名称，或两档共用的名称，会被忽略。
- `explode=` 和 `double=` 为本次检定调整这两条规则的点数，仅在规则带 `min` 时提供。
- `reroll=` 为本次检定点名某个 `reroll` 项，仅在你声明过重掷时提供。没有任何一项对应的名称会被忽略。

所有值都受文件声明约束：越界值会拉回最近边界，规则集未提供的属性则忽略，而不拒绝检定。保存记录显示实际使用的值：以数字表示的难度和阈值、限制后的奖励骰；只有实际替换或追加属性时才保留 `with=`，只有检定实际调整过点数时才显示该点数，只有它点名的重掷正是实际进行的那一次、且确实重掷了骰子时才保留 `reroll=`；`halfOrMore` 大失败在结果之外出了岔子时会记下 `complication="true"`。Engine 始终自行掷骰，替换模型填写的骰池结果；该类型没有优势，因此忽略 `mode="advantage"`，玩家在回合前自行掷出的骰子也不适用。

#### 范围之外的机制及原因

以下都无法表示为“数出达到目标的骰子”，因此各需自己的结算类型：

- **取最高骰**(如 Blades in the Dark) 需要检定结果尚未具备的部分成功档位。
- **与属性比较的姿态骰池**(如 Lasers and Feelings) 每次检定决定“高于还是低于”，是不同的比较方式。
- **符号骰**(如 Genesys) 根本不产生数字。
- **对抗骰池**同时结算两个角色，而一次检定只有一个掷骰者。
- **掷低于目标与开放式百分骰**使用相反的比较方向。
- **带狂野骰的求和骰池**(如 OpenD6) 将骰子相加并特殊处理其中一颗。

该类型过去遗漏的另外两项现在都已建模，下方“消费资源改变掷骰”说明其方式。系统自身的规则，如“花一点换一次成功”或“花一点把失败的骰子再掷一次”，是 `resolution.spend`。情境需要时系统免费给予的重掷，是上文的 `resolution.reroll`。属于角色实际选取内容的重掷，则是目录条目的 `mechanics.check`，用该条目自己的成本购买。

### 消费资源改变掷骰

有些系统允许玩家为即将进行的掷骰支付资源，例如一点意志换一次自动成功。`resolution.spend` 将其声明为系统常设规则，而非角色另行购得的特性：

```json
"spend": [{ "pool": "resolve", "amount": 1, "successes": 1, "perCheck": 2 }]
```

- `pool` 是某个 `live.pools`。不能是初始为空的池，否则开始游玩时无资源可花。
- `amount` 是一次购买的成本。购买所得为 `successes`、`dice` 和 `reroll`，至少要有其中一项。成功在骰子计数及抵消之后加入，因为它们并非掷出。骰子则在池自身范围内随骰池一起掷。`reroll` 写作 `{ "upTo": 6, "mode": "once" }`，读法与 `resolution.reroll` 相同；无论检定购买几次，它都只买一次：同一颗骰子重掷两遍，仍然只算一次重掷。
- `perCheck` 是单次检定可购买的次数，因此最多购买 `amount * perCheck` 点的效果。这个上限防止满池买出绝不会失败的掷骰。它可以是 1 到 10 的数字；可以是人物卡上的值，写法与派生值读取的形式相同（`{ "abilityScore": "nerve" }`、`{ "derived": "focus" }`）；也可以是 `"pool"`，即检定有几颗骰子就能买几次，指的是计入伤势、奖励骰或调整值之前人物卡给出的数字。人物卡上的值按掷骰者读取，向下取整并限制在 0 到 100 之间；上限算出为 0 的角色什么也买不到，也不支付任何费用。
- 仅限 `dice-pool`。求和掷骰没有可加的成功次数，也没有可加入骰子的池，因此声明 `spend` 的 `dice-sum` 会在导入时被拒绝。
- 最多四项，且两项不能指向同一资源池，否则检定无法说明指的是哪项。
- 购买 `reroll` 的消耗、不是数字的 `perCheck`，以及超过两项，对打包规则集要求 Capability API 1.38。

**它写在检定本身。** Game Master 写 `[skill_check: skill="Nerve" dc="2" spend="resolve:1"]`，而非独立 `[sheet:]` 命令，因为骰子在应用人物卡命令之前就已掷出，另写命令将无从改变。一次结算同时掷骰并支付改变骰子的费用。

### 改变掷骰的护符

`resolution.spend` 是系统规则。角色实际选取的条目也能通过目录项的 `mechanics.check` 改变检定：

```json
"mechanics": {
  "kind": "utility",
  "cost": [{ "pool": "blood", "amount": 1 }],
  "perCostStep": { "flat": 1 },
  "check": { "reroll": { "upTo": 1, "mode": "once" }, "successes": 1 }
}
```

- `reroll` 重掷不高于 `upTo` 的骰子。`once` 对每颗只替换一次，并接受新点数；`until` 则继续重掷。`upTo` 必须是低于最高面的实际点数，否则会永远重掷整个池；无论文件怎么写，Engine 都限制单次检定的重掷数量。
- `dice` 在掷池之前增加骰子，`successes` 在计数之后增加成功，`threshold` 则在 `target` 允许范围内设定本次单骰目标。
- `explode` 和 `double` 为这一次掷骰调整这两条规则的点数，仅在你的规则带 `min` 时有效，且不能越过它：`"check": { "explode": 8 }` 是让八点再掷一次的条目。和 `threshold` 一样，它们优先于 Game Master 在标签上写的值。Capability API 1.37。
- 这些键至少声明一个，否则条目没有效果而被拒绝。
- 只有 `dice-pool` 能实现这些内容，因此含 `mechanics.check` 的 `dice-sum` 会在导入时被拒绝。

**成本是条目自身的 `cost`**，通过与使用其他内容完全相同的机制支付：资源池，以及该条目写入的每个计数器的一次使用量。`perCostStep` 表示条目会缩放；声明它时按实际支付了几份价格购买几份，未声明时无论提出多少都只买一次。

**Game Master 在检定中点名：** `[skill_check: skill="Brawl" dc="3" use="Potence" spend="blood:3"]`。同样不能另写人物卡命令，因为骰子在任何记账处理之前就已掷出。

**要么全部成功，要么完全不买。** 资源池不足时不购买也不扣除，仍进行原本的掷骰。不足以构成整数次购买的点数也买不到任何东西。请求超过 `perCheck` 时会限制到上限而非拒绝，且只支付上限对应金额。全部由 Engine 计算，Game Master 只写玩家声明要花的资源，不碰骰子。记录显示真实支付量、应用条目、未掷骰获得的成功数，以及重掷了多少骰子。角色未选取的护符，或 Engine 无法读取目录的护符，完全不生效，不会凭信任应用。

### 人物卡

- `sections` 对编辑器内容分组。字段、派生值和列表用 `section` 指定自己所在的分区，属性、技能和豁免也可以这样做：编辑器、游戏内人物卡和 Game Master 的人物卡块都会在该标题下显示它们（未给它们指定分区的人物卡，显示方式与以前完全相同）。分区还可以带 `untrained` 规则（见下文）。
- `abilities` 是核心属性值。`skills` 和 `saves` 可各自指定使用的属性、`cap` 和 `untrained` 规则（见下文）。
- `fields` 是单值，类型有 `number`、`text`、`longtext`、`boolean`、`enum`(固定候选列表) 和 `dice`(如 `1d8` 的文本)。
- `derived` 从其他值计算，不可手动覆盖。运算有 `sum`、`min`、`max`、`scale`(乘算并取整)、`stepTable`(按阈值查表，如等级对应熟练加值) 和 `enumTable`(为枚举字段或实时状态的每个值给出一个数字，见下文“实时状态”)。
- `lists` 是自定义列的表，如装备、法术或特性。带 `pools` 的列表让每行成为有自身上限的资源，适用于有限使用次数的职业特性。
- `live` 是游玩中变化的内容：`pools`(生命、法术位、Grit)、`tracks`(如力竭的刻度数值，或勾格子的伤势轨道)、`text`(如角色正专注什么的短备注)、`conditions` 和 `states`(从封闭集合中取一个值，如形态或姿态)。

凡是读取数字的地方都用值引用指定。它是恰好含一个键的对象，键为 `const`、`field`、`derived`、`abilityScore`、`abilityMod`、`abilityModFromField`、`skillMod`、`saveMod`、`listSum`、`livePool`、`liveTrack` 或 `itemStat`。例如，最大值来自派生值的池：`"max": { "derived": "grit_max" }`。

- `listSum` 把列表的某个数字列相加：`{ "listSum": { "list": "gear", "column": "bulk", "onlyWhen": "packed" } }`。`onlyWhen` 可选，指定一个布尔列，只计入该列为真的行。空单元格按该列的默认值计算，被 `hideWhen` 隐藏的列表不加任何数。Ember Roads 就这样根据角色打包的装备算出 Burden，因此它永远不会像手动输入的数字那样与列表脱节。目录的缩放列不能读取列表合计，经由派生值也不行：列表里可能有缩放单元格，无论是自己的，还是反过来读取它的，重新计算将永远无法收敛。
- `livePool` 读取某个 `live.pools` 的剩余量（列表行的池不在其中）。`liveTrack` 读取某个 `live.tracks`，可选 `read`：`"value"` 是当前所处位置（默认）；`"filled"` 是高出 `min` 多少；`"remaining"` 是距离 `max` 还有多少；仅限伤势轨道的 `"penalty"` 是当前生效的惩罚。被隐藏的池或轨道读作 0。Gravewatch 的 “Harm still to take” 写作 `{ "liveTrack": "harm", "read": "remaining" }`。
- 实时读取使用掷出检定、战斗开始或写入 Game Master 人物卡块那一刻的实时状态。尚无实时状态时（人物卡编辑器、导入审核），读取游玩开始时的状态：池按其 `start` 为满或为空，轨道处于其 `default`，伤势轨道没有标记。
- 在实时状态存在之前就要算出的内容，都不能读取实时值：池或轨道的 `max`、熟练加值，以及目录的缩放列或缩放。经由读取实时值的派生值，或经由被实时值限制上限的技能，同样不行；导入会指出是哪个值这样做了。
- 这三项对打包规则集要求 Capability API 1.39。
- `itemStat` 读取角色持有的物品，例如所穿护甲的 guard，并受与实时读取相同的规则约束。见[人物卡上的物品](#items-on-the-sheet)。Capability API 1.52。

**未受训练时的检定。** 角色没有受过训练的技能或豁免（其档位为第一档）照常掷骰，除非规则集另有规定：在技能或豁免上，或在其所在的分区上写 `untrained`(技能或豁免自身的规则优先)：

```json
"sections": [{ "id": "labour", "label": "Labour", "untrained": { "by": -1 } }],
"skills": [
  { "id": "dig", "label": "Dig", "ability": "sinew", "section": "labour", "untrained": "refuse" },
  { "id": "listen", "label": "Listen", "ability": "nerve", "untrained": "harder" }
]
```

- `"normal"`：照常掷骰（默认）。
- `{ "by": -3 }`：加到检定的数字上，范围 -20 到 20：骰池上是骰子，求和上是固定值。它属于人物卡显示、每次检定都会掷出的那个数字，在任何 `cap` 之前计入。
- `"harder"`：难一档。骰池上的单骰目标提高一点（且保持在目标范围之内），因此需要目标可调的骰池规则集；其他情况都会在导入时被拒绝，因为它永远不可能改变掷骰。
- `"refuse"`：完全不掷这次检定。Engine 把 Game Master 的请求连同 `reason="untrained"` 写回，检定就此了结：之后不会有任何东西再掷它，Game Master 会被告知角色无法尝试，叙述也会这样交代。技能检定接口对这类检定返回 400(`skill_check_untrained`)，以它为条件的分支块两半文字都不保留。人物卡编辑器不为它显示数字。
- 没有人物卡的角色没有可读取的训练，因此任何未受训练规则都不适用于他们。
- Game Master 的提醒会按分区或技能列出未受训练规则，让它只要求角色能够进行的检定。

属性、技能和豁免上的分区，以及任何位置的 `untrained`，对打包规则集要求 Capability API 1.41。

技能或豁免可以带 `cap`，这是一个值引用，表示其检定最多能达到的数字：角色无法超越的评级，或压制它的轨道。Gravewatch 把 Soothe 限制为守望者剩余的 Resolve：`"cap": { "livePool": "resolve" }`。限制后的数字就是人物卡显示、每次检定掷出的数字；`with=` 替换先在限制前的数字上计算，再重新限制。上限不能读取技能或豁免调整值，它所读取的派生值，以及声明在该派生值之前的任何派生值也都不能，因此人物卡仍然自上而下只计算一遍。Capability API 1.39。

`hideWhen` 按另一字段的值隐藏字段、派生值、列表、池或普通轨道，方式恰好是以下三种之一：`equals` 某个值；`notEquals` 某个值（字段为其他任何值时都隐藏，“只适用于这类角色”就这样表达）；或 `in` 一组值。指定的每个值都必须是该字段能取的值，否则规则永远不会匹配；这会逐个值检查，对 `notEquals` 和 `in` 来说还包括数字字段的范围和文本字段的长度。规则读取人物卡编辑器显示的值：未设置的字段取空白人物卡的初始值，规则集已不再提供的枚举值取字段默认值。5e 文件用 `equals` 为不施法的角色隐藏法术位；Gravewatch 用 `"hideWhen": { "field": "watch", "notEquals": "night" }`，让所有不值夜班的守望者人物卡上都不出现灯油。层不能移除这些规则所比较的任何值。伤势轨道不能隐藏，因为无论人物卡显示什么，掷骰和战斗都会读取它。`notEquals` 和 `in` 要求 Capability API 1.39。

普通轨道的 `max` 可以是值引用而非数字，这样角色拥有的评级就决定了它的长度：`"max": { "field": "willpower_rating" }`。其 `min` 和 `default` 仍是数字。带命名等级的伤势轨道，`max` 等于等级数；编号格子的轨道长度等于自身的 `max`(见下文“编号格子代替命名等级”)。`alwaysShow: true` 让轨道即使处于默认值也会印在 Game Master 的人物卡块中，适合每回合都重要的评级；没有它时，处于默认值的轨道会被省略，因为三个为零的死亡豁免什么也没说明。这三项对打包规则集要求 Capability API 1.37。

### 实时状态：形态、姿态、灯光

枚举字段在创建角色时选定，之后保持不变；状态只有开和关。**实时状态**则是从封闭集合中取的一个值，会在游玩中变化：变形者的形态、战斗姿态、提灯的亮度。

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

- `values` 为 2 到 40 个，各自显示为 `valueLabels` 中的对应项，或显示其本身。它们遵循所有标签都遵循的规则（单行、无方括号、无宏花括号），且不得包含双引号，因为人物卡命令会给它们加引号。最多十二个状态。
- `default` 是每张人物卡的起点，省略时取第一个值。只有离开默认值的状态才会保存；已保存但状态不再提供的值读作默认值。
- `hideWhen` 按字段的值把状态从人物卡上拿掉，与池的做法相同：不显示、不能被命令设置，被表（见下文）读取时视为没有值。
- Game Master 用 `[sheet: who="Name" op="state" state="Stance" value="Reckless"]` 设置状态，状态用 ID 或标签指定，值用其本身或标签指定。人物卡没有的状态或值会被拒绝（`unknown-state`、`unknown-value`）。只有规则集含有状态时，Engine 才会教授这条命令并列出每个值；每个人物卡块都会写出每个状态的值，包括默认值，因为“处于人形”是叙述每回合都需要的事实。
- 游戏的人物卡画面为每个状态提供选择器。
- 休息通过指定该状态的恢复步骤把它复原：`{ "state": "stance", "to": "default" }`，或让 `"to"` 取某个值。

**随之变化的数字。** `"op": "enumTable"` 的派生值为枚举字段或实时状态的每个值给出一个数字：

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

- `from` 恰好指定 `field`(枚举字段) 或 `liveState` 之一。`table` 的每一行都是该来源能取的值，共 1 到 40 行。表中未列出的值，以及被人物卡隐藏的状态，都读作 `default`(省略时为 0)。
- 字段在创建角色时固定，因此以字段为键的表可以用来提供最大值：Gravewatch 让值晨班的守望者多一点 Resolve。实时状态会在游玩中变化，因此以它为键的表是一种实时读取，与 `liveTrack` 一样：在实时状态存在之前就要算出的内容，无论直接还是经由其他派生值，都不能读取它。这就是键名是 `liveState` 而非 `state` 的原因：对实时状态的每次读取都带 `live` 字样，你和导入都能据此分辨最大值不能使用什么。
- 要让这个数字作用于骰子，请将 `resolution.adjust` 指向它（见“来自人物卡的调整值”）。Ember Roads 把 Stance 加到每次 Brawn 掷骰上；Gravewatch 在提灯关闭或熄灭时，从每次 Nerve 掷骰中扣除。
- 层可以从表中有对应行的字段里移除某个值。该层开启期间，这一行只是永远不会被读取。

实时状态、`enumTable` 以及指定 `state` 的休息步骤，对打包规则集要求 Capability API 1.42。

### 伤势轨道：用轨道而非数字表示生命

许多系统根本不计生命点，而是有一列每格比前一格更严重的格子，受伤时标一格。给 `live.tracks` 条目添加 `kinds`，再加上命名的 `levels` 或编号的 `boxes`(见下文) 之一，它就从刻度数字变为这种轨道：

**系统需要哪种形式？** 池记录承受多少伤害；轨道记录每份伤害的量与种类。如果伤势分为冲击、致命或恶化，并且因为恶化伤害恢复更慢、无法吸收或最终致死，种类在打击之后仍有意义，就必须在掷骰后保留它，而只有标记能携带种类。点数池做不到：扣伤后只剩更小数字，人物卡不记得哪些点属于哪类。因此，生命为池时 `combat.damageKinds` 会被拒绝，而非静默忽略。池仍可有 `damageTypes`，对手仍能抵抗或免疫，因为那关乎一击实际造成多少伤害，而非事后伤势是什么。

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

- `levels` 为 1 到 16 级，最好在前、最差在后。每级有 `label` 和不大于 0 的整数 `penalty`。这类系统用很大的负数表示失去行动能力，因此 `-99` 也有效。
- `kinds` 为轨道能接受的 1 到 6 类伤害，每类含 `id`、格子使用的简短 `label` 和 `severity`。严重度必须互不相同；数值除排序外没有含义，间隔随意。
- 两者必须配套。既没有 `levels` 也没有 `boxes` 的 `kinds` 会被拒绝，因为无处标记；没有 `kinds` 的 `levels` 或 `boxes` 也会被拒绝，因为标记必须有种类。轨道只能有 `levels` 或 `boxes` 之一，不能两者都有。
- **区分两个术语。** `kinds` 定义规则集允许标记成为哪些种类。标记则是游玩时某一种类实际占据轨道的实例。定义保存种类，人物卡保存标记。
- 伤势轨道开始时没有标记，因此 `min` 为 0。带命名等级的轨道长度就是级数，因此 `max` 为 `levels.length`；不符的文件会被拒绝而非悄悄修正，避免同时携带两个冲突长度。格子轨道的长度等于自身的 `max`，它可以是人物卡计算出的值（见下文“编号格子代替命名等级”）。

模糊理解会得到错误轨道，因此以下是**精确规则**：

- 标记按**最严重优先**排序保存。七级轨道最多容纳七个标记。
- 标记按已有标记的**严重度顺序插入**，从不直接追加末尾。它占据严重度允许的最高等级，并将较轻标记向下推。
- 生效惩罚取**最低已标记等级**，而非各格之和。上面轨道的三个标记是 `-3`，不是 `0 + -1 + -3`。
- `amount` 是某一类标记的个数，**逐个应用**，因此中途填满和原本已满的轨道使用相同规则。
- 向**已满**轨道标记时，不新增标记，而将**最轻标记提升一档**。按自身种类阶梯上升一档，与新增标记原本是什么种类无关。
- 超过最高严重度的标记停在最高级，无法落下的那一个计作**溢出**。溢出会保存，因此重载不会忘记已受伤害。
- **治疗使用同一命令和负数数量。** 先清溢出再清任何标记。指定轨道某一种类的治疗只清该种类的标记，因此“冲击伤恢复，致命伤留下”就是带负数数量的 `kind="bashing"`；未指定轨道所具有种类的治疗，先清较轻标记。

**游玩时标记。** Game Master 写 `[sheet: op="damage" track="harm" kind="knock" amount="1"]`，用负 `amount` 治疗。在按格填充的轨道上，它还会加上 `box="3"`，指明这次打击落在哪一格。原先指定 `pool=` 的 `damage` 形式不变。伤势轨道拒绝普通 `track` 命令，因为单一数字无法说明新增标记是什么。玩家也可在人物卡手动标记和清除格子，符合这些系统的预期。

**战斗也能标记。** 将 `combat.health` 指向轨道而非池，命中一击就按 `combat.damageKinds.marks` 指定格数，以及该块将伤害类型映射到的种类进行标记。轨道填满的角色倒下，濒死规则读取这一状态。在按格填充的轨道上，`per-point` 伤害会指定落在哪一格，`per-blow` 则瞄准第一格；没有任何格子能承受的打击会让角色倒下，在采用这类轨道的系统中，这就是被击倒出局的含义。治疗清一个标记，即最轻的那个，不论种类。轨道没有临时点数缓冲，因此拒绝临时点数。Engine 把轨道视为剩余级数，所以战斗的其他部分、倒下、复活、日志和战后摘要都不变。

**休息可以治疗伤势轨道。** 用 `"to"` 指定轨道的恢复步骤会连同溢出一起清到指定标记数；用负数 `"by"` 则先清溢出，清除指定数量，因此 `"by": { "const": -2 }` 清除两个。同时指定 `kind` 的步骤只清该种类的标记：Gravewatch 的 “Catch your breath” 写作 `{ "track": "harm", "kind": "knock", "to": "min" }`，治好冲击伤而留下撕裂伤。会增加标记的步骤不做任何事，因为休息不造成伤害，正数 `"by"` 也是如此：它会被一声不响地跳过，所以请写上负号。

#### 编号格子代替命名等级

如果轨道的长度取决于角色有多坚韧，就用 `boxes` 代替 `levels`。格子数等于自身的 `max`，它可以是人物卡计算出的值（但绝不能读取实时状态），最多 64：

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

- 生效惩罚按 `filled` 或 `remaining` 的格子数，从 `table`(与派生值相同的阶梯表，惩罚不大于 0) 中读出，无论这个数是多少，所以即使没有任何标记，表也可以产生代价。它不取决于标记了哪些格子。
- 格子带编号：人物卡显示 “Box 3”，Game Master 的人物卡块写出已标记几格、共几格，而不是说出等级名称。

#### 按格填充，以及拒绝标记的轨道

- `"fill": "indexed"` 把标记放在命令用 `box=` 指定的格子上（未指定时为第一格），或其上方下一个空格，标记一旦放下就不再移动。治疗清除最轻标记中位置最高的那一个。人物卡画面允许玩家点击任何空格。
- `"onFull": "refuse"` 在没有空格时拒绝新标记，而不是将最轻标记升级。整个命令都会被拒绝，因此只剩两个空格时，三个标记一个也不会落下。按编号填充的轨道总是拒绝；请在其旁边写上 `"onFull": "refuse"` 加以说明。

#### 由列表增加的等级

在带命名等级的轨道上写 `"extra": { "list": "scars", "countColumn": "levels", "penaltyColumn": "penalty" }`，人物卡上的列表就能为每个角色单独延长它。每一行按顺序以其惩罚增加其计数的等级（最多 16），放在惩罚至少同样好的最后一级之后；惩罚相同时以该级命名，否则以惩罚命名。Gravewatch 的伤疤就是这样运作的。被人物卡隐藏的列表不增加任何等级；如果某行在其等级带有标记时被删除，放不下的标记会作为溢出保留：编辑人物卡不能抹消伤害。

`boxes`、`fill`、`onFull`、`extra` 以及休息步骤的 `kind`，对打包规则集要求 Capability API 1.40。

### 掷骰惩罚

`resolution.penaltyFrom` 指定伤势轨道，其惩罚应用于规则集掷出的每次检定。它需要声明而非推定，因此省略时与伤势轨道出现之前完全一样。

惩罚如何作用，由结算类型决定，与人物卡自身数字相同：

- `dice-pool` 是**从骰池减骰**，下限为自身 `pool.min`。`pool.min` 为 1 时，即使最底一档也掷一颗；`pool.min` 为 0 时则不掷骰直接失败。
- `dice-sum` 是**掷骰固定调整值**，合入属性和训练原本添加的数字。

指定的必须是伤势轨道。普通轨道没有可应用的惩罚，导入时会拒绝。结果说明用了哪个惩罚，让玩家理解为何少掷骰；Game Master 的人物卡块也显示档位及代价。

### 来自人物卡的调整值

人物卡上有些数字会伴随它所涉及的每一次掷骰：每次攀爬都背着的沉重行囊、每次潜行都有的护甲惩罚、对一切都有效的祝福。`resolution.adjust` 就用来表达这一点，两种结算类型都适用：

```json
"adjust": [{ "value": { "derived": "burdened" }, "abilities": ["brawn"] }]
```

- `value` 是人物卡上的值，写法与派生值读取的形式相同，包括实时状态。负数表示扣除。Ember Roads 把角色打包装备的 bulk 加总成 Burden，再用阶梯表让每三点使每次 Brawn 掷骰减一。以实时状态为键的表，就是形态或姿态作用于骰子的方式（见“实时状态”）。
- `abilities` 可选。有它时，调整值只作用于使用其中某个属性的检定：该属性本身、使用该属性的技能或豁免、通过 `with=` 用该属性掷的技能，或包含该属性的属性组合。没有它时，作用于每次检定。
- 它在伤势惩罚所在的位置应用：骰池上是骰子，受同一个 `pool.min` 下限约束；求和上是固定数字，计入记录加总的调整值中。最多八项，每次检定把它们相加。
- 规则集战斗在战斗开始时，把它加到从人物卡构建的每次掷骰上：用属性或技能进行的攻击、豁免、对抗检定，以及从属性、技能或豁免读取的先攻，采用针对该属性的项；其他任何掷骰只采用适用于所有掷骰的项。
- 没有人物卡的角色得不到它，正如他们也没有调整值一样。

结果会说明人物卡加了多少（记录上的 `adjust="-2"`，以及骰子卡上的一行），让玩家明白掷骰为何是这个结果。它对打包规则集要求 Capability API 1.38。

### 休息

休息是恢复步骤和清除内容的列表。每步指定一个目标(`pool`、`poolGroup`、`listPools`、`track` 或 `state`)，并设置为某值(`"to": "max"`、`"to": "min"` 或数字)，或改变它(`"by": { "const": 1 }` 或 `"by": { "fractionOfMax": 0.5 }`)。指定伤势轨道的步骤只能治疗，因此其 `by` 为负数，见上文。指定实时状态的步骤只能设置它，取 `"default"` 或其某个值。

### Game Master 文本

- `checkGuidance` 替换内置的检定请求说明段落。请说明是什么系统、何时掷骰。Game Master 仅指定技能和难度，Engine 依据人物卡计算并掷骰，所以不要要求模型做算术。
- `sheetGuidance` 在提示词中介绍人物卡，用来说明重要资源及何时消耗。
- `worldGuidance` 可选，在生成世界时只读一次，让 GM 创造的背景符合规则，例如无火药、魔法稀有、死者行走。它从不进入回合提示词。
- `sheetSummary` 选择 GM 对各角色可见的字段、派生值和列表行。Engine 始终显示属性调整值、受训练技能和豁免，以及实时数值。其余保持简短，因为每回合都会发送。摘要列表的 `nameColumn` 可以是文本列或枚举列（枚举显示其值的标签），`columns` 指定最多三个该行自身的其他列，打印在名称之后，因此人物卡块显示为 `Gear: Crowbar 1d6`，而不只有名称。布尔列为真时打印其标签。两者对打包规则集都要求 Capability API 1.37。
- 不必在 `sheetGuidance` 中教授人物卡命令。Engine 会使用文件声明的名称自行教授每条命令；规则集有伤势轨道时，它会加上用于标记轨道的 `op="damage"` 命令，并列出轨道的等级和伤害种类。规则集有实时状态时，它会加上 `op="state"` 命令，并列出每个状态可取的值。

## 目录：人物卡列表的现成条目

逐行输入法术列表、装备表或整页职业特性很费力。目录是随规则集提供的具名现成条目集合。人物卡编辑器在目录供给的每个列表提供选择器，选中一个就填好对应行。

目录可选，规则集最多十二个。Engine 不知道它们各自关于什么；每个 ID、列、筛选条件和词语都来自文件。

### 头部

头部放在文件顶层、与 `gm` 并列的 `catalogs` 中。

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

- `id` 与 `label`：ID 遵循人物卡 ID 规则，标签是选择器名称。
- `holds`：取 `"rows"`(默认，也是本次发布前全部目录的形式)、`"creatures"` 或 `"items"`。生物目录是战斗读取的图鉴，物品目录则列出队伍携带的东西。两者都不写人物卡、不声明 `feeds`，也从不出现在选择器。见下方[生物](#creatures-a-bestiary-a-fight-reads)和[物品](#items-what-a-party-carries)。
- `feeds`：该目录条目可写入的 1 到 8 个人物卡列表。行目录必填，生物目录和物品目录禁止。条目不能写入未列出的列表，也不能写入列无法容纳的值。
- `filters`：可选，最多八个，说明选择器可按什么缩小范围。筛选项为 `number`、`text` 或 `tags`(多个词)。`startFrom` 指定选择器打开时使用的人物卡字段，所以 Calling 为 Tinker 的角色先看到 Tinker 条目。
- `units`：可选，说明条目 `mechanics` 块中的射程或区域尺寸在系统中代表什么。

### 条目

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

- `id`：小写字母、数字和单个连字符，在目录内唯一。
- `label` 与 `summary`：选择器展示的内容。摘要可选、单行、最多 300 字符。
- `filters`：头部声明的筛选值。`number` 接受数字，`text` 接受一个字符串，`tags` 接受字符串列表。
- `rows`：选中条目时写入的 1 到 6 行。`list` 是目录某个 `feeds`，`values` 按该列表的列 ID 定键。
- `creature`：在 `holds` 为生物的目录中，用对手代替行。条目恰好包含 `rows` 或 `creature` 之一；生物不带 `mechanics`，而在自身动作中说明行为。

每个值都按目标列表的列检查，因此列名拼错或数字越界会连同来源条目一起报告。规则集文件内的条目在加载时验证；导入文件就是导入时验证。包的独立目录文件在选择器首次请求时检查；有错误时显示原因而非任何条目。

### 一个条目，多个列表

有限使用次数的特性在人物卡上是两行：特性本身及跟踪次数的计数器，但仍只需选择一次。

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

### 规则集持续更新的值

行一旦被选取，数值便属于玩家。值得保留一个例外：随角色变化的上限，例如次数等于属性值，或随等级增长的职业资源。行可以在 `scaled` 映射中指定自身最多四个数字列，由人物卡编辑器维护正确值。

```json
{
  "list": "tricks",
  "values": { "name": "Last Ember", "uses": 1, "recharge": "camp" },
  "scaled": { "uses": { "from": { "abilityScore": "heart" } } }
}
```

- 键是列表某个 `number` 列。
- `from` 是普通值引用，使用与其他位置相同的封闭词汇。更复杂的计算由人物卡声明为 `derived` 值，再让 `from` 引用，例如 `"from": { "derived": "lay_on_hands_max" }`。这里不增加新算术。
- `table` 可选，将引用值按阶梯表查找，从而让等级给出数字：`"scaled": { "max": { "from": { "field": "level" }, "table": [[1, 2], [3, 3], [6, 4]] } }`。
- `values` 仍须为该列提供普通数字，缺失会被拒绝。这是尚不知道具体人物卡时的值，也是没有该引用的人物卡保留的值。
- 带 `scaled` 的行必须是该条目写入该列表的唯一一行，确保人物卡上带标记的行始终只对应一份规格。

值在编辑人物卡时计算，读取时不计算，因此已存行始终就是其声明的数字。值按目标列的 `min`、`max` 限制，整数列则向下取整。上例中 Heart 3 有三次，Heart 0 或以下没有次数。行仍以 0 次保留；上限为 0 的计数器不是资源池，因此游玩时没有可消耗内容。

打包规则集的缩放列要求 Capability API 1.23。导入社区规则集由读取它的 Engine 验证，无须另行声明。

### 选中的行是副本

每个选中行复制到人物卡时，会增加 `_catalog` 键，保存 `<catalog id>/<entry id>`。列 ID 始终以字母开头，因此该键绝不会是你自定义的列。

副本属于角色。玩家之后可编辑任何部分，未安装规则集时人物卡仍可用，发布新版也绝不会重写任何人的角色。选择器读取此标记以显示人物卡已有内容，下方 Refresh 也读取它。

### 从规则集刷新

选中行保留标记，因此人物卡编辑器能在新文本与现有行不同时提醒玩家。列表下方简短说明有多少行更新，**Review**(检查) 按钮逐项并列显示人物卡和规则集内容，每行有勾选项。玩家点击 **Update selected**(更新所选) 前不会写入，而且只写入勾选行中不同的列。其余内容全部保留，包括标记。

比较范围有意保持狭窄：

- 只比较 `text`、`longtext`、`dice` 和 `enum` 列的现有值。已有的 `number` 和 `boolean` 值属于玩家，包括 0 和 false 在内都会保留。对于行中尚不存在的列，可以提议加入符合类型的值，包括数字和开关。随人物卡缩放的列不参与比较，因为它已经跟随人物卡变化。
- 只涉及条目设置的列。条目未填写的列始终不会改动，无论人物卡中存了什么。
- 列本身会拒绝的值，例如已不再提供的 `enum` 选项或超过 `maxLength` 的文本，会跳过，不会写入。
- 在该列表中携带同一标记的各行之间，按位置将人物卡行匹配到它源自的条目行。只要人物卡中这些行的数量仍等于条目写入的数量，这种匹配就成立。否则，只有条目向该列表写入单行时才能匹配。如果玩家删除了双行条目中的一行，就保留该条目不动，不去猜测。
- 如果行所属的条目已不在目录中，该行会静默保留原状。

因此，修改条目措辞或名称可以触达已经选择它的角色，前提是玩家接受。改变数字的含义则不能、也不会这样传播：行归玩家所有之后，那一列也就属于玩家。

### `mechanics`：条目的数值效果

条目可以带可选的 `mechanics` 块，用数值说明其效果：`kind`(`attack`、`heal`、`buff`、`debuff`、`utility`、`rider`)、`range`、`area`、`targets`、`targetCount`、`friendlyFire`、`amount`(`2d6` 等骰子表达式或固定数值)、`damageType`、`attackRoll`、`autoHit`、`save`(人物卡中的一种豁免，以及成功后的效果)、`applies`(为接触到的对象施加的状态)、`temporary`(生命池的临时点数)、`scales`(随人物卡增长的数量)、`cost`(使用时消耗哪个池)、`perCostStep`、`budget`(消耗行动经济的哪部分)、`concentration`、`reaction`、`plus`、`free`、`gives`、`standard`、`rider` 和 `check`。

选择器会用一行展示这个块。其余内容由谁读取，取决于规则集选择启用了哪个块：

- 带有 [`combat` 块](#combat-a-fight-your-own-rules-resolve)时，战斗读取其中的战斗效果。`range`、`area` 和 `friendlyFire` 在有位置的战场上生效；`reaction` 表示该条目用于回应某件事，而不是在回合内执行，并指定它所等待的时刻（见[窗口](#windows-holding-the-fight-open)）；只写 `true` 的条目没有指定时刻，不会出现在任何菜单上。`check` 按前文所述应用于技能检定。
- 只有 [`battle` 块](#battles-lending-the-sheet-to-marinaras-combat)时，战斗读取 `kind`、`range`、`area`、`friendlyFire`、`amount`、`damageType` 和 `cost`，因为 Marinara 自身的战斗系统只能使用这些部分。

词汇范围是封闭的。上述列表之外的键或值会被拒绝，而不是静默忽略。

`cost` 也是 Game Master 在战斗外使用 `use` 命令时支付的费用，下一节会说明。

### `use` 命令：让 Game Master 支付你写下的费用

Game Master 在叙述时使用 `[sheet: ...]` 命令保持各人物卡最新：`spend`、`restore`(`heal` 含义相同)、`damage`、`temp`、`track`、`condition`、`note` 和 `rest`。提供目录的规则集还会多一个命令：

```
[sheet: who="Mira" op="use" name="Fireball"]
[sheet: who="Mira" op="use" name="Fireball" pool="3rd-level slots"]
```

`op="cast"` 与 `op="use"` 含义相同，`spell=` 与 `name=` 也相同。因此，无须让格式认识“法术”这个词，Game Master 自然选用的措辞也能工作。

名称会与该角色人物卡中来自你所提供目录的行进行匹配，不区分大小写。行会响应 Game Master 看到的名称(依次取该列表的 `sheetSummary` 名称列、列表的 `pools.nameColumn`、第一个文本列)，也响应原条目的 `label`，因此玩家重命名行后仍能找到它。没有任何行响应的名称，以及两个不同条目都响应的名称，均会被拒绝。

它消耗以下内容：

- 条目 `mechanics.cost` 中的每一项。指定实时池的费用项从该池支付；指定池组的费用项，按声明顺序从该组中第一个付得起的池支付。不会自动向更高的池升级，因为池组不一定是逐级阶梯。
- 再从同一条目写入的每个列表行池中各扣一份，例如追踪特性使用次数的计数器。这就是上文 `Last Ember` 条目的第二行。最大值为 0 的计数器没有可用次数，因此会拒绝命令，不会免费放行。

`pool=` 表示升阶使用：从同组另一个池支付相同的单项费用。仅当费用恰好有一项，且指定池与该项属于同一组时才接受。其他情况会被拒绝，不会另行解释。

支付要么全部成功，要么全部取消。如果任一部分付不起，整个命令都会被拒绝，不作任何更改，并告知玩家。戏法或被动特性等完全没有费用的条目会被接受，也不会改变任何数值。

### 内联或独立文件

小目录内联放在 `ruleset.json` 头部的 `entries` 中。长目录存于独立文件，头部改用 `asset` 指向它。每个目录必须且只能选其中一种。

```json
{ "id": "knacks", "label": "Knacks", "feeds": ["knacks"], "asset": "catalogs/knacks.json" }
```

路径始终是 `catalogs/<the catalog's id>.json`。文件本身如下：

```json
{ "schemaVersion": 1, "catalog": "knacks", "entries": [] }
```

独立目录文件用于通过官方目录发布的包：包必须在 `contributions.assets.paths` 中将该文件与 `ruleset.json` 一同列出，并且需要 Capability API 1.21。生物目录无论内联还是独立文件，都需要 Capability API 1.27，物品目录需要 1.49。**以单个文件导入或通过 GitHub 仓库分享的规则集，必须内联携带目录**，所以目录也得装进整个规则集文件 256 KB 的限制内。这大约能容纳数百个短条目。

限制为每个规则集 12 个目录、无论哪种方式每个目录 2000 个条目、每个目录文件 1 MB。

<a id="items-what-a-party-carries"></a>

## 物品：队伍携带的东西

护甲、武器、药水、装备、弹药和钱都是物品。可选的 `items` 块声明规则集中每件物品所用的词汇，`holds: "items"` 的目录则列出物品本身。两者都需要 Capability API 1.49；该块中管理 Game Master 自创物品的 `rarityCaps` 和 `propose` 需要 1.51；读取角色所持物品的值（`itemStat`）需要 1.52；物品在穿戴或携带时对检定的作用，以及该块的 `bonus` 上限，需要 1.53；物品对穿戴者的要求及其改变的属性需要 1.54；武器的 `attack` 需要 1.55；物品在战斗中的作用需要 1.56；武器射出什么、装填多少需要 1.57；武器的模式、副手攻击、下限和命中时施加的状态需要 1.58；在战斗中使用物品的效果及其储存的充能需要 1.59；恢复池的使用需要 1.60；休息时恢复充能，或用尽时会损坏的物品需要 1.61；先要求检定的使用（`gate`）需要 1.62；战利品表和生物的战利品需要 1.63；掉落钱币的战利品项，或移除钱币的层需要 1.64；市场、物品的 `sold` 地点和 `service` 需要 1.65；集市日（钱袋、`buyBack`、卖家的 `stock`、`limited` 数量，或不指定地点的 `sold`）需要 1.69；在手持武器上触发的附加效果（`weapons`）需要 1.70。

### 物品块

该块放在文件顶层的 `items` 中。下面是 Ember Roads 的物品块，略有删减：

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

- `categories`：1 到 24 个。每件物品恰好属于一个类别，如护甲、武器或药水。
- `rarities`：可选，最多 12 个。按从最常见到最稀有的顺序列出。
- `tags`：可选，最多 48 个。物品可能具有的特性，如可投掷或镀银。
- `stats`：可选，最多 24 个。物品携带的数字和词语，声明方式与列表列完全相同：`number`、`text`、`boolean`、`enum` 或 `dice`，类型支持时还带范围、取值和默认值。`promptVisible`(默认 `true`) 决定 Game Master 是否会在物品旁看到该数值。只应让玩家看到的内容，例如物品藏在哪里，请设为 `false`。
- `slots`：可选，最多 12 个。物品穿戴或手持的位置，以及角色有多少个该槽位，1 到 20。
- `binding`：可选。同调、灌注，或其他任何限制一个角色同时能绑定多少件物品的机制。`label` 是它的名称，`max` 是从角色人物卡读取的值，如 `{ "abilityScore": "nerve" }` 或派生值。它不能读取实时状态，因为角色能绑定多少物品不会随每次打击而改变。
- `carry`：可选。`stat` 指定作为物品重量的数字数值，其 `min` 须为 0 或以上。`encumberedAbove` 是角色在超重之前能携带多少，`limit`(可选) 是最多能携带多少。两者都像 `binding.max` 一样从人物卡读取。没有 `carry` 时，重量没有意义，也没有人会超重。
- `currencies`：可选，最多六个货币体系，每个体系 1 到 10 种钱币。
  - 同一体系的钱币按 `value` 相互兑换，它以该体系最小的钱币计数。因此最小的钱币价值为 1，同一体系中没有两种钱币价值相同。
  - 两个体系之间从不兑换。另一个国家的钱币，或某个设定中的人情，都是独立的体系。
  - 钱币的 ID 在所有体系中唯一，因为物品的价格只写钱币名。
  - `perWeight`(可选，且只能与 `carry` 同时使用) 是该体系多少枚钱币重一个携带数值单位。
  - 钱币是队伍的钱，和任何物品一样装在背包里（见下文“金钱”）。
- `rarityCaps`：可选，每个稀有度最多一项。Game Master 自创的物品在该稀有度下最多能给出多少：`stats` 中列出的每个数字数值的最大值（在该数值自身范围之内，取整数的数值须为整数），以及 `bonus`(Capability API 1.53)，即其穿戴或携带调整值所能增加的最大固定加值。在带 `bonus` 的稀有度下，以骰子表示的加值会被省略，因为骰子无法限制到一个数字；惩罚从不受限。自创物品受此约束；目录列出的物品属于你，从不受限。
- `propose`：默认为 `true`。设为 `false` 时禁止 Game Master 自创规则集的物品。
- `native`：默认为 `true`。设为 `false` 时，在使用该规则集的游戏中关闭 Game Mode 自带的无类型物品（见**什么会读取物品**）。Game Master 仍可自创物品，用规则集的词汇书写。
- `freeform`：玩家手动输入的物品会成为什么。`"plain"`(默认) 将它保留为没有规则的物品，与现在一样。`"refuse"` 只允许规则集的物品。
- `lootTables`(Capability API 1.63)：可选，最多 24 个。打赢的战斗中的生物，以及 Game Master 的 `[loot:]` 会掉落什么（见下文“战利品”）。

### 一件物品

物品目录不声明 `feeds`：

```json
{ "id": "outfitter", "label": "Outfitter", "holds": "items", "entries": [] }
```

每个条目携带 `item`，而不是 `rows` 或 `creature`：

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

- `id`、`label`、`summary` 和 `filters` 的作用与任何条目相同。标签就是物品的名称。
- `category` 必填。`rarity` 和 `tags` 可选，三者都使用该块自己的 ID。只有该块声明了稀有度，物品才能有稀有度。
- `stats`：该块各数值的值。每个值都必须是对应数值能取的值。
- `slots`：物品占用每种槽位的数量，绝不超过角色拥有的数量。
- `stack`：一叠最多容纳多少，1 到 999,999。没有它时，一叠的容量与任何 Game Mode 物品叠相同。带 `charges` 的物品每叠只有一件。
- `cost`：某种钱币的整数 `amount`，用其 `unit` ID 指定。选择器会显示它，Game Master 也会在物品旁看到它（见下文“金钱”）。
- `sold`(Capability API 1.65)：`{ "place": "city" }`，即市场中出售该物品的最小地点，优先于任何作出不同规定的规则；在 1.69 中，它也可以改为或同时说明一个地点一天备有多少件（`"limited": 2` 或 `"limited": "1d3"`）（见下文“市场”）。
- `service`(Capability API 1.65)：住宿、船票、祝福等设为 `true`：像物品一样购买，但从不携带。购买它只是付钱；其他任何操作都不会把它放进背包（添加它会以 `service` 为由被拒绝），选择器也不列出服务。服务有 `cost`，但没有任何与携带或使用相关的内容（`slots`、`stack`、`binds`、`worn`、`carried`、`requires`、`attack`、`use`、`charges`）。
- `binds`：物品必须先绑定，穿戴时才会产生任何作用。`restriction`(可选) 用文字说明谁可以绑定它，`cursed: true` 标记一旦绑定就无法摆脱的物品。只有带 `binding` 的规则集才能有需要绑定的物品。
- `worn` 和 `carried`(可选，Capability API 1.53)：物品在穿戴时及仅携带时对持有者检定和豁免的作用，它设定或提高的属性（1.54），以及它在战斗中的作用（1.56）。见[战斗外的检定](#checks-outside-a-fight)和[战斗中的护甲与穿戴效果](#armor-and-worn-effects-in-a-fight)。
- `requires`(可选，Capability API 1.54)：物品对穿戴者的要求，以及未达到要求时适用的效果。见[战斗外的检定](#checks-outside-a-fight)。
- `attack`(可选，Capability API 1.55)：物品穿戴时在战斗中作为武器的作用，它射出什么、装填多少（1.57），以及它的其他模式、副手攻击、伤害下限和命中时施加的状态（1.58）。见[战斗中的武器](#weapons-in-a-fight)、[弹药与装填](#ammunition-and-reloading)，以及[模式、第二把武器、下限和命中时的状态](#modes-a-second-weapon-a-floor-and-conditions-on-a-hit)。
- `use` 和 `charges`(可选，Capability API 1.59)：在战斗中和战斗外使用该物品的效果、它为该使用储存的充能，以及它恢复的池（1.60）。见[在战斗中使用物品](#using-items-in-a-fight)和[在战斗外使用物品](#using-items-outside-a-fight)。

物品不携带 `mechanics`：它的作用写在自己的 `item` 块中。

### 什么会读取物品

以上所有内容都会在导入规则集时检查，物品目录也随规则集一起提供。游戏中，Game Mode 的物品栏会读取它们：

- **你的物品就在背包里。** 一叠物品可以是你的某件物品，无论玩家如何称呼，它始终是那件物品。物品栏的 **From the ruleset**(来自规则集) 按钮会打开一个选择器，列出目录中的所有物品，带目录的搜索和筛选；隐藏某个条目的层，在这里也会隐藏它。
- **名称能找到你的物品。** 玩家输入的名称，或 Game Master 在 `[inventory: action="add"]` 中写的名称，只要是你某件物品的 `label`(不区分大小写)，就会添加那件物品。两件物品标签相同时，名称找到的是目录中先列出的那一件。
- **`stack`** 会被遵守：添加、设置、合并或给予超过上限时，会填满这一叠并开始新的一叠。
- **`freeform: "refuse"`** 让玩家只能使用你的物品：选择器，以及属于你物品的名称。
- **`native: false`** 让 Game Master 只能使用你的物品和它自创的物品：两者都不是的名称会被拒绝（`not-ruleset-item`），但已持有的东西仍可以继续添加，它的指令也会这样说明。战斗不再询问模型物品栏中的物品有什么作用，因此只有你的物品参与战斗：在规则集战斗中是手持的武器和物品的使用（见下文“战斗中的武器”），在 Classic 或 Tactical 战斗中是带 `use` 的物品（见下文“Classic 与 Tactical 战斗中的物品”）。玩家输入的内容仍遵循 `freeform`，而队伍携带的东西无论哪种情况都会在新会话中回来。
- **物品是什么**会显示在选中的那一叠上：按标签显示其类别、稀有度和标签，它给出的数值、摘要，以及一叠能容纳多少。选择器还会显示它的 `cost`。Game Master 会看到物品栏中你的每件物品，连同类别、稀有度、标签，以及你保留 `promptVisible` 的数值，例如 `Hand axe [Weapon, Common, Thrown; Damage 1d6, Reach close]`。

- **`slots`**：占用槽位的物品可以由携带者装备，前提是对应槽位有空；较大的一叠中，会取出一件单独成叠来穿戴。物品栏会按角色显示每个已占用的槽位。
- **`binding`**：带 `binds` 的物品可以绑定，数量上限为从持有者自己的人物卡读取的 `binding.max`(没有人物卡的角色读取空白人物卡)。`cursed` 物品一旦绑定就保持绑定：玩家不能解除绑定、脱下、送人或移除它，只有 Game Master 能结束诅咒。
- **`carry`**：物品的重量是它的 `carry.stat` 值（没有该值的物品没有重量），角色的负重是其背包的重量，与从其人物卡读取的 `encumberedAbove` 和 `limit` 比较。由玩家，或由不带 `who=` 的 Game Master 添加到物品栏共享视图中的物品，会交给能携带它而不超重的人（先是玩家，再按顺序是队伍成员）；没有人能全部拿下时，按各人剩余的空间分配。任何人都不会超过自己的 `limit`：没有人能携带的东西会被留下，并告知 Game Master。像四分之一磅这样的重量，请给重量数值设 `"integer": false`。
- **Game Master 用你的词汇自创物品。** 除非你设置 `propose: false`，否则它的 `[inventory: action="add"]` 可以描述一件新物品：先用 `like=` 指定作为起点的某件你的物品，再写任意 `category=`、`rarity=`、`tags=`、`stats=`(`id=value` 对)、`slots=`(`id=count`)、`binds=`(`yes`、`cursed` 或 `no`)、`worn=` 和 `carried=`(用 `;` 分隔的改动，每项为 `+N`、`-N`、`advantage`、`disadvantage`，或针对豁免的 `fails`，作用于按名称指定的技能或豁免，或作用于代表全部的 `checks` 或 `saves`；在属性名上写 `+N` 或 `-N` 以提高或降低属性；规则集有战斗时，还可以在攻击上写 `+N`、`-N`、优势或劣势，在防御或你对防御的称呼上写 `+N` 或 `-N`：`worn="+1 Sneak; disadvantage on Sway checks"`、`carried="+1 Brawn"`、`worn="+1 Guard"`，或 `none`) 以及 `summary=`，每部分都用 ID 或标签指定。如果防御已经计入某个物品数值（Ember Roads 的 Guard 会加总所穿戴物品的 `guard`），护甲就是靠该数值提高防御的，因此与这种数值并列的穿戴防御改动会被省略，答复也会说明：小模型会为同一个 +1 同时写 `guard=1` 和 `+1 Guard`。它也能读懂小模型常写的替代写法：名称后的数字（`Brawn +1`）、攻击或防御后的 “bonus”（`+1 attack roll bonus`）、表示空标签、空数值或空槽位列表的 `none`，以及没有同名数值时写在 `stats=` 里的 `worn=`、`carried=` 或 `summary=`。Engine 只保留你的块中有的内容：未知的类别、标签、数值、槽位、技能或豁免会被省略；没有的稀有度变为最低稀有度；数字先限制在其数值的范围内，再按其稀有度限制到 `rarityCaps`；穿戴或携带加值限制到其稀有度的 `bonus`(`like` 作为起点的部分也一样)；名称就是你某件物品的，直接就是那件物品。答复会告诉 Game Master 改动了什么（绝不涉及你不向它展示的数值），物品详情会向玩家显示每一处改动。游戏会保存这件物品，因此在本局游戏余下的时间里，同一名称都指这件物品；只要还有人持有，新会话也会保留它。
- 规则集有 `slots` 时，Game Master 可以用物品栏命令 `equip` 和 `unequip` 你的物品；有 `binding` 时，可以 `bind` 和 `unbind` 它们：它只会被告知规则集具有的那些操作。它能看到每个角色的负重、已绑定物品和槽位，以及穿戴或绑定了什么。

- **物品在穿戴或携带时的作用**显示在选中的那一叠和选择器中（“While worn: -1 on checks (Sneak)”），Game Master 也会在物品旁看到它（`worn: -1 on checks (Sneak)`）。战斗外的检定会应用它（见下文“战斗外的检定”）。
- **武器的作用**以同样方式显示（“Attack (Action): Brawn to hit, 1d6 + Brawn cut damage”），手持时战斗会提供它（见下文“战斗中的武器”）。
- **使用物品的效果**也以同样方式显示（“Use (Action): heals 1d4 + 1, range 0 paces, used up”），并附带剩余充能。战斗会提供它，物品栏的 **Use**(使用) 按钮和 Game Master 的 `[inventory: action="use"]` 也一样（见下文“在战斗中使用物品”和“在战斗外使用物品”）。

在 Classic 或 Tactical 战斗中，每件带 `use` 的物品都按其描述生效，没有它的物品不会提供（见下文“Classic 与 Tactical 战斗中的物品”）。无论 `native` 如何设置：人物卡都能读取你的物品（见下文）；在规则集战斗中，手持的武器就是一次攻击，射出从背包中取出的东西；物品在穿戴或携带时的作用在规则集战斗中也会计入（见下文“战斗中的护甲与穿戴效果”）；带 `use` 的物品在战斗中和战斗外都按其自身规则使用（见下文“在战斗中使用物品”和“在战斗外使用物品”）；你的 `currencies` 就是队伍的钱（见下文“金钱”）。

<a id="items-on-the-sheet"></a>

### 人物卡上的物品

值引用可以用 `itemStat` 读取角色持有的物品（Capability API 1.52）。Ember Roads 把旅人所穿护甲的 guard 加到其 Guard 上，也就是打击必须超过的数值：

```json
{
  "id": "guard",
  "label": "Guard",
  "op": "sum",
  "of": [{ "const": 6 }, { "abilityMod": "wits" }, { "itemStat": { "stat": "guard", "from": "worn", "pick": "sum" } }]
}
```

- `from`：`"worn"` 是角色身上的物品：占用槽位的物品在装备时、带 `binds` 的物品在绑定时、两者兼有的物品在两者都满足时。两者都不是的物品从不算作穿戴。`"carried"` 是其持有的其余物品，`"all"` 是两者合计。
- `pick`：`"sum"` 把每件物品的值乘以数量后相加，`"max"` 和 `"min"` 读取单件的最高或最低值，`"count"` 按数量清点物品。
- `stat` 指定某个物品数值。`sum`、`max` 和 `min` 需要 `number` 数值。`count` 可以省略它以清点所有物品，也可以指定任意数值，只清点给出该数值的物品。
- `slot`、`category` 和 `tag`(均可选) 只保留占用该槽位、属于该类别或带有该标签的物品。
- `default`(可选，省略时为 0) 是没有选中物品，或选中的物品都不给出该数值时读到的值。

物品会在游玩中变化，因此 `itemStat` 受与实时读取相同的规则约束：池或轨道的 `max`、熟练加值、`binding.max`、`carry` 的数字，以及目录的缩放列或缩放都不能读取它，经由派生值也不行。其他任何读取数字的地方都可以：派生值、检定的 `adjust`、技能的 `cap`、战斗的防御、吸收或先攻。

角色的物品就是其自己背包里的物品。玩家的角色卡（聊天设定为玩家扮演的那张，否则为第一张）读取玩家的背包，其他每张角色卡读取各自的背包。只有规则集的物品才计入，因为普通物品没有可读取的数值。屏幕上的人物卡、检定、Game Master 的人物卡块，以及开始时的战斗，都读取每个角色当时持有的东西。在游戏之外，即人物卡编辑器或导入审核中，没有人持有任何东西，`itemStat` 读取其 `default`。

<a id="checks-outside-a-fight"></a>

### 战斗外的检定

Game Master 在战斗外要求的检定或豁免，读取的不只是人物卡：还有角色自身生效的状态、已达到的等级、物品在穿戴或携带时的作用，以及未达到穿戴物品要求时的代价。它使用与战斗状态相同的词汇（物品相关的键和范围收窄需要 Capability API 1.53）。

物品用 `worn` 说明穿在身上时（需要绑定的还须已绑定）的作用，用 `carried` 说明仅携带时的作用。Ember Roads 的皮外套在你潜行时会吱嘎作响，而路石让商队的人信任你：

```json
"worn": { "modifiers": [{ "to": "checks", "skills": ["sneak"], "flat": -1 }] }
```

```json
"carried": { "modifiers": [{ "to": "checks", "skills": ["sway"], "flat": 1 }] }
```

两者都使用检定所读取的那部分状态内容：

- `effects`：`own-checks-advantage`、`own-checks-disadvantage`、`own-saves-advantage` 和 `own-saves-disadvantage`。
- `modifiers`：用 `to` 指定改动检定或豁免，带 `flat`、`dice`(及 `minus`)、`mode`，以及各自的 `skills` 或 `saves`，与状态的写法完全相同。
- `failsSaves`：持有者无须掷骰便失败的豁免。
- `skills` 和 `saves`：效果以及未自行指定范围的调整值涉及哪些技能或豁免。
- `abilities`(Capability API 1.54)：按 ID 指定它改变的属性，每项为 `{ "set": N }`(至少为 N：更高的分值保持不变) 或 `{ "add": N }`(负数则降低)。它们在计算人物卡之前应用，因此读取该属性的一切都读取改变后的值：屏幕上的人物卡、派生值、检定、Game Master 的人物卡块，以及开始时的战斗。先加上增加值，再取最高的 `set`，属性自身的 `min` 和 `max` 仍然有效。池或轨道的最大值以及熟练加值只根据人物卡计算，因此物品不会改变它们。

Ember Roads 的牛皮护手能让无力的手握得像赶牲口的人一样牢：

```json
"worn": { "abilities": { "brawn": { "set": 2 } } }
```

物品在战斗中的作用（防御、攻击、速度、战斗的其他效果，以及它能挡下的伤害和状态）见[战斗中的护甲与穿戴效果](#armor-and-worn-effects-in-a-fight)；战斗外的检定只读取上述部分。无论某人持有同一物品多少叠，它都只生效一次。

物品还可以用 `requires` 对穿戴者提出要求：其人物卡上的某个值（`value`，任意值引用，读取方式与游玩中的一切相同）、该值的最低要求（`atLeast`），以及未达到时适用的效果（`otherwise`，使用相同词汇，但不能改变属性，因为要求本身可能读取属性）。最多四项。对力弱的守望者来说，Gravewatch 的墓地铁锹是份重活：

```json
"requires": [
  {
    "value": { "abilityScore": "sinew" },
    "atLeast": 3,
    "otherwise": { "modifiers": [{ "to": "checks", "skills": ["dig"], "flat": -1 }] }
  }
]
```

要求读取的是穿戴者的物品改变之后的属性，因此把 Sinew 设为 3 的护手就能满足铁锹的要求。

检定时：

- 与该检定相关的每个调整值都会加上其数字，并掷出其骰子：求和检定上是固定数字，骰池上是骰子。属性检定，以及规则集无法指名的检定，只读取未收窄到任何范围的内容；豁免只读取与豁免相关的内容。
- 来自各个来源的优势和劣势（包括 Game Master 的 `mode=`）会相互抵消：两种各有任意一个，检定就只掷一次。只有本来就会掷两次的规则集（`resolution.advantage`）才会偏向一方。
- 被状态或物品判定失败的豁免无须掷骰直接失败，也不为它消耗任何东西。
- 记录会说明是什么改变了它：数字用 `effects="-1"`，改变它的来源用 `from="Leather coat"`，无须掷骰便失败的豁免用 `automatic="true"`。骰子卡显示相同内容，Game Master 会在下一回合读到。它会被告知 Engine 已应用这些内容，因此不会再加一次。

Gravewatch 在骰池上展示了同样的做法：已绑定的 Dawn bell 为 Ward 加一颗骰子，Rattled 则从 Soothe 和 Barter 中减去一颗。

<a id="weapons-in-a-fight"></a>

### 战斗中的武器

物品可以是武器（Capability API 1.55）。它的 `attack` 与战斗块的[攻击行](#combat-a-fight-your-own-rules-resolve)形状相同，只是用值代替列；物品穿戴时，[规则集战斗](#combat-a-fight-your-own-rules-resolve)会把它作为攻击提供：拿在它所占用的手中，需要绑定的还须已绑定。收起或仅携带的武器不提供任何东西，攻击行和生物自身的行动与以前完全一样。Ember Roads 的手斧：

```json
"attack": {
  "budget": "act",
  "toHit": { "abilities": { "stat": "swing" } },
  "damage": { "dice": { "stat": "damage" }, "abilities": { "stat": "swing" }, "type": "cut" },
  "reach": 2,
  "range": { "normal": 10, "long": 20 }
}
```

- `budget`：它消耗的额度，取 `combat.economy.budgets` 之一。
- `toHit`：命中时加上什么。`abilities` 是一个或多个属性 ID，取其中最好的一个（灵巧武器会列出两个）。`skill` 加上该技能检定所加的值，并像攻击行一样，用攻击的属性代替技能自己的属性。`proficiency` 是持有者人物卡上的值：读数大于 0 时，加上规则集的熟练加值（`{ "const": 1 }` 表示始终加）。`bonus` 是一个数字。`target` 是骰池战斗中这把武器攻击专用的单骰目标（Gravewatch 的铁锹从 6 起计每颗骰子，而不是骰池的 7），只有 `target` 可调的 `dice-pool` 规则集才能给出；在求和战斗中，用它的 `bonus` 表达同样的意思。
- `damage`：它造成的 `dice`、加上其伤害 `abilities` 中最好的一个、`bonus` 和 `type`。求和战斗需要 `dice`；在骰池战斗中，命中造成的伤害等于其成功数，`dice` 则再加上相应数量的骰池自身骰子。
- `reach` 和 `range`(`normal`，以及表示超出后仍能打到多远的 `long`) 使用你的 `combat.distance` 单位，因此需要声明它。两者都有的武器是投掷武器，与攻击行相同：近处挥击，远处投掷。两者都没有时触及一格。
- `versatile`：当它占用的每种槽位，在持有者所穿戴的物品中还能再容纳同样多时，改为造成的 `dice`：一手持矛、另一只手空着。
- `strikes`：一次消耗能买几次打击，从持有者的人物卡读取，与攻击行的 `strikes` 相同。没有它的武器每次消耗一次打击，无论使用者有多少次攻击。

这些数字或词语中的任何一个都可以改为读取物品自己的数值：`{ "stat": "damage" }` 就是物品的 `damage`，物品没有给出的数值视同没有写这个值。`abilities` 和 `skill` 读取取值为属性或技能 ID 的枚举数值，`type` 读取文本或枚举数值。因此，Game Master 用 `like=` 某件武器自创的物品也是武器，用自己的数值作战，并像任何数值一样受 `rarityCaps` 限制（你直接写下的值按原样复制）。自创时没有槽位也没有绑定的物品从不算作穿戴，因此不会带有攻击。小模型往往会描述一件武器却省略 `like=`，所以在你的某个武器类别中自创、没有起点的物品，会按名称最相近的该类别武器作战（任一方向共用一个词即可，所以 crossbow 和 bow 相近），找不到就按该类别的第一件武器，并采用该攻击读取、而提议中省略的数值，答复也会说明这一点（“It fights as Hand axe does.”）。Game Master 的提议格式也会告诉它，照某件武器做出的武器会像它一样作战。

武器的标签会随其打击一起传递。生物的 `resist` 或 `immune` 项可以说明什么能穿透它：`{ "type": "tearing", "except": ["silver"] }` 表示来自带 `silver` 标签武器的伤害全额承受，来自其他东西的则被抵抗。Gravewatch 的墓地尸妖就是这样写的，银棺钉能够穿透它。

武器详情和 Game Master 看到的那一行会说明它的作用：`attack (Action): Brawn to hit, 1d6 + Brawn cut, reach 2 paces, range 10 to 20 paces`；Ember Roads 的野猪矛会带 `1d8 with a hand free`，Gravewatch 的铁锹旁会有 `at 6`。

<a id="ammunition-and-reloading"></a>

#### 弹药与装填

武器可以射出某种东西，也可以自己保留已装填的数量（Capability API 1.57）。Ember Roads 的猎弓射箭，Gravewatch 的守夜手枪装一颗弹丸，从守望者的弹丸和火药中装填：

```json
"ammo": { "tag": "arrow", "recover": 0.5 }
```

```json
"ammo": { "tag": "shot" },
"clip": { "max": 1, "reload": "act" }
```

- `ammo`：武器射出什么，用某个物品 `tags` 指定。每次攻击从持有者携带的（无论是否穿戴）带该标签的物品中取 `perAttack` 个（未写时为 1），先从背包中第一叠取，再取下一叠。只有携带足够时才会提供该武器，战斗菜单会显示剩余数量。队伍打赢战斗后，每人射出的东西中有 `recover`(0 到 1) 的比例回到其原来那一叠：比例按整场战斗累计，每叠只向下取整一次，因此从同一个箭袋射出五支箭，收回一半就是两支。打输或逃跑的战斗不会收回任何东西。
- `clip`：武器自身保留的已装填数量。`max` 是它能装多少，可以直接写下，或从物品的某个数字数值读取；`reload` 是装填一次消耗的额度。攻击消耗已装填的弹药，而不是从背包取，只有装填量足够一次攻击时才会提供该武器。战斗菜单上的 **Reload**(装填) 选项消耗 `reload` 并将其装满：有 `ammo` 时从背包取（背包剩余较少时就取剩下的全部），没有时直接装满。数量保存在武器的物品栏那一叠上，因此在一场战斗中打空的手枪，下一场仍是空的。从未开过火的武器是已装填的；玩家把它倒入有多件同类武器的一叠时也一样，因为装填数属于单件武器。弹匣里的弹药在战斗后不会捡回，因此 `recover` 适用于没有弹匣的武器。

战斗射出、装填和捡回的东西，会在每一步执行时写入物品栏，与队伍的生命值一样，冒险日志也会记下用掉了什么。如果某一步对应的那一叠已经不在，或数量少于战斗所预计的，这一步会被拒绝，就像战斗找不到已消耗的物品时一样。战斗日志记录每次射击、装填和捡回（“Hunting bow: 2 left to shoot.”、“Ada loads 1 into Watch pistol: 1 of 1 loaded.”），物品详情和 Game Master 看到的那一行会说明武器射出什么、装了多少：`ammunition Arrow (1 an attack, 50% picked up after a won fight)` 和 `holds 1, reload (Act)`。

<a id="modes-a-second-weapon-a-floor-and-conditions-on-a-hit"></a>

#### 模式、第二把武器、下限和命中时的状态

武器可以有其他攻击方式、副手中的搭档、造成伤害的下限，以及施加给被命中者的状态（Capability API 1.58）。Ember Roads 的猎弓可以齐射，Gravewatch 的银棺钉可以两手各持一枚，并标记被它深深刺中的目标：

```json
"modes": [{ "id": "volley", "label": "Volley", "ammo": 2, "toHit": -2, "targets": 2 }]
```

```json
"offHand": true,
"onHit": [{ "condition": "marked", "atLeast": 2, "rounds": 2 }]
```

- `modes`：最多六种其他攻击方式，每种有 `id`、`label` 及其改变的内容。`ammo` 是该模式下一次攻击射出的数量，因此武器须有 `ammo` 或 `clip` 作为弹药来源。`toHit` 是命中加值，骰池战斗中是骰子，求和战斗中是数字。在骰池目标可调时，`target` 将骰池战斗的单骰目标移动这么多（以武器自身的目标为基准，没有则以骰池的为准；`+1` 表示更难）。`targets` 是它可以瞄准几个目标，每个目标各作一次攻击检定。选定攻击后（有先攻方式时在选定方式之后），战斗菜单会询问用哪种模式，每种都附带预期效果，并只提供持有者弹药足够的模式。Engine 扮演的队伍成员会评估瞄准单一目标的每种模式，而把瞄准多个目标的模式留给玩家。在窗口中，攻击按原样进行。
- `offHand`：副手武器。战斗块指定第二次攻击消耗的额度，以及其伤害是否保留为正的属性值：

  ```json
  "offHand": { "budget": "quick", "ability": "full" }
  ```

  持有者本回合用一件标为 `offHand` 的武器攻击过之后，其穿戴的其他每件此类武器都会提供一次消耗该额度的第二次攻击，在菜单上名为 “Silver coffin nail, off hand”。`ability` 为 `full`(默认) 或 `penalty-only`；后者只在伤害属性为减值时才加上它，与 5e 的双武器战斗相同。副手攻击只是一次打击，无论主攻击买了几次打击，也绝不会用于打击走开的人。标为 `offHand` 的武器需要战斗块的 `offHand`。
- `floor`：一次命中至少造成的伤害，可以直接写下，或从物品的某个数字数值读取。骰池战斗中吸收后的伤害，或求和战斗的伤害，会在抗性减半之前提高到这个值，日志也会说明。消耗型打击掷出的是攻击者自身的数字，从不提高。
- `onHit`：打击造成的伤害达到 `atLeast` 时，目标会受到的最多四种状态，持续其自身的 `rounds` 个回合；没有它时，持续到被某种东西移除。伤害指吸收和抗性之后实际作用于目标的部分；夺取型打击夺取的是先攻而不是造成伤害，因此不会施加状态。

武器详情和 Game Master 看到的那一行会说明全部内容：`modes Volley (2 shots, -2 to hit, up to 2 targets)`、`off hand (Quick)`、`at least 1 on a hit before resistance` 和 `Marked for 2 rounds when a hit deals 2 or more`。

<a id="armor-and-worn-effects-in-a-fight"></a>

### 战斗中的护甲与穿戴效果

物品在穿戴或携带时的作用也可以改变战斗（Capability API 1.56），作用于其持有者，与其状态和等级并列。两者都使用战斗所读取的那部分状态内容：

- `effects`：状态的任何效果，但等级不能有的四种除外（`half-move-to-stand`、`ends-on-damage`、`cannot-target-source`、`cannot-approach-source`），因为没有人把物品施加给持有者，它也从不自行结束：`own-attacks-advantage`、`attacks-against-disadvantage`、`resist-all`、`speed-zero` 等。
- `modifiers`：用 `to` 改动防御、攻击和速度，以及检定和豁免，与状态的写法完全相同；在 `dice-pool` 战斗中，对攻击的改动是固定数量的骰子，与状态相同。
- `resist`、`vulnerable` 和 `immune`：持有者受到一半、双倍或完全不受的伤害种类，与生物的写法相同（声明了 `combat.damageTypes` 时还会据此检查）。`conditionImmunities`：永远不会施加给持有者的状态。

Gravewatch 受诅咒的寡妇戒指让每次攻击少一颗骰子，已绑定的 Dawn bell 让持有者不会陷入 rattled，而携带 Ember Roads 的路石能挡住灼热：

```json
"worn": { "modifiers": [{ "to": "attacks", "flat": -1 }] }
```

```json
"worn": { "modifiers": [{ "to": "checks", "skills": ["ward"], "flat": 1 }], "conditionImmunities": ["rattled"] }
```

```json
"carried": { "modifiers": [{ "to": "checks", "skills": ["sway"], "flat": 1 }], "resist": ["burn"] }
```

与检定一样，战斗对每件物品只读取一次，并以那一叠的名称标注（日志中掷骰旁和防御上的 “Widow's ring”）；未满足要求时的 `otherwise` 在战斗中同样计入，因此对穿戴者过重的护甲会降低速度。物品详情和 Game Master 看到的那一行会说明全部内容（“While worn: -1 on attacks”、“worn: +1 on checks (Ward), immune to Rattled”）。只是增加防御或吸收的护甲，是用 `itemStat` 读取的物品数值（见“人物卡上的物品”），就像 Ember Roads 的 Guard 读取所穿的外套一样。

**硬度。** 在先攻是会被攻击改变的数字时，`combat.pool.hardness` 是战斗者的硬度，在战斗开始时从其人物卡读取（通过 `itemStat` 读取其护甲的硬度），生物则给出自己的 `hardness`。骰子数低于硬度的消耗型打击虽然命中，却没有任何效果：攻击者的数字仍会回到基准值，日志会说明（“Ada's Grave spade lands on the warden with 5 dice, below a hardness of 6, and does nothing.”），菜单也不会为它预测伤害。只有带消耗型方式的规则集才能使用它，以人物卡编写的生物从人物卡获取它。

<a id="using-items-in-a-fight"></a>

### 在战斗中使用物品

物品可以在战斗中使用（Capability API 1.59）：按在伤口上的药膏、吞下的补剂、敲响的铃铛。它的 `use` 用目录条目 `mechanics` 的词汇说明使用效果（见[战斗会从 `mechanics` 读取什么](#what-a-fight-reads-from-mechanics)）：`kind`(`heal`、`attack`、`buff` 或 `debuff`)、`amount`、`damageType`、`plus`、`attackRoll`、`save`、`applies`、`temporary`、`range`、`area`、`targets`、`targetCount` 和 `friendlyFire`，以及它消耗的 `budget`，或 `free`。Ember Roads 的药膏，以及 Gravewatch 的暖身补剂和 Dawn bell：

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

- 使用出现在战斗菜单的 **Items**(物品) 下，以物品名称命名（“Poultice”），无论那一叠叫什么。占用槽位或需要绑定的物品在穿戴时（需要绑定的还须已绑定）使用，就像铃铛那样；其他物品在携带时即可使用。
- 它消耗一份额度，或为 `free`，并以自身作为代价：`cost`、`perCostStep`、`check`、`concentration`、`reaction`、`scales`、`gives`、`standard` 或 `rider` 属于人物卡持有的目录条目，写在使用上会被拒绝。
- 带 `attackRoll` 的使用上，`toHit` 是命中加值，写法与武器的 `toHit` 相同（见“战斗中的武器”）：属性、技能、加值，在骰池战斗中还有单骰目标。没有它时，掷骰不加任何值。
- `saveDifficulty`：它所要求的豁免要对抗的数字（它自己的豁免、结束其所施加状态的豁免，或没有 `difficulty` 的子项豁免），可以直接写下，或从物品的某个数字数值读取。人物卡上的条目从其目录来源读取这个数字；物品没有来源，所以自己写明。
- `consumes: true` 每使用一次就从物品那一叠中减去一件，菜单会显示剩余数量。用掉最后一件时，那一叠也随之消失。
- `restore`(Capability API 1.60) 为它作用到的每张人物卡恢复某个池的一部分，就像血袋补回血液：`{ "pool": "resolve", "amount": { "flat": 1 } }`，`dice` 和 `flat` 的写法与 `amount` 相同。它指定某个实时池，但绝不能是生命池（生命通过治疗恢复），并且只能用于有益的使用，即 `heal` 或 `buff`。生物没有池，因此什么也得不到。日志会说明（“Ada gets back 1 Resolve, and is on 3 of 4.”），Engine 扮演的队伍成员不会对已满的池使用它。Gravewatch 的暖身补剂在治疗之外还恢复一点 Resolve。
- `charges` 消耗物品自身 `charges` 中相应的数量，后者的 `max` 是它能储存多少，1 到 100，可以直接写下，或从物品的某个数字数值读取（给出的该数值小于 1 的物品会被拒绝，超过 100 的按 100 计）。数量保存在物品的物品栏那一叠上，因此一场战斗中敲过两次的铃铛，下一场只剩一次充能，而从未敲过的铃铛是满的。充能属于单件物品，因此带充能的物品 `stack` 为 1。一次使用要么用掉物品，要么消耗充能，不会两者兼有，而物品的充能总是由其使用消耗。`charges` 还有两个键（Capability API 1.61），Gravewatch 的 Dawn bell 就带有这两个键：

  ```json
  "charges": { "max": 3, "recharge": { "rests": ["vigil"], "amount": "max" }, "breaksOn": { "die": 20, "atMost": 1 } }
  ```

  - `recharge`：哪些 `rests` 会恢复充能，以及恢复多少：`"max"` 将物品充满，数量（`{ "dice": "1d6", "flat": 1 }`）则恢复相应的数量，绝不超过 `max`。休息会恢复该角色携带的、指定了这种休息的每件物品的充能，无论是通过人物卡的 Rest 按钮还是 Game Master 的 `[sheet: op="rest"]` 进行；人物卡和背包会一起写入，重新生成的回复只会恢复一次。休息后，人物卡会说明恢复了什么（“Dawn bell 3/3 charges”），Game Master 也会在物品栏中每件物品旁看到其剩余充能（“2 of 3 charges left”）。
  - `breaksOn`：使用消耗最后一点充能时，Engine 掷一颗 d`die`，结果不高于 `atMost` 时物品损坏，从背包中消失，战斗中和战斗外都是如此。战斗日志会说明（“Dawn bell breaks (a 1 on its die).”），Use 按钮的报告也会说明（“Its last charge spent, it breaks.”），冒险日志会把它记为遗失。
- `gate`(Capability API 1.62) 是使用者在物品生效前必须通过的检定，就像高于阅读者自身水平的法术卷轴所要求的那样。`check` 指定掷什么：`{ "skill": "ward" }`、`{ "ability": "wits" }`，或人物卡上的某个值（`{ "value": { "derived": "spell_mod" } }`，用于因角色而异的数字，例如 5e 施法者的施法调整值）。`difficulty` 取 1 到 100，可以直接写下，或从物品的某个数字数值读取（没有给出该数值的物品不会出现在菜单上，其 Use 按钮会直接说明原因）。使用者人物卡上的某个值至少为 `atLeast` 时，`unless` 跳过检定。检定失败时，物品白白用掉（或白白消耗充能）。在战斗中，检定在消耗物品时掷出，使用战斗自己的骰子（骰池战斗中是骰池，此时 `difficulty` 是所需成功数），以及战斗开始时读取的人物卡数字；使用者的状态和穿戴物品对该检定的加值都会计入，包括技能自身的。日志会说明（“Ada rolls Ward to use Page of the vigil litany: 2 successes from 2 dice at 7 or more (8, 9), needing 2 successes, a success.”）。Gravewatch 的守夜祷文残页就带有它：

  ```json
  "gate": { "check": { "skill": "ward" }, "difficulty": 2, "unless": { "value": { "abilityScore": "nerve" }, "atLeast": 3 } }
  ```

战斗用掉的物品和剩余的充能，会像武器的射击一样在每一步执行时写入物品栏，冒险日志也会记下用掉了什么。战斗日志记录每次使用（“Poultice: 1 of 2 left.”、“Dawn bell: 2 of 3 left.”）。Engine 扮演的队伍成员只对受伤的人使用治疗，也没有人会用物品打击走开的人。物品详情和 Game Master 看到的那一行会说明使用效果：“Use (Action): heals 1d4 + 1, range 0 paces, used up” 和 `use (Act): Steel 7 save negates it, Rattled, 1 of 3 charges`，详情还会显示剩余充能。Game Master 用 `like=` 你的某件物品自创的物品，会像那件物品一样使用，充能等一应俱全。

### Classic 与 Tactical 战斗中的物品

不自行结算战斗的规则集，会把战斗交给 Game Mode 自己的 Classic 和 Tactical 战斗；在那里，战斗开始时会询问模型物品栏中的物品有什么作用。你的物品不靠猜测：每件带 `use` 的物品都按其描述、以 Engine 自己的数值生效，只有不属于你的物品才会请 Game Master 猜测（`native: false` 时一件也不会）。

- `heal` 治疗、`attack` 伤害的量，是按其 `amount` 平均值换算的目标最大生命值比例：平均值 7，即基础武器的 `1d8+3`，略多于五分之一，最少二十分之一，最多为全部。这与战斗桥接读取目录条目数值时使用的尺度相同。`damageType` 就是元素。
- `applies` 中的第一个状态以其自身名称作为状态效果施加，持续其 `rounds` 回合（没有时为 2）；`buff` 或 `debuff` 就只是这个状态效果，像 Engine 自身的状态效果一样提高或降低防御。
- `targets` 说明对谁使用。没有它时，治疗或增益用于友方，攻击或减益用于敌方。
- `consumes: true` 从那一叠中减去一件，没有它的使用会把物品留在原处。
- 命中检定、豁免、范围、临时点数和恢复池，在这些战斗中都没有对应位置，与目录条目的 `mechanics` 在其中的情况相同：菜单上的物品描述仍会写出它们。
- 与规则集战斗中一样，占用槽位的物品只在穿戴时使用，需要绑定的物品只在绑定时使用。用掉的物品从穿戴的那件中扣除，绝不从背包里的备用品中扣。
- 使用会消耗 `charges` 的物品，在还剩可用次数时提供，Items 菜单会显示其使用次数。每次使用都从该物品那一叠中消耗，先用玩家自己的；消耗最后一点时掷其 `breaksOn`：损坏后它会从背包中消失，冒险日志把它记为遗失。
- 先要求检定的物品（`gate`）在队伍成员使用时由服务器掷出检定，使用该成员的人物卡和穿戴物品（没有自己角色卡的同伴用空白人物卡掷骰，只有玩家自己的单位才会回退到玩家的角色卡），与 Use 按钮的掷法相同；`unless` 成立时跳过。失败时，物品被消耗（或其充能被消耗）且没有任何效果，日志会说明：“Ada rolls Ward to use Page of the vigil litany: 0 against 2, failed, and it is used up for nothing.”
- 没有 `use` 的物品不会在这些战斗中提供，没有 `amount` 的治疗也一样。

Engine 自行计算每件物品的效果，因此画面发送的内容永远决定不了你的物品做什么。

### 战利品

在每个 Game Mode 游戏中，打赢的战斗都会掉落一次战利品，放进队伍的背包。没有规则集时，战利品来自 Game Mode 自己的表（难度越高越多、越稀有）；未声明战利品表且保持 `native` 开启的规则集也是如此。声明了战利品表的规则集改为掉落自己的物品；设为 `native: false` 且没有表的规则集什么也不掉落。

`items` 块中的 `lootTables`(Capability API 1.63) 列出这些表，Gravewatch 的陪葬品就是这样写的：

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

- `rolls` 是该表抽取几次：0 到 20 的数字，或骰子（`"1d2"`），默认为 1。
- 每次抽取按 `weight`(默认为 1) 与其他项比较，抽中一行。一行可以用 `<catalog>/<entry>` 指定你的某件物品；可以是按 `rarity`、`category` 和 `tag` 筛选的 `filter`，在所有匹配的物品中均匀抽取；也可以是 `coins`(Capability API 1.64)，用 ID 指定你的某种钱币。`count` 是掉落数量，数字或骰子，默认为 1。被层移除的物品或钱币不会掉落，找不到任何物品的筛选也一样。
- 生物图鉴中的生物用 `loot` 指定它携带的表（见下文“生物”）。打赢的规则集战斗会为每个被击败的生物掷其表；Engine 不按你的规则结算的战斗没有图鉴生物，因此什么也不掉落。
- 掉落的东西像添加一样进入共享视图，先问玩家的背包，没有人能携带的东西会被留下。Game Master 收到的战后摘要会说明掉落了什么、已经放进背包，冒险日志也会记录，并显示通知。
- Game Master 在故事中用 `[loot: table="grave_goods" who="Ada"]` 掷表（`who=` 指定某个角色的背包），与其物品栏标签一样以回合开始时为基准，因此重新生成的回复只掉落一次。每件掉落的物品都按添加来答复，它的指令会列出你的表。

### 金钱

你的 `currencies` 就是队伍的钱。钱币是背包里的物品，每种钱币和其他物品一样成一叠，以其 `label` 命名：

- 其体系中每 `perWeight` 枚钱币重一个携带数值单位，计入持有者的负重，进入时按携带规则分配。没有 `perWeight` 的体系没有重量。
- 玩家从选择器的 **Coins**(钱币) 列表中添加钱币，并像任何一叠物品一样拆分、给予和合并它们。
- 物品栏各叠物品上方有一行，显示视图中每个体系的钱币，以及它们折合该体系最小钱币的价值。Game Master 会在每个背包旁看到这一价值（`Coin worth 432 pennies`，空钱袋则为 `no coins`，玩家的背包即使是空的也会列出），并在物品旁看到物品的 `cost`(`costs 3 shillings`)。
- Game Master 用 `[inventory: action="pay" amount="3 shillings" who="Ada"]` 付款，用 `action="earn"` 收款。`amount` 是数量加钱币，钱币用 ID 或标签指定，单数复数均可，不区分大小写（`1 penny`、`12 pennies`）。付款从一个背包中支出（不带 `who=` 时为玩家的背包），且只使用该钱币自己的体系：先付能用上的最大面额钱币，再把剩下的、足以付清余款的最小钱币破开，找零以该体系较小的钱币返还，面额从大到小。答复会说明付了什么、找回什么：从只有一枚 crown 的钱袋中付 3 shillings，就是 `Paid with crowns ×1; shillings ×2 back.`。背包付不起的付款会被拒绝（`cannot-afford`），你没有的钱币（`unknown-coin`），以及在没有货币的规则集中的任何付款（`no-currencies`）也一样。收款就是添加该钱币，放进 `who` 的背包，或按携带规则分配。冒险日志会列出支出和收入，并显示通知。它的指令会列出你的钱币，只有带钱币的规则集才有这些指令。
- 购买就是先付款再添加；在有市场的规则集中，则是 `buy`(见下文)。没有商店画面。

### 市场

`items` 块中的 `market`(Capability API 1.65) 说明一个地点卖什么、卖什么价，而不是每家店都按标价出售一切。下面是 Gravewatch 的市场，略有删减：

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

- `prices`：1 到 8 个档位，每个都是物品 `cost` 的倍率（`times`，大于 0），其中一个为 `default`。价格是以该体系最小钱币计的成本乘以档位倍率，在最小钱币上取整（有成本的东西绝不低于一枚钱币）；如果成本所用的钱币能正好付清，就以它表示，否则以层所保留的、能正好付清的最大钱币表示：在 `dear` 档位下，成本 8 shillings 的外套是 12 shillings，成本 50 gp 的 5e 药水是 50 gold pieces，绝不会是 5 platinum。讨价还价或卖家的心情改变的是档位，而不是数字。
- `places`：1 到 12 种地点规模，从小到大，用你自己的措辞。
- `sold`：规定筛选选中的物品从多小的地点起就有出售。筛选可以指定 `rarity`、`category` 和 `tag` 中的任意几项，匹配具有其所指定全部各项的物品；第一条匹配某物品的规则起决定作用。物品自身的 `sold` 优先于这些规则，没有任何规则指定的物品在任何地方都有出售。
- `sellers`：卖家种类，每种带其 `sells`(若干筛选，任意一个选中即可)、有这种卖家的最小 `place`(没有时为任何地方)，以及可选的 `only`，即只卖给谁：买家人物卡上的某个值（连同其实时状态和所携带的东西）至少为 `atLeast`，并以 `label` 告知 Game Master。没有 `sellers` 时，一个地点的任何人都出售该地点出售的任何东西。
- 没有 `cost` 的物品不出售，被层隐藏的物品，或其钱币被层移除的物品也不出售。

**队伍在哪里。** Game Master 用 `[place: name="Millbrook" size="market town"]` 说明场景所在的地点及其规模，规模用 ID 或标签指定；没有市场的地方（道路、荒野）则省略规模。Engine 就地答复每个标签，拒绝你没有的规模；从最近一次对话开始算起，它在玩家可见的自身回复中最后答复的地点，就是当前生效的地点，直到另有说明。玩家输入的地点标签从不计入。重新生成的回复只读取它所替换的那次叙述之前的内容。

**购买。** Game Master 用 `[inventory: action="buy" item="Hand axe" count="1" level="dear" seller="smith" who="Ada"]` 购买，`level`(没有时为默认档位) 和 `seller`(没有时为此地任何出售该物品的卖家) 可选。Engine 按回复结束时生效的地点定价，像付款一样从买家的钱袋（不带 `who=` 时为玩家的钱袋）中付款并找零，然后把物品放进其背包；如果是服务，则只付款。两者要么都发生，要么都不发生。答复会说明花了多少（`price="12 shillings"`）、付了什么、找回什么，并显示通知。以下情况购买会被拒绝，且不作任何更改：未说明地点（`no-place`）；你没有的档位、物品或卖家（`unknown-level`、`unknown-item`）；没有价格（`not-for-sale`）；地点太小，不卖该物品（`not-here`）；此地没有出售它的卖家（`no-seller`）；买家不满足卖家的 `only`(`not-to-you`)；钱袋付不起（`cannot-afford`）；或背包装不下。

**Game Master 看到什么。** 场景位于有市场的地点时，MARKET 块会写出地点及其规模、价格档位，以及此地的每个卖家，并按默认档位从便宜到贵列出各自出售的十来样东西；他们出售的任何东西都可以按名称购买。它的指令会教授 `[place:]` 和 `buy`，只有带市场的规则集才有这些指令。

#### 集市日

集市日（Capability API 1.69）让一个地点的货架会变化、会售罄，并允许队伍把东西卖回去。下面是 Gravewatch 的集市日，略有删减：

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

- **一天。** 在一个地点的一天，持续到 Game Master 点名另一个地点为止；在同一地点再说一次并带上 `day="new"`，就开始新的一天（`[place: name="Millbrook" size="market town" day="new"]`）。以下所有内容都根据玩家可见的 Game Master 回复按天计算，因此备选回复、重新生成的回复或分支都不需要保存任何东西，同一天掷出的货架总是相同。
- **`limited`**，写在 `sold` 规则或物品自身的 `sold` 上：一个地点一天备有多少件，可以是数字，或为当天掷出的骰子。规则可以只写限量而不写地点（任何地方都卖，每次几件）；物品的地点取自第一条指定地点的规则，限量取自第一条指定限量的规则。买完之后，购买会被拒绝（`sold-out`），直到新的一天。
- **`stock`**，写在卖家上：你的某个战利品表，每天掷一次，决定该卖家在常规货品之外还有什么存货。掷出的东西（只有物品，从不包括钱币）只由该卖家出售，数量就是掷出的数量，无论 `sells` 和 `sold` 怎么规定；卖家的 `only` 仍然适用。
- **`buyBack`**：卖家按物品默认价格的多少比例（大于 0，最多为 1）收购它。此地出售这类东西的卖家会收购它（没有卖家设置时任何人都会）。没有 `buyBack` 时，没有人回收任何东西。
- **`purse`**，写在地点规模上：此地所有卖家一天合计最多付出多少，以你的某种钱币计。以该钱币所在体系支付的出售都计入其中；没有钱袋时，收购没有上限。
- **出售。** Game Master 用 `[inventory: action="sell" item="Silver coffin nail" count="1" seller="smith" who="Ada"]` 出售。Engine 为它定价，从出售者的背包中取出（不带 `who=` 时为玩家的背包；绝不取已绑定的诅咒物品），并按面额从大到小付入钱币；否则拒绝且不作任何更改：未说明地点、此地无人收购（`not-bought`）、没有价格（`not-for-sale`）、今天的钱袋已花光（`purse-spent`）、没有持有，或背包装不下这些钱币。答复会说明卖了多少钱（`price="4 shillings"`），冒险日志将它列为已售出，并显示通知。
- MARKET 块还会写出每个卖家今天的存货、限量物品还剩多少，以及此地卖家的收购价和今天的钱袋还剩多少；它的指令会教授 `sell` 和 `day="new"`。

<a id="using-items-outside-a-fight"></a>

### 在战斗外使用物品

同一个 `use` 在战斗外也有效（`restore` 需要 Capability API 1.60；其余不需要新的声明）。对带 `use` 的物品，物品栏的 **Use** 按钮和 Game Master 的 `[inventory: action="use" item="Poultice" who="Juno"]` 都会用 Engine 的骰子，对携带者产生该物品的效果：

- 治疗恢复生命：在生命池上恢复其数量，在伤势轨道上清除一个标记，与战斗中的治疗相同。`temporary` 点数加到生命池上，`restore` 恢复其池，它 `applies` 的每个状态都会施加，直到被某种东西移除。
- 造成伤害或瞄准对方阵营的使用（`kind` 为 `attack` 或 `debuff`，或 `targets` 为 `enemy`）不会作用于任何人：战斗外棋盘上没有可以打的人。物品仍会被使用，Game Master 会被告知它的作用，以便叙述。
- 它与战斗中完全一样，从那一叠中减去一件或消耗其充能；已经用完的物品，或占用槽位、需要绑定却未穿戴的物品，会被拒绝且不作任何更改。人物卡和背包会一起写入。
- `gate` 像检定一样先掷：使用规则集自己的骰子和人物卡的数字，以及使用者的伤势惩罚、其状态和穿戴或携带物品对该检定的作用。失败时物品白白用掉，那一行会说明（“Ada uses Page of the vigil litany: Ward check 1 against 2, failed; it is used up for nothing. 1 left.”）。

发生的事用一行说明：“Juno uses Poultice: heals 4 (Grit 6/11). 1 left.” Use 按钮会在 “I use my Poultice.” 之后，用 `[item_used]` 块把它发送给 Game Master，聊天会将其显示为徽章；Game Master 自己的标签也以它答复，与其他标签一样以回合开始时为基准，因此重新生成的回复绝不会把一件物品用两次。没有 `use` 的物品，或在没有规则集的游戏中，仍然只是简单地说：“I use my rope.”

<a id="battles-lending-the-sheet-to-marinaras-combat"></a>

## 战斗：将人物卡借给 Marinara 的战斗系统

默认情况下，战斗对人物卡一无所知。它照常构建参战者，角色打完一场战斗后，人物卡上的生命值可能完全没变。

可选的 `battle` 块只从一个方向改变这一点：将人物卡数值借给战斗，并将战斗结果写回。**它不会让战斗遵循你的规则。** 骰子计算仍属于 Marinara，谁打中谁、造成多少伤害也一样。因此生命按最大值的比例传递，而非直接使用你的原始数值：人物卡上半血的角色，会以 Marinara 为其构建的生命条的一半开始战斗。你的 9 点生命池绝不会原样放进一次攻击就造成 12 点伤害的战斗。

```json
"battle": {
  "health": { "pool": "grit" },
  "energy": { "pool": "luck" },
  "skills": [{ "list": "knacks" }]
}
```

- `health`：必填。作为角色战斗生命值的实时池。必须来自 `sheet.live.pools`，不能是各行为池的列表。
- `energy`：可选。供战斗消耗的实时池，成为 MP 条。必须与 `health` 是不同的池，因为战斗不能将生命值当作燃料消耗。
- `slots`：可选。战斗每次消耗一个的实时池，各自具有 1 到 9 的 `level`：`[{ "pool": "slots_1", "level": 1 }]`。每个等级和池只能使用一次。
- `skills`：可选，最多八项。其行会成为角色战斗技能的人物卡列表。只计算来自你所提供目录的行，而且该行背后的条目必须具有 `mechanics` 块：手动填写的行没有数值意义。`onlyWhen` 指定行中必须设为真的布尔列，例如已准备的法术。`alwaysWhen` 指定一个列和值，允许行例外通过，例如无需准备即可施放的法术。它是 `onlyWhen` 的例外，所以未同时指定后者时会被拒绝。

### 带入与带出的内容

**带入：** 对游戏中已有人物卡的每位队友，生命池占最大值的比例决定其在 Marinara 自身生命条上的起点；能量池成为 MP；每个法术位池成为对应等级的法术位；带标记的行成为技能。最大生命、攻击、防御、速度和等级仍采用 Marinara 自身数值。生命池为零的角色以倒下状态开始战斗，因为人物卡就是这样记录的；生命大于零的角色则至少以 1 点生命开始，因此小比例不会因舍入而让人倒下。

**带出：** 战斗结束后，将参战者最终生命条的比例读回生命池自身的刻度，再把与开战时的差额作为伤害或治疗应用。能量和法术位是计数而非比例，所以原样写回。所有更改都经过人物卡自身按钮所遵循的规则，人物卡拒绝的更改会跳过并报告，而不会强行应用。如果战斗没有改变某位参战者的生命值，就完全不写生命变化，因此两次转换本身绝不会改变人物卡。

**均不涉及：** 攻击检定、豁免、专注，以及更高费用会增加什么效果。这些内容保存在 `mechanics` 块中，供真正的战斗系统日后读取；这项桥接功能不会应用它们，规则集也不应声称会应用。

放弃的战斗不会写回任何内容。如果你删除了战斗开始所在的消息，或战斗始终没有结束，人物卡就与原先完全一致：视同战斗没有发生。

### 条目如何成为技能

目录条目的 `mechanics` 块按以下方式读取：

- `kind` 成为技能类型。`utility` 条目及所有标为 `reaction` 的条目都会排除，因为 Marinara 的战斗系统没有对应的处理位置。
- `amount` 决定命中威力，但使用的是相对参战者自身攻击力的倍率，而非伤害数值。更大的骰子不会打得更轻，倍率也保持在生成技能原本使用的范围内。
- `range` 和 `area.size` 除以目录的 `units.distance.perCell` 后得到网格格数，绝不会向下取整成零。爆发取其半径，锥形取其一半，直线取一格。凡有范围的效果都以覆盖的所有敌人为目标，并遵守 `friendlyFire`。
- `damageType` 成为技能元素。`targets` 不会带入：Marinara 的战斗系统根据技能类型，决定治疗、增益或攻击可以指向谁。
- 能量池上的 `cost` 成为 MP 费用，多项能量费用会相加。恰好一个法术位的 `cost` 会消耗对应等级的一个法术位。Marinara 的战斗系统只收取一种能量数值或一个法术位，从不两者兼收。因此，消耗两个法术位、两个不同等级的法术位、或法术位加能量的条目，会从战斗中排除。任何其他池的费用也一样，例如生命或职业资源，否则 Engine 就会免费提供它。
- `buff` 或 `debuff` 成为 Marinara 自身的增益或减益。条目文本承诺的其他效果，例如清除人物卡上的状态，不会在战斗中应用。如果条目的效果只有在战斗外才有意义，就不要为它添加 `mechanics`。

`coverage.combat` 独立设置，含义仍和以前一样：只有战斗真正遵循你的系统规则时才设置它。

<a id="combat-a-fight-your-own-rules-resolve"></a>

## 战斗：由你自己的规则结算

上面的 `battle` 块将人物卡数值借给战斗，计算方式仍属于 Marinara。可选的 `combat` 块则是另一回事：它说明如何用你的规则结算战斗。它为 Engine 所拥有的一种战斗类型提供参数，正如 `resolution` 为检定类型提供参数一样，其中每个名称都由你决定。共有两种类型：`attack-vs-defense` 把骰子相加后与一个数字比较；`dice-pool` 则让战斗掷出 `dice-pool` 规则集自己的骰池并计算成功数（见下文“以骰池进行的战斗”）。

**规则集声明了 `combat` 的游戏，会按你的块进行战斗。** 队伍数值取自各自的人物卡；敌人来自你的生物图鉴或威胁尺度；每个回合都按你的骰子结算；角色消耗或损失的任何内容都立即写回人物卡，因此中途关闭标签页也不会丢失。战斗画面使用你的措辞：菜单中是你的攻击和能力，使用你的行动额度和状态，日志展示实际算式。尚未实现的内容列在“尚未支持”中。

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

这就是 Ember Roads 块的全部内容，Ember Roads 游戏按它进行战斗。5e 草案将相同的键用于 d20 系统：

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

### 各个键

- `kind`：`"attack-vs-defense"`(一方投骰对抗另一方的防御，命中则造成伤害) 或 `"dice-pool"`(见“以骰池进行的战斗”)。下面每个键在两种类型下含义相同，只是按该类型读取数字的方式来读。
- `health`：必填。战斗扣除的对象。可以是 `{ "pool": "grit" }`，表示向下扣减的实时池；若有临时缓冲，伤害先扣缓冲。也可以是 `{ "track": "harm" }`，表示需要标记的伤势轨道。轨道必须同时指定 `damageKinds`，且不提供临时点数。
- `defense`：必填的值引用。可以是玩家输入的字段，或由你计算的派生值。
- `initiative`：必填。开始时投一次的骰子，以及可选的调整值引用。同值时调整值较高者优先，再按战斗建立时的顺序排列。`"each": "round"` 在每个新轮次开始时让所有人重新投先攻，使用当时的调整值（因此读取伤势轨道的调整值在受伤后会更慢），该轮从新顺序中的第一位开始。Capability API 1.47。`dice-pool` 战斗可以改为以骰池投先攻：`pool`(值引用) 是骰子数量，其成功数加上 `plus` 就是先攻数字，此时旁边不能有 `dice` 或 `modifier`。`resource` 需要 `pool`，它会保留这个数字并允许攻击改变它（见下文“会被攻击改变的先攻”）。Capability API 1.48。
- `attackRoll`：在 `attack-vs-defense` 下必填，在 `dice-pool` 下会被拒绝，因为后者掷的是结算规则自己的骰池。指定骰子、是否投两次取其一(`advantage`)、单骰最大与最小点数的效果(`naturals.max`：`critical`、`hit` 或 `none`；`naturals.min`：`miss` 或 `none`)，以及重击对伤害的影响(`critical`：`double-dice` 再投一次伤害骰，`max-dice` 加一次最高骰面，`none` 则是普通命中)。特殊骰面和检定一样，需要单颗骰子。战斗内的豁免也投相同的骰子。
- `economy`：必填。`budgets` 是一个回合可持有的行动额度，包含 ID、标签、`per`(`turn` 在持有者自己的回合开始时补充，`round` 在新轮开始时补充)和 `count`。你首先声明的额度是主额度，标准行动消耗它。`movement` 是可选的值引用，表示按你自己的距离单位，一回合可以走多远。它由棋盘上的战斗读取(见“位置”)。
- `attacks`：可选。其行为武器的人物卡列表。`name` 是给行命名的文本列，`damage.dice` 是骰子列；`toHit.ability`、`toHit.proficiency`、`toHit.bonus`、`damage.ability`、`damage.bonus` 和 `damage.type` 各自指定同一列表中的列。`ability` 列是保存某个自定义属性 ID 的 `enum`，不属于属性 ID 的值不作任何加成。`proficiency` 列是 `boolean`，设为真时加入熟练加值。没有可读骰子的行不是攻击，所以同一列表中的绳索仍只是绳索。`toHit.skill` 指定保存某个技能 ID 的 `enum` 列：该行加上这项技能检定所加的值；如果行指定了自己的 `ability`，就用它代替技能的属性，与检定的 `with=` 替换方式完全相同（Capability API 1.47）。`strikes` 是可选的值引用，说明消耗一次该列表额度能获得几次打击。手中没有剩余打击时选取一行，会消耗额度并将其余打击交到手中；只要手中还有，所有声明了 `strikes` 的行都不再消耗额度，因此换武器、换目标、在两次攻击间行走，都自然地由菜单支持。`strikesCappedBy` 指定一个布尔列，无论列表一次给出几次打击，该列都将自己所在的行限制为一次。这用于无论使用者攻击次数多少，每回合都只能发射一次的武器；它正是为了表达 SRD 5.1 的 Loading 属性而设。若列表本来就一次消耗只给一次打击，该设置毫无意义，会被拒绝。手中的打击属于参战者，不属于某个列表：如果角色的两个武器列表都声明 `strikes`，不论挥动哪一行，都消耗同一份剩余打击。它们在获得它们的回合结束时清除。未声明此项的列表一次消耗获得一次打击，与该功能出现前的所有战斗相同。

  ```json
  {
    "list": "attacks",
    "budget": "action",
    "name": "name",
    "strikes": { "field": "attacks_per_action" },
    "damage": { "dice": { "column": "damage" } }
  }
  ```

- `abilities`：可选。其带目录标记的行为能力的人物卡列表，使用 `onlyWhen` 和 `alwaysWhen`，完全按 `battle.skills` 的方式筛选。每个能力的效果取自条目自己的 `mechanics`；该块指定默认消耗的 `budget`、需要攻击检定的条目所加的 `toHit`，以及条目豁免所对抗的 `saveDifficulty`。如果条目要求豁免，无论是自己的豁免，还是结束其施加状态的豁免，而落入的列表没有 `saveDifficulty`，就会被拒绝：没有难度的豁免总会成功。
- `standard`：可选，取自封闭列表 `dash`、`disengage`、`dodge`、`help`、`hide`、`ready`。`dodge`(攻击闪避者时投两次取较差结果)和 `help`(受助盟友下次攻击投两次取较好结果)始终会结算。`dash`(再获得相同移动额度)和 `disengage`(本回合走开时无人能借机攻击你)在棋盘上结算，无棋盘时仅记录。`hide` 和 `ready` 会接受，但目前没有效果。
- `standardEffects`：可选，表达标准行动标记本身无法涵盖的部分。目前仅 `dodge` 有此项：`{ "dodge": { "saves": ["dex_save"] } }` 指定闪避期间哪些自定义豁免投两次取较好结果。只能指定人物卡已声明的豁免，并且 `standard` 列表必须含有 `dodge`。省略后闪避与原来完全相同：更难被命中，仅此而已。
- `conditions`：可选。将你自己的状态 ID 映射到效果，使人物卡状态和战斗状态成为同一份记录，中毒角色战后仍然中毒。效果取自封闭列表：`own-attacks-advantage`、`own-attacks-disadvantage`、`attacks-against-advantage`、`attacks-against-disadvantage`、`attacks-against-adjacent-advantage`、`attacks-against-far-disadvantage`、`attacks-from-adjacent-critical`、`cannot-act`、`cannot-react`、`speed-zero`、`half-move-to-stand`、`ends-on-damage`、`own-saves-advantage`、`own-saves-disadvantage`、`own-checks-advantage`、`own-checks-disadvantage`、`resist-all`、`cannot-target-source` 和 `cannot-approach-source`。`failsSaves` 指定在该状态下无须投骰便失败的豁免。需要距离或移动的六项(`attacks-against-adjacent-advantage`、`attacks-against-far-disadvantage`、`attacks-from-adjacent-critical`、`speed-zero`、`half-move-to-stand`、`cannot-approach-source`)由有棋盘的战斗读取，没有棋盘时不产生效果(见“位置”)。`cannot-react` 会将持有者排除在走动打开的响应窗口之外，因此从不会征询其回应。效果旁还有四个键：
  - `modifiers`：状态持续期间它改变的数字。见下文“状态改变的数字”。
  - `saves`：两种豁免效果，以及任何针对豁免的调整值，涉及哪些自定义豁免。省略时表示全部；两者都没有却指定它，会被拒绝。
  - `skills`：两种检定效果，以及任何未自行指定技能的检定调整值，涉及哪些自定义技能。省略时表示全部；两者都没有却指定它，会被拒绝。战斗中的对抗掷的是战斗自己的检定，而不是技能，因此收窄到技能的内容永远不会作用于对抗；战斗外针对这些技能的检定会读取它（见下文“战斗外的检定”）。Capability API 1.53。
  - `whileSourceInSight`：只有施加者在持有者视线内时才生效的部分。`true` 限制整个状态；写该状态自身效果的列表，则只限制所列效果，其余继续有效。这适合无论是否看得到来源，都阻止你走近的恐惧。指定状态并不具备的效果会被拒绝。没有棋盘就没有可打断的视线，因此两种写法都会让所有效果生效。
  - `endsWhenSourceDown`：施加者倒下的瞬间移除状态。

  `own-saves-advantage` 及其相反效果与攻击一样，将豁免投两次取其一，彼此抵消。`own-checks-advantage` 及其相反效果对持有者在对抗中的一方做同样的事（战斗在对抗中进行属性检定），也作用于其在战斗外的检定。`resist-all` 在目标自身抗性的基础上将所有种类的伤害减半，并按相同方式与易伤抵消。`cannot-target-source` 阻止持有者将任何东西指向施加该状态的人；`cannot-approach-source` 阻止持有者走到比当前格更靠近施加者的位置，连路径也算：绕到同样远的一格仍会提供，但先靠近对方再穿到另一侧的路不会提供。

  ```json
  { "condition": "restrained", "effects": ["own-saves-disadvantage"], "saves": ["dex_save"] }
  ```

  **状态改变的数字。** `modifiers` 中的每一项用 `to` 指定改变什么，并说明改变多少：
  - `defense`：针对持有者的攻击必须达到的数字。只能是固定数字。
  - `attacks`、`saves`、`checks`：持有者自己的攻击检定、豁免和对抗检定。可以是固定数字，或每次掷骰时都掷的 `dice`(`"1d4"`)；`minus: true` 表示减去这些骰子而不是加上。
  - `speed`：持有者能走多远，用你自己的距离单位（`flat`），或用 `times` 0.5 或 2 表示减半或加倍，在所有固定改动之后应用。和 `speed-zero` 一样，只在棋盘上读取。

  针对 `checks` 的调整值可以指定自己的 `skills`，针对 `saves` 的可以指定自己的 `saves`，以此代替状态的范围进行收窄。两者都可以用 `mode: "advantage"` 或 `"disadvantage"` 掷两次，与同名效果一起计算并以相同方式抵消，旁边可以有数字，也可以没有；因此一个状态可以同时让 Sneak 更难、让 Climb 更容易。这三个键需要 Capability API 1.53。

  ```json
  { "condition": "muddy", "modifiers": [{ "to": "checks", "skills": ["sneak"], "mode": "disadvantage" }] }
  ```

  固定数字自带正负号，0 会被拒绝。每个调整值都会显示在日志中它所改变的掷骰旁，并附上状态名称（“12 + 5 + 3 (Blessed) = 20”），菜单的命中几率和获胜几率也会计入它。状态之间可以叠加；同一个状态出现两次仍是一个状态。

  ```json
  "conditions": [
    { "condition": "blessed", "modifiers": [{ "to": "attacks", "dice": "1d4" }, { "to": "saves", "dice": "1d4" }] },
    { "condition": "shielded", "modifiers": [{ "to": "defense", "flat": 5 }] },
    { "condition": "slowed", "modifiers": [{ "to": "speed", "times": 0.5 }, { "to": "defense", "flat": -2 }] }
  ]
  ```

- `levels`：可选。实时轨道的等级，轨道足够高时视为状态；力竭这类逐级恶化的状态就是这样表达的。每一项指定一条普通 `track`(不是伤势轨道)、开始生效的等级 `at`，以及它的作用，使用与状态相同的 `effects`、`modifiers`、`failsSaves`、`saves` 和 `skills`。轨道已达到的每个等级都会计入，因此随着轨道上升而累加。等级没有来源，只有在它读取的值下降时才结束，因此 `half-move-to-stand`、`ends-on-damage`、`cannot-target-source` 和 `cannot-approach-source` 写在等级上会被拒绝。只有带人物卡的参战者才有轨道。日志以轨道和数字命名等级（“- 1 (Heat 3)”）。

  等级也可以不读 `track`，而读取一个 `derived` 值（Capability API 1.54），该值根据角色的实时状态和物品计算，因此不需要任何人去勾选：Ember Roads 会让携带 10 点或更多 bulk 的人变慢。

  ```json
  { "derived": "bulk_carried", "at": 10, "modifiers": [{ "to": "speed", "flat": -2 }, { "to": "checks", "skills": ["sneak"], "flat": -1 }] }
  ```

  因角色而异的阈值（“超过 Strength 的五倍”）用一个减去它的派生值表达：用 `scale` 乘以 -1 得到要减去的部分，两者的 `sum` 就是等级读取的值。

  ```json
  "levels": [
    { "track": "exhaustion", "at": 1, "effects": ["own-checks-disadvantage"] },
    { "track": "exhaustion", "at": 2, "modifiers": [{ "to": "speed", "times": 0.5 }] },
    { "track": "exhaustion", "at": 3, "effects": ["own-attacks-disadvantage", "own-saves-disadvantage"] }
  ]
  ```

  调整值、等级和检定效果需要 Capability API 1.45。

- `checks` 和 `contests`：可选。对抗读取什么，以及战斗中任何人都可以发起的对抗：擒抱、推撞、挣脱。见下文“对抗”。Capability API 1.43。
- `concentration`：可选。指定记录维持内容的实时 `text` 字段、伤害强制要求的 `save`、难度下限 `floor`，以及所受伤害的比例 `fromDamage`，在后者更高时用它确定难度。开始第二个需要专注的能力会结束第一个；豁免失败也会结束专注，并一并移除它维持的状态。
- `dying`：可选，`kind: "saves"`。指定记录投骰次数的两条轨道(所需次数取各轨道自身最大值)、`dice`、`succeedAt`、最大与最小骰面的效果(`naturals.max`：`revive-1` 或 `success`；`naturals.min`：`one-failure` 或 `two-failures`)、倒下时受伤的代价(`damageWhileDown`、`criticalWhileDown`)，以及倒下角色所处的 `condition`。没有此块时，角色降到零就只是倒下，治疗即可恢复。
- `damageTypes`：可选。你的系统拥有的伤害类型，不区分大小写匹配。
- `damageKinds`：`health` 指定伤势轨道时必填，指定池时则拒绝，因为只有标记能携带种类，点数池没有位置保存种类(见前文“伤势轨道”)。它说明一次打击标为轨道的哪个 `kinds`，以及勾几格。`default` 用于一切未映射的情况，包括完全不带类型的打击；`byType` 将你的 `damageTypes` 映射到标记种类，键和类型本身一样不区分大小写，所以 `"Fire"` 和 `"fire"` 是同一个键，同时指定会被拒绝。`marks` 没有默认值，因为两种选择含义相反：`"per-point"` 表示伤害骰计算生命等级，3 点打击勾三格，减轻 1 点也有价值；`"per-blow"` 表示打击要么命中要么未中，无论打得多重都只勾一格。带多个伤害子项的打击也只标一格，采用实际生效的最严重种类。请明确你的系统采用哪一种。`{ "default": "bashing", "byType": { "fire": "aggravated" }, "marks": "per-point" }`。
- `pool`：在 `dice-pool` 下必填，在 `attack-vs-defense` 下会被拒绝。见“以骰池进行的战斗”。
- `spendLimits`：可选。在战斗中，一个参战者每 `turn` 或每 `round` 最多能从某个实时池消耗多少，无论哪种池都适用：`[{ "pool": "blood", "max": { "derived": "blood_per_turn" }, "per": "turn" }]`。`max` 在战斗开始时读取一次。超出剩余限额的费用视为付不起，因此不会出现在菜单上，无论池中还有多少。每回合的限额在持有者自己的回合开始时重新计算，每轮的限额在新一轮开始时重新计算。以直接数值编写的对手不从人物卡支付任何东西，因此限额从不约束它；以人物卡编写的对手从自己的池中支付，与其他人一样受限额约束。Capability API 1.47。
- `threat`：可选，生物图鉴需要它。`tiers` 是选取敌人的尺度，包含 ID、标签、`health` 区间、`defense`、`toHit`、`damagePerRound` 区间和 `saveDifficulty`。你提供的每个生物都指定其中一档；没人预先编写的敌人则会拉回 Game Master 要求的档位，因此不会超出你的尺度。`damagePerRound` 区间按生物一轮内对一个目标造成的伤害读取，包含完整动作序列。

### 以骰池进行的战斗：`dice-pool`

`dice-pool` 规则集计算成功数而不是把骰子相加，它的战斗也可以如此。`"kind": "dice-pool"` 要求 `resolution.kind` 为 `"dice-pool"`，并以骰池检定已有的方式读取人物卡：**掷骰所加的每个数字都是骰子数量，所要达到的每个数字都是成功次数。** 因此不需要改任何名称：

- 命中数字（攻击行的、能力列表 `toHit` 的、生物行动的）就是攻击所掷的骰池。骰子、单骰须达到的目标，以及哪些点数计两次、爆骰、抵消和大失败，都取自你的 `resolution`。`resolution.penaltyFrom` 指定的伤势轨道会从骰池中扣除其惩罚，与检定相同。
- `defense` 是攻击所需的成功次数，绝不少于一次。状态对 `defense` 的调整值和 `cover.bonus` 都会加到它上面。
- 超过所需的每次成功增加一颗伤害骰。大失败一律未命中，无论计了多少成功。没有重击：强力命中的价值就在于额外的骰子。
- 豁免的数字就是其骰池，难度就是所需的成功次数。对抗把双方的检定都作为骰池掷出，成功更多的一方获胜。
- 状态对攻击、豁免或检定的调整值是增减的骰子，因此是 `flat` 数字：掷骰得出的调整值（`dice`）会被拒绝。
- 伤害就是你的骰子：数量的 `dice`(生物的、条目的) 说明掷几颗，且必须是你的骰子；其 `flat` 部分是自动成功，从不掷出。攻击行的伤害骰列只读取其骰子数量，`damage.ability` 把该属性的评级作为骰子加入，`damage.bonus` 增加自动成功。`perCostStep` 和 `scales` 一如既往地增加骰子。治疗和临时点数是数量，照旧相加。
- 先攻是总和（`initiative.dice` 加 `modifier`），或在你要求时为骰池（`initiative.pool`，见下文），濒死规则保留自己的骰子。

除此之外，骰池战斗还会掷的内容由其 `pool` 块决定：

```json
"pool": {
  "advantage": true,
  "damageTarget": 6,
  "soak": { "roll": true, "byKind": { "knock": { "abilityMod": "sinew" } } }
}
```

- `advantage`：偏向一方的掷骰是否掷两次并保留成功更多的那次（大失败视为少于任何未大失败的掷骰）。没有它时，无论状态怎么说，战斗都只掷一次。
- `damageTarget`：伤害骰和吸收骰所对抗的单骰目标。省略时取结算规则的默认目标。这些骰子只把达到或超过它的点数各计一次，别无其他：它们不会计两次、爆骰、抵消或大失败。
- `soak`：目标按伤害种类从一次命中造成的伤害中减去多少。`all` 是适用于任何种类的值引用，`byKind` 则为生命轨道的每个种类各给一个，对其种类优先于 `all`；两者都未指定的种类不吸收。`"roll": true` 时，目标对抗伤害目标掷相应数量的骰子，每次成功减去一点。`"roll": false` 时，这个数字在掷伤害骰之前从伤害骰中扣除，因此从不触及自动成功；而掷骰吸收会像减去其他成功一样减去自动成功。任何值都不会低于零，没有计到任何成功的打击也不会掷骰吸收。生物用 `soak: { "all": 1, "byKind": { "knock": 3 } }` 给出自己的数字，这需要该块的 `soak` 说明如何吸收。抗性、易伤、免疫和 `resist-all` 在吸收之后作用于剩下的部分。
- `hardness`：在先攻是会被攻击改变的数字时，表示战斗者硬度的值引用。见[战斗中的护甲与穿戴效果](#armor-and-worn-effects-in-a-fight)。

每个目标的伤害都为该目标单独掷出，因为命中超出所需成功数所赢得的部分，以及目标吸收的部分，都只属于该目标。打击的第二个子项和附加效果各自是一个独立的、有自己种类的伤害骰池，不会从命中获得额外骰子。菜单会精确预测骰池：达到所需成功数的几率，以及第一个目标吸收之后伤害骰的价值。

Gravewatch 就是这样写的：它的守望者掷一个评级加一个专业技能，用 Sinew 吸收冲击伤，完全不吸收撕裂伤，每回合可以消耗一点 Resolve，并且每轮重新投先攻。

**逆序声明没有实现。** 有些系统要求所有人在任何人行动之前先声明行动，最慢的人先声明。在每个参战者轮到时才选一个行动的战斗中，先声明不会改变任何规则读取的内容，因此没有需要用键表达的东西。

### 会被攻击改变的先攻

有些骰池系统在整场战斗中把先攻作为一个数字保留，并允许攻击改变它：一种攻击方式从目标那里夺取先攻，另一种把攻击者自己的先攻作为伤害消耗掉，跌到某条线的人就崩溃了。`dice-pool` 战斗可以表达这一点（Capability API 1.48）：

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

- **开场。** 战斗开始时，所有人掷 `pool` 颗骰子，成功数加上 `plus` 就是各自的数字，因此 `resource` 需要 `pool`：相加的骰子只能排出顺序，不是骰子数量。生物的 `initiativeModifier` 就是它的骰池，因为骰池战斗所加的每个数字都是骰子。这个数字会一直保留：从不重新投掷（`resource` 旁边不能有 `each`），每轮开始时按当时的数字排序。
- **方式。** 每种攻击（攻击行、作命中检定并造成伤害的能力或目录条目、造成伤害的生物行动，以及由其他行动组成的行动）都按每种方式各提供一次，玩家先选方式再选目标。最多四种，每种要么夺取、要么消耗，且至少有一种是夺取。未指定方式的选择按第一种进行；指定了攻击未提供的方式会被拒绝。攻击在结束之前一直保持它发起时的方式，即使有回应在它落下之前改变了攻击者的数字。
- **夺取型方式。** 命中时照常掷伤害（命中带来的额外骰子、武器的骰子、按种类吸收、自动成功），计得的数值从目标的数字中扣除，而不是从生命中扣除。攻击者获得全部数值，再加上 `gain`。
- **消耗型方式。** 只在攻击者的数字高于崩溃线时提供（未给出 `crash` 时为 0），且从不用于由其他行动组成的行动，因为一个数字只能花在一次打击上。命中时，伤害就是攻击者发起攻击时的数字，以骰子对抗伤害目标，此外别无其他：没有武器骰、没有命中带来的额外骰子、没有自动成功、没有吸收。它按攻击自身的伤害种类标记生命。攻击结束后，在任何目标上命中过的数字回到 `base`；一处也没命中的，则按发起时的数字失去 `onMiss` 所规定的量（阶梯表：`[at least, lose]` 对，升序排列；低于第一档时不失去任何东西）。
- **崩溃。** 跌到 `crash.at` 或以下的数字就崩溃了：对其施加 `condition`(你自己的状态之一，可选)，把他们打到这里的人获得 `bonus`。让攻击者自己崩溃的未命中不给任何人奖励。崩溃的参战者不能使用消耗型方式。以下情况会解除崩溃，连同状态一起：其数字再次升到线以上时；经过其自己的 `recoverAfter` 个回合后（该回合开始时其数字回到 `base`）；以及战斗结束时，无论以何种方式结束。开场投掷就处于线上或线以下的人，以崩溃状态开始战斗。`base` 必须高于 `crash.at`，否则回到基准值又会让他们崩溃。

菜单显示每种方式各自的预测：命中几率，以及夺取型方式会夺取多少，或消耗型方式会造成多少伤害。状态面板显示每个人的数字，日志说明每次变化及原因：“Ada gains 5 initiative for crashing Rats, and is on 14.” Engine 的选择器会向前看一个回合来评估夺取型打击：比较先夺取再消耗，与现在消耗、之后从基准值再消耗。

<a id="what-a-fight-reads-from-mechanics"></a>

### 战斗会从 `mechanics` 读取什么

`kind` 决定 `amount` 是伤害还是治疗；所有标为 `reaction` 的条目都不会出现在菜单中，`utility` 条目也一样，除非它改变回合自身可持有的内容(见下文)。`attackRoll` 使用列表的 `toHit` 对目标防御投骰；`autoHit` 完全跳过这一步。`save` 使用目标自己的豁免对抗列表的 `saveDifficulty`，`onSuccess` 决定成功时承受一半还是零。`targetCount` 表示能指向几个目标。没有攻击检定的能力(人人豁免的范围、直接命中的效果)只为所有人投一次伤害骰；对各目标分别作攻击检定的能力，则每次命中都重新投伤害骰。`applies` 为受影响对象施加状态，每项具有 `duration`：`instant`(没有自己的计时，直到被移除才结束)、`until-save`(须同时有 `saveEnds`)或 `{ "rounds": n }`；还可用 `saveEnds` 指定豁免及其在 `turn-end` 或 `turn-start` 重复。轮数在持有者自己的每个回合结束时倒数；`{ "rounds": 1, "at": "turn-start" }` 则在每个回合开始时倒数，“直到你下个回合开始”就这样表达。`endsAfter` 让状态在第一次 `own-attack`(持有者的下一次攻击检定)、`attacked`(针对持有者的下一次攻击检定) 或 `own-save`(持有者的下一次豁免) 之后移除，无论其计时如何，因此 `{ "duration": { "rounds": 1 }, "endsAfter": "own-attack" }` 表示“在其下个回合结束前的下一次攻击时”。持续一次攻击的状态会覆盖整次攻击，包括其伤害，而同一次打击新施加的状态会保留。两者都需要 Capability API 1.45。`temporary` 为生命池给予临时点数，永不叠加，只保留较大的缓冲。`scales` 按读取值所对应的表项增加额外骰子。`cost` 通过人物卡自身的 `use` 命令支付，`budget` 则覆盖其消耗的行动额度部分。

`plus` 在同一次打击的 `amount` 之外，增加最多三项数值，各自投骰、各自指定类型("and 2d6 fire")。子项形如 `{ "dice": "2d6", "flat": 1, "type": "fire" }`，还可以具有自己的 `save`：`{ "save": "con_save", "difficulty": 13, "onSuccess": "none" | "half" }`。无论行动已经要求目标做过什么，目标都要为此再作豁免：成功时 `none` 使该子项归零，`half` 留下一半，两者都不改变打击的其他部分。未写 `difficulty` 时，先回退到行动自身豁免使用的数值，再回退到列表的 `saveDifficulty`。重击按首项数值相同的规则翻倍各子项骰子；未写 `type` 的子项沿用打击自己的伤害类型。整个打击依然只按合计伤害做一次专注检定，以及一次倒下检查。子项必须依附 `amount`，`heal` 不携带子项。

```json
{
  "kind": "attack",
  "attackRoll": true,
  "amount": { "dice": "1d8" },
  "damageType": "piercing",
  "plus": [{ "dice": "2d6", "type": "fire" }]
}
```

三个键说明条目对回合自身的行动经济有何影响。声明其中任意一个的 `utility` 条目会被提供，不会被丢弃：

- `free`：完全不消耗行动额度。仍支付所声明的 `cost`，且不能同时指定 `budget`。
- `gives`：`[{ "budget": "action", "count": 1 }]`，最多四项。使用瞬间向对应额度增加数量，增加后的上限为本回合原本可持有的量加赠送量，因此无法存到后续回合。
- `standard`：`{ "actions": ["dash", "disengage", "hide"], "budget": "bonus" }`。持有者可以用该额度执行这些标准行动。它们以 `standard:<id>@<budget>` 的形式和普通行动并列提供；如果条目只有这种许可，条目自身就不会出现在菜单中，因为许可并不是人可以执行的动作。

新的 `kind: "rider"` 条目是被动效果：无人执行，永不上菜单，并会自动为每个周期内第一次符合条件的命中增加一个伤害子项。它携带 `rider`，不携带其他可执行内容：

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

`sources` 指定适用的攻击列表，`requires` 指定这些行中一个求值为真的列。因此，只在特定武器上触发的附加效果可以声明哪些武器适用，而不要求 Engine 理解什么是武器。两者都不写，就表示持有者造成的任何命中。从背包中拿在手上的武器不是行，因此 `weapons`(Capability API 1.70) 还会指定它在哪些物品标签上触发：`"weapons": { "tags": ["finesse",
"ranged"] }` 表示用带有其中任一标签的手持武器命中时触发，包括其副手打击。只写 `weapons` 时不在任何行上触发，只写行时不在任何手持武器上触发。`when` 满足任意一项即可：`advantage` 表示攻击检定最终的优势倾向；`ally-adjacent` 表示攻击者一名站立且能行动的盟友，在棋盘上须位于目标一格内，没有棋盘时则可在任何位置。`oncePer` 是 `turn`(每个人的每次回合开始时都会刷新，因此在别人行动时打出的攻击也能附加)或 `round`。`amount` 随条目自己的 `scales` 增长，`type` 是伤害类型，默认沿用打击自身的类型。

<a id="creatures-a-bestiary-a-fight-reads"></a>

### 生物：供战斗读取的生物图鉴

声明 `"holds": "creatures"` 的目录携带敌人，而非人物卡行。它不向任何列表供给内容，人物卡编辑器的选择器也从不提供它；其中每个数值都用 `combat` 块已经声明的键书写。它需要 `combat` 块和 `threat` 尺度，因为生物要归入你自己的某个档位。

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

下面是直接用数值编写生物的方式。如果改用规则集自己的定义写成 `sheet`，就从该人物卡获取 `health`、`defense`、`initiativeModifier`、`speed`、`abilities`、`saves`、`checks`、`soak` 和 `hardness`(见下文“用规则集自身定义编写生物”)。

- `health`：数值，或在创建战斗时投一次的 `{ "dice": "3d6", "flat": 2 }`。预测读取平均值，因此菜单不会承诺尚未投出的骰子结果。
- `defense`、`initiativeModifier`、`speed`：分别为攻击检定对抗的数值、先攻所加的值（或在以骰池投先攻时，所掷的骰子数），以及按你自己的距离单位一回合能走多远。
- `abilities` 和 `saves`：以人物卡声明的属性 ID 和豁免 ID 为键。未指定的豁免读作零。
- `checks`：它在对抗中所加的值，以 `combat.checks` 的 ID 为键。未指定的读作零。Capability API 1.43。
- `soak`：仅用于 `dice-pool` 战斗，表示它吸收什么：`all` 表示任何伤害，`byKind` 表示生命轨道的某个种类。Capability API 1.47。
- `hardness`：在先攻是会被攻击改变的数字时，骰子数低于它的消耗型打击没有任何效果。Capability API 1.56。
- `resist`、`vulnerable`、`immune`：伤害类型，不区分大小写匹配；如果声明了 `combat.damageTypes`，还会据此验证。`resist` 或 `immune` 项还可以说明什么能穿透它，`{ "type": "tearing", "except": ["silver"] }`：来自带有其中某个物品标签的武器物品的打击按原样承受（Capability API 1.55，见[战斗中的武器](#weapons-in-a-fight)）。`conditionImmunities` 指定你自己的状态。
- `tier`：属于 `combat.threat` 的哪一档。
- `loot`(Capability API 1.63)：打赢战斗时为它掷的战利品表 ID(见上文“战利品”)。
- `traits`：展示给 Game Master 的简短名称与文本对。它们从不自动结算，因此凡是涉及数值的内容都应写入行动。
- `signaturePoints`：在自己的回合开始时补回的点数，用于支付 `signature` 行动。
- `riders`：最多四项，与目录条目的 `rider` 相同，只是写在块上。每项形如 `{ "id": "pack", "name": "Pack", "on": "hit", "oncePer": "turn" | "round", "amount": { "dice": "1d6" } }`，可选 `type`，还可用 `actions` 指定它在本块哪些行动上触发。块的附加效果不读取人物卡列表，所以没有 `sources` 和 `requires` 这两个键。写成人物卡的生物则和角色一样，获得其列表携带的附加效果。
- `actions`：最多十二项，每项有自己的 `id`。行动包含手写数值块所包含的内容(`toHit`、`autoHit`、`damage`、`save`、`applies`、`targetCount`、`reach`、`range`、`area`)，以及四项生物独有内容。`reach` 是打击触及距离，`range` 是投掷或射击距离，`area` 是落点形状，全都使用你自己的距离单位。`range` 可以是普通数值，或在付出惩罚后仍能打得更远时写成 `{ "normal": 30, "long": 120 }`；`area` 为 `{ "shape": "burst" | "cone" | "line", "size": n, "friendlyFire": false }`(见“位置”)：
  - `uses`：`{ "per": "encounter" | "day", "count": n }`。耗尽后该行动离开菜单。
  - `recharge`：`{ "dice": { "count": 1, "sides": 6 }, "from": 5 }`。开战时可用，使用后消耗；在生物自己的回合开始时投骰，达到 `from` 或更高便恢复。无论结果如何，日志都记录骰子。
  - `sequence`：按顺序执行同一块的其他行动，每个行动有自己的目标。**一个行动攻击两次的生物就是这样编写的。** 一次额度支付整个序列。序列不携带自己的效果，也绝不能指定另一个序列。
  - `signature`：`{ "cost": n }`，使用生物自己的点数而非行动额度支付，并且仅在别人行动时可用：战斗会在一回合与下一回合之间的窗口提供它(见“窗口”)。
  - `reaction`：与目录条目的 `mechanics.reaction` 相同的对象（见“窗口”）：该行动不出现在任何回合的菜单上，而是在它指定的时刻提供。反应不能同时是招牌行动，任何序列也不能包含反应。
  - `self: true`：它作用于生物自身而不是别人，因此不带 `targetCount` 或 `area`。两者都需要 Capability API 1.46。Parry 就同时使用这两者，并带有一个让防御加 2 的 `parrying` 状态：

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
- 豁免需要在行动本身指定难度：行动强制的豁免使用 `save.difficulty`；若行动没有自己的豁免，而状态在豁免成功时结束，则使用 `saveDifficulty`。即使生物有人物卡，块行动仍使用直接数值编写，所以该数值放在行动上。子项自己的豁免可以省略 `difficulty`，回退到同一个数值。
- `damage.plus` 与目录条目的 `plus` 是同一种子项列表，读取方式完全相同：`"damage": { "dice": "1d6", "flat": 2, "type": "piercing", "plus": [{ "dice": "1d4", "type": "fire" }] }` 表示带热量的啃咬，热量有独立数值，独立受到抗性影响，也独立翻倍。

5e 草案自己的生物图鉴是在 `docs/development/ruleset-5e-2014.example.json` 中手写的五个生物，覆盖动作序列、充能、带状态的豁免、抗性与免疫、有限次数、招牌点数，以及一个以人物卡编写的生物。

#### 用规则集自身定义编写生物

生物不必用直接数值书写。改给它一份与角色人物卡形状完全一致的 `sheet`，战斗就会按构建队友的方式构建它：生命、防御、豁免、先攻、速度，以及列表中的每项攻击和能力，都由你自己的人物卡公式得出。如果规则集中的敌人与角色使用相同的属性、技能和列表，无论具体是什么，都可以这样说明。Ember Roads 的 Toll Warden 如下：

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

- **每部分都可省略：** `abilities`、`skills`、`saves`、`bonuses`、`fields` 和 `lists`，以人物卡声明的 ID 为键。所有省略内容都读取人物卡自身默认值，与空白角色完全一样。守卫的 Grit 为 9，是因为 `grit_max` 将 4、Toughness 和 Brawn 相加；Guard 为 7 也是同类计算结果。
- **每个数值只有一个来源。** 带人物卡的生物不能再提供 `health`、`defense`、`initiativeModifier`、`speed`、`abilities`、`saves` 或 `checks`，否则 Engine 拒绝文件。它可以没有自己的 `actions`，因为它的列表决定它能做什么。没有人物卡的生物仍须提供前三项和至少一个行动。
- **按作者编写的数据验证。** 每个 ID 都必须由人物卡声明；技能或豁免设为你为其提供的某个熟练档次；字段、数值、加值或列须持有声明允许的内容(范围内的整数、某个选项值等)；列表行数不得超过允许数量。没有 `live` 部分，因为生物已消耗的内容由战斗保管。
- **行可以来自目录。** `_catalog: "<catalog>/<entry>"` 和角色人物卡一样，指定行所选自的条目；战斗也从该条目读取这行的费用和效果。目录必须向该列表供给内容。目录内联时，条目必须在其中；目录为独立文件时，若行指定文件中没有的条目，该行就不给生物任何效果。生物图鉴中人物卡所引用的目录，会与图鉴一起为战斗加载。
- **条目在人物卡之外声明的内容仍有效：** `tier`、`traits`、`actions`、`signaturePoints`、`riders`、`resist`、`vulnerable`、`immune` 和 `conditionImmunities`。
- **从自己的池中支付。** 池以满额开始，生物为列表提供的行动消耗它们；无论由 Engine 还是 Game Master 决策，都会像队友一样获得更大额支付的选项(用更高法术位施法)。守卫的 Hold the Line 消耗 Luck。
- **使用伤势轨道时，生命就是轨道。** 打击先应用 `resist`、`vulnerable` 和 `immune`，再按 `damageKinds` 标记生物自身轨道，因此免疫某种伤害的生物不会因它获得标记。
- **它仍是敌人。** 降到零时出局而非濒死，从不作死亡豁免；画面显示原本就会展示的敌人信息，不显示其人物卡。即便有同名角色，也不会将它的消耗写回任何地方。
- **计算结果完全没有生命的人物卡**会被排除出战斗，并在开场日志说明原因，不会作为无人能伤害的存在入场。
- 如果某层从枚举字段中移除一个值，使用该值的生物不会因此丢失：该层启用期间，该字段和角色一样读作默认值，不会因此拒绝生物。
- Game Master 也可以创作一个，并按其档位约束(见“无人预先编写的敌人”)。
- 提供这种生物的包须声明 Capability API 1.34。

5e 草案的 Toll Sergeant 在 d20 人物卡上采用相同做法：其 Armor Class、生命、豁免，以及每行动两次挥击，都来自自身字段和攻击列表。

#### 无人预先编写的敌人

Game Master 创作敌人时，Engine 会在任何投骰发生前，将提案拉回你的 `threat` 尺度：生命限制在该档区间内，防御、命中和豁免难度最多高于该档自身值 2 点，并持续降低伤害，直到生物的最佳一轮(最强动作序列或最强单个行动，按一个目标衡量)落入该档的 `damagePerRound`。它先减少骰子数量，再减固定值，再从序列减少一次打击，最后才缩小骰子面数，绝不减到什么都没有。规则集不存在的名称会丢弃，包括未知伤害类型、状态和豁免，以及前六个之后的所有行动。未声明的档位回退到尺度最底档。每次更改都返回一句易懂的话，因此日志可以说明做了什么。

创作的敌人也能像图鉴生物一样写成 `sheet`，这让即兴创造的法师获得法术位和法术。Game Master 会看到人物卡 ID 及各项允许的值、战斗读取的列表，以及目录为这些列表提供的名称，因此法术按名称指定，而非用描述代替：凡是指定目录条目的行，不区分大小写，都会成为该条目，再叠上 Game Master 自己的值(例如法术已准备)。因为由模型编写，人物卡会宽松读取：规则集没有的名称丢弃，值调整到字段或列允许的范围，写在人物卡旁边的数值不采用。

不是首领的创作生物，只能使用规则集向它开放的内容。带 `startFrom` 的目录筛选器指定组织条目的人物卡字段(5e 包的法术列表按 `class`)；创作生物只保留筛选器匹配自身该字段值的条目，匹配方式与选择器按它打开时相同。因此 Sorcerer 永远不会获得整个法术列表，未指定职业的生物也不会从按职业组织的目录中得到任何内容。之后，无须再询问 Game Master，就会填补尚未作出的选择：对于只有选中后行才有效的每个列表(战斗能力来源带有 `onlyWhen`)，从向它开放且能用自身池支付的条目中，为每个现有池补足少量条目(能力越高的生物越多)，也补足能随意使用的条目。获得的内容受它战斗时同样使用的性情与能力影响：保护型或支援型选择支撑己方的内容，鲁莽型选择伤害，条理型或耐心型选择牵制敌人的内容；能力越高，越可能携带反应、反制或其他改变回合的效果。抽取使用战斗自身的种子，因此同一战斗总会以相同方式填充。Game Master 在这类列表中按名指定的行视为已选中。

首领可能是例外，因此完全由 Game Master 编写：不会删去任何内容，也不会填补任何内容。

之后，两者都按档位约束：

- 通过生命读取自的唯一字段，将生命调整到档位区间。池的最大值要么直接是该字段，要么是恰好包含一个字段的 `sum`(5e 的最大生命、Ember Roads 的 Toughness)。不含单一字段的生命公式保持原样，并在日志中说明。伤势轨道长度由你决定，永不更改。
- 生物构建完成后，将防御、命中和豁免难度限制为最多高于档位自身值 2 点，并降低伤害，直到计入它所能负担的最大支付后，其最佳一轮仍处于该档 `damagePerRound` 内。先削减更大支付带来的增量，再减骰子、固定值、一次打击，最后才缩小骰子面数。

你自己的生物图鉴永远不会被限幅。那是你编写的数据，所以 Engine 按原样采用。

### 位置：棋盘上的战斗

在你的块声明棋盘每格代表多远之前，战斗都靠想象进行。声明 `distance` 后便可在网格上作战，此时移动、触及、射程、范围、视线、掩护，以及对走开者的打击才开始有意义。每项数值都由你写下，Engine 只提供棋盘。

```json
"distance": { "label": "ft", "perCell": 5 },
"ranged": { "long": "disadvantage", "adjacentFoe": "disadvantage" },
"cover": { "bonus": 2 },
"opportunity": { "budget": "reaction" }
```

Ember Roads 只声明其中一行，没有其他内容，这正说明其余部分均非必填。

```json
"distance": { "label": "paces", "perCell": 2 }
```

**格子。** `distance.perCell` 表示一格相当于你自己单位的多少，`label` 是该单位的名称。块所在世界中的所有距离都用它：`economy.movement`、生物的 `speed`、武器的 `reach` 和 `range`，以及生物行动的 `reach` 和 `range`。声明了自身 `units.distance` 的目录，用自己的 `perCell` 转换 `mechanics.range` 和 `area.size`；没有声明的目录则用这里的设置。大于零的距离会舍入到最近格数，但绝不舍入成零，因此任何有正数距离的内容至少触及一格。零不是短距离，而是保留自身含义：`mechanics.range` 为 0 表示自身或接触(接触别人可以触及邻格)；武器行的 `reach` 或 `range` 列为 0，则表示该行没有这种距离。

**是否使用棋盘。** 两个条件必须同时满足：你的块声明了 `distance`，且玩家游戏设置为 Tactical 战斗样式。使用 Classic 样式或规则集未声明 `distance` 时，战斗与以前一样靠想象进行：任何人都可以指向任何人，完全不读取下面的内容。

**玩家看到什么。** 棋盘会绘出 Tactical 样式自身的地形。每格都是可用指针或方向键访问的按钮，说明它是什么、谁在上面，以及尚未完成的选择如何影响它。行走时会点亮菜单提供的格子，每格以你自己的单位标出费用，绘出到达路径，并将路径中有人会出手攻击的格子标为琥珀色，在棋盘下方列出攻击者。需要目标的选项会在棋盘和列表中同时高亮可选对象。带 `area` 的选项瞄准格子，指针下的格子会说明将卷入谁，包括友军。剩余移动额度显示在行动额度旁边，也使用你的单位。画面不自行测量任何内容：每个格子、费用、路径、目标和瞄准位置都由服务器发送。

**移动。** 每回合的移动额度，队友取 `economy.movement`，生物取自己的 `speed`，除以 `perCell` 后向下取整；但只要还能移动，就至少为一格。额度在持有者自己的回合开始时补满，可在行动前、行动间和行动后使用：走动、攻击、再走动。踏入一格费用为 1，崎岖地面则更高。八个方向的费用相同，因为这正是所面向的桌面网格玩法。可以经过友军，但不能停在任何人的格子上；敌人视作墙；不能进入实体障碍，也不能从两个实体格之间斜切墙角。

**触及与射程。** 武器行从 `combat.attacks[].reach` 和 `.range` 获取它们，各自可以是同一列表中的列，或对每行相同的数值：

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

某列在一行读作 0，表示该行没有这种距离，这让普通剑能和投掷斧放在同一列表。完全没有触及距离的行可触及一格。生物行动使用自己的 `reach` 或 `range`，目录能力使用 `mechanics.range`(0 表示自身或接触，指向别人时为一格)。

两者都有的行是投掷武器：触及范围内为挥击，超出后为射击。因此，下述射击规则不会影响拿在手中的挥击；它还能打击经过的人，而弓不能。

生物行动还可以携带其落点 `area`，使用你自己的单位：`{ "shape": "cone",
"size": 15 }`，并用 `"friendlyFire": false` 放过己方。这让吐息武器成为棋盘上的真实锥形，而非一个目标数。序列不带自身形状，由它指定的各行动分别携带。没有棋盘的战斗会忽略形状，使用 `targetCount`，所以生物条目可以两者兼有，并在两种情况下都准确表达。

**形状能送到多远。** 由 `range` 指定：投到一百英尺外的球需要射程。没有射程时，爆发在放下的位置、即行动者自己的格子生效；锥形或直线则可以瞄准所画长度内的任何位置，因为此时格子只表示方向。这既适用于生物，也适用于目录条目的 `mechanics.area`。

`ranged` 说明超出普通 `normal` 距离射击，或邻格有敌人时射击的代价。两项都可为 `"disadvantage"` 或 `"normal"`；省略整个块，则两者都没有代价。挥击永远不是射击，所以两条规则都不影响它；在自身触及范围内使用的投掷武器也一样。

**范围。** 条目的 `mechanics.area` 成为棋盘上的真实形状，瞄准格子而非某个人，`targetCount` 对此没有意义：触及多少人由形状决定。除非条目指定 `"friendlyFire": false`，否则格子中的所有人都会卷入，不分敌友。

```
burst, size 2, aimed at X        cone, size 3, aimed right      line, size 3, aimed right
. . . . .                        . . . .                        . . . .
. # # # .                        . . # .                        A # # #
. # X # .                        A # # #                        . . . .
. # # # .                        . . # .
. . . . .                        . . . .
```

爆发包含瞄准格周围大小范围内的每一格。锥形从行动者朝该格延伸，每一步的宽度等于其距离。直线朝同一方向延伸，宽一格。三种形状遇到实体障碍都会停止。

**视线与掩护。** 以两者之间的直线格子判断：任何实体障碍都会阻挡射击并阻止范围扩散到其后，目标也就不会出现在菜单上。有掩护价值的地形会为攻击所对抗的防御增加 `cover.bonus`，日志会说明。没有四分之三掩护、全掩护或高低差。

**打击走开的人。** 声明 `opportunity.budget` 后，如果参战者走出某个站立、可行动、持有该额度并有近战手段的敌人的触及范围，行走会停在当前位置，询问该敌人是否攻击。选择攻击则消耗额度，并与其自己回合的同一攻击完全一样地结算；放行不消耗任何内容。不论哪种情况，之后行走都从暂停处继续，支付实际经过的每格费用；如果打击让移动者倒下，行走就在倒下处结束。整次行走对每个敌人只给一次机会，无论路径反复离开同一触及范围多少次。`disengage` 在本回合余下时间阻止它；未声明 `opportunity` 的规则集完全没有这一机制。

这次询问就是一个窗口，会暂停整场战斗：被询问者全部回答之前，其他任何内容都不推进。队友的窗口由玩家回答，旁边提供攻击和 Pass；其他人的窗口由其操作者回答，Game Master 所扮演的首领则通过 Game Master 自己的决策回答。见下文“窗口”。

**敌人如何使用棋盘。** 没人操控的敌人会将每个可达格与从那里可用的每个选项组合评估，为行走中会遭遇的每次打击扣分；若当前格已经可以做到最佳，就倾向于不移动。没有任何目标在范围内时，它会靠近；若 `standard` 列表有 `dash`，就先疾走。

**可能看到的拒绝原因。** `out-of-reach`(超过触及范围)、`no-line-of-sight`(中间有实体障碍)、`unreachable`(无法支付行走费用或不能停留的格子)，以及 `bad-cell`(将形状瞄准不允许的位置)。

### 服务器如何用你的块处理战斗

规则集声明了 `combat` 的游戏，会在 Engine 一直使用的同一份已保存战斗中，按该块结算：

- **谁参战。** Game Master 指定参战者，Engine 从每位队友自己的人物卡读取数值。没有该规则集人物卡的成员会按名拒绝，不会给它你没写过的数值。
- **敌人数值的来源**按以下顺序：Game Master 在图鉴中指定的生物；标签与敌人自身名称匹配的生物；Game Master 为本场战斗提议、并经限幅拉回威胁尺度的数值块；最后是用档位自身数值构建的普通生物。每次回退和限幅都以易懂的话记录，让战斗能说明做了什么。如果规则集既没有图鉴条目、没有提案，也没有威胁尺度，就拒绝战斗，不会凭空创造敌人。
- **人物卡就是记录。** 每个接受的行动之后，生命、池、状态、专注以及濒死规则的计数都通过人物卡自身规则写入。因此战斗中途重新加载也会准确显示战斗留下的结果，不存在可能与它冲突的战后结算总账。
- **菜单是合法性的唯一依据。** 所有行动者，无论玩家还是敌人，都从你的块生成的同一菜单选择 ID。Engine 扮演的敌人用 Engine 自己的战术选择；Game Master 扮演的敌人则被要求从同一菜单选一个 ID，能看到你的数值，却不会被告知骰子将产生什么结果。
- **你的骰子。** 战斗持有自己的种子和游标，因此从磁盘读回后，会继续使用原本将投出的骰子序列。

### 屏幕显示

战斗画面使用你的措辞。菜单包含你的攻击、能力、对抗和列出的标准行动，每项说明从你的行动额度和池中消耗什么。行动顺序、轮次、你命名的每种状态及剩余轮数、临时点数、专注，以及濒死规则的两项计数都会显示。日志按你的术语打印真实算式："Juno attacks Rust jackal with Road axe: 8 (5 + 3) + 3 = 11 against Guard 6, a hit." 每个接受的行动都会立即写入人物卡，因此中途重新加载也精确无误，之后还会告知 Game Master 不要再次修改这些数值。

有位置的战斗绘制在棋盘上，而非立绘舞台上；玩家如何操作见“位置”。棋盘、菜单和日志中的每个距离都使用你的单位："Juno moves to 4, 6 for 6 paces and has 2 paces left."

### 对抗：擒抱、推撞、挣脱

有些动作不是对抗防御的攻击，而是一场对抗：双方都掷骰，结果更好的一方如愿以偿。擒抱某人、把对方推倒或推开，以及挣脱，都是同一种形状，因此规则集把它们都写成数据：

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

- **`checks`** 是对抗读取的数字，每个都是人物卡上的值，像防御或豁免一样在战斗开始时读取一次。以直接数值编写的生物给出自己的值（`"checks": { "brawn": 2 }`）；带人物卡的生物从人物卡读取。最多十二个。
- **掷骰。** 双方都掷你的 `attackRoll.dice`，并加上此处可用检定中最好的一个（`attacker.checks`、`defender.checks`）。总数较高者获胜；平局时防守方获胜，除非 `ties` 写为 `"attacker"`。日志会写出双方：“Juno tries Grab on Ash-hound: 12 (6 + 6) + 3 = 15 with Brawn against 2 (1 + 1) + 2 = 4 with Brawn, and wins.”
- **获胜**时执行 `onWin` 所规定的内容，至少一项：
  - `applies` 以获胜者为来源，对失败者施加状态，因此标为 `endsWhenSourceDown` 的状态会在抓住对方的人倒下时结束。`rounds` 为状态设定计时；没有它时，持续到被某种东西结束。
  - `ends` 移除行动者或目标身上的状态。
  - `push` 将失败者沿远离获胜者的方向直线推开，距离按你的距离单位计，在碰到任何实体障碍、任何站着的人、棋盘边缘或窄得挤不过去的角落前停下。这是强制移动：不消耗其任何移动额度，也没有人会借机打击它。没有棋盘的战斗不会移动任何人。
    失败没有任何效果，失败的擒抱就是这样。
- **`from: { "holding": "held" }`** 让对抗只瞄准对行动者施加该状态的人，并且只在该状态持续时才出现在菜单上：挣脱就是这样表达的。
- **`reach`** 以你的距离单位表示它能触及多远；省略时为相邻格。`reach` 和 `push` 需要 `combat.distance`。
- **`strike: true`** 让它在一个行动买到多次打击时，像攻击一样占用其中一次：先选它时，消耗额度并把其余打击留在手中；手中已有打击时选它，则消耗其中一次。5e 参考规则集的擒抱和推撞就是这样做的。
- **在菜单上**，对抗有自己的分组，并显示获胜几率。Engine 扮演的对手会像评估其他选项一样评估对抗，但比较保守：擒抱或推撞是为后续做铺垫，挣脱价值最高，但只在被抓住时才有意义。扮演对手的 Game Master 从同一菜单中选择。
- Game Master 自创的对手，其检定与命中几率一样受其档位约束。

对抗对打包规则集要求 Capability API 1.43。

<a id="windows-holding-the-fight-open"></a>

### 窗口：让战斗等待

有些时刻属于当前行动者之外的人。Engine 会为其保留战斗、等待选择，而不是代为决定；这段暂停就是窗口。

六种情况会打开窗口，其中两种来自你已经声明的内容：

- **有人脱离。** 行走离开可以攻击的敌人触及范围时，在那一步停下并询问敌人。见上文“打击走开的人”。
- **一回合与下一回合之间。** 回合结束后、下一回合开始前，会询问所有持有 `signaturePoints` 且能负担自身某个 `signature` 行动的敌人是否购买一次。这是唯一能购买的时刻：招牌行动不在任何人的回合菜单里，包括它自己的菜单。
- **有人使用某样东西。** 在结算前，会询问对方阵营中所有持有等待此刻条目的人，不论它指向谁，只要该条目能触及使用者。标准行动和对抗不会打开这一时刻。
- **某个效果指向某人。** 在结算前，会询问被指向的对方阵营中，所有持有等待此刻条目的人。友军治疗你并不是需要应对的威胁，因此友军行动不会打开窗口。
- **攻击检定命中某人。** 在造成伤害之前，如果被命中者持有等待此刻的条目，就会询问他们。他们选择的内容对这次攻击生效：见下文“命中之后的时刻”。
- **某个效果伤害了某人。** 结算后，无论是谁造成的，都会询问所有受伤且持有等待此刻条目的人。受伤是发生在你身上的事实；但反指向伤害来源的条目仍不能指向友军。

最后四种情况由目录条目或生物自身的行动通过指定所等待的时刻来请求。一个行动同时打开 `used` 和 `aimed` 时，先询问使用；如果没有人取消它，再询问它所指向的人，两者都答复后才结算。被命中在每次掷骰命中时询问，受伤则在整个行动结束后询问。

无论由什么打开，窗口都会这样运作：

- **窗口打开期间，其他任何内容都不推进。** 当前回合的行动者、回合结束、其他窗口都不能推进。战斗等待。
- **按行动顺序逐个询问，每人只问一次。** 放弃始终是有效答复，不收费用。如果轮到的人没有可用选择，就直接跳过，不再询问。
- **从暂停处精确恢复。** 行走继续完成剩余格子，并支付实际经过的每一格。
- **谁操控，谁回答。** 自己队友的窗口由你回答，菜单中选项旁会有 Pass；敌人由其操作者回答，Game Master 的首领通过 Game Master 回答，放过这个时机也是可选答复之一。
- **随战斗保存。** 行走中关闭游戏，回来后仍是相同的人等待询问、相同的格子等待行走。

前两种不需要额外声明：带 `opportunity.budget` 的规则集获得其中一种，带 `signaturePoints` 的图鉴获得另一种；两者都没有的规则集从不会看到它们。

**声明条目等待哪个时刻。** 将 `mechanics.reaction` 写成对象，而非 `true`：

```json
"reaction": { "on": "aimed", "at": "source", "cancels": true }
```

- `on` 为 `used`、`aimed`、`hit` 或 `harmed`，决定将条目放入哪个窗口菜单。Engine 只监听这四个时刻。仍然只写 `"reaction": true` 的条目，仅表示不在回合内执行，不足以将它放入任何位置，所以仍不上任何菜单。
- `at` 为 `source`(默认)或 `chosen`。`source` 将执行内容指向引发该时刻的人，并填入目标，无须任何人再选；`chosen` 则保留条目自己的目标设置并询问。
- `cancels` 阻止窗口保留的事情发生。只有 `used` 或 `aimed` 条目能指定它：已经发生的时刻不能撤销，而命中时攻击检定已经掷出。
- `against` 把条目回应的范围收窄到使用自特定目录的东西：`"against": { "catalogs": ["spells"] }` 只回应法术，别无其他。背后没有条目的行动（列表上的武器、生物自身的行动）不来自任何目录，因此永远不会为指定了目录的条目打开这一时刻。省略 `against` 时，条目回应任何东西。

也请指定 `budget`，否则消耗列表默认额度。反应几乎总是使用自己的额度，这正是阻止一个回合内使用多次的机制。

**询问任何人之前，费用就已经支付。** 取消行动阻止的是事情发生，不是撤销购买：额度和池已经消耗。如果你的系统会退款，目前还无法表达。

反制这样写，在棋盘上它只回应处于其自身 `range` 之内的人：

```json
"mechanics": {
  "kind": "utility",
  "range": 8,
  "free": true,
  "cost": [{ "pool": "luck", "amount": 1 }],
  "reaction": { "on": "used", "cancels": true, "against": { "catalogs": ["knacks"] } }
}
```

**命中之后的时刻。** `hit` 上的条目用于表达 Shield、Parry 和 Uncanny Dodge。攻击在掷骰之后暂停，在造成任何伤害之前询问被命中者。他们选择的内容对这次攻击生效：不会重新掷骰，而是按其当前的防御重新判定，因此增加防御的状态可以把命中变成未命中，而总是命中的自然点数仍然命中。作为回应施加、持续一次攻击的状态（`endsAfter: "attacked"`）覆盖这次攻击，包括其伤害，并随这次攻击用掉；“这次攻击的伤害减半”就是这样表达的：作用于一次攻击的 `resist-all`。

```json
"mechanics": {
  "kind": "buff",
  "targets": "self",
  "budget": "reaction",
  "reaction": { "on": "hit" },
  "applies": [{ "condition": "shielded", "duration": { "rounds": 1, "at": "turn-start" } }]
}
```

攻击从暂停处精确继续：剩余的目标，以及对由其他行动组成的生物行动而言，剩余的部分。每次命中持有此类回应者的掷骰都会依次暂停，窗口打开期间保存的战斗也会以相同方式恢复。在窗口内进行的攻击（打击走开的人、招牌行动、回应）从不暂停，因为窗口内打开的任何东西都不会再开另一个窗口。窗口打开时，日志会显示掷骰结果及其对抗的数值；如果回应改变了防御，还会显示现在是否未命中。Engine 扮演的对手只在防护能把命中变成未命中时才会使用它。

指定时刻的包需要 Capability API 1.33，使用 `used` 或 `against` 的包需要 1.44。等待 `hit` 的包需要 1.46。

### 尚未支持

在此明确列出，因为规则集不应声称 Engine 会做尚未支持的事：

- **超出简易棋盘的内容：** 没有四分之三掩护、全掩护、高低差、飞越障碍、挤过狭处、坐骑、躲藏或突袭；除了自身的行走和对抗的推撞，没有任何东西会移动任何人。没有人会拖着自己抓住的生物走。
- **对抗很简单。** 没有体型限制，也不会打开窗口（没有人可以回应对抗）。
- **条目只能等待 `used`、`aimed`、`hit` 和 `harmed` 四个时刻**(见上文“窗口”)。这是 Engine 为条目感知的时刻；另外两种窗口，即有人脱离和两回合之间的暂停，由战斗本身打开，并不是条目可以请求的时刻。没有豁免投骰、死亡、回合开始或坠落时刻。
- **反制会直接阻止它回应的东西。** 它无法区分同一目录中的不同条目，也没有用来阻止比自己更强之物的检定。
- **不会连锁。** 战斗保留一个窗口，而非窗口栈，因此窗口内发生的事情不会再开另一个窗口：反制不能再被反制，反应造成的效果不会打开后续时刻，窗口内进行的攻击命中时也不会暂停。
- **只有攻击检定才算命中。** 不经攻击检定就生效的东西（直接命中的飞镖、所有人都要豁免的范围效果）不会打开 `hit` 时刻，而回应无法区分挥击与射击，因此规则只允许用剑格挡的 Parry 也能格挡箭矢。
- **伤害只能减半，不能减去一个数量。** 回应可以让一次攻击的伤害减半（作用于该次攻击的 `resist-all`），但不能从中减去一个掷出的数字。
- **没有退款。** 被取消行动的费用已经消耗。
- **状态能改变的内容是封闭列表。** 防御、攻击检定、豁免、对抗检定和速度（等级以及物品在战斗中的作用也是如此），此外别无其他：没有对持有者所造成伤害的加值，也没有降低角色上限或让其退出战斗的等级。
- **直接用数值编写的生物没有伤势轨道。** 即使规则集用轨道表示生命，这类生物仍然扣点数；给它 `sheet` 后，受到的打击才会先应用自身 `resist`、`vulnerable` 和 `immune`，再标记格子。
- **附加效果自行触发。** `on` 只有 `hit` 一个值，因此周期内第一次符合条件的命中就使用它，不存在询问是否消耗的时刻。
- **会被攻击改变的数字很简单。** 消耗型打击只掷这个数字，别无其他，因此任何武器都不会改变它，包括武器的 `floor`(结实的目标会吸收夺取型打击，其硬度会挡住消耗型打击，但除此之外没有东西能削减这两者)；无论跌得多深，崩溃都在固定回合数后解除；在窗口中进行的攻击（打击脱离的人、反应、招牌动作）都按第一种方式进行。
- **自创的对手不吸收伤害，也没有硬度。** Game Master 为一场战斗临时创造的生物受你的威胁尺度约束，而威胁尺度不涉及吸收或硬度，因此它两者都没有。

## 层：规则集自身的变体

层是规则集的具名变体，由玩家创建游戏时开启，例如 Low magic、Hard winter，或更残酷的难度。层位于规则集文件的可选 `layers` 数组中，因此随文件一起流转，不会从使用过它的游戏中丢失。向导在规则集下将其显示为开关，选择与规则集本身一样，在游戏整个生命周期中固定。

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

**层能做什么。** 列表是封闭的，每个效果要么缩小范围，要么添加文本：

- `gm.guidance` 追加到 `gm.checkGuidance`，位于你自己的文本及任何先前层之后。`gm.worldGuidance` 以同样方式追加到 `gm.worldGuidance`。
- `fields` 从 **enum** 字段移除值。`removeValues` 指定字段已有的值，必须至少保留一个；若字段的 `default` 也被移除，层须指定一个仍保留的 `default` 代替。
- `difficultyLadder` 以另一套梯度替换原梯度，形状须符合自身的结算类型：`dice-sum` 用 `{label, dc}`，`dice-pool` 用 `{label, successes, target?}`。它接受与原梯度完全相同的验证。若多个启用层都声明了它，最后一个优先。
- `catalogs` 从人物卡编辑器选择器中隐藏条目。每条规则指定该目录已声明的某个 `filters`，以及恰好一种比较：`number` 筛选用 `above` 或 `below`，`text` 或 `tags` 筛选用 `equals` 或 `notIn`。完全没有设置该筛选项的条目永不隐藏。
- `currencies` 移除钱币：`removeUnits` 指定单种钱币，`removeFamilies` 指定整个体系。体系中最小的钱币只能随整个体系一起移除，因此剩下的每个体系都仍能付款和找零。该层开启期间，没有人会收到、用来支付、选取或掉落被移除的钱币，钱袋那一行也不显示它。以它标出的价格，会按相同价值改用剩下的、能正好付清的最大钱币表示：Gravewatch 的 long night 层移除了 crown，因此标价 3 crowns 的守夜手枪要 15 shillings。整个体系都被移除的物品没有价格。

**层不能做什么。** 不能添加枚举值、字段、技能、池或休息，不能改变结算类型、触及实时状态或战斗数值，也不能增加模型调用。层所_添加_的值，对人物卡的其他读取者都会是未知值，因此只能移除值。超出这份列表的内容，应当修改规则集本身，或成为第二个规则集。

**冲突。** `conflicts` 指定不能同时开启的层。只在成对关系的一侧声明就够了。向导会禁用另一个开关；如果保存的选择不知为何包含两者，就丢弃声明**较晚**的那个，因此相同的两个选择总会得到相同规则。

**已经持有被移除值的人物卡会保留它。** 不会重写角色。编辑器只是停止提供该值，而已经拥有它的角色仍按原值显示。在新游戏中关闭该层后，该值又能选择。隐藏的目录条目也一样：只从选择器中排除，玩家已经选择的行仍留在人物卡上。层所移除、但已有人携带的钱币，仍留在其背包里。

**限制。** 每个规则集最多 12 个层，每层两段指引字符串合计最多 4000 字符。声明 `layers` 或基础 `gm.worldGuidance` 的打包规则集需要 Capability API 1.25，层会移除钱币的规则集需要 1.64。导入的规则集由读取它的 Engine 验证，因此不需要这项声明。

**其他人编写的层**(例如为不是自己编写的规则集提供独立文件的 Low Magic 层)会在以后添加。目前层随其所属规则集一起提供。

<a id="trying-your-ruleset"></a>

## 试用你的规则集

社区规则集使用与导入智能体相同的开关。打开 **Settings**(设置) > **Advanced**(高级) > **Danger Zone**(危险区域)，确认 **Allow custom Agent imports**(允许导入自定义智能体) 已开启。导入还需要 localhost 访问，或已配置的 **Admin Access**(管理员访问)。

1. 打开 **Agents**(智能体) 面板，选择 **Import agents**(导入智能体) 按钮，即面板顶部按钮行中的下载图标。
2. 选择 **Game Mode ruleset**(Game Mode 规则集)，再选择 JSON 文件。
3. 阅读审核信息，其中包含名称、版本、许可证、规则集覆盖范围，以及 Game Master 文本。选择 **Import**(导入)。

规则集会出现在面板的 **Rules**(规则) 区域，以及新游戏设置向导的 **Rules** 选项中。文件导入的规则集归为 `local/<your id>`，因此绝不会与官方规则集或别人的规则集混淆。

### 修改已导入的规则集

已经导入的版本永不重写。如果更改文件后仍以相同 `version` 再次导入，会被拒绝，并要求提高版本号。这是有意设计的：游戏绑定创建时的精确版本，因此进行中的战役不会突然使用不同的计算方式。

所以起草时的循环是：编辑、提高 `version`、导入、开始新游戏。旧版本会与新版本并列安装，直到你从 **Rules** 区域移除规则集。移除仍被游戏使用的规则集后，该游戏会提示规则集缺失，直到再次导入。

如果改变了人物卡结构(添加、删除或重命名内容)，也请提高 `sheet.version`。现有人物卡会宽容读取：新人物卡不认识的值会保留，缺少的值使用默认值。

## 分享你的规则集

**作为文件。** 将 JSON 文件发给朋友，对方按相同方式导入即可。

**通过 GitHub 仓库。** 如果将作品保存在公开 GitHub 仓库，请把每个规则集放在仓库顶层的 `rulesets` 文件夹中，每个规则集一个文件：

```text
your-repository/
  agents.json        (optional, only if you also share agents)
  rulesets/
    ember-roads.json
    another-system.json
```

用户通过自定义智能体仓库列表添加你的仓库一次，审核内容后，便可在以后同步获取新版本。自定义仓库列表是高级功能，需要服务器运营者使用 `ENABLE_CUSTOM_AGENT_REPOS=true` 开启。仓库中的规则集归到仓库所有者名下，例如 `alice/ember-roads`，所以两个作者都能发布名为 `v20` 的规则集而不冲突。

有两项限制。仓库的 `rulesets` 目录下最多直接存放 32 个 JSON 文件，超过则拒绝该仓库。名为 `local` 的账号不能发布规则集，因为 `local/` 保留给从文件导入的规则集。

**进入官方目录。** 广泛游玩且授权明确的系统，可以通过 **Download Agents**(下载智能体) 提供给所有人。这需要向 [Marinara-Agents](https://github.com/Pasta-Devs/Marinara-Agents) 仓库提交拉取请求。布局请参考其中的 `ruleset-5e-2014` 包。

## 授权许可

只发布你有权分享的规则文本。许多系统会按开放许可证发布参考文档，你可以复制的是该文档。将许可证 ID 及许可证要求的署名文本放入 `license`。不要从非开放授权的规则书复制文本。规则集主要需要名称和数值，Game Master 文本应使用你自己的措辞。

## 故障排查

- **导入提示某个名称不存在。** 文件中某处引用了未声明的 ID，例如技能引用已删除的属性。消息会给出该位置的路径。
- **导入提示同一版本已安装但内容不同。** 提高 `version` 后重新导入。
- **设置向导中没有我的规则集。** 检查 **Allow custom Agent imports** 是否开启。关闭时，导入的规则集不会提供给新游戏；已经使用它的游戏仍继续工作。
- **游戏提示规则集缺失。** 没有安装游戏创建时所用的精确版本。请重新导入该版本文件。
