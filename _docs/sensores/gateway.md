---
title: Gateway
category: Sensores
order: 3
type: 1
published: true
---

#Gateway

Equipamento agregador que está conectado à rede do MakerLab e recebe todas as mensagens MQTT enviadas pelos módulos de sensorização.

Dispõe também de uma ligação à rede da Universidade de Aveiro através da VPN da mesma, por onde os dados recebidos são encaminhados para o broker da instância Eclipse Hono alojada no Instituto de Telecomunicações.

Assim, de um ponto de vista de alto-nível, o equipamento efetua as funções de uma bridge MQTT. 

Durante o desenvolvimento do projeto foi utilizado um Raspberry Pi (2B ou 4B) para esse efeito.

### Broker MQTT

O broker local para o qual toda a telemetria é enviada pelos microcontroladores pode estar alojado em qualquer servidor acessível dentro da rede do MakerLab.

Durante o desenvolvimento do projeto o local escolhido foi a própria gateway, sendo que o software selecionado foi o Mosquitto (imagem Docker: eclipse-mosquitto)

As configurações utilizadas podem ser encontradas no repositório respetivo.

### Receção, desencriptação, e encaminhamento de mensagens

A lógica de receção, desencriptação, e encaminhamento de mensagens é assegurada por um programa multi-threaded em Python (gateway.py).

A thread principal conecta-se ao broker MQTT local e subscreve o tópico ‘telemetry’ e todos os seus subtópicos (i.e. ‘telemetry/#’).

As mensagens dos microcontroladores com valores de um determinada métrica deverão ser enviadas para o tópico: ‘telemetry/<ID do sensor/métrica>’ e, conforme o exposto na secção anterior, ter sido encriptadas com a sua chave única AES-128 e posteriormente codificadas em base64. 

Exemplo:
```
telemetry/7d245a97-66c7-49eb-9940-dbb9cb24f5ec HLhUwpmNgorg7W6lf4WZhcSXMThCJ8wHcv+wavM=
```
<br>

Para o processamento das mensagens é utilizado um modelo producer/consumer com múltiplas threads.

No momento de receção, a mensagem é mantida numa fila até uma das worker threads se encontrar disponível para a desencriptar, verificar a sua integridade, e proceder ao seu encaminhamento para o broker da instância Eclipse Hono.

A mensagem, uma vez desencriptada, deverá ter o seguinte formato (no caso do LoPy é assegurado automaticamente):

```
{'value': <valor observado pelo sensor>}    // Exemplo: {'value': 1018.56}
```

Existe ainda uma thread adicional para manter o estado da conexão com o broker remoto.
Durante o desenvolvimento do projeto foi utilizado um máximo de 20 worker threads concorrentes.

### Configuração

De forma a assegurar um elevado nível de parametrização, o ficheiro gateway_config.json (exemplificado em baixo) permite a configuração de várias facetas da lógica de receção e encaminhamento.

```
{
    "remote": {   // Configurações do broker remoto
            "host": "iot.av.it.pt",
            "port": 1883,
            "tenant_id": "detimotic",
            "device_prefix": "device_", // prefixo ao UUID da métrica
            "telemetry_topic": "telemetry",
            "value_description": "value"
    },
    "local": {   // Configurações do broker local
            "host": "localhost",
            "port": 1883,
            "uname": "detimotic",
            "telemetry_topic": "telemetry",
            "value_description": "value",
            "max_workers": 20
    },
    "security": {
            "kdf_iterations": 1974  // Iterações do PBKDF2
    }
}
```
*Exemplo de configuração do ficheiro gateway_config.json*

Existe ainda o ficheiro escondido .secret_config.json (exemplificado em baixo) que não deverá ser público e onde são introduzidos os parâmetros de autenticação e segurança.

```
{
    "local_broker_pw": "mqtt_password",
    "hono_sensors_pw" : "pw_that_proves_gw_identity_to_hono",
    "secret_key": "secure_key_for_generating_aes_keys",
    "secret_salt": "random_str123"
}
```
*Exemplo de configuração do ficheiro .secret_config.json*
