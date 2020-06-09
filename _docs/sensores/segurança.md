---
title: Segurança
category: Sensores
order: 2
type: 1
published: true
---

De modo a garantir a segurança nas comunicações MQTT com dados de telemetria dos microcontroladores para a gateway, a payload das mensagens é encriptada utilizando AES-128 e de acordo com o esquema em baixo.

A escolha de algoritmo visou limitar o impacto na performance dos microcontroladores sem comprometer a segurança do sistema.

Esta solução impede que terceiros consigam visualizar informação sensorial à qual poderiam não ter acesso normalmente.

Adicionalmente, o facto de cada sensor possuir a sua chave de encriptação única impossibilita o envio de dados sem o conhecimento da mesma, pelo que impede terceiros de ‘se fazerem passar’ por um sensor existente e enviar dados falsos.


![Alt text](/images/posts/Segurança.png?raw=true "Title")
*Diagrama ilustrativo da arquitetura de segurança nas comunicações*

De modo a evitar a manutenção de uma lista de chaves de encriptação na gateway, a chave de cada sensor é dedutível através do seu UUID e de um conjunto de chaves secretas que apenas a API e a gateway conhecem. 

