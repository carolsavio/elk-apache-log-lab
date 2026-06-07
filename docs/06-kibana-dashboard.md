# Etapa 6 — Kibana Dashboards

## Objetivo
Criar um Data View e um dashboard com visualizações dos logs do Apache no Kibana.

---

## 1. Acessar o Kibana

```
http://<SEU-IP-PUBLICO>:5601
```

Login:
- Usuário: `elastic`
- Senha: sua senha

---

## 2. Verificar dados disponíveis

Antes de criar o Data View, confirme os índices existentes:

```
curl -k -u elastic:<SUA-SENHA> "https://localhost:9200/_cat/indices?v&h=index,docs.count"
```

Anote o nome do índice com mais documentos — geralmente `filebeat-8.x.x`.

---

## 3. Criar o Data View

No Kibana:
```
☰ → Management → Stack Management → Kibana → Data Views
```

Clique em **Create data view**:

| Campo | Valor |
|---|---|
| Name | `filebeat-apache` |
| Index pattern | `filebeat-*` |
| Timestamp field | `@timestamp` |

Clique em **Save data view to Kibana**.

---

## 4. Explorar os dados no Discover

```
☰ → Analytics → Discover
```

- Selecione o data view `filebeat-apache`
- Mude o período para **Last 24 hours**
- Confirme que os logs aparecem com campos como `url.original`, `http.response.status_code`, `source.ip`

---

## 5. Criar o Dashboard

```
☰ → Analytics → Dashboard → Create dashboard
```

---

### Visualização 1 — Status HTTP (Pizza)

Clique em **Create visualization**:

- Tipo: **Pie**
- Arraste `http.response.status_code` para **Slice by**
- Arraste `http.response.status_code` para a área central
- Clique em **Save and return**

![Dashboard em pizza do http.response](/docs/screenshots/elastic-dashboard-pie.png)

---

### Visualização 2 — Requisições por tempo (Barras)

Clique em **Create visualization**:

- Tipo: **Bar**
- Arraste `@timestamp` para a área central
- O Kibana cria automaticamente o gráfico por tempo
- Clique em **Save and return**

---

### Visualização 3 — Top IPs (Tabela)

Clique em **Create visualization**:

- Tipo: **Table**
- Arraste `source.ip` para a área central
- Clique em **Save and return**

---

## 6. Salvar o Dashboard

Clique em **Save** no canto superior direito:

- Title: `Apache Logs Dashboard`
- Clique em **Save**

![Dashboard no elastic](/docs/screenshots/elastic-dashboard.png)

