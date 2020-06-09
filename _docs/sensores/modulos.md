---
title: Módulos Sensoriais
category: Sensores
order: 1
type: 1
published: true
---

<br>

![Alt text](/images/posts/Sensores_diagrama.png?raw=true "Title")

<br>

### Microcontrolador

Dadas as suas capacidades de input/output, comutação, e memória e também o facto de existirem unidades disponíveis para requisição no armazém de componentes do DETI MakerLab, o microcontrolador escolhido para o desenvolvimento deste projeto foi o LoPy versão 1:

- Baseado no Espressif ESP-32
- Dispõe de Wi-Fi e Bluetooth integrados
- Programável em Micro-Python
- Suporta aplicações multi-threaded
- Possui vários ADCs e I2C Bus

A release de firmware utilizada foi a ‘1.18.2’, com a versão ‘v1.8.6-849-07a52e4’.

No entanto, estudantes e docentes do departamento poderão utilizar qualquer microcontrolador para efetuar instalações em novas salas no futuro, bastando para isso seguirem a especificação contida nas secções seguintes deste documento.

### Sensores instalados

Em cada montagem foram instalados os seguintes sensores:

|  Sensor  |  Métrica  |  Precisão  |  Comunicação  |  Alimentação  |   |
| --- | --- | --- | --- | --- | --- |
|  LMV324  (SEN-12642)  |  Som (dB)  |  ---  |  Medições Analógicas  |  5V  |   |
|  TSL2561  |  Luminosidade (lux)  |  ---  |  I2C  |  3.3V  |   |
|        BME860  |  Temperatura (ºC)  |  +- 1ºC  |        I2C  |        3.3V  |  |
|  |Humidade (%RH)  |  +- 3%RH  |  |
|  |Pressão (hPa)  |  +- 1hPa  |  |
|  |Indoor Air Quality (Índice - Gases VOC)  |  ---  |  |

*Listagem de sensores requisitados e instalados*

Adicionalmente, é registado ainda o número de dispositivos BLE visíveis dentro do alcance do módulo Bluetooth integrado no LoPy. 

Durante o desenvolvimento foi também instalado e programado com sucesso um sensor de temperatura e humidade DHT-11.

Foi igualmente preparado código-fonte para um sensor de concentração de gás Dióxido de Carbono MG812, que não foi possível instalar.

No entanto, a plataforma é dinâmica e não restringe os tipos de sensores que podem ser utilizados, pelo que cada montagem futura poderá possuir um conjunto distinto de sensores.

### Montagem

As duas montagens exemplo preparadas ao longo do desenvolvimento do projeto foram realizadas de acordo com o seguinte diagrama:

![Alt text](/images/posts/montagem.png?raw=true "Title")
*Diagrama de montagem dos módulos de sensorização integrados*

De notar a necessidade de utilização de uma pequena placa de alimentação externa para o(s) sensor(es) que necessitem de input de 5V não fornecido pelo microcontrolador. Por conseguinte, é importante que os sinais de output desses sensores sejam sujeitos a um divisor de tensão de forma a limitar a voltagem a 3.3V e proteger o LoPy. 

### Configuração e Utilização

O código-fonte de um módulo sensorial é dividido em duas partes lógicas:

- Inicialização, conexão, e envio de mensagens
- Lógica inerente a cada sensor individual

A listagem e configuração dos sensores é realizada no ficheiro auxiliar ‘conf.json’.

Cada sensor instalado poderá disponibilizar várias métricas ao sistema, sendo que todas necessitam de ser registadas através de pedido à equipa de administração (que utiliza a plataforma de gestão para esse efeito).

Após o registo é fornecido, para cada métrica, um par ‘ID / Chave de encriptação’ que deverá ser introduzido no ficheiro supracitado conforme o exemplo seguinte.

```
{
    "isu_id": "cc2f8ad2-e719-4366-a157-5463c9097dfc", // UUID do módulo
    "modules": [
    {
      "name": "tsl2561",
      "active": true,
      "wait_time": 6000, // em milisegundos
      "metrics": {
        "lux": {         // nome arbitrário - denomina uma métrica
             "id": "bb60686d-614a-43af-bf51-cc41477326d9", // UUID da métrica
             "key": "ck5PZQLoy0DleIYQkoRvkw==" // Chave AES-128
          },
         // ...
      }
    }, 
    // ...
}
```
*Exemplo de configuração do ficheiro conf.json*

Após ter adicionado um novo sensor ao ficheiro de configuração, a programação da lógica de obtenção de medições do mesmo é feita através da adição de um novo ficheiro python no diretório sensors com o nome configurado no campo name.

Seguindo a configuração acima como exemplo, o código associado ao sensor é colocado em ‘sensors/tsl2561.py’.

Durante o seu processo de inicialização, o microcontrolador tenta importar todos os ficheiros associados a sensores que constem no ficheiro de configuração e executa uma vez o método setup de cada um deles.
Posteriormente, o método loop será invocado repetidamente com o intervalo configurado em wait_time.

```
from lib.tsl2561_driver import *
lux_sensor = None

def setup(dm):
   global lux_sensor
   lux_sensor = device()
   lux_sensor.init()

def loop(dm): # Recebe 'dm' como argumento para poder publicar telemetria
   value = lux_sensor.getLux()
   print("Lux value: " + str(value))
   dm.publish("lux", value) # nome 'lux' é traduzido pelo ID no conf.json
```
*Exemplo de programação da lógica de obtenção de dados de um sensor*

A telemetria poderá ser publicada com o auxílio do objeto detimotic (dm) fornecido como argumento às funções. Para tal basta indicar o valor a enviar e o nome da métrica correspondente, já que um sensor pode dispor de várias métricas associadas. O nome é depois traduzido automaticamente pelo ID equivalente.

Resumindo, as acções necessárias para a adição de um novo sensor à lógica de execução do microcontrolador são apenas:

- Configuração do mesmo no ficheiro conf.json
- Adição de um ficheiro sensors/<name>.py com a interface demonstrada no exemplo

De forma a assegurar um elevado nível de parametrização, o ficheiro detimotic_conf.json (exemplificado em baixo) permite a configuração de várias facetas da lógica de conexão e envio de informação.

```
{
  "wifi": {
    "ssid": "MakerLab",
    "passw": "p4ssw0rd"
  },
  "gateway": {
    "addr": "192.168.0.2",
    "uname": "detimotic",
    "passw": "mqtt_password",
    "port": 1883,
    "telemetry_topic": "telemetry",
    "ping_freq": 10000 // em milisegundos
  },
  "watchdog": 5000 // em milisegundos
}
```
*Exemplo de configuração do ficheiro detimotic/detimotic_conf.json*

Para assegurar a recuperação do funcionamento do módulo no caso de um erro imprevisto que seja fatal à lógica do mesmo, é utilizado um watchdog timer que o reinicia e cujo limite é igualmente configurável.