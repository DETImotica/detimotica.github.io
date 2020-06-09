---
title: Plataforma de monitorização dos dados sensoriais
category: Frontend
order: 3
type: 2
---

### Dados e fontes de dados

Relativamente à plataforma para a monitorização e visualização dos dados provenientes dos sensores, foi escolhido o Grafana como ferramenta a ser utilizada, por ser uma solução open-source que permite uma monitorização eficaz de dados. Os primeiros passos a serem tomados para a monitorização de dados,foi a integração de dados dos sensores com o Grafana, usando como data source o InfluxDB, sendo desta forma obtidos os dados diretamente da base de dados onde são enviados os dados dos sensores.

Para a integração desta fonte de dados na plataforma, bastou apenas adicionar uma data source no Grafana, sendo necessário definir o endereço IP e porto da InfluxDB API onde são enviados os dados dos sensores, assim como, o nome da base de dados, credenciais de acesso e HTTP mode. Depois, foram criadas várias dashboards para os diferentes sensores, sendo os dados obtidos através de queries à base de dados, usando a sintaxe InfluxQL, bastante semelhante à sintaxe da linguagem SQL, inseridas na plataforma através da ferramenta Query Editor disponibilizada pelo Grafana.

Os dados podem ser formatados em modo de tabela ou timeseries, mas como o InfluxDB é uma base de dados timeseries, foi escolhido o último modo para a apresentação dos dados.

![query-editor](/images/posts/query_editor.png){: .center-image}

Após esta primeira fase, o próximo objectivo passou pela obtenção dos dados diretamente da API, em vez da base de dados, procurando-se assim a criação de uma camada de abstração, na qual a plataforma consome os dados fornecidos pelos endpoints da API e o utilizador, por sua vez, não tem que inserir queries para obter os dados sensoriais. Como tal, foi utilizado o plugin Simple JSON Datasource que permite a obtenção de dados através da execução de pedidos JSON a endereços de Backend arbitrários.

Para a plataforma obter os dados diretamente da API, foi necessário implementar endpoints específicos, que fossem de acordo com aquilo que estava especificado na documentação do plugin. Para uma melhor organização da API, foi usado o conceito de Blueprint da framework Flask, sendo uma das utilidades deste conceito, a subdivisão da API em módulos, sendo então implementado um Blueprint para os endpoints relativos ao Grafana.

Foi então necessário implementar um endpoint para o teste da conexão da plataforma com a API, no momento de adicionar o datasource no Grafana. Por outro lado, houve a necessidade de implementar dois endpoints para a obtenção de dados diretamente da API: um endpoint que devolve a lista de todas as métricas (sensores) existentes e outro que devolve um conjunto de data points juntamente com as respetivas timestamps, relativos ao(s) sensor(es) alvo e ao intervalo de tempo pedido. De notar, que para não ser feito um envio em grande escala de dados sensoriais, os dados são agrupados em intervalos de 30 segundos, sendo feita a média desses valores. Foi ainda necessário a inclusão de um endpoint para comentários sobre intervalos de tempo no gráfico, que retorna um JSON vazio, uma vez que esta funcionalidade não é utilizada no projeto.

### Dados e fontes de dados

Com a implementação dos endpoints necessários e respetivos testes para o consumo de dados através da API, a criação de dashboards e respetivos painéis revelou-se uma tarefa simples e direta. Para uma melhor organização da informação, foram criadas duas dashboards (uma para cada sala), em que cada uma contém vários painéis relativos aos sensores existentes nessa sala. Para a adição de um novo painel, basta selecionar um ou mais sensores da lista de sensores existentes, apresentados com o formato SalaX_SensorY (Metrica_Sensor), sendo esta lista obtida através do endpoint /search. Após esta seleção, os dados são automaticamente apresentados, passando os próximos passos por definir o intervalo de tempo e os aspetos visuais do painel. Um painel no Grafana apresenta vários modos de visualização como gráfico, tabela e heatmap, mas em todos os sensores optou-se por um gráfico como modo de visualização. Por fim, é possível optar pela configuração de alertas relativos ao painel, tema que irá ser abordado no subcapítulo seguinte.


