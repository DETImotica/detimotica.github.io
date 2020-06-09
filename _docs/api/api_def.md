---
title: REST API
category: API
published: true
order: 5
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

### Integração de controlo de acessos

### Algumas considerações

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

