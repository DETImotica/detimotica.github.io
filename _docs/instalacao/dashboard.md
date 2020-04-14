---
title: Dashboard
category: Instalação e Utilização
order: 1
type: 3
---

A dashboard é uma maneira de mostrar visualmente os dados provenientes dos sensores, recorrendo a gráficos, por exemplo, sendo a principal forma de controlo e monitorização.

### Instalação (Linux/Debian)

### Parte 1 - Iniciar o servidor localmente

1. Instalar o *package* .deb
```
sudo apt-get install -y adduser libfontconfig1
wget https://dl.grafana.com/oss/release/grafana_6.6.2_amd64.deb
sudo dpkg -i grafana_6.6.2_amd64.deb
```

2. Iniciar o servidor
```
sudo service grafana-server start
```

3. Confirmar o estado atual do servidor
```
sudo service grafana-server status
```

### Parte 2 - Encerrar o servidor local

1. Encerrar o servidor
```
sudo service grafana-server stop
```

2. Confirmar o estado atual do servidor
```
sudo service grafana-server status
```

### Utilização

### Parte 1 - Aceder ao Grafana

Depois de abrir o browser, acesse o porto 3000 (porto *default*): [http://localhost:3000/](http://localhost:3000/) 

### Parte 2 - Autenticação
Credenciais

Username: **admin**

Password: **admin**  

### Referências
- [https://grafana.com/docs/grafana/latest/installation/debian/](https://grafana.com/docs/grafana/latest/installation/debian/)
- [https://grafana.com/docs/grafana/latest/installation/windows/](https://grafana.com/docs/grafana/latest/installation/windows/)
