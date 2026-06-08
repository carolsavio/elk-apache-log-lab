# Troubleshooting - Problemas Reais Encontrados no Lab

> Este documento registra todos os problemas encontrados durante a execução do lab, com diagnóstico e solução aplicada. Esses erros são reais e aconteceram durante a instalação em uma instância EC2 t3.micro.


## 1 - Senha e enrollment token do Elasticsearch não anotados

**Sintoma:** A senha do usuário `elastic` e o enrollment token do Kibana são exibidos apenas uma vez, durante a primeira inicialização do Elasticsearch. Se você não anotar, perde. Foi o que aconteceu comigo, na primeira vez executando o laboratório esse ponto importante passou batido.

E a **Solução** foi gerar novamente via CLI:

```bash
# Resetar senha do usuário elastic
sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic

# Gerar novo enrollment token para o Kibana
sudo /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
```

A **lição** que ficou é a de sempre prestar atenção nas linhas do terminal e anotar credenciais imediatamente. Em ambiente de produção, usar um gerenciador de secrets.

---

## 2 - Certificado SSL não encontrado pelo curl

**Sintoma:**
```
curl: The file '/etc/elasticsearch/certs/http_ca.crt' provided to --cacert does not exist
```

**Causa:** O arquivo existe mas o usuário comum não tem permissão de leitura.

**Solução alternativa 1 - copiar para local acessível:**
```
sudo cp /etc/elasticsearch/certs/http_ca.crt /tmp/http_ca.crt
sudo chmod 644 /tmp/http_ca.crt
curl -u elastic:<SENHA> --cacert /tmp/http_ca.crt https://localhost:9200
```

**Solução alternativa 2 - ignorar verificação (apenas para lab):**
```
curl -k -u elastic:<SENHA> https://localhost:9200
```

A **lição** que fica é que em produção **nunca** se deve usar `-k`. 
Para lab local é aceitável.


---

## 3 - Logstash inviável no t3.micro - problema crítico de memória

**Sintoma:** Logstash travado por mais de 1 hora sem inicializar, sem mensagem de erro clara.

**Diagnóstico:**
```
ps aux --sort=-%mem | head -5
# Logstash consumindo 37% da RAM (352MB) apenas para a JVM
# Elasticsearch consumindo 30% (288MB)
# Total: ~640MB de 908MB disponíveis ← sem sobra para o Kibana
```
Esse foi o problema que me travou por algum tempo neste lab.
Foram muitas tentativas se solução.

**Tentativas de solução:**
- Reduzir heap de 256m para 128m → ainda travava
- Parar Kibana para liberar RAM → Logstash subia mas Kibana não conseguia subir depois
- `pkill -9 logstash` → travava também, necessário abrir segundo terminal SSH

A **solução definitiva** foi desabilitar o Logstash e usar o módulo Apache nativo do Filebeat, que faz o parsing dos logs e envia direto para o Elasticsearch sem precisar do Logstash.

```
sudo systemctl stop logstash
sudo systemctl disable logstash
```

**Reconfigurar Filebeat com módulo Apache:**
```
sudo filebeat modules enable apache
# Editar /etc/filebeat/modules.d/apache.yml com os paths corretos
sudo systemctl restart filebeat
```

**Impacto:** Pipeline funcional, sem Logstash. Para uso em produção com parsing avançado (grok customizado, enriquecimento GeoIP), recomenda-se instância com mínimo 4GB de RAM. 


**Lição:** O t3.micro (1GB RAM) é insuficiente para rodar o stack ELK completo. **Mínimo recomendado: t3.small (2GB RAM).** O swap de 4GB ajuda mas não resolve o problema de latência causado pelo uso intenso de swap.

Ou seja, no proximo lab usarei uma máquina com mais recursos e adicionarei a documentação nesse repositório. 

---

## 4. Pipeline do Logstash com erro de GeoIP no ECS v8

**Sintoma:**
```
GeoIP Filter in ECS-Compatibility mode requires a `target` when 
`source` is not an `ip` sub-field
```

**Causa:** O Logstash 8.x no modo ECS v8 exige o campo `target` explícito no filtro geoip.

