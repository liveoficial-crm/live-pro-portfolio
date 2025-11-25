# Projeto de experiência e jornada do cliente:

## Etapas Projeto:

### Etapa atual:
- **1 - Histórico Consolidado**

**Próximas Etapas:**
- **2 - Criação da Fila**
- **3 - Leitura e Tratamento dos dados**
- **4 - Consolidação da ficha unificada**
- **5 - Método de validação**
- **6 - Sincronização em todos os sistemas**


## 1° Etapa - Histórico Consolidado e Fila Inicial

### Validação de Payload

#### Campos Obrigatórios para Chaveamento

Na validação inicial, o payload deve obrigatoriamente conter:
- **Sistema de origem** (`source`)
- **Tipo de interação** (`interaction_type`) 

**Pelo menos um dos seguintes campos prioritários** (definirá a chave do registro), **Ordem de Prioridade (do mais alto ao mais baixo):**
1. **Documento** - Documento Internacionalizado dos clientes (Prioridade Máxima)
2. **Celular** - DDI + DDD + número (sempre usar "phone", nunca "telefone")
3. **Email** - endereço de email válido
4. **Nome** - Nome completo (Fallback - apenas para ter registro básico)

> ⚠️ **Importante:** Nome **NUNCA** pode ser usado como chave principal. Ele serve apenas como fallback para identificar que existiu um registro quando nenhum outro campo estiver disponível.

#### Estrutura do Payload

**Exemplo Payload de entrada na API:**
```json
{
  "data": {
    "source": "e-commerce|cigam|omnichat|zendesk",
    "interaction_type": "purchase|exchange|lead|other",
    "document": "703.345.961-06", // Pode vir tratado ou não api irá tratar depois
    "phone": "+55 (47) 99999-9999", // Pode vir tratado ou não api irá tratar depois
    "email": "email@email.com",
    "nome": "José Silva",
    "birthday": "2001/07/04",
    "gender": "M",
    "profession": "Painter",
    "address": "Av Paraná, 370 - Centro, Lucas do Rio Verde, MT, Brasil - 78455-000"
    // Outros campos adicionais...
  }
}
```

**Registro armazenado no banco:**
```json
{
  "_id": "691cbc55faa197f9263490a4", // Auto-gerado pelo MongoDB
  "key": "document|phone|email",
  "data": {
    "document": "70334596106",
    "phone": "5547999999999", 
    "email": "email@email.com",
    
    "additional_fields": {
      "nome": "José Silva",
      "birthday": "2001-07-04",
      "gender": "M",
      "profession": "Painter",
      "address": "Av Paraná, 370 - Centro, Lucas do Rio Verde, MT, Brasil - 78455-000"
    }
  },

  "metadata": {
    "source": "e-commerce|cigam|omnichat|zendesk",
    "interaction_type": "purchase|exchange|lead|other",
    "interacted_at": "2025-11-01T11:47:21.000+00:00"
  }
}
```

#### Lógica de Validação

1. **Detecção da Chave:** O primeiro campo válido encontrado na ordem de prioridade determina a `key` do registro
2. **Validação de Campos:** O payload deve conter pelo menos um dos campos obrigatórios (documento, celular, email)
3. **Campos Adicionais:** Todos os campos extras podem ser livremente registrados em `additional_fields` (nesta etapa os dados entram "sujos")
4. **Registro Inválido:** Caso nenhum campo obrigatório seja encontrado, o payload é marcado como inválido

### Criação de Task para Workers

Após a validação e armazenamento, uma task é automaticamente criada na fila para os workers do serviço com os seguintes objetivos:

- **Validar** os dados postados no payload
- **Enriquecer** os dados consultando fontes externas (tabela `sources` com coluna `enrich_data = true`)

### Exemplo:

| id  | status | name           | enrich_data |
|-----|--------|----------------|-------------|
| 1   | true   | Unknown        | false       |
| 2   | true   | LIVE! Pro API  | false       |
| 3   | true   | Manual Entry   | false       |
| 4   | true   | CSV Import     | false       |
| 5   | true   | Systextil      | false       |
| 6   | true   | Cigam          | true        |
| 7   | true   | E-commerce     | true        |
| 8   | true   | App E-commerce | true        |
| 9   | true   | Shoplive       | false       |
| 10  | true   | App Experience | false       |
| 11  | true   | LIVE! Pro      | true        |
| 12  | true   | Omnichat       | true        |
| 13  | true   | Zendesk        | true        |
| 100 | true   | Dito           | false       |

