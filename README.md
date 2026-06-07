# 🔍 ELK Stack — Central de Logs com Apache

![Apache](https://img.shields.io/badge/Apache-D22128?style=flat&logo=apache&logoColor=white)
![Filebeat](https://img.shields.io/badge/Filebeat-00BFB3?style=flat&logo=elastic&logoColor=white)
![Logstash](https://img.shields.io/badge/Logstash-FEC514?style=flat&logo=logstash&logoColor=black)
![Elasticsearch](https://img.shields.io/badge/Elasticsearch-005571?style=flat&logo=elasticsearch&logoColor=white)
![Kibana](https://img.shields.io/badge/Kibana-E8478B?style=flat&logo=kibana&logoColor=white)
![WIP](https://img.shields.io/badge/🚧-WIP-E08B00?style=flat)

```
ELK STACK
Lab de Observabilidade e Análise de Logs

Apache  →  Filebeat  →  Logstash  →  Elasticsearch  →  Kibana
 ``` 

> Lab prático de centralização e monitoramento de logs usando o stack ELK em uma instância EC2 na AWS.

---

## 📌 Objetivo

Construir uma central de logs capaz de:

- Coletar logs de acesso e erro do **Apache2** em tempo real
- Processar e enviar os eventos via **Filebeat**
- Armazenar e indexar os dados no **Elasticsearch**
- Visualizar dashboards interativos no **Kibana**