**Solução:** Adicionar `target` no bloco geoip do pipeline:
```
geoip {
  source => "clientip"
  target => "geoip"   # <- adicionei esta linha
}
```

**Lição:** A documentação do Logstash 8.x mudou o comportamento padrão do ECS. 
Sempre consultar a versão correta da documentação.

---

## 5 - Filebeat falhando por dependência do Kibana

**Sintoma:**
```
Exiting: error connecting to Kibana: fail to get the Kibana version
dial tcp 127.0.0.1:5601: connect: connection refused
```

**Causa:** O `filebeat.yml` tinha `setup.dashboards.enabled: true`, que obriga o Filebeat a conectar no Kibana antes de inicializar, mesmo que o Kibana esteja parado.

A **solução** foi comentar as linhas de setup do Kibana no `filebeat.yml`:
```
#setup.kibana:
#  host: "http://localhost:5601"

#setup.dashboards.enabled: true
```

**Lição:** O Filebeat tem duas funções: coletar logs e configurar dashboards. Para operação normal, a configuração de dashboards pode ser desabilitada.

---

## 6 - Filebeat com erro de módulo sem filesets habilitados

**Sintoma:**
```
module apache is configured but has no enabled filesets
```

**Causa:** O módulo Apache estava ativado mas o arquivo `/etc/filebeat/modules.d/apache.yml` não tinha os filesets configurados.

**Solução:** Editar o arquivo do módulo:
```
sudo nano /etc/filebeat/modules.d/apache.yml
```

Conteúdo correto:
```yaml
- module: apache
  access:
    enabled: true
    var.paths: ["/var/log/apache2/access.log"]
  error:
    enabled: true
    var.paths: ["/var/log/apache2/error.log"]
```

**Lição:** Habilitar o módulo com `filebeat modules enable apache` não configura os paths automaticamente, é necessário editar o arquivo `.yml` do módulo.

---

## 8. Kibana falhando com "duplicated mapping key" no YAML

**Sintoma:**
```
FATAL CLI ERROR YAMLException: duplicated mapping key (195:1)
elasticsearch.hosts: ["https://...
```

**Causa:** Durante as minhas tentativas de identificar e corrigir as configuração do meu laboratório após as falhas de memória no logstash, a chave `elasticsearch.hosts` foi adicionada múltiplas vezes no `kibana.yml`. O YAML não aceita chaves duplicadas.

**Diagnóstico:**

```
grep -n "elasticsearch.hosts" /etc/kibana/kibana.yml
# Retornou 3 ocorrências nas linhas 43, 188 e 195
```

**Solução:** Remover as linhas duplicadas com sed:
```
sudo sed -i '195d' /etc/kibana/kibana.yml
sudo sed -i '188d' /etc/kibana/kibana.yml
# Verificar que sobrou apenas uma entrada
grep -n "elasticsearch.hosts" /etc/kibana/kibana.yml
```

**Lição:** Ao adicionar configurações no final de arquivos YAML com `echo >>` ou `tee -a`, sempre verificar se a chave já existe antes.

---

## 8 - Conflito entre serviceAccountToken e username no Kibana

**Sintoma:**
```
FATAL Error: serviceAccountToken cannot be specified when "username" is also set
```

**Causa:** Durante o enrollment inicial do Kibana, foi gravado um `serviceAccountToken` no `kibana.yml`. Depois adicionei `elasticsearch.username` e `elasticsearch.password`, as duas formas de autenticação são mutuamente exclusivas.

**Diagnóstico:**
```
grep -n "serviceAccountToken\|elasticsearch.username\|elasticsearch.password" /etc/kibana/kibana.yml
```

**Solução:** Remover o serviceAccountToken e usar apenas username/password:
```
sudo sed -i '188d' /etc/kibana/kibana.yml  # linha do serviceAccountToken
```

**Lição:** O Kibana aceita dois métodos de autenticação com o Elasticsearch: token de enrollment (automático) ou username/password (manual). Nunca os dois juntos.

---

## 9 - Kibana com "socket hang up" ao conectar no Elasticsearch

