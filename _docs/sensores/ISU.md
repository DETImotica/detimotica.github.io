---
title: ISU
category: Sensores
order: 3
type: 1
published: true
---

## Integrated Sensor Unit (ISU)

A unidade integrada de sensorização (ISU) tem integrada vários sensores que comunicam de diferentes formas com o microcontrolador LoPy v1. De momento, os sensores e respetivos circuitos encontram-se montados numa breadboard, de modo, a permitir a fácil montagem e alteração dos circuitos.    

### Componentes
<br>

#### Alimentação
Os sensores podem ser alimentados por 3.3V ou 5V, consoante o modelo do sensor. De modo a permitir uma alimentação independente do microcontrolador, está a ser usada uma fonte de alimentação para breadboard que disponibiliza 5V num power rail e 3.3V no outro.
 
|           Sensor            |         Alimentação         |
|:---------------------------:|:---------------------------:|
| Sensor LMV324/SEN-12642     | 5V                          |
| Sensor DHT11                | 3.3V                        |
| Sensor MG812                | 5V                          |
| Sensor TSL2561              | 3.3V                        |
| Sensor BME860               | 3.3V                        |

![fonte-de-alimentacao](/images/posts/fonte_alimentacao.jpg){: .center-image}


#### Comunicação
Os sensores comunicam de diferentes formas com o microcontrolador. Os sensores (slave) podem comunicar pelo protocolo de comunicação série I2C com o microcontrolador (master), ou então pode ser utilizada uma comunicação analógica entre sensor e microcontrolador. No caso do DHT11, é usado o protocolo de comunicação One Wire.
 
|           Sensor            |         Comunicação         |
|:---------------------------:|:---------------------------:|
| Sensor LMV324/SEN-12642     | Analog                      |
| Sensor DHT11                | One Wire                    |
| Sensor MG812                | Analog                      |
| Sensor TSL2561              | I2C                         |
| Sensor BME860               | I2C (SPI também suportado)  |


### Montagem
<br>

Uma representação da montagem dos sensores encontra-se disponível na imagem em baixo. De notar também, que a tensão de saída dos sensores alimentados com 5V, isto é os sensores que disponibilizam uma saída de dados analógica, foi reduzida para 3.3V, passando por um divisor de tensão, de modo a não danificar o microcontrolador que só pode receber no máximo 3.3V. No caso do DHT11, foi necessário uma resistência de pullup na saída de dados.

![montagem](/images/posts/montagem.png){: .center-image}

### Configuração no microcontrolador
<br>

Além do código desenvolvido e bibliotecas utilizadas para cada sensor, que irão ser explorados noutro documento, de modo a o microcontrolador poder gerir facilmente os sensores instalados, é utilizado um ficheiro de configuração json que é exemplificado a seguir:

```
{
    "isu_id": <"isu_id">,
    "modules": [
    {
      "name": <"sensor_name">,
      "active": true,
      "wait_time": <time_ms>,
      "metrics": {
        <"metric_name">: <"sensor_id">
        ...
      }
    }, ...
}
```

