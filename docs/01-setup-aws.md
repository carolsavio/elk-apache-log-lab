# Etapa 1 — Criar a instância EC2 na AWS

## Objetivo
Provisionar uma VPS Ubuntu 22.04 na AWS para hospedar o lab ELK.

---

## Pré-requisitos
- Conta AWS com free tier ativo
- Acesso ao console AWS

---

## Passo a passo

### 1. Acessar o EC2
- Acesse [console.aws.amazon.com](https://console.aws.amazon.com)
- No campo de busca, digite **EC2**
- No painel lateral, clique em **Instances**
- Clique em **Launch instances**

---

### 2. Configurar a instância

**Name and tags**
```
elk-lab
```

**Application and OS Images**
- Clique em **Ubuntu**
- Selecione: `Ubuntu Server 22.04 LTS (HVM), SSD Volume Type`
- Architecture: `64-bit (x86)`

**Instance type**
```
t3.micro
```

**Key pair**
- Clique em **Create new key pair**
- Key pair name: `the-elk-lab-key`
- Key pair type: `RSA`
- Private key format: `.pem`
- Clique em **Create key pair** — o arquivo baixa automaticamente
- ⚠️ Guarde esse arquivo. Sem ele você não acessa a instância.

---

### 3. Network settings

- Clique em **Edit**
- Auto-assign public IP: **Enable**
- Selecione **Create security group**

Adicione as seguintes regras:

| Type | Protocol | Port | Source |
|---|---|---|---|
| SSH | TCP | 22 | My IP |
| HTTP | TCP | 80 | Anywhere |
| Custom TCP | TCP | 5601 | My IP |
| Custom TCP | TCP | 9200 | My IP |
| Custom TCP | TCP | 5044 | Anywhere |

---

### 4. Configure storage
- 1 volume: **20 GiB**
- Volume type: `gp3`

![Sumário de instância](/docs/screenshots/instance-summary.png)
---

### 5. Lançar a instância
- Clique em **Launch instance**
- Aguarde o **Instance state** ficar `Running`
- Aguarde o **Status check** mostrar `3/3 checks passed`

![Instência rodando](/docs/screenshots/instance-running.png)

---

## Informações para anotar

| Campo | Valor |
|---|---|
| Instance ID | `i-0xxxxxxxxxxxxxxxxx` |
| Public IPv4 | `xxx.xxx.xxx.xxx` |
| Key pair | `the-elk-lab-key.pem` |