### Fluxo de Exemplo

**Cenário:** Novo cadastro na Cigam

1. **Dados disponíveis postados:**
```json
{
  "data": {
    "source": "cigam",
    "interaction_type": "purchase",
    "document": "11.222.333/0001-01"
  }
}
```

2. **Validação:** Payload é aceito (possui documento válido)
3. **Armazenamento:** Registro criado no histórico com `key: "document"`
4. **Worker acionado:** Task criada para buscar dados adicionais na Cigam
5. **Enriquecimento:** Worker acessa API da Cigam e complementa o registro

### Próximos Passos - Regras de Consolidação

Nessa proxima etapa após consolidar o histórico validaremos informações críticas e elaboraremos as regras para unificação da ficha cadastral no banco.

**Linha de Pensamento inicial** - Regras de Prioridade para Consolidação:

1. **Documento (Prioridade Máxima)**
   - Identificador principal (CPF)
   - Precisa bater exatamente
   - Dado mais confiável para identificação

2. **Celular + Email (Alta Confiabilidade)**
   - Ambos precisam coincidir
   - Desempate: documento (decrescente) e ID (decrescente)

3. **Celular + Nome (Confiabilidade Moderada)**
   - Telefone e nome devem coincidir
   - Desempate: documento (desc), email (desc), ID (desc)

4. **Email + Nome (Confiabilidade Moderada)**
   - Email e nome devem coincidir
   - Desempate: documento (desc), celular (desc), ID (desc)

5. **Apenas Nome (Prioridade Baixa)**
   - Nome igual, mas apenas quando não há documento
   - Evita erros com nomes comuns
   - Usado apenas se nenhum outro identificador estiver disponível

#### Exemplo de Consolidação

**Situação:** Lead com cadastro básico → Cliente com cadastro completo

**Registro Consolidado Inicial (Lead):**
| id | merged_into_id | status | lead | document | document_format | name | first_name | birthday | gender | phone | email | source_id | update_source_id | created_at | updated_at | deleted_at |
|----|----------------|--------|------|----------|-----------------|------|------------|----------|--------|-------|-------|---------|------------------|------------|------------|------------|
| 123 | NULL | true | true | NULL | NULL | José Silva | José | 2001-07-04 | M | 5547999999999 | email@email.com | 1 | 1 | 2025-11-01 10:00:00 | 2025-11-01 10:00:00 | NULL |

**Novo Registro (Pós-Venda):**

| id | merged_into_id | status | lead | document | document_format | name | first_name | birthday | gender | phone | email | source_id | update_source_id | created_at | updated_at | deleted_at |
|----|----------------|--------|------|----------|-----------------|------|------------|----------|--------|-------|-------|---------|------------------|------------|------------|------------|
| 123 | 456 | false | true | NULL | NULL | José Silva | José | 2001-07-04 | M | 5547999999999 | email@email.com | 1 | 1 | 2025-11-01 10:00:00 | 2025-11-01 10:00:00 | NULL |
| 456 | NULL | true | false | 70334596106 | CPF | José Silva | José | 2001-07-04 | M | 5547999999999 | email@email.com | 2 | 2 | 2025-11-01 11:00:00 | 2025-11-01 11:00:00 | NULL |

**Resultado:** As regras identificam a similaridade (celular + email coincidem) e consolidam os registros, atualizando o status de lead para cliente.

> 💡 **Nota:** As regras acima servem como linha de pensamento inicial. Com o histórico consolidado, poderemos elaborar regras mais precisas e eficazes.


<!-- # 2° Etapa - Fila de processamento

## Arquitetura da Fila

### Estrutura da Fila
- **Fila Principal**: Ordem cronológica de entrada dos payloads validados
- **Fila de Prioridade**: Processamento acelerado para eventos críticos
- **Fila de Erro**: Reprocessamento de payloads com falhas

### Mecanismo de Processamento
- **FIFO (First In, First Out)**: Processamento na ordem de chegada para fila principal
- **Priorização**: Eventos urgentes pulam para fila de prioridade, e.g: cadastro no e-commerce
- **Retry Logic**: Tentativas automáticas de reprocessamento em casos de falhas
- **Rate Limiting**: Controle de volume para evitar sobrecarga de sistemas

### Monitoramento e Controle
- **Dashboard de Fila**: Visualização em tempo real do status de processamento
- **Alertas Automáticos**: Notificação em caso de acúmulo ou falhas
- **Métricas de Performance**: Tempo médio de processamento e throughput