![dashboard-grafana](/images/posts/dashboardgrafana.png){: .center-image}
### Alertas

A configuração de alertas no Grafana revelou-se como uma tarefa de extrema importância, pois permitiu alertar de forma quase instantânea os desenvolvedores de sistemas de IoT que utilizam o sistema, no caso de falhas ou anomalias nos seus sensores, e por outro lado, permitiu que estas servissem como trigger para o envio dinâmico de notificações push para a aplicação móvel.  

O Grafana já disponibiliza por defeito a configuração de alertas para um painel, mas o plugin que permite o consumo de dados através de uma API não é compatível com o sistema de alertas do Grafana. Por isso, houve uma necessidade de fazer alterações no plugin disponibilizado, passando a solução por transformá-lo num Backend Plugin para permitir o suporte do sistema de alertas, sendo usado o trabalho desenvolvido numa branch do repositório GitHub do plugin como referência.

Após a incorporação do sistema de alertas no plugin, foi necessário configurar os canais de notificações, podendo estas notificações serem enviadas para o Email, Discord, Slack, Webhooks, entre outros sistemas que se encontram listados no site do Grafana. Foi precisamente com a configuração de um canal do tipo webhook que é feito o envio dinâmico de notificações push para a aplicação móvel. Para isso, foi necessário implementar um endpoint na API que funciona como um Webhook Listener, fazendo a leitura do conteúdo enviado nos alertas do Grafana e transformando esse conteúdo numa notificação FirebaseCM que é enviada para um tópico específico relativo ao sensor em questão.

O próximo passo, passou pela configuração nos painéis dos alertas, podendo a lista de alertas configurados e o seu estado ser consultado na tab de Alerting, em Alert Rules. Os alertas no Grafana funcionam como uma máquina de estados sendo o estado normal o de OK, e no caso de violação de uma regra definida pelo utilizador, existe uma transição do estado de OK para Pending. Neste estado não é enviada logo a notificação, sendo a regra avaliada constantemente e no caso desta continuar a ser violada durante um intervalo de tempo definido pelo utilizador, é então feita a transição do estado Pending para Alerting, sendo enviada a notificação. Outro estado existente é o No Data, sendo feita a transição automática para este estado, no caso de não haver dados, ou estes serem todos nulos num painel. A configuração passa por definir uma ou mais condições que são avaliadas a cada intervalo de tempo definido, assim como um intervalo de tempo na qual esta deve estar no estado de Pending. São também definidos os canais de notificações a ser usados e o conteúdo da mensagem.

![grafana-alerts](/images/posts/grafana_alerts.png){: .center-image}

### Login e acesso

A integração da autenticação na API com o Grafana foi realizada com recurso à funcionalidade de autenticação por um proxy de autenticação (Auth Proxy) que o serviço de dashboards disponibiliza.
Esta integração utiliza as capacidades de um proxy Nginx que expõe todos os serviços web internos existentes no servidor e um endpoint especial, “/wsauthverify”, que associa uma resposta de validação de sessão por parte da API para a localização do proxy correspondente:

- O utilizador entra na localização que corresponde a um acesso ao servidor do Grafana. Este verifica o estado da sua sessão, com recurso ao endpoint de verificação. No caso deste já se encontrar autenticado na API, então este é, novamente, redirecionado automaticamente para a página inicial associada às dashboards.

- No caso de não estar autenticado, este é redirecionado para o endpoint de autenticação. O processo é efetuado e, no final, como já referido na secção de sessão autenticada, na explicitação dos endpoints da API, uma cookie especial de sessão é criada para o efeito. A API, por fim, redireciona-o para a página principal das dashboards.

De notar que o acesso à plataforma de dashboards é apenas admitida a administradores.
