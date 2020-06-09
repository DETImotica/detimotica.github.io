---
title: Plataforma de Gestão
category: Frontend
order: 1
type: 2
---

Destinada apenas a administradores, a Plataforma de Gestão exemplifica algumas das inúmeras variantes possibilitadas pela API. Desde uma simples visualização das salas disponíveis, passando pela criação de tipos de métricas e pelo sistema de notificações para os utilizadores da Aplicação Móvel, finalizando na gestão das políticas de acesso aos diversos sensores, entre outros, a Plataforma de Gestão permite aceder e controlar a maioria da informação gerada.
<br>A framework Django foi a escolhida, pelo facto de possibilitar a interligação do código desenvolvido em Python com a construção da página através de HTML, CSS e JavaScript. A passagem de parâmetros obtidos nos requests feitos pela biblioteca do Python para o próprio documento HTML foi um dos passos essenciais para desenvolver a Plataforma de Gestão com sucesso.

### Páginas da Plataforma

## Página Inicial

Após ser autenticado com êxito na Plataforma de Gestão, o utilizador será redirecionado para a Página Inicial. Nesta, poderá visualizar os cinco atalhos para as secções referidas em seguida, bem como quatro referências para: Grafana, Documentação da API, Microsite desenvolvido para o projeto e o repositório do GitHub com o mesmo.

![Exemplo da Pagina Inicial da Plataforma.](/images/plataformaGestao/paginaInicial.png?raw=true?style=centerme "Página Inicial")
<br>*Exemplo da Página Inicial da Plataforma.*

## Salas

A página denominada “Salas” lista todas as salas disponíveis numa tabela onde é possível ver o UUID atribuído à divisão (gerado do lado do servidor), bem como a sua designação e descrição. Os UUIDs das divisões são obtidos recorrendo ao endpoint [GET ] /rooms, com os metadados de cada sala a resultarem do [GET] /room/<UUID>.
<br>É possível criar novas salas e editar os metadados da mesma. Clicando no botão “Adicionar”, preenchendo os campos “Nome” e “Descrição” e pressionando o ícone com a checkmark, o endpoint [POST] /room será chamado, contendo um JSON com os metadados fornecidos pelo utilizador e gerando um UUID que será atribuído àquela sala em específico. A tarefa de alterar as informações de uma sala em específico é bastante semelhante, recorrendo no entanto ao [POST] /room/<UUID> do respetivo repartimento.
<br>Para além disso, existe ainda a hipótese de extinguir salas listadas, bastando para isso clicar no ícone vermelho que ativará o [DELETE] /room/<UUID>.

![Exemplo da criacao de um Sala na Plataforma de Gestao.](/images/plataformaGestao/criacaoSala.png?raw=true?style=centerme "Criação de uma Sala")
<br>*Exemplo da criação de um Sala na Plataforma de Gestão.*

## Sensores

Na secção sensores começa por ser apresentada uma lista das salas disponíveis, contendo apenas a nomenclatura de cada uma. Esta escolha baseia-se no facto dos sensores estarem organizados por divisões. À semelhança da lista de salas referida anteriormente, são obtidos os UUIDs de cada sala através do [GET] /rooms, sendo depois efetuado um novo [GET] /room/<UUID> para cada UUID obtido e apresentado o “Nome” associado ao mesmo.

![Exemplo da primeira pagina apresentada na seccao Sensores.](/images/plataformaGestao/sensorsFirst.png?raw=true?style=centerme "Primeira Página dos Sensores")
<br>*Exemplo da primeira página apresentada na secção Sensores.*

Ao clicar no compartimento que pretende, será apresentada uma nova página onde é possível visualizar todas as informações disponíveis da própria divisão, bem como dos sensores contidos na mesma. É ainda possível efetuar diversas ações sobre os mesmos.
<br>Esta segunda página recebe o UUID da sala em forma de parâmetro e um [GET] /room/<UUID> devolve os metadados da mesma, permitindo assim apresentar os detalhes da divisão. Para além do [POST] /room/<UUID> com as informações transmitidas num JSON para editar a sala e do [DELETE] /room/<UUID> para apagar a mesma, é ainda possível aceder à página das políticas de acesso do compartimento em si. Uma generalização da gestão das políticas de acesso pode ser encontrada aqui.
<br>Já os sensores, são apresentados numa tabela que possui o seu UUID, o “Tipo de Métrica”, as “Unidades” relativas ao Tipo de Métrica e a “Descrição” associada ao aparelho. Primeiramente, é efetuado um [GET] /room/<UUID>/sensors que retorna uma lista com todos os UUIDs dos dispositivos associados à sala em questão. Posto isso, é efetuado um [GET] /sensor/<UUID> para cada sensor da lista obtida que devolve então as informações necessárias para preencher a tabela.
<br>Analogamente ao que acontece na tabela das salas, é permitido adicionar, alterar e remover um sensor. O primeiro ocorre através do [POST] /sensor e tem o UUID gerado de forma automática, com os detalhes a serem novamente transmitidos no formato JSON, tal como no [POST] /sensor/<UUID> em que se alteram os metadados de um sensor. Para remover algum dos aparelhos listados, é chamado o endpoint [DELETE] /sensor/<UUID>, removendo então o sensor com sucesso.
<br>Um dos campos de preenchimento obrigatório na criação e/ou alteração de um dispositivo é o “Tipo de Métrica”. Esse campo é apresentado por um dropdown que contém todas as métricas disponíveis, métricas essas carregadas através do [GET] /types, seguido de um [GET] /type/<ID> para cada tipo de métrica.

