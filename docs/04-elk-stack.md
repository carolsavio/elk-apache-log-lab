# Etapa 4 — Instalar Elasticsearch + Kibana

## Objetivo
Instalar e configurar o Elasticsearch e o Kibana, garantindo que os dois serviços estejam ativos e se comunicando.

---

## 1. Instalar o Elasticsearch

```
sudo apt install -y elasticsearch
```

> ⚠️ Na saída do comando, o Elasticsearch exibe a senha do usuário `elastic` e o enrollment token do Kibana **uma única vez**. Anote imediatamente.


![Instalando elasticsearch](/docs/screenshots/installing-elasticsearch.png)
![Password gerado pelo elasticsearch](/docs/screenshots/passwdprint.png)

Caso perca, gere novamente:
```
# Resetar senha
sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic

# Novo enrollment token
sudo /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
```

---

## 2. Reduzir heap da JVM (essencial no t3.micro)

```
sudo nano /etc/elasticsearch/jvm.options.d/heap.options
```

Adicione:
```
-Xms512m
-Xmx512m
```

---

## 3. Iniciar o Elasticsearch

```
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch
sudo systemctl start elasticsearch
```

Aguarde 30-60 segundos e verifique:

```
sudo systemctl status elasticsearch
```

Deve aparecer `active (running)`.

---

## 4. Testar o Elasticsearch

```
curl -k -u elastic:<SUA-SENHA> https://localhost:9200
```

Saída esperada:
```json
{
  "name" : "ip-xxx-xxx-xxx-xxx",
  "cluster_name" : "elasticsearch",
  "version" : {
    "number" : "8.19.x",
    ...
  },
  "tagline" : "You Know, for Search"
}
```

---

## 5. Instalar o Kibana

```
sudo apt install -y kibana
```
![Instalando o Kibana](/docs/screenshots/installing-kibana.png)

---

## 6. Configurar o Kibana

```
sudo nano /etc/kibana/kibana.yml
```

Adicione no final do arquivo:
```
server.port: 5601
server.host: "0.0.0.0"
elasticsearch.hosts: ["https://localhost:9200"]
elasticsearch.ssl.verificationMode: none
elasticsearch.username: "kibana_system"
elasticsearch.password: "<SENHA-DO-KIBANA-SYSTEM>"
```
![Edição do Kibana no nano](/docs/screenshots/kibana-external-connections.png)

Gerar senha do kibana_system:
```
sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u kibana_system
```

> ⚠️ Verifique se as chaves `elasticsearch.hosts` e `elasticsearch.username` não estão duplicadas no arquivo. Use `grep -n "elasticsearch.hosts" /etc/kibana/kibana.yml` para confirmar.

---

## 7. Iniciar o Kibana

```
sudo systemctl daemon-reload
sudo systemctl enable kibana
sudo systemctl start kibana
```

Acompanhar inicialização (pode demorar 2-5(ou mais) minutos no t3.micro):

```
sudo journalctl -u kibana -f
```

Aguarde a linha:
```
[INFO][status] Kibana is now available
```
![Kibana running](/docs/screenshots/kibanna-running.png)

---

## 8. Acessar o Kibana

> O Kibana pode demorar até 5 minutos para carregar na primeira vez no t3.micro. Se aparecer "Kibana server is not ready yet", aguarde e pressione F5.

O primeiro acesso passa por 3 telas:

**Tela 1 — Enrollment token**
- Cole o enrollment token gerado anteriormente
- Clique em **Configure Elastic**

Caso o token tenha expirado (válido por 30 minutos), gere um novo:
```
sudo /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
```
![Tela do Elastic com Token do Kibana](/docs/screenshots/paste-token-on-kibana.png)

**Tela 2 — Verification code**
- O Kibana pede um código de 6 dígitos para confirmar a conexão
- Gere o código:
```
sudo /usr/share/kibana/bin/kibana-verification-code
```
- Digite o código na tela do navegador

![Kibana código de verificação](/docs/screenshots/kibana-verification-code.png)

**Tela 3 — Login**
- Usuário: `elastic`
- Senha: a que você anotou

![Tela de login do elastic](/docs/screenshots/elastic-login.png)

Após o login aparece o painel **"Welcome to Elastic"**.

![](/docs/screenshots/elastic-welcome-screen.png)

> ⚠️ Se o fluxo do enrollment token não funcionar, configure manualmente o `kibana.yml` com `elasticsearch.username` e `elasticsearch.password` conforme descrito na etapa 6, e reinicie o Kibana. Veja detalhes em [troubleshooting.md](troubleshooting.md).

---

