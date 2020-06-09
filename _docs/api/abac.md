---
title: Controlo de Acesso
category: API
order: 4
type: 1
---

### Descrição
  
A API está diretamente integrada com um módulo de controlo de acessos, que serve para a gestão das permissões dos utilizadores, por forma a que o acesso aos recursos da API, como Salas e Sensores, possa ser personalizado para cada um por criação de políticas. 
  
### Definição de políticas
  
No que toca à definição de uma política, **esta deve ser uma mensagem JSON** com entidades ou partes que devem ser processadas e, depois, persistidas de maneira a que, após a criação, pedidos HTTP sejam mapeados para um formato de uma **consulta** e enviados ao motor de controlo de acessos, **também formatada em JSON**, por forma a correspondê-los a políticas existentes e, consequentemente, retornar o resultado final para concessão ao acesso do recurso.
  
Na utilização do *toolkit*, é necessário considerar cinco entidades-chave em cada mensagem.
  
- **Recursos** (*Resources*) - atributos que permitem definir recursos do sistema (O que se está a ser acedido?). A sua inclusão na criação de uma política não é obrigatória: por defeito, é aplicada a qualquer recurso. É dada por uma lista de dicionários.
  
- **Sujeitos** (*Subjects*) - atributos relativos à caracterização dos utilizadores da qual tenta aceder ao recurso (Quem acede aos recursos?). A sua inclusão numa política é obrigatória, pois a identidade do indivíduo que acede a determinado recurso é crucial no sistema (como explicitado acima, todos os serviços se apoiam na autenticação com o IdP da UA, pelo servidor de OAuth). O seu valor deve ser uma lista de dicionários.
  
- **Ações** (*Actions*) - atributos que definem o que vai ser realizado com o recurso (O que se pretende fazer com o pedido de acesso aos recursos?). A criação de uma política não necessita da sua definição, sendo que, por omissão, a política engloba todas as ações.
  
- **Contexto** (*Context*) - atributos que se relacionam com as circunstâncias de acesso (Em que contexto é que esta política é satisfeita?). A sua definição não é obrigatória, sendo que, nesse caso, a política abrange pedidos de qualquer contexto.
  
- **Efeito** (*Effect*) - Se a política se destina a satisfazer o pedido pela positiva (allow) ou pela negativa (deny). Não sendo obrigatória para definir uma política, **por defeito, o efeito é dado como allow**. Definido como uma string.
  
É possível, ainda, incluir uma **Descrição** (*Description*) na política. **Recomenda-se o uso desta entidade** para se conseguir identificar as políticas de uma forma fácil e legível ao administrador, sem ter de analisar a política em si.
  
De notar que um dicionário dentro de cada parte inclui atributos, sendo que para fazer corresponder a uma consulta, todos os atributos dados devem ser satisfeitos dentro do dicionário. A lista define um conjunto em que, para o pedido satisfazer a política numa consulta, um dos atributos ou elementos desta deve ser correspondido.  
Portanto, um dicionário pode ser visto como um elemento individual de cada entidade ou parte (como um recurso nos recursos dados), em que a política é satisfeita se qualquer um elemento da lista de entidades fizer corresponder.

Assim, **cada elemento das partes** são avaliadas **OR lógico**, e os **atributos dentro dos dicionários** são avaliados posteriormente com um **AND lógico**.
	
**Observação:** Na criação de uma política, este formato interno é validado, sendo que um erro é assinalado com o motivo para a falha (é retornado um tuplo com um booleano e mensagem de erro).
  
### Definição de atributos
  
Existem os seguintes atributos para avaliação, por cada elemento da entidade, para definição numa política:
  
| **Entidade**      | **Atributo**         | Valor                                     | **Descrição**                                                   |
| :-------------: | ---------------- | ----------------------------------------- | ----------------------------------------------------------- |
| **resources**      | sensor           | String (UUID)                             | ID interno de um sensor                                     |
|                | room             | String (UUID)                             | ID interno de uma sala                                      |
|               | type             | Inteiro                                   | ID interno de um tipo de métrica                            |
| **subjects**      | email            | String                                    | Email institucional da UA                                   |
|               | admin            | Booleano                                  | Administrador do sistema                                    |
|               | student          | Booleano                                  | Estudante na UA                                             |
|               | student\_courses | Lista de inteiros                         | Códigos de unidades curriculares inscritas do estudante     |
|               | teacher          | Booleano                                  | Docente na UA                                               |
|               | teacher\_courses | Lista de inteiros                         | Códigos de unidades curriculares lecionadas pelo docente    |
| **actions**         | \-               | Lista de Strings: ‘GET’, ‘POST’, ‘DELETE’ | Métodos HTTP                                                |
| **context**      | day              | Dicionário                                | Intervalo de datas no formato ‘MM/DD/AAAA’ ou ‘AAAA/MM/DD’. |
|                   | from: String     |
|               | to:     String   |
|               | hour             | Dicionário                                | Intervalo de hora do dia no formato ‘HH:MM:SS’              |
|               | from: String     |
|               | to:     String   |
|               | ip               | String: ’internal’ ou ‘external’          | IP interno (interno apenas) ou externo (interno e externo)  |
| **effect**        | \-               | String: ‘allow’ ou ‘deny’                 | Conceder o pedido ou rejeitá-lo                             |
| **description** | \-               | String                                    | Descrição de uma política                                   |
  
### Políticas
  
#### Criação de políticas
  
Na criação de uma política, a mensagem é validada em termos de sintaxe geral de JSON.
  