![Exemplo da criacao de um Sensor na Plataforma de Gestao.](/images/plataformaGestao/criacaoSensor.png?raw=true?style=centerme "Criação de um Sensor")
<br>*Exemplo da criação de um Sensor na Plataforma de Gestão.*

Na figura acima, é possível visualizar três ações extras que não se encontram na tabela das salas, sendo que a terceira está associada à página das políticas de acesso do sensor. A primeira, representada por um ícone de uma notificação, permite enviar uma mensagem composta por “Assunto” e “Corpo”. Essa mensagem será recebida por todos os utilizadores da Aplicação Móvel que subscreveram o sensor em questão. Ao contrário dos outros endpoints que efetuam ações do tipo [POST], para além das componentes da mensagem, também o UUID do sensor é enviado no JSON no corpo do [POST] /mobile/notifications.
<br>A ação seguinte é relativa à chave de encriptação, revelando a mesma ao pressionar o ícone. Para obter a chave de encriptação AES-128, é chamado o endpoint [GET] /sensor/<UUID>/key.

![Exemplo do formulario para enviar uma notificacao e da chave de encriptacao de um Sensor.](/images/plataformaGestao/notifySubscribers.png?raw=true?style=centerme "Notificar Subscritores")![Exemplo do formulario para enviar uma notificacao e da chave de encriptacao de um Sensor.](/images/plataformaGestao/key.png?raw=true?style=centerme "Notificar Subscritores")
<br>*Exemplo do formulário para enviar uma notificação e da chave de encriptação de um Sensor.*

## Tipos de Métricas

Relativamente aos tipos de métricas, o processo é semelhante ao efetuado na página das salas. É primeiramente efetuado um [GET] /types que devolve uma array com os IDs de todos os tipos de métricas disponíveis. Percorrendo todos os elementos da lista, são efetuados diversos [GET] /type/<ID> para obter os metadados de cada tipo de métrica, nomeadamente o seu nome, descrição e unidades.
<br>É então construída uma tabela com os metadados obtidos, sendo permitido adicionar, alterar ou ainda apagar um tipo de métrica na mesma. Os campos preenchidos ao criar ou alterar uma métrica são passados através de um JSON nos endpoints [POST] /types e /type/<ID>, respetivamente, com o primeiro a obter um ID criado automaticamente. De notar que o campo das unidades não é passível de alterações diretas, pois este é completo pelo conjunto de unidades de todos os sensores referentes àquele tipo. Ao clicar no ícone de remoção de um tipo, acionará o [DELETE] /type/<ID>, excluindo o mesmo.
<br>Tal como nos sensores e nas divisões, é também possível criar diferentes políticas de acesso para cada tipo de métrica. O terceiro e último ícone redireciona o utilizador para a página de gestão das mesmas.

![Exemplo da adicao de um novo Tipo de Sensor na Plataforma de Gestao.](/images/plataformaGestao/criacaoType.png?raw=true?style=centerme "Criação de um Tipo de Sensor")
<br>*Exemplo da adição de um novo Tipo de Sensor na Plataforma de Gestão.*

## Políticas de Controlo de Acessos

Como referido na secção dos Sensores e dos Tipos de Métricas, é possível definir e gerir políticas de acesso para salas, sensores e tipos de métricas. Primeiramente, é executado um [GET] /<room/sensor/type/>/<UUID/ID> para obter os metadados do alvo a que queremos aplicar ou modificar uma ou várias políticas de acesso. É assim possível exibir o ID no caso dos sensores e a nomenclatura no caso das salas e tipos de métrica no início da página.
<br>Para obter todas as políticas de acesso associadas a um determinado objeto (sala, sensor ou tipo de métrica), é convocado o endpoint [GET] /accessPolicies com o objeto e o UUID ou ID pretendidos a serem transmitidos por JSON, filtrando assim as políticas relevantes.
<br>Não destoando das restantes páginas, as ações de adicionar e de alterar uma política de acesso utilizam o [POST] /accessPolicy e [POST] /accessPolicy/<UUID>, com os detalhes das mesmas no corpo do pedido. No primeiro caso, um UUID é atribuído à política no momento da sua criação. Já para remover uma restrição de acesso, o [DELETE] /accessPolicy/<UUID> é o endpoint designado para a tarefa.

![Exemplo de uma pagina de gestão das Politicas de Controlo de Acessos.](/images/plataformaGestao/criacaoType.png?raw=true?style=centerme "Página das Políticas de Controlo de Acessos")
<br>*Exemplo de uma página de gestão das Políticas de Controlo de Acessos.*

O formulário para restringir ou alargar o acesso de uma política está dividido em cinco áreas: “Sujeitos”, “Contexto”, “Descrição”, “Efeito” e “Métodos”. O último é referente ao tipo de pedidos que serão alvos da política, podendo estes ser GET, POST e/ou DELETE. O “Efeito” tem apenas duas opções, sendo estas a de permissão ou proibição, caso queira alargar ou restringir o acesso, respetivamente.
<br>A secção do “Contexto” é composta por três campos, sendo um deles o “IP”, com as hipóteses “Interno” e “Externo”. Ao escolher a segunda, não existirá qualquer restrição a nível de endereços visto serem permitidos IPs externos. Já a opção “Interno”, bloqueará qualquer IP exterior à rede da Universidade de Aveiro, sendo por isso necessário estar presente na rede da mesma, seja presencialmente ou utilizando, por exemplo, a VPN da Universidade de Aveiro. 
<br>Os outros dois campos são o “Dia” e a “Hora”, sendo possível combinar os dois para impedir ou permitir diariamente, durante o intervalo de dias estabelecido, o acesso à sala/sensor/tipo de métrica em causa, ao longo das horas pré-definidas. De notar que as horas determinadas estarão no fuso horário UTC, e que, o final do intervalo de dias é exclusivo.
<br>No que diz respeito aos “Sujeitos”, é possível especificar os mesmos de até quatro maneiras diferentes. A primeira opção, passa por determinar se os “Administradores” devem ou não ser alvos da política pretendida. As duas seguintes, “Docentes” e “Estudantes”, quando ativas, bloqueiam ou permitem todos os docentes e estudantes por default. É, no entanto, possível especificar a(s) cadeira(s) que queremos no escopo da política, introduzindo o(s) código(s) da(s) mesma(s) na caixa de texto respetiva. Já o campo “Email” permite associar emails à política, vinculando assim indivíduos e não grupos à mesma.

![Exemplo de uma Politica de Controlo de Acesso na Plataforma de Gestao.](/images/plataformaGestao/createPolicy.png?raw=true?style=centerme "Criação de uma Política de Controlo de Acessos")
<br>*Exemplo de uma Política de Controlo de Acesso na Plataforma de Gestão.*

## Utilizadores

Qualquer utilizador que aceda ao endpoint /login ficará registado. É possível fazê-lo diretamente, entrando na Aplicação Móvel, ou tentando ingressar na Plataforma de Gestão e/ou Grafana, com estes dois últimos a recusarem a autenticação de qualquer pessoa que não tenha o estatuto de administrador. 
<br>Para obter uma lista de todos os utilizadores que já efetuaram o login, bem como as suas informações, recorreu-se ao [GET] /users/full. Estas informações contêm o email e um booleano denominado “admin” que declaram se o utilizador em questão é ou não um administrador do sistema. Para além do “ID”, “Email” e do “Admin”, a tabela presente nesta página é composta ainda por uma coluna com um botão que permite alternar o valor do booleano.
<br>É assim possível visualizar todos os utilizadores que têm acesso à plataforma visto esta estar apenas disponível a administradores.

![Exemplo da pagina Utilizadores da Plataforma de Gestao.](/images/plataformaGestao/usersPage.png?raw=true?style=centerme "Página de Utilizadores")
<br>*Exemplo da página Utilizadores da Plataforma de Gestão.*

## Notificações

Para além das notificações referentes a um sensor em específico, o sistema de notificações possui uma página reserva apenas para o mesmo. Nela é possível preencher dois campos, o “Assunto” e o “Corpo”, tal como nas notificações dos sensores, mas existindo uma grande diferença: esta página não tem qualquer UUID ou ID associado. Sendo assim, a notificação é enviada através do [POST] /mobile/notifications com o seu JSON a conter apenas as duas secções da mensagem.
<br>Todos os utilizadores da Aplicação Móvel receberão, por isso, quaisquer notificações enviadas a partir desta página, sendo idealizada para alertas e informações gerais.

![Exemplo da pagina de Notificacoes da Plataforma de Gestao.](/images/plataformaGestao/notifications.png?raw=true?style=centerme "Página de Utilizadores")
<br>*Exemplo da página de Notificações da Plataforma de Gestão.*