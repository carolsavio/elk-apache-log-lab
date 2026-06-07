# Etapa 5 — Apache2 + Filebeat + Pipeline de Logs

## Objetivo
Instalar o Apache2 como gerador de logs e o Filebeat como agente de coleta, configurando o módulo Apache nativo para enviar os logs diretamente ao Elasticsearch.

> **Nota:** O Logstash foi removido da arquitetura por inviabilidade de memória no t3.micro. O módulo Apache do Filebeat substitui o Logstash fazendo o parsing dos logs nativamente. Veja detalhes em [troubleshooting.md](/docs/troubleshooting.md).

---

## 1. Instalar o Apache2

```
sudo apt install -y apache2
sudo systemctl enable apache2
sudo systemctl start apache2
```

Verificar:
```
sudo systemctl status apache2
```

Testar no navegador:
```
http://<SEU-IP-PUBLICO>
```

Deve aparecer a página padrão **"Apache2 Ubuntu Default Page"**.

![Apache Default](/docs/screenshots/apache-default.png)

---

## 2. Verificar geração de logs

```
sudo tail -f /var/log/apache2/access.log
```

Acesse a página do Apache no navegador e observe os logs aparecendo em tempo real. Pressione `Ctrl+C` para parar.

---

## 3. Instalar o Filebeat

```
sudo apt install -y filebeat
```

---

## 4. Habilitar o módulo Apache

```bash
sudo filebeat modules enable apache
```

---

## 5. Configurar o módulo Apache

```bash
sudo nano /etc/filebeat/modules.d/apache.yml
```

Conteúdo:
```yaml
- module: apache
  access:
    enabled: true
    var.paths: ["/var/log/apache2/access.log"]
  error:
    enabled: true
    var.paths: ["/var/log/apache2/error.log"]
```

---

## 6. Configurar o filebeat.yml

```bash
sudo nano /etc/filebeat/filebeat.yml
```

Substitua todo o conteúdo por:
```yaml
filebeat.modules:
  - module: apache
    access:
      enabled: true
      var.paths: ["/var/log/apache2/access.log"]
    error:
      enabled: true
      var.paths: ["/var/log/apache2/error.log"]

filebeat.config.modules:
  path: /etc/filebeat/modules.d/*.yml
  reload.enabled: false

output.elasticsearch:
  hosts: ["https://localhost:9200"]
  username: "elastic"
  password: "<SUA-SENHA>"
  ssl.verification_mode: none

logging.level: info
logging.to_files: true
logging.files:
  path: /var/log/filebeat
  name: filebeat
  keepfiles: 7

tags: ["apache", "lab", "elk"]
```
>Leia mais em **Referencias** no fim da página
---

## 7. Configurar pipelines e índices

```
sudo filebeat setup --pipelines --modules apache -e
sudo filebeat setup --index-management -e
```

---

## 8. Iniciar o Filebeat

```
sudo systemctl enable filebeat
sudo systemctl start filebeat
sudo systemctl status filebeat
```

Deve aparecer `active (running)`.

---

## 9. Gerar logs de teste

```
# Mix de status codes
for i in {1..100}; do
  curl -s http://localhost/ > /dev/null
  curl -s http://localhost/pagina-nao-existe > /dev/null
  curl -s http://localhost/admin > /dev/null
  curl -s http://localhost/.env > /dev/null
done

# Simular scanner de vulnerabilidades
for path in /phpmyadmin /admin /login /shell /backdoor /.git /config.php; do
  curl -s http://localhost$path > /dev/null
done
```

---

## 10. Confirmar logs no Elasticsearch

```
curl -k -u elastic:<SUA-SENHA> "https://localhost:9200/_cat/indices?v&h=index,docs.count"
```

Procure pelo índice `filebeat-*` com documentos indexados.


---
## Referências
- [Filebeat Modules Configuration](https://www.elastic.co/guide/en/beats/filebeat/current/configuration-filebeat-modules.html)
- [Filebeat Apache Module](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-apache.html)
- [Filebeat Elasticsearch Output](https://www.elastic.co/guide/en/beats/filebeat/current/elasticsearch-output.html)
- [Filebeat Logging Configuration](https://www.elastic.co/guide/en/beats/filebeat/current/configuration-logging.html)