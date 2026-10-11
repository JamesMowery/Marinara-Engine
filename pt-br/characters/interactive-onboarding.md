# Onboarding interativo

Este guia explica o **onboarding interativo**, um recurso do card de personagem que permite aos jogadores montar a própria persona respondendo às perguntas do autor do card ao começar um chat de Roleplay. Ele mostra como os autores configuram o recurso no Character Editor (editor de personagens) e o que os jogadores veem no assistente de configuração de novo chat.

## O que ele faz

A persona é o personagem que representa você em um chat. Veja [Personas do usuário](personas.md). Normalmente, você mesmo escreve uma persona antes de jogar. Com o onboarding interativo, o card de personagem traz no lugar disso um modelo de persona. O modelo é formado pelos cinco campos de persona (**Description**, **Personality**, **Backstory**, **Appearance** e **Scenario**), com lacunas para o jogador preencher, e pelas perguntas que preenchem essas lacunas.

Quando um jogador começa um chat de Roleplay com o card, ele pode escolher **Create your persona** (criar a sua persona), responder às perguntas e receber uma persona nova e comum. As respostas são escritas no texto uma única vez. O resultado é uma persona normal, que o jogador pode editar, manter ou excluir como qualquer outra. O card em si não muda.

O onboarding usa a sintaxe de macros que já existe. Ele não roda nenhum script.

## Configurar o recurso como autor

1. Abra o personagem no Character Editor. Veja [Criando e editando personagens](creating-and-editing-characters.md).
2. Abra a aba **Onboarding**, ao lado de **Card**.
3. Ative o botão liga/desliga **Interactive onboarding** (onboarding interativo). Um guia curto, **How onboarding works** (como o onboarding funciona), aparece abaixo do botão.

Desativar o botão esconde os campos de onboarding, mas mantém o que você escreveu.

### Adicionar perguntas

Em **Onboarding Questions** (perguntas de onboarding), clique em **Add Question** (adicionar pergunta). Cada pergunta funciona como uma variável de preset. Veja [Editor de presets e gerenciador de prompts](../prompts/presets.md). Um card pode ter até 100 perguntas.

- **Variable Name** (nome da variável) é o nome que você usa nos campos, como `faction`. Escolha qualquer nome com letras, números e sublinhados que comece com uma letra ou um sublinhado. Os nomes diferenciam maiúsculas de minúsculas: `Gender` e `gender` são diferentes.
- **Question (shown to user)** (a pergunta mostrada ao usuário) é o que o jogador lê. Ela pode usar respostas anteriores, como `What does {{player}} look like?`. Aqui, `{{char}}` é o nome do card e `{{user}}` é o nome do jogador.
- **Options** (opções) transformam a pergunta em uma escolha. A primeira opção vem selecionada por padrão. Sem opções, o jogador digita uma resposta livre.
- **Value** (valor) é o que entra na persona. As maiúsculas e minúsculas ficam do jeito que você digitar; as macros dentro dele são preenchidas como no resto dos campos. Deixe em branco para usar o rótulo da opção. Digite um valor em minúsculas, como `ranger`, se a resposta entrar no meio de uma frase.
- **Help text for players** (texto de ajuda para os jogadores) é um texto opcional mostrado abaixo da opção.
- **Allow own answer** (permitir resposta própria) acrescenta a escolha **Write your own** (escrever a sua) ao lado das suas opções.
- **Multi-Select** (seleção múltipla) permite que o jogador escolha várias opções. Elas são unidas pelo separador.

Uma pergunta chamada `player` define o nome da persona. Se você não criar essa pergunta, o aplicativo pergunta primeiro o nome do jogador.

O editor avisa quando um nome está vazio, é inválido, foi usado duas vezes ou pertence a uma macro embutida, e também quando nenhum campo usa uma pergunta.

### Escrever os campos da persona

Escreva os cinco campos como os de uma persona sua e coloque `{{name}}` onde deve entrar uma resposta:

```
Name: {{player}}
Class: {{class}}
Role: {{player}} is a {{class}} sworn to {{faction}}.
```

Quando você clica em um campo, as **Available variables** (variáveis disponíveis) dele aparecem logo abaixo. Clique em uma para inseri-la onde está o cursor.

Use `{{#if}}` para mostrar um texto só para algumas respostas:

```
{{#if knowsHer == yes}}Grew up next door to {{char}}. {{relationToAna}}{{/if}}
```

Uma pergunta que só aparece dentro de um `{{#if}}` só é feita quando essa condição é verdadeira. No exemplo, `relationToAna` é uma pergunta de acompanhamento que aparece quando o jogador responde **yes**. As comparações ignoram maiúsculas e minúsculas.

`{{user}}` e `{{char}}` continuam como macros na persona criada, então funcionam como em qualquer persona. Outras macros, como `{{random}}`, `{{time}}` ou `{{getvar}}`, são preenchidas uma única vez, quando a persona é criada, ou ficam vazias se não tiverem valor nesse momento.

A **Rendered preview** (prévia renderizada), no fim, mostra a persona montada a partir de respostas de exemplo e lista as perguntas feitas, na ordem.

## Criar uma persona como jogador

1. Comece um novo chat de Roleplay. Na etapa **Persona & Characters** (persona e personagens), os cards com onboarding mostram um pequeno ícone de pessoa com um sinal de mais na lista de personagens.
2. Adicione o personagem. Uma linha **Create your persona** (criar a sua persona) aparece no topo de **Your Persona** (a sua persona), com o nome do card a que ela pertence.
3. Selecione essa linha e clique em **Next** (avançar).
4. Uma janela **Create your persona** mostra as perguntas do card. As perguntas de acompanhamento aparecem assim que as suas respostas as tornam relevantes.
5. Clique em **Create persona** (criar persona). A nova persona é salva e selecionada para o chat, e o assistente de configuração continua.

Se você escolher qualquer outra persona ou **None** (nenhuma, com a legenda "Stay anonymous"), nenhuma pergunta é feita. **Back** (voltar), na janela de perguntas, volta ao seletor de persona. Cada card com onboarding no chat ganha a própria linha **Create your persona**.

O onboarding é oferecido no assistente de configuração de novo chat de Roleplay. Quick Start, a configuração do Conversation e começar um chat direto de um card de personagem ainda não oferecem o recurso.

## Compartilhar cards com onboarding

O onboarding fica guardado com o card. As exportações **Marinara Native** e **Compatible PNG Card** mantêm o recurso, então um jogador que importa o card também recebe as perguntas. A exportação **Compatible JSON** deixa o onboarding de fora. Um card importado com mais de 100 perguntas mantém as 100 primeiras. Veja [Importar e exportar cards de personagem](import-export.md).

## Guias relacionados

- [Criando e editando personagens](creating-and-editing-characters.md)
- [Personas do usuário: criar e editar](personas.md)
- [Escolhendo a persona de um chat](choosing-your-persona.md)
- [Macros de prompt](../prompts/macros.md)
- [Importar e exportar cards de personagem](import-export.md)