# 3° Etapa - Leitura e Tratamento de Dados

## Normalização de Dados

### Padronização de Formatos
- **Documentos**: Formatação e validação de documento, nacional e internacional
- **Telefones**: limpeza de caracteres não númericos
- **Emails**: Validação de formato e limpeza de caracteres
- **Datas**: Padronização para formato ISO 8601

### Limpeza de Dados
- **Remoção de Duplicatas**: Identificação e eliminação de registros repetidos
- **Correção de Inconsistências**: Padronização de valores similares
- **Tratamento de Valores Nulos**: Estratégias específicas para campos vazios

### Enriquecimento de Dados
- **Geolocalização**: Determinação de cidade/estado baseado em CEP/telefone
- **Demografia**: Extração de informações de idade, gênero quando disponível
- **Categorização**: Classificação automática de tipos de interação

## Validação de Integridade
- **Verificação de Chaves**: Confirmação da consistência das main keys
- **Validação Cruzada**: Confirmação entre diferentes fontes de informação
- **Auditoria de Alterações**: Log de todas as modificações realizadas

---

# 4° Etapa - Consolidação da Ficha Unificada

## Criação do Perfil Único

### Algoritmo de Unificação
- **Match por Documento**: Unificação direta quando documentos coincidem
- **Match por Telefone**: Consolidação baseada em números de telefone
- **Match por Email**: Unificação através de endereços de email
- **Fuzzy Matching**: Algoritmos para casos aproximados e similares

### Resolução de Conflitos
- **Prioridade de Dados**: Estratégia para dados conflitantes entre fontes
- **Merge Inteligente**: Preservação da informação mais recente/precisa
- **Flag de Auditoria**: Marcação de registros que passaram por merge

---

# 5° Etapa - Sistema de Validação

## Criação de Métodos para Validação

### Validação Automática
- **Regras de Negócio**: Verificação contra políticas específicas
- **Validação de Referência**: Checagem contra bases de dados externas
- **Análise de Consistência**: Detecção de anomalias nos dados

### Validação Semiautomática
- **Confirmação do cliente**: Em casos especificos enviar notificação ao cliente solicitando confirmação cadastral
- **Flag de Supervisão**: Marcação de registros que precisam revisão humana
- **Score de Confiança**: Indicador de qualidade da consolidação
- **Queue de Revisão**: Interface para validação manual quando necessária

### Métricas de Qualidade
- **Taxa de Sucesso**: Percentual de payloads processados com sucesso
- **Tempo de Processamento**: Velocidade média de consolidação
- **Taxa de Erro**: Frequência de falhas no pipeline
- **Satisfação de Dados**: Indicador de completude e precisão

---

# 6° Etapa - Sincronização Multisistema

## Distribuição de Dados

### Sistemas Destino
- **Zendesk**: Dados de suporte e atendimento
- **Omnichat**: Informações de chat e comunicação
- **LIVE! Pro**: Dados de visitação e retorno à loja
- **E-commerce**: Informações de compra e navegação
- **Experience**: Dados de eventos e campanhas

### Mecanismo de Sincronização
- **Real-time**: Atualização imediata para eventos críticos
- **Batch**: Sincronização programada para dados não urgentes
- **Delta Sync**: Envio apenas das alterações (mudanças incrementais)

### Garantias de Consitência
- **Two-Phase Commit**: Garantia de transação atômica
- **Rollback Mechanism**: Reversão automática em caso de falha
- **Retry Logic**: Tentativas inteligentes de reenvio
- **Dead Letter Queue**: Tratamento de falhas persistentes

### Monitoramento de Sincronização
- **Health Checks**: Verificação contínua do status de cada sistema
- **Delay Metrics**: Monitoramento de latência na distribuição
- **Success/Failure Rates**: Taxas de sucesso por sistema de destino

---

## Próximos Passos e Considerações

### Implementação
1. **Desenvolvimento Modular**: Implementação etapa por etapa
2. **Testes Isolados**: Validação de cada componente separadamente
3. **Integração Progressiva**: Testes de integração incrementais
4. **Rollout Gradual**: Implementação em ambientes de menor para maior criticidade

### Monitoramento e Otimização
- **Dashboards Executivos**: Visão geral do performance do sistema
- **Alertas Proativos**: Notificação de problemas antes do impacto ao usuário
- **Otimização Contínua**: Melhorias baseadas em métricas e feedback -->