**Sintoma:**
```
Unable to retrieve version information from Elasticsearch nodes. socket hang up
Local: 127.0.0.1:XXXXX, Remote: 127.0.0.1:9200
```

**Causa:** A linha `elasticsearch.hosts` estava comentada no `kibana.yml` — o Kibana não sabia onde encontrar o Elasticsearch.

**Diagnóstico:**
```
grep -n "elasticsearch.hosts" /etc/kibana/kibana.yml
# 43:#elasticsearch.hosts: ["http://localhost:9200"]  ← comentada!
```

**Solução:**
```
echo 'elasticsearch.hosts: ["https://localhost:9200"]' | sudo tee -a /etc/kibana/kibana.yml
sudo systemctl restart kibana
```

**Lição:** O `kibana.yml` padrão tem quase todas as configurações comentadas. É fácil achar que algo está configurado quando na verdade está comentado.

---

## 10 - Swap excessivo causando lentidão geral

**Sintoma:** Kibana com timeout de 48 segundos no event loop, requisições expirando, interface travando.

**Diagnóstico:**
```
free -h
# Swap: 4.0Gi  1.4Gi usado
# RAM: apenas 78Mi disponível
```

**Causa:** Com Elasticsearch + Logstash + Kibana rodando simultaneamente, a RAM de 1GB foi completamente esgotada e o sistema passou a usar intensamente o swap, que é muito mais lento.

**Solução aplicada:** Parar o Logstash permanentemente e usar Filebeat direto para o Elasticsearch como comentei anteriormente:
```
sudo systemctl stop logstash
sudo systemctl disable logstash
sudo sync && echo 3 | sudo tee /proc/sys/vm/drop_caches
```

**Recomendação para produção:**

| Instância | RAM | Viável para |
|---|---|---|
| t3.micro | 1GB | Filebeat + Elasticsearch apenas |
| t3.small | 2GB | Filebeat + Elasticsearch + Kibana |
| t3.medium | 4GB | Stack ELK completo com Logstash | ← Use esta no mínimo.

---

## 11 - Data View não criado com padrão errado

**Sintoma:** Ao criar o Data View no Kibana com o padrão `apache-logs-*`, aparecia erro "doesn't match any data streams, indices, or index aliases".

**Causa:** Com o Logstash desabilitado, os logs foram indexados pelo Filebeat no índice `filebeat-8.19.16` (data stream), não no índice `apache-logs-*` que era criado pelo Logstash.

**Solução:** Verificar os índices existentes e usar o padrão correto:
```
curl -k -u elastic:<SENHA> "https://localhost:9200/_cat/indices?v&h=index,docs.count"
```

Usar o padrão `filebeat-*` no Data View do Kibana.

**Lição:** O nome do índice depende de qual componente está fazendo a ingestão. Logstash e Filebeat criam índices com nomes diferentes por padrão.

---

## Resumo dos problemas por categoria

| Categoria | Quantidade | Principais causas |
|---|---|---|
| Memória insuficiente | 3 | t3.micro com 1GB RAM |
| Configuração YAML | 3 | Chaves duplicadas, comentadas ou conflitantes |
| Autenticação | 2 | Credenciais perdidas, métodos conflitantes |
| Dependências de serviço | 2 | Ordem de inicialização, serviços parados |
| Compatibilidade de versão | 1 | ECS v8 mudou comportamento do GeoIP |
| Permissões | 1 | Certificado SSL sem leitura para usuário comum |

---

## Tempo total de execução

| Etapa | Tempo estimado | Tempo real |
|---|---|---|
| Setup AWS + SSH | 30 min | ~30 min |
| Instalação ELK | 45 min | ~2h (problemas de memória e config) |
| Apache + Filebeat | 30 min | ~1h30 (Logstash inviável) |
| Kibana Dashboard | 20 min | ~1h (timeouts e erros de config) |
| **Total** | **~2h** | **~5h** |

> O tempo real foi 2.5x maior que o estimado, principalmente devido às limitações de hardware do t3.micro e aos problemas de configuração do kibana.yml que foram se acumulando.

---

*Documentar erros é tão importante quanto documentar sucessos. Esses problemas são reais e definitivamente fazem parte do aprendizado.*
