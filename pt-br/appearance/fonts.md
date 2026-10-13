# Fontes personalizadas e Google Fonts

Neste guia você aprende a trocar a fonte que Marinara Engine usa em todo o aplicativo. Use a fonte que já vem instalada, adicione arquivos de fonte próprios ou baixe uma fonte do Google Fonts pelo nome.

## Como escolher a fonte do aplicativo

A opção de fonte fica em **Settings** (Configurações), na aba **Appearance** (aparência), dentro da seção **Text & Scale** (texto e escala).

1. Abra **Settings** e clique na aba **Appearance**.
2. Localize a seção **Text & Scale**.
3. Abra o menu suspenso **Font**.
4. Escolha uma fonte da lista.

A opção padrão é **Default (Inter)**. Inter é uma fonte limpa, pensada para leitura na tela. As fontes personalizadas que você adiciona aparecem no mesmo menu suspenso **Font**, logo abaixo da opção padrão.

Os campos de prompt do editor de presets e o Example Dialogue de um personagem usam uma fonte monoespaçada enquanto a fonte padrão está selecionada. Escolha qualquer outra fonte e eles também passam a usá-la. Os outros campos de prompt, como o modelo de prompt de um agente, continuam monoespaçados.

Os widgets do chat têm uma opção **Font** própria em **Settings → Appearance → App → Chat widget style** (Configurações → Aparência → Aplicativo → Estilo dos widgets do chat). Ative **Apply preset font** (aplicar fonte do preset) nessa mesma seção para usar essa fonte também nas mensagens, nos campos de entrada e nos controles do chat. Deixe esse botão liga/desliga desativado para que eles mantenham a fonte habitual. Veja [Estilos prontos para janelas de chat](custom-css-themes.md#ready-made-chat-window-styles).

A fonte escolhida é sincronizada entre os dispositivos. Ao escolher uma fonte, todos os navegadores e dispositivos ligados ao mesmo servidor Marinara passam a usá-la. Para entender como essa sincronização funciona, veja o guia [Visão geral das configurações](../settings/settings-overview.md).

## Como adicionar fontes próprias

Para adicionar uma fonte personalizada, coloque o arquivo de fonte em uma pasta do servidor, ou seja, a máquina que executa Marinara.

1. Localize a pasta `data/fonts/` dentro da pasta de dados do Marinara, na máquina do servidor.
2. Copie o arquivo de fonte para essa pasta.
3. Volte para **Settings**, depois **Appearance**, depois **Text & Scale**.
4. Abra o menu suspenso **Font**. A fonte já aparece na lista.
5. Selecione a fonte.

Marinara lê estes tipos de arquivo de fonte: `.ttf`, `.otf`, `.woff` e `.woff2`. Arquivos com qualquer outra extensão são ignorados.

Marinara monta o nome de exibição a partir do nome do arquivo. Por exemplo, um arquivo chamado `OpenSans-Bold.ttf` aparece como "Open Sans". Então dê nomes claros aos arquivos para manter a lista organizada.

Os arquivos de fonte da pasta `data/fonts/` ficam no servidor. Todo dispositivo que se conecta ao mesmo servidor Marinara pode usá-los. A fonte escolhida também é sincronizada entre esses dispositivos, então todos mostram a mesma fonte.

## Como baixar do Google Fonts

Marinara busca uma fonte direto do Google Fonts para você. Para isso funcionar, o servidor precisa de acesso à internet.

1. Abra **Settings**, depois **Appearance**, depois **Text & Scale**.
2. Localize o campo **Google Fonts**.
3. Digite o nome exato da fonte, por exemplo `Fira Code` ou `Lora`.
4. Clique em **Add** (adicionar).
5. Espere o download terminar. A nova fonte então aparece no menu suspenso **Font**.

Digite o nome exatamente como o Google Fonts escreve. O link **Browse fonts at fonts.google.com** fica ao lado do campo. Ele abre o site do Google Fonts em uma nova aba, para você consultar os nomes.

O nome aceita apenas letras, números e espaços. Se você baixar a mesma fonte de novo mais tarde, Marinara substitui a cópia antiga em vez de criar uma duplicata.

Se o download falhar, leia a mensagem de erro. Quando Marinara não consegue alcançar o Google Fonts, ela pede que você verifique a conexão com a internet. Quando a mensagem diz que a fonte não foi encontrada, há duas causas possíveis. O nome pode não corresponder a nenhuma fonte do Google Fonts. Ou a fonte pode não ter o peso regular (400), que é o estilo normal, sem negrito. Confira a grafia e veja no site do Google Fonts se a fonte oferece o estilo Regular.

## O botão Open Fonts Folder só funciona localmente

Ao lado do menu suspenso **Font** existe o botão **Open Fonts Folder** (abrir a pasta de fontes). Ele abre a pasta `data/fonts/` no explorador de arquivos da máquina do servidor.

Esse botão age no servidor, não no dispositivo em que você está usando Marinara. Se Marinara roda no seu próprio computador, ele abre a pasta para você. Se você acessa de um celular ou de um segundo computador, o botão não faz nada de útil. Nesse caso, copie os arquivos de fonte para a pasta `data/fonts/` do servidor por conta própria.

## Guias relacionados

- [Configurações de aparência](appearance-settings.md)
