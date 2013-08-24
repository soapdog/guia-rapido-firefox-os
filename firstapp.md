#  Nosso Primeiro App {#firstapp}

![Memos, um bloco de notas minimalista](images/originals/memos-app.png)

Nesse capítulo vamos reconstruir o aplicativo **Memos** que é um bloco de notas que eu criei para servir de exemplo em minhas palestras. Esse programa possui três telas. A primeira está acima e é a tela principal do programa que lista os títulos das notas. Ao clicar em uma dessas notas ou clicar no sinal de adição somos direcionados para a tela de edição onde podemos alterar o título e conteúdo da nota como pode ser visto abaixo:

![Memos, tela de edição](images/originals/memos-editing-screen.png)

Nesta tela de edição o usuário pode também deletar a nota sendo mostrada ao clicar no ícone da lixeira e confirmar o desejo de deletar a nota como a captura de tela mostrada a seguir.

![Memos, confirmar a remoção de uma nota](images/originals/memos-delete-screen.png)

O código do Memos está disponível no [meu Github](https://github.com/soapdog/memos-for-firefoxos) para quem quiser baixar e olhar logo tudo de uma vez.

O Memos utiliza [IndexedDB](https://developer.mozilla.org/en-US/docs/IndexedDB/Using_IndexedDB) para armazenar as notas e o [Gaia Building Blocks](http://buildingfirefoxos.com/building-blocks) para a construção da sua interface. Em uma atualização futura deste livro eu falarei mais sobre os building blocks, nesta primeira versão simplesmente construiremos o app.

O primeiro passo é separarmos uma pasta para o aplicativo. Vamos chamar a pasta de **memos**.

## Criando o manifesto

O manifesto do Memos é bem simples. Crie um arquivo chamado **manifest.webapp** no a pasta **memos**. Manifestos são arquivos do tipo [JSON](http://json.org) que descrevem um aplicativo nele colocamos o nome, a descrição, os ícones utilizados e muitas outras coisas importantes como quais permissões o programa necessita para funcionar e qual arquivo é utilizado para carregar o app.

Abaixo podemos ver o conteúdo do manifesto do Memos, atenção ao copiar pois é fácil errar uma vírgula e tornar o seu JSON inválido. Para validar o seu JSON você pode utilizar várias ferramentas, uma delas que é especifica para validação de manifestos que é o [http://appmanifest.org/](http://appmanifest.org/). Para aprender mais sobre manifestos visite [a página na MDN sobre manifestos](https://developer.mozilla.org/pt-BR/docs/Apps/Manifest).

<<[Manifesto do programa Memos (*manifest.webapp*)](code/memos/manifest.webapp)

Vamos explicar cada um dos campos do manifesto acima.

|Campo		|Descrição	                                                                        |
|-----------|-----------------------------------------------------------------------------------|
|name		|Esse campo dita o nome do aplicativo.                                              |
|version	|Essa é a versão atual do seu aplicativo. Mudar a versão causa um update no app.    |
|launch_path|Qual arquivo que deve ser carregado quando o seu app é iniciado                    |
|permissions|Quais permissões seu app precisa. Mais informações sobre permissões adiante		|
|developer  |Quem desenvolveu esse aplicativo 													|
|icons		|Os ícones para cada tamanho necessário 											|


A parte mais interessante desse manifesto é a entrada de permissões onde pedimos a permissão para *storage* que permite que utilizemos o IndexedDB sem limitação de espaço[^storage-permission] (graças a isso podemos armazenar quantas notas quisermos no nosso programa).

[^storage-permission]: Para saber mais sobre as permissões que você pode pedir olhe [a página na MDN sobre permissões de aplicativos](https://developer.mozilla.org/en-US/docs/Web/Apps/App_permissions).

Com o manifesto pronto podemos passar para o HTML.

## Estruturando o HTML

Antes de colocarmos a mão na massa e montarmos o HTML utilizado pelo memos vamo falar rapidamente sobre o [Gaia Building Blocks](http://buildingfirefoxos.com/building-blocks) que é uma iniciativa de construir um conjunto de css e js reutilizáveis com o *look and feel* do Firefox OS para você aproveitar nos seus próprios apps.

No Firefox OS, assim como na web em geral, você não é obrigado a utilizar o *look and feel* do Firefox OS. Utilizar ou não os Building Blocks é uma decisão sua que passa por questões de *branding*, conveniência de uso, adequação ao que você precisa entre outras, o importante é entender que você não sofre nenhum tipo de repreensão no Firefox Marketplace por não utilizar a cara do Firefox OS. Eu como não sou um bom designer opto sempre por utilizar um pacote pronto como esse (ou contratar um designer).

A estrutura do HTML do nosso programa foi construída para se adequar aos padrões adotados pelo Gaia Building Blocks onde cada tela é uma *section* e os elementos seguem uma formulazinha. O ideal agora é que você baixe o código fonte do Memos a partir do [meu Github](https://github.com/soapdog/memos-for-firefoxos) para que você tenha os arquivos do Building Blocks.

W> Aviso: A versão que eu usei do Building Blocks não é a atual e eu modifiquei alguns arquivos portanto o código que vamos mostrar só vai funcionar com a versão que está no repositório do Memos.

### Incluíndo os Building Blocks

Antes de mais nada, copie as pastas **shared** e **style** para a pasta Memos para que possamos utilizar o Building Blocks. Vamos começar o nosso arquivo **index.html** com os *includes* necessários para o programa.

~~~~~~~~
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <link rel="stylesheet" type="text/css" href="/style/base.css" />
    <link rel="stylesheet" type="text/css" href="/style/ui.css" />
    <link rel="stylesheet" type="text/css" href="/style/building_blocks.css" />
    <link rel="stylesheet" type="text/css" href="shared/style/headers.css" />
    <link rel="stylesheet" type="text/css" href="shared/style_unstable/lists.css" />
    <link rel="stylesheet" type="text/css" href="shared/style_unstable/toolbars.css" />
    <link rel="stylesheet" type="text/css" href="shared/style/input_areas.css" />
    <link rel="stylesheet" type="text/css" href="shared/style/confirm.css" />
    <title>Memos</title>
</head>
~~~~~~~~

Na *linha 01* declaramos o tipo do documento como sendo HTML 5. Da *linha 05 até a linha 15* incluímos os CSS dos diversos componentes que são utilizados no aplicativo tais como cabeçalhos, listas, áreas de entrada de dados entre outros.

### Construíndo a tela principal

Agora podemos passar a implementação das telas. Como falamos anteriormente, cada tela do programa é uma **<section>** dentro do **body** do HTML que deve ter um atributo *role* com valor *application* tipo `<body role="application">`. Isso é utilizado pelos seletores dos CSS do Building Blocks. Vamos construír a primeira tela (e declarar o body).

~~~~~~~~
<body role="application">

<section role="region" id="memo-list">
    <header>
        <menu type="toolbar">
            <a id="new-memo" href="#"><span class="icon icon-add">add</span></a>
        </menu>
        <h1>Memos</h1>
    </header>
    <article id="memoList" data-type="list"></article>
</section>
~~~~~~~~

Nossa tela tem um **header** que possui um botão para adicionar novas notas e o nome do programa. Possui também um **article** que é utilizado para conter a lista de notas armazanadas no app. Nós utilizaremos as IDs do **article** e do **botão** para capturar eventos quando chegarmos na parte em JavaScript.

Repare que a criação da tela é um HTML bem tranquilo de se entender, construir a mesma tela em outras linguagens é muito mais trabalhoso. Simplesmente declaramos nossos componentes e damos IDs para elementos que desejamos referenciar posteriormente. 

Agora que temos a tela principal pronta, vamos montar a tela de edição que é mais complicada.

### Montando a tela de edição

~~~~~~~~
<section role="region" id="memo-detail" class="skin-dark hidden">
    <header>
        <button id="back-to-list"><span class="icon icon-back">back</span>
        </button>
        <menu type="toolbar">
            <a id="share-memo" href="#"><span class="icon icon-share">share</span>
            </a>
        </menu>
        <form action="#">
            <input id="memo-title" placeholder="Memo Title" required="required" type="text">
            <button type="reset">Remove text</button>
        </form>
    </header>
    <p id="memo-area">
        <textarea placeholder="Memo content" id="memo-content"></textarea>
    </p>
    <div role="toolbar">
        <ul>
            <li>
                <button id="delete-memo" class="icon-delete">Delete</button>
            </li>
        </ul>
    </div>
    <form id="delete-memo-dialog" role="dialog" data-type="confirm" class="hidden">
        <section>
            <h1>Confirmation</h1>
            <p>Are you sure you want to delete this memo?</p>
        </section>
        <menu>
            <button id="cancel-delete-action">Cancel</button>
            <button id="confirm-delete-action" class="danger">Delete</button>
        </menu>
    </form>
</section>
~~~~~~~~

Essa tela de edição contém a tela de diálogo utilizada quando o usuário tenta deletar uma nota por isso ela é mais complicada. 

No topo da tela que é marcado pelo **header** temos o botão de voltar para a tela principal, uma caixa de entrada de texto que é utilizada para mostrar e modificar o título da nota e um botão utilizado para enviar a nota por email.

Depois da toolbar que fica no topo, temos um parágrafo contendo uma área para a entrada de texto da nota e então uma outra toolbar com um botão para deletar a nota que está aberta.

Esses três elementos e seus nós filhos formam a tela de edição e após essa tela temos um **form** que na verdade representa a caixa de diálogo utilizada pela tela de confirmação da remoção da nota. Essa caixa de diálogo é bem simples contendo uma mensagem informativa e um botão para cancelar a ação de deletar a nota e um para confirmar.

Ao fecharmos essa **section** terminamos todas as telas do programa e o restante do código HTML serve apenas para incluir os arquivos de JavaScript utilizados pelo programa.

~~~~~~~~
<script src="/js/model.js"></script>
<script src="/js/app.js"></script>
</body>
</html>
~~~~~~~~

## Construíndo o JavaScript

Agora vamos programar de verdade ao dar vida ao nosso app. Para efeitos de organização separei o código em dois arquivos de JavaScript:

* **model.js:** que contém as rotinas para lidar com o armazenamento e alteração das notas porém não contém a lógica do programa ou algo relacionado a sua interface e tratamento de entrada de dados.
* **app.js:** responsável por ligar os elementos do HTML às rotinas correspondentes e contém a lógica do app.

Os dois arquivos devem ser postos em uma pasta chamada **js** ao lado das pastas **style** e **shared**.

### model.js

No Firefox OS utilizaremos o [IndexedDB](https://developer.mozilla.org/en-US/docs/IndexedDB/Using_IndexedDB) para guardar as notas. Como pedimos a permissão de *storage* podemos grava quantas notas a memória do aparelho permitir.

A parte do código do model.js que mostrarei abaixo é responsável por abrir a conexão e criar o *storage* se necessário.

A> Importante: Esse código foi escrito para ser entendido facilmente e não representa as melhores práticas de programação para JavaScript. Variáveis globais são utilizadas (ARGH!) entre outros problemas. Fora isso o tratamento de erros é basicamente inexistente. O mais importante desse livro é ensinar o *worlflow* de como programar apps para Firefox OS.

~~~~~~~
const dbName = "memos";
const dbVersion = 1;

var db;
var request = indexedDB.open(dbName, dbVersion);

request.onerror = function (event) {
    console.error("Can't open indexedDB!!!", event);
};
request.onsuccess = function (event) {
    console.log("Database opened ok");
    db = event.target.result;
};

request.onupgradeneeded = function (event) {

    console.log("Running onUpgradeNeeded");

    db = event.target.result;

    if (!db.objectStoreNames.contains("memos")) {

        console.log("Creating objectStore for memos");

        var objectStore = db.createObjectStore("memos", {
            keyPath: "id",
            autoIncrement: true
        });
        objectStore.createIndex("title", "title", {
            unique: false
        });

        console.log("Adding sample memo");
        var sampleMemo1 = new Memo();
        sampleMemo1.title = "Welcome Memo";
        sampleMemo1.content = "This is a note taking app. Use the plus sign in the topleft corner of the main screen to add a new memo. Click a memo to edit it. All your changes are automatically saved.";

        objectStore.add(sampleMemo1);
    }
}
~~~~~~~










## Vestindo nosso app com CSS

## Adicionando o Javascript

## Testando o app no simulador