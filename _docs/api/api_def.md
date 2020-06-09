---
title: REST API
category: API
published: true
order: 3
type: 1
---

### Introdução

Relativamente à API, esta surge no sistema como a interface da camada de sensores, possuindo então um conjunto de endpoints desenvolvidos com recurso à framework Flask, que permitem que os programadores e utilizadores finais acedam aos dados dos mesmos, e ainda que possam interagir com esta, por exemplo, criando novas salas, atribuindo sensores a espaços ou alargando o número de ações que dados utilizadores podem realizar . 

Todos os endpoints são autenticados, isto é, apenas utilizadores que pertencem à Universidade de Aveiro são capazes de usufruir do sistema. Para isso, foi feita uma integração de um servidor de autorização, implementando por cima desta serviços de cache de identificação do utilizador e validação das sessões, que permitem também a autenticação do sujeito que concedeu a autorização.

Para além disso, a API está diretamente integrada com o módulo de controlo de acessos, que serve para a gestão das permissões dos utilizadores, por forma a que o acesso aos recursos da API, como Salas e Sensores, possa ser personalizado para cada um por criação de políticas.

A API conecta-se também a uma Base de Dados auxiliar para garantir a persistência da meta-informação associada aos dados do sistema, e ainda guardar as relações entre os mesmos .

Por fim, a API é configurada por ficheiros para o efeito, e é executada a partir de um Web Server Gateway Interface (WSGI), que a expõe e intermedia todos os pedidos a esta assincronamente, garantindo uma performance significativamente maior do que se fosse executada diretamente. A configuração do WSGI garante, ainda, segurança nas comunicações ao nível dos pedidos.


### Desenho da API

Especificando os endpoints desta solução, durante a sua definição adotou-se o estilo de desenho de APIs REST: para tal, mapeou-se cada um dos principais recursos e as suas operações num conjunto de endpoints que tem sempre como Base o nome do recurso (Ex: /<nome_recurso>).

Posteriormente, fez-se uso das opções de ação nativas nos requests do protocolo HTTP/HTTPS para diferenciar, num mesmo endpoint, o resultado que é esperado da sua execução, deixando deste modo a API simples e intuitiva de utilizar. Resumo nas Tabelas abaixo.

| Recurso | Endpoint Base Path |
| --- | --- |
| Salas | /room |
| Sensores | /sensor |
| Tipos de Sensores | /type |
| Utilizadores  | /user |
| Políticas de Acesso | /accessPolicy |

<i>Resumo dos recursos na API</i>


| Ação | Resultado Esperado |
| --- | --- |
| GET | devolva valores de um recurso |
| POST | modificação (alteração dos campos do recurso) |
| DELETE | apague um dado recurso |

<i>Resumo das ações na API</i>

<br/>
Exemplos:

[POST] /sensor - criar um novo sensor

[POST] /sensor/"id" - modificar um sensor com o id "id" já existente

Além disto cada recurso pode ainda possuir Opções ou Recursos Internos: nestes dois casos, decidiu-se que deve ser também indicada a opção, após a escolha do recurso ( [ / recurso_base ] [ / opção/recurso_secundário ] ).

Exemplos:

[GET] /sensor/"id"/measure/"tipo_medição" - obter os valores de um sensor "id" segundo a forma especificada dentro da opção "tipo_medição"

[GET] /room/"id"/sensors - obter os sensores existentes na sala "id"

Para terminar ainda se implementou alguns endpoints diretos, que participam na autenticação dos utilizadores na API. Estes são descritos mais abaixo.


### Autorização e autenticação

Todos os pedidos de acesso a recursos pelos endpoints da API são autenticados.
  
Para isto, é necessária a definição de endpoints especiais de sessão que consigam realizar um mecanismo capaz de autenticar o utilizador final. Adicionalmente, a definição de uma API com controlo de acessos por atributos requer que exista recolha de atributos relacionados com o utilizador que, depois, sejam utilizados para definir políticas que controlem finamente os recursos existentes.
  
