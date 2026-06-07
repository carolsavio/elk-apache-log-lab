# Etapa 2 — Primeiro acesso SSH + Configuração inicial

## Objetivo
Conectar na instância via SSH, atualizar o sistema e configurar o swap de 4GB necessário para compensar a limitação de RAM do t3.micro.

---

## 1. Ajustar permissão da chave

No terminal **local**:

```
chmod 400 ~/Downloads/the-elk-lab-key.pem
```
> Se salvou em outro diretório, ajuste para o caminho correto. 

---

## 2. Conectar via SSH

```
ssh -i ~/Downloads/the-elk-lab-key.pem ubuntu@<SEU-IP-PUBLICO>
```

Na primeira conexão confirme com `yes` quando perguntar sobre o fingerprint.

Prompt esperado após conexão:
```
ubuntu@ip-xxx-xxx-xxx-xxx:~$
```

---

## 3. Atualizar o sistema

```bash
sudo apt update && sudo apt upgrade -y
```

> Se aparecer tela roxa perguntando sobre reiniciar serviços, pressione Enter para aceitar o padrão.

---

## 4. Criar swap de 4GB

O t3.micro tem apenas 1GB de RAM. O swap é essencial para o Elasticsearch não travar.

```
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

Tornar permanente (sobrevive a reboots):

```
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```
![Terminal com comando swap](/docs/screenshots/swap-permanent.png)

Verificar:

```
free -h
```

Saída esperada:
```
               total        used        free
Mem:           908Mi        ...
Swap:          4.0Gi        0B         4.0Gi
```
![Teste de memória](/docs/screenshots/free-h.png)

---

## 5. Ajuste de memória virtual para o Elasticsearch

```bash
sudo sysctl -w vm.max_map_count=262144
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
```

---