No caso de ser válida, começa o processamento de cada entidade/parte da política para ser serializada e persistida como uma Policy, que é o objeto utilizado pelo framework
  
Em cada entidade desta, para cada atributo, **são utilizadas funções internas**, dadas pela ferramenta, para a criação de regras **(‘Rules’)** que serão avaliadas pelo motor aquando de um pedido que lhe origine uma consulta (ou seja, **avaliadas pelo PDP**).
  
As seguintes regras são persistidas na base de dados, por entidade:
  
| **Entidade**    | **Atributo**                   | **Regra de correspondência**                                                                           |
| :-----------: | -------------------------- | -------------------------------------------------------------------------------------------------- |
| **resources**    | sensor                     | Igualdade da String                                                                                |
|               | room                       | Igualdade da String                                                                                |
|               | type                       | Igualdade                                                                                          |
| **subjects**  | email                      | Igualdade da String                                                                                |
|             | admin                      | Veracidade / Falsidade                                                                             |
|             | student                    | Veracidade / Falsidade                                                                             |
|             | student\_courses           | Qualquer valor da lista da consulta na lista dada                                                  |
|             | teacher                    | Veracidade / Falsidade                                                                             |
|             | teacher\_courses           | Qualquer valor da lista da consulta na lista dada                                                  |
| **actions**       | \-                         | Atributo encontra-se na lista dada                                                                 |
| **context**    | day                        | Valor epoch dado em from <= Valor epoch na consulta <= Valor epoch dado em to                      |
|              | hour                       | Valor dado em from (segundos) <= Valor na consulta (segundos) <= Valor dado em to (segundos)       |
|             | ip                         | Se ‘internal’, IP dado está dentro da rede interna (avaliação CIDR de qualquer das redes internas) |
|             | Se ‘external’, qualquer IP |
  
De notar que:
- O atributo de sujeito **‘admin’** é sempre incluído na política, de forma a fazer a priorização do atributo de administrador nas políticas. Assim, qualquer política criada que não contenha este atributo não afeta os administradores, a não ser que este seja dado como Verdadeiro.
  
No final, e assumindo que não existiu qualquer erro, à mensagem JSON não processada e original é adicionada um ID interno, e esta é guardada **numa coleção de uma base de dados orientada a documentos** (MongoDB), para conseguir ser legível e limpa na vistas de controlo de acessos da plataforma de gestão. A política processada e criada como um objeto é serializada e persistida, em JSON, com o mesmo ID interno criado anteriormente, numa coleção interna utilizada pela ferramenta de controlo de acesso. 

Estas duas coleções são **sempre sincronizadas** aquando da manipulação de uma política com as funções de atualização ou remoção, chamadas a partir dos endpoints específicos para o controlo de acessos. 
  
  
#### Exemplo de uma política
  
```json
{
    'subjects': [
                    {
                        'teacher': True,
                        'courses': ['49984']
                    }
                ],
    'actions':  [
                'GET',
                'POST'
                ],
    'resources': [
                    {
                        'sensor': '144f7484-7446-4e8f-b58e-c25221904dea'
                    }
                 ],
    'context':   {
                    'hour': {
                                'from': '08:30:00',
                                'to': '18:30:00'
                            }, 
                    'ip': 'internal'
                 },
    'effect': 'allow',
    'description': 'Permitir que um docente que lecione PEI possa ler ou modificar atributos do sensor 144f das 8h30 às 18h30 dentro da UA'
}
```
  
### Consultas ao motor
  
Numa consulta, é utilizado um objeto do tipo **PDP** (implementado no módulo de controlo de acessos) para efetuar uma interrogação ao acesso a partir de um pedido HTTP. Numa chamada à função para o efeito, esta começa por obter os atributos de utilizador necessários, seguido da construção de uma **Inquiry**, que passa por fazer uma recolha e processamento de atributos correspondentes a cada uma das entidades/partes que são necessárias para definir o pedido feito do ponto de vista do motor de controlo de acessos.
  
A tabela abaixo explica a obtenção dos valores de atributos na execução de uma consulta:
  
| **Entidade**   | **Atributo**         | **Valor**                                                                                                                                                   |
| :----------: | ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **resource**    | sensor           | Valor do sensor pedido, obtido pelo caminho do endpoint, como atributo opcional na chamada ou atributo de recursos na base de dados relacional          |
|            | room             | Valor da sala pedida, obtido pelo caminho do endpoint, como atributo opcional na chamada ou atributo de recursos na base de dados relacional            |
|            | type             | Valor do tipo de métrica pedido, obtido pelo caminho do endpoint, como atributo opcional na chamada ou atributo de recursos na base de dados relacional |
| **subject**    | email            | Atributo de utilizador (Identity@UA ou cache)                                                                                                           |
|               | admin            | Atributo de utilizador (base de dados relacional)                                                                                                       |
|            | student          | Atributo processado de utilizador (Identity@UA ou cache): verificado se utilizador está inscrito em alguma UC                                           |
|            | student\_courses | Atributo de utilizador (Identity@UA ou cache)                                                                                                           |
|            | teacher          | Atributo processado de utilizador (Identity@UA ou cache): verificado se utilizador leciona alguma UC                                                    |
|            | teacher\_courses | Atributo de utilizador (Identity@UA ou cache)                                                                                                           |
| **action**      | \-               | Método HTTP do pedido                                                                                                                                   |
| **context**   | day              | Epoch da data no momento do pedido                                                                                                                      |
|            | hour             | Calculada a partir da data no momento do pedido                                                                                                         |
|            | ip               | Primeiro IP do cabeçalho de proxy X-Forwarded (IP original do pedido)                                                                                   |
