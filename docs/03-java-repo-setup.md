# Etapa 3 — Java 17 + Repositório Elastic

## Objetivo
Instalar o Java 17 (dependência do ELK) e adicionar o repositório oficial da Elastic para instalar os pacotes nas etapas seguintes.

---

## 1. Instalar o Java 17

```bash
sudo apt install -y openjdk-17-jdk
```

Verificar:

```bash
java -version
```

Saída esperada:
```
openjdk version "17.x.x" 2024-xx-xx
OpenJDK Runtime Environment ...
```
![Print da versão do Java](/docs/screenshots/java-version.png)

---

## 2. Adicionar a chave GPG da Elastic

```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | \
  sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
```

Sem saída = sucesso.

![GPG Key](/docs/screenshots/installing-gpg-key.png)
---

## 3. Adicionar o repositório oficial

```
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] \
  https://artifacts.elastic.co/packages/8.x/apt stable main" | \
  sudo tee /etc/apt/sources.list.d/elastic-8.x.list
```

---

## 4. Atualizar lista de pacotes

```
sudo apt update
```

Confirmar que o repositório foi reconhecido — procure por uma linha assim na saída:
```
Get:... artifacts.elastic.co/packages/8.x/apt stable InRelease
```

---

## 5. Confirmar disponibilidade dos pacotes

```bash
apt-cache policy elasticsearch
```

Deve mostrar a versão `8.x.x` como candidata para instalação.

![Instalando elasticsearch](/docs/screenshots/installing-elasticsearch.png)

![Password gerado pelo elasticsearch](/docs/screenshots/passwdprint.png)