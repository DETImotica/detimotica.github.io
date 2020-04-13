---
title: Sistemas de Base de Dados
category: API
order: 2
type: 1
---

No que toca a Sistemas de Gestão de Bases de Dados (SGBD), o módulo intermédio utiliza os seguintes pontos de armazenamento:

- Relacional - PostgreSQL versão 12.2
    - Integrado no módulo aplicacional, instalado e configurado via containers Docker.
    - Utilizada para registo genérico de utilizadores, sensores existentes e associação de instalação em salas e como PIP (Policy Information Point) no modelo de controlo de acessos ABAC.

- Time-series - InfluxDB versão 1.7.10
    - Base de dados remota, que integra o *broker* Eclipse Hono
    - Armazena os dados de sensorização do projeto

- Documentos (NoSQL) - MongoDB versão 4.2
    - Integrada, instalada usando um contentor Docker
    - Usada como subcomponente fundamental no módulo de controlo de acessos. Tem a função de PRP (Policy Retrieval Point) na implementação de controlo de acessos.