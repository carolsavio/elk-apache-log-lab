# 🔍 ELK Stack — Central de Logs com Apache

![Apache](https://img.shields.io/badge/Apache-D22128?style=flat&logo=apache&logoColor=white)
![Filebeat](https://img.shields.io/badge/Filebeat-00BFB3?style=flat&logo=elastic&logoColor=white)
![Logstash](https://img.shields.io/badge/Logstash-FEC514?style=flat&logo=logstash&logoColor=black)
![Elasticsearch](https://img.shields.io/badge/Elasticsearch-005571?style=flat&logo=elasticsearch&logoColor=white)
![Kibana](https://img.shields.io/badge/Kibana-E8478B?style=flat&logo=kibana&logoColor=white)
![WIP](https://img.shields.io/badge/🚧-WIP-E08B00?style=flat)

## Arquitetura

```
┌─────────────┐      ┌──────────┐      ┌───────────────┐      ┌─────────┐
│   Apache2   │ ───▶ │ Filebeat │ ───▶ │ Elasticsearch │ ───▶ │ Kibana  │
│ access.log  │      │ (agente) │      │  (indexação)  │      │(dashbrd)│
│ error.log   │      └──────────┘      └───────────────┘      └─────────┘
└─────────────┘
         EC2 t3.micro — Ubuntu 22.04 LTS — AWS us-east-1
```

> Lab prático de centralização e monitoramento de logs usando o stack ELK em uma instância EC2 na AWS.

---

## Objetivo

Construir uma central de logs capaz de:

- Coletar logs de acesso e erro do **Apache2** em tempo real
- Processar e enviar os eventos via **Filebeat**
- Armazenar e indexar os dados no **Elasticsearch**
- Visualizar dashboards interativos no **Kibana**

---


> **Nota:** O Logstash foi removido da arquitetura final por inviabilidade de memória no t3.micro (1GB RAM). O Filebeat com módulo Apache nativo faz o parsing dos logs e envia diretamente para o Elasticsearch. Veja detalhes no [troubleshooting](/docs/troubleshooting.md).

---

## Ambiente

| Item | Detalhe |
|---|---|
| Provedor | AWS EC2 |
| Instância | t3.micro (free tier) |
| RAM | 1 GB + 4 GB swap |
| Sistema | Ubuntu 22.04 LTS |
| Região | us-east-1 |
| Stack | Elasticsearch 8.19 + Kibana 8.19 + Filebeat 8.19 |

---

##  Como reproduzir

### Pré-requisitos

- Conta AWS com free tier ativo
- Par de chaves `.pem` criado no console EC2
- Portas abertas no Security Group: `22`, `80`, `5601`, `9200`

### Passo a passo

A documentação está dividida por etapas na pasta `docs/`:

| Etapa | Documento | Descrição |
|---|---|---|
| 1 | [01-setup-aws.md](docs/01-setup-aws.md) | Criar instância EC2 |
| 2 | [02-acesso-ssh.md](docs/02-ssh-access.md) | Primeiro acesso + swap |
| 3 | [03-java-repositorio.md](docs/03-java-repo-setup.md) | Java 17 + repo Elastic |
| 4 | [04-elk-stack.md](docs/04-elk-stack.md) | Elasticsearch + Kibana |
| 5 | [05-apache-filebeat.md](docs/05-apache-filebeat.md) | Apache2 + Filebeat |
| 6 | [06-kibana-dashboard.md](docs/06-kibana-dashboard.md) | Dashboards no Kibana |
| - | [troubleshooting.md](docs/troubleshooting.md) | Problemas e soluções |


---

## Dashboard

O dashboard final no Kibana contém:

- **Gráfico de pizza** — distribuição de status HTTP (200, 404, 500...)
- **Gráfico de barras** — volume de requisições ao longo do tempo
- **Tabela** — Top IPs com mais requisições

> Screenshots na pasta `screenshots/`

![Dashboard pie](/docs/screenshots/elastic-dashboard-pie.png)
![Dashboard pronto](/docs/screenshots/elastic-dashboard.png)

---

## Estrutura do repositório

```
elk-apache-log-lab/
├── README.md
├── docs/
│   ├── screenshots/
│   ├── 01-setup-aws.md
│   ├── 02-acesso-ssh.md
│   ├── 03-java-repositorio.md
│   ├── 04-elk-stack.md
│   ├── 05-apache-filebeat.md
│   ├── 06-kibana-dashboard.md
│   └── troubleshooting.md
├── configs/
│   ├── filebeat.yml
│   └── logstash-apache.conf
└── .gitignore
```

---

## Aprendizados

- Gerenciamento de memória em ambiente com recursos limitados
- Debugging de serviços via `journalctl` e `systemctl`
- Configuração de pipelines de logs com Filebeat
- Criação de índices e Data Views no Kibana
- Resolução de conflitos de configuração YAML
- Tomada de decisão técnica: substituição do Logstash pelo módulo nativo do Filebeat

---

## Referências

- [Elasticsearch Docs](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)
- [Filebeat Apache Module](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-apache.html)
- [Kibana Docs](https://www.elastic.co/guide/en/kibana/current/index.html)

---

*Projeto desenvolvido como lab prático de Blue Team / Monitoramento / DevSecOps.*