Com isto, foi utilizado o Identity@UA, que é um servidor OAuth (Open Authorization), versão 1.0a, que integra a autenticação dos utilizadores associados à Universidade de Aveiro pelo IdP (Identity Provider). Isto permite que a API tenha, a partir de um token de acesso, permissão por parte do end-user para recolher atributos de identidade.
  
Por si só, este método não faz qualquer autenticação - para o efeito, uma sessão é criada (a partir de uma cookie de sessão), internamente mapeada para um token de acesso, que fica em cache, e validada sempre que o utilizador fizer um pedido à API.
  
Para mais informação acerca da implementação do servidor OAuth, por favor, ver a [página](https://identity.ua.pt/oauth) do Identity@UA e a [documentação](http://api.web.ua.pt/pt/services/universidade_de_aveiro/oauth) do serviço de OAuth.
  
#### Processo de autenticação na API
  
O utilizador inicia a sessão chamando o endpoint associado para o efeito **(/login)**. A API faz um pré-processamento para averiguar se o utilizador já está autenticado com sucesso no sistema. No caso de estar, então o processo é finalizado diretamente com a resposta de redirecionamento ou de sucesso final.
  

**Nota:** Dependendo com que propósito foi chamado, o seu URL poderá ter parâmetros **app** (ID de aplicação) e **redirect_url** (página de redirecionamento após autenticação bem sucedida). Se estes existirem, o primeiro passo consiste em incluí-los na cookie de sessão. Esta fase aplica-se à autenticação dentro da plataforma de gestão e do serviço de dashboards, que diferem do passo final.
  
  
Neste ponto, a API começa o processo OAuth, pede o token de pedido e a chave correspondente, e redireciona o utilizador para o IdP da Universidade.

De seguida, o utilizador introduz as suas credenciais, valida-as e responde ao consentimento para permissão de utilização de dados pela API, enviado pelo servidor de OAuth.

Este, seguidamente, comunica à API pelo endpoint de callback, configurado aquando do registo inicial da API no servidor (/auth_callback). 
  
No processo deste endpoint, é verificado se o utilizador consentiu os dados.
  
- Se respondeu negativamente no ecrã de consentimento, o URL que o servidor OAuth envia à API deve conter o parâmetro **consent=no**. Neste caso, o processo de login **aborta** e envia uma resposta ao utilizador que o avisa que o processo falhou, informando-o que não consentiu a permissão.
  
- No caso de ter aceite o consentimento, é feito um pedido ao access token com o token de pedido e a chave passada no início do processo. O servidor retorna o token de acesso e chave.
  
Com este token, de seguida, é feito um pedido de todos os atributos do utilizador, seleciona-se e processa-se os que são relevantes para a definição do conjunto de atributos para efeitos de controlo de acesso do utilizador, e guarda-se o resultado em cache de dados local.

Os atributos básicos que identificam o utilizador são guardados na cookie da sessão, e internamente mapeados com estas (access token e chave) noutra cache local da sessão.
  
Ainda falta persistir dados básicos de utilizador na base de dados relacional de atributos para haver um registo de utilizadores que, no passado, utilizaram a plataforma, e conseguir admitir administradores do sistema.

Assim sendo, se o utilizador ainda não existir na base de dados, este é inserido com as informações básicas (email e ID), sem o papel de administrador.
  
Por fim, para concluir o processo de login com sucesso, é necessário responder de acordo com o início do processo:
  
- Se foi identificado que a autenticação veio da plataforma de gestão, então é necessário passar a string da cookie de sessão pelo URL como parâmetro. Como a plataforma de gestão apenas se aplica a administradores, isto é verificado. 
  
    - Se sim, faz-se um redirecionamento para o URL que se encontra na sessão, como o valor de “redirect_url”. Se não, à localização é juntado o caminho para uma página de erro “Forbidden”.
  
    - Se foi verificado que este início de sessão veio de uma chamada ao endpoint pela entrada numa das páginas que correspondem ao caminho do serviço de dashboards, então é feita uma cookie especial para o efeito, sendo que a cookie de sessão original é a do servidor que expõe esse serviço. Por fim, a resposta redireciona para a página inicial das dashboards.
  
- Se veio de outro tipo de chamada (direta do endpoint no browser ou na aplicação), então o endpoint responde com sucesso, em que inclui nessa resposta os dados básicos do utilizador.
  
![login_flow](/images/login_flow.png?raw=true "Flowchart do processo de autenticação"){: .center-image}
  
#### Processo de desautenticação
  
O processo de logout inicia quando o utilizador final chama o endpoint **/logout**. Primeiro, é verificado se este não se encontra autenticado.
  
- No caso de se encontrar, é informado da situação, e o processo termina.
  
No caso de não estar autenticado, então o processo de desautenticação começa:
  
- Elimina-se as entradas relacionadas com o utilizador da cache de sessão, expirando assim o token de acesso (do sistema interno).
  
- Elimina-se da cache de atributos todos os dados relacionados com o indivíduo autenticado
  
- Limpa-se a sessão
  
- Se existir uma outra cookie associada ao processo de criação da sessão autenticada, esta é expirada.
  
- Por fim, é retornada uma resposta de desautenticação bem sucedida.
  
  
![logout_flow](/images/logout_flow.png?raw=true "Flowchart do processo de desautenticação"){: .center-image}
  
### Integração de controlo de acessos
  
A API combina regras internas pré-definidas de acesso administrativo a endpoints e a utilização um submódulo de controlo de acessos baseado em atributos (ABAC), desenvolvido em Python.

Este utiliza uma framework denominada Vakt, que define um conjunto de funcionalidades capazes de fornecer uma solução de controlo de acesso por políticas baseada em dicionários Python e estruturas do tipo JSON, que unifica o desenvolvimento da API no que toca às mensagens transmitidas em todo o fluxo de dados com as componentes de frontend.
  
  
Mais informação pode ser consultada [aqui](../abac)
  

#### Regras internas de acesso
  
A especificação de endpoints da API reúne um conjunto de pressupostos que são internos e que definem, à partida, o acesso que deve ser dado em endpoints.
  
- No que toca a endpoints que manipulam internamente recursos como utilizadores, sensores, salas e tipos de métricas, são definidos logo à partida como endpoints apenas para administradores: antes de processar o pedido HTTP para um destes endpoints, é verificado, na base de dados interna de atributos, se o utilizador que o fez é administrador. 
  
    - No caso de não o ser, é-lhe respondido que o acesso ao endpoint é interdito, ou seja, é-lhe enviada a resposta HTTP ao pedido efetuado com o código de estado 403 (Forbidden).
      
    - Assim, é feito um pré-processamento leve deste pedido, ao invés de processar o endpoint, tratar erros expectáveis a falhar e pedidos desnecessários à camada de controlo. Portanto, o estabelecimento desta regra interna na API é justificável.
      
- Outra regra define, de uma maneira semelhante, que um utilizador só consegue aceder a qualquer endpoint se estiver autenticado, com exceção aos endpoints relacionados com o estabelecimento dessa sessão.
  
- Dentro do motor de controlo de acessos, denominado PolicyManager, na primeira inicialização, é criada automaticamente uma política que define que administradores têm acesso a toda a API, se esta não existir.
      
    - Isto é necessário porque o PDP (Policy Decision Point) - nome da classe implementada para o cliente efetuar consultas - rejeita todos os pedidos por omissão. Como, dentro dos utilizadores da plataforma, os administradores devem ter a mais alta prioridade, esta política deve estar sempre criada.
      
    - Depois, é possível retirar acesso a outros administradores com a criação de outras políticas para fazer a restrição necessária.
  

#### Algumas considerações
  
Em termos de endpoints na API, averiguou-se algumas considerações:
  
- Um utilizador que tenha acesso a um sensor específico, também tem acesso à sala e ao tipo de métrica desse sensor
  
- Um utilizador que tenha acesso a uma sala específica, tem acesso a todos os sensores da mesma sala
  
- Um utilizador que tenha acesso a um tipo de métrica específico, tem acesso a todos os sensores do mesmo tipo
  
Por isto, é necessário coordenar a classe PDP do módulo de controlo de acessos com implementações dos endpoints na API para que a análise de políticas na consulta faça corresponder estas regras:
  
- Um endpoint que liste salas ou tipos ou retorne metadados de uma sala ou tipo tem de verificar se o utilizador tem acesso a algum sensor correspondente
  
- Endpoints relacionados com a obtenção de dados dos sensores têm sempre de incluir o tipo e a sala

### Resumo das Funcionalidades

Nesta seção, é sucintamente descrito o que cada endpoint é esperado que faça. Para efeitos de simplificação, não se indica que ação do protocolo http/https deve ser usada, nem quais são os parâmetros necessários no corpo do pedido, nem quais são as possíveis formas de erros e seus códigos.
Caso seja necessário e precise de procurar saber mais acerca destes detalhes, é possível recorrer à especificação em Swagger (Documentação da API).

#### Rooms

- /rooms - Devolve todas as salas a que os utilizadores da API têm acesso

- /room - Serve para um administrador poder criar uma nova sala na API com os detalhes especificados

- /room/< ID > -  Serve para que um utilizador possa gerir/aceder aos detalhes de uma determinada sala com o identificador interno “ID”; <br/>
<i>Ações Possíveis:</i> Ver/Alterar os detalhes da Sala, Apagar a Sala

- /room/< ID >/sensors (/full)* - Serve para que um utilizador possa aceder/gerir os sensores associados a uma sala com o identificador interno “ID”; <br/>
<i>Ações Possíveis:</i> Ver/Adicionar/Remover Sensores da Sala <br/>
(/full) - Opção opcional que apenas serve para que o endpoint devolva toda a informação de cada sensor

#### Sensors

- /sensors - Devolve todos os sensores a que os utilizadores da API têm acesso

- /sensor -  Serve para um administrador criar um novo sensor na API com os detalhes especificados

- /sensor/< ID > (/key)* - Serve para que os utilizadores possam gerir/aceder aos detalhes de um determinado sensor com o identificador interno “ID";<br/> 
<i>Ações Possíveis:</i> Ver/Alterar os detalhes do Sensor, Apagar o Sensor<br/>
(/key) - Opção opcional para o endpoint devolver a chave de encriptação do sensor

- /sensor/< ID >/measure/< Opção > - Serve para que os utilizadores possam aceder às medições de um sensor com o identificador interno “ID” segundo uma determinada “Opção”;<br/>
<i>Ações Possíveis:</i> Obter a última medição, Obter todas as medições num dado intervalo, Obter a média das medições num intervalo 

#### Types

- /types - Devolve todos os tipos de sensores a que os utilizadores da API têm acesso

- /type -  Serve para os administradores criarem um novo tipo de sensores na API com os detalhes especificados

- /type/< ID > - Serve para que os utilizadores possam gerir/aceder aos detalhes de um determinado tipo de sensores com o identificador interno “ID;<br/>
<i>Ações Possíveis:</i> Ver/Alterar os detalhes do Tipo de Sensor, Apagar o Tipo de Sensor

#### Users

- /users (/full)* - Devolve todos os Utilizadores da API, apenas aos Administradores <br/>
(/full) - Opção opcional que apenas serve para que o endpoint devolva toda a informação de cada Utilizador

- /user/< ID > - Serve para que os utilizadores possam gerir/aceder aos detalhes de um determinado Utilizador da API com o identificador interno “ID; <br/>
<i>Ações Possíveis:</i> Ver detalhes do Utilizador, Atribuir/Retirar o papel de Administrador, Remover Utilizador

#### Access Policies

- /accessPolicies - Devolve todos as políticas de acesso aos recursos definidas na API, apenas aos Administradores

- /accessPolicy -  Serve para os administradores criarem uma nova política de acesso com base nas condições enviadas<br/>
<i>Ações Possíveis:</i> Combinação de Restrições: a nível de um grupo/um utilizador ; a nível de um determinado recurso/conjunto de recursos ; a nível de um determinado contexto (ex. horas) ; a nível da ação que vai ser realizada no recurso

- /accessPolicy/< ID > - Serve para que os utilizadores possam gerir uma determinada política de acesso com o identificador interno “ID”;<br/> 
<i>Ações Possíveis:</i> Substituir a política existente por uma nova ; Remover a política de acesso

