---
title: Componentes
category: API
order: 1
type: 1
---

O módulo da API permite a disponibilização estruturada e organizada dos dados sensoriais, de pontos de entrada para gestão de políticas de acesso e de queries a atributos, com mecanismos de autorização/autenticação integrados. É maioritariamente desenvolvido com recurso a programas na linguagem Python (versão 3.6).

### Arquitetura

O módulo aplicacional integra as seguintes componentes:

- Definição de uma API REST, com recurso ao pacote Flask e alguns plugins associados, que corre por trás do Gunicorn WSGI (*Web Server Gateway Interface*)

- Mecanismo de controlo de acesso por atributos (ABAC) baseado no *kit* de software Vakt, com um Sistema de Gestão de Bases de Dados No-SQL integrado para armazenamento de políticas

- Adaptador para Sistemas de Gestão de Bases de Dados
    
    - **Relacional**, que suporta o catálogo de informação de sensores e salas do departamento, sendo também o Ponto de Informação de Políticas para controlo de acesso
    
    - **Time-series**, que armazena os dados de sensorização recolhidos pelo *broker* agregador (Eclipse Hono)

Todos estes módulos estão a ser desenvolvidos a pensar numa instalação integrada com base em microserviços, em que cada componente é instalado com recurso a contentores Docker.