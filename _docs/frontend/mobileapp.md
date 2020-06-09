---
title: Aplicação Móvel
category: Frontend
order: 2
type: 2
---

A aplicação móvel associada ao projecto tem por objectivo principal fornecer aos utilizadores comuns acesso aos dados dos sensores instalados, de forma conveniente e rápida.
Devido à sua capacidade de criação de aplicações de forma rápida e de fácil compreensão, foi utilizada a framework [Flutter](https://flutter.dev/), recorrendo, também a vários plugins da mesma.

As principais funcionalidades implementadas foram:
-   Listagem de todas as salas monitorizadas pelo sistema
-   Disponibilização dos dados sensoriais registados numa dada sala, bem como avaliação dos mesmos
-   Recepção de alertas e configuração de quais alertas a receber
-   Fazer pedidos autenticados à API

Deste modo, o desenvolvimento da aplicação móvel pode ser dividido em três componentes fundamentais:

-   Autenticação
-   Aquisição e Apresentação de Informação Sensorial
-   Sistema de Alertas

### Componentes da Aplicação

## Autenticação

Ao entrar na aplicação, o utilizador é redireccionado para uma página de login. Foi utilizado um plugin externo para poder ser mostrado na aplicação a página do idp da universidade de aveiro, através da qual o utilizador efectua o login. O url passado a este plugin é o endpoint de autenticação da API, que quando chamado, redirecciona o utilizador para o idp. Uma vez introduzidas as credenciais, estas são validadas conforme explicado na secção referente à API.
Uma vez efectuado o login, a aplicação recebe um id de sessão que é usado para todos os pedidos subsequentes feitos à API.
Em qualquer altura, após estar autenticado, o utilizador pode fazer o logout na aplicação

| Plugins utilizados | Utilização |
| --- | --- |
| [http](https://pub.dev/packages/http) | Efectuar pedidos HTTP à API |
| [webview_flutter](https://pub.dev/packages/webview_flutter) | Apresentar a página do idp da universidade para login |
*Resumo dos Plugin utilizados na aplicação móvel para autenticação*

## Aquisição e Apresentação de Informação Sensorial

O aspecto mais importante da aplicação móvel é a possibilidade de visualizar os dados sensoriais de forma rápida e conveniente. Isto é possível a qualquer utilizador autenticado no sistema, que terá acesso aos dados dos sensores que lhe sejam permitidos.
Uma vez efectuado o login, o utilizador é levado para a página de listagem de salas. Ao iniciar, a aplicação efectua uma sequência de pedidos à API (ilustrada no Diagrama 4.15), com o objectivo de recolher informação sobre todas as salas registadas, os seus detalhes e informação sobre os sensores instalados nas mesmas.

![Sequência de aquisição dos dados das salas](/images/posts/App_AquisiçãoDeDados.png){: .center-image}

Nesta altura, é possível, também procurar uma sala específica por nome.
Por cada uma das salas listadas, o utilizador pode expandir para ter mais informações, provenientes da API, tais como uma descrição da sala e todos os sensores nela instalados.
Ao seleccionar uma sala, o utilizador é levado para a página de detalhes dos sensores, onde são listadas as informações sensoriais para cada sensor instalado. Para cada sensor, é efectuado um pedido à API, sendo recolhidos os dados do último minuto. Posteriormente, estes dados são apresentados num gráfico. É, ainda calculado o ganho em valor absoluto e percentagem. Finalmente, para cada métrica adequada, é avaliado o valor medido actualmente, num de cinco níveis, para indicar se o valor se encontra demasiado alto ou demasiado baixo.
A Figura 4.16 ilustra esta interface.

![Página de detalhes sensoriais](/images/posts/app.jpg){: .center-image}

Usando o sistema de alertas, é, também possível activar ou desactivar (encontrando-se desactivado por defeito) notificações para cada sensor.

| Endpoints relevantes | Utilização |
| --- | --- |
| /rooms | Usado para preencher a lista de salas monitorizadas a que o utilizador tem acesso |
| /room/ | Obter os detalhes de uma sala identificada por |
| /room/{id}/sensors | Obter a lista de ids de sensores para uma sala identificada por |
| /sensor/ | Obter os detalhes de um sensor identificado por |
| /sensor/{id}/measure/interval | Obtenção dos valores registados pelo sensor identificado por ao longo do último minuto |
*Endpoints relevantes utilizados pela aplicação móvel*

<br>

| Plugins utilizados | Utilização |
| --- | --- |
| [http](https://pub.dev/packages/http) | Efectuar pedidos HTTP à API |
| [typicons_flutter](https://pub.dev/packages/typicons_flutter) | Utilização de diversos ícones relevantes |
| [flutter_sparkline](https://pub.dev/packages/flutter_sparkline) | Apresentação de informação sensorial em gráficos |
*Resumo dos plugins utilizados na aplicação móvel para apresentação dos dados*

## Sistema de Alertas

A aplicação móvel também pode receber alertas provenientes do sistema. Estes alertas são de dois tipos: alertas de controlo ou de sensor. Qualquer utilizador da aplicação recebe os alertas de controlo que são enviados, por exemplo, por um administrador. No entanto, o utilizador pode escolher sobre quais sensores pretende receber notificações.
Para poder receber os alertas de sensores, a aplicação liga-se ao projecto Firebase criado para o propósito. Quando o sistema envia os alertas, estes são, também, para o projecto Firebase referido, que, por sua vez envia no tópico do respectivo sensor que é identificado pelo seu ID. Se o utilizador pretender receber notificações de um dado sensor, deve-se subscrever ao tópico do sensor que pretende. Como foi dito, todos os utilizadores estão, sempre, subscritos ao tópico de controlo. Na página de sensores de uma sala, ao lado de cada sensor, o utilizador pode activar ou desactivar os alertas do sensor em questão.
Devido à simplicidade da informação, optámos por guardar a lista de sensores subscritos na memória interna do telemóvel, num ficheiro, contendo os ids dos mesmos.
Para os dois tipos de alertas, uma vez recebidos pelo Firebase, são reenviados para a aplicação, que apresenta uma notificação Android no telemóvel.
Na figura 4.17 apresenta-se um exemplo de uma notificação de sensor e uma de controlo.

![Exemplos de notificações](/images/notificacoes.jpg){: .center-image}

| Plugins utilizados | Utilização |
| --- | --- |
| firebase_messaging | Interacção com o projecto Firebase |
| flutter_local_notifications | Apresentação dos alertas numa notificação android |
| path_provider | Identificar caminhos no sistema de ficheiros |
*Resumo dos plugins utilizados na aplicação móvel para as notificações*
