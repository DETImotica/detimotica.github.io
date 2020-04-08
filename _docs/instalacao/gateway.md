---
title: Sensor Gateway
category: Instalação e Utilização
order: 3
type: 3
---

## Instalação (Linux)

### Parte 1 - Broker MQTT

1. Instalar o *docker*
```
sudo apt install docker
```

2. Descarregar a imagem *eclipse-mosquitto*
```
docker pull eclipse-mosquitto
```

3. Criar os diretórios necessários
```
mkdir -p /var/mosquitto/log
mkdir -p /var/mosquitto/data
chmod ugo+w /var/mosquitto
```

4. Criar os ficheiros de configuração e passwd
```
wget https://raw.githubusercontent.com/DETImotica/Sensor_Gateway/master/configuration/var/mosquitto/mosquitto.conf -P /var/mosquitto
wget https://raw.githubusercontent.com/DETImotica/Sensor_Gateway/master/configuration/var/mosquitto/data/passwd -P /var/mosquitto/data
```

5. Iniciar um docker container em modo daemon
```
docker run -d -it -p 1883:1883 -p 9001:9001 -v /var/mosquitto/mosquitto.conf:/mosquitto/config/mosquitto.conf -v /var/mosquitto/data:/mosquitto/data -v /var/mosquitto/log:/mosquitto/log eclipse-mosquitto
```

6. Instalar o pacote *mosquitto-clients* de forma a poder subsrever e publicar em tópicos pela linha de comandos (Opcional) 
```
sudo apt install mosquitto-clients
```

Testado em: Raspberry Pi 2B+ (Raspbian Stretch) e Raspberry Pi 4B (Raspbian Stretch)


### Parte 2 - Módulo Gateway

1. Clonar o repositório
```
git clone https://github.com/DETImotica/Sensor_Gateway.git
```
2. Navegar até ao diretório 'src' do mesmo
```
cd Sensor_Gateway/src
```
3. Instalar as dependências do projeto
```
pip3 install -r requirements.txt
```


<br>
## Utilização

### Parte 1 - Broker MQTT

<br>
#### Credenciais (temporário)
Username: **detimotic**

Password: **testpw**  

É necessário colocar o container em execução sempre que se inicia a máquina em questão (Instalação - Parte 1 - Passo 5)

#### Subscrever a tópicos
Utilizando o *mosquitto-clients*:
```
mosquitto_sub -h <address> -v -t "<topic>" -u detimotic -P testpw
```
Substituir *\<topic\>* pelo tópico desejado
 
Substituir *\<address\>* pelo endereço IP da gateway ou *localhost* se for executado na própria  


#### Publicar em tópicos
Utilizando o *mosquitto-clients*:
```
mosquitto_pub -h <address> -t "<topic>" -u detimotic -P testpw -m "<message>"
```
Substituir *\<topic\>* pelo tópico desejado
 
Substituir *\<address\>* pelo endereço IP da gateway ou *localhost* se for executado na própria  

### Parte 2 - Módulo Gateway

1. Executar o ficheiro gateway.py com o parâmetro *password* definido
```
python3 gateway.py -p '<password>'
```

O parâmetro *password* identifica a palavra-chave associada aos sensores no Eclipse Hono.


### Referências
- [https://selfhostedhome.com/using-two-mqtt-brokers-with-mqtt-broker-bridging/](https://selfhostedhome.com/using-two-mqtt-brokers-with-mqtt-broker-bridging/)
