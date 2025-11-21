# Projeto de experiência e jornada do cliente:

## Etapas Projeto:
- **1 - Histórico Consolidado (mongoDB)**
- **2 - Criação da Fila**
- **3 - Leitura e Tratamento dos dados**
- **4 - Consolidação da ficha unificada**
- **5 - Método de validação**
- **6 - Sincronização em todos os sistemas**

# 1° Etapa - Histórico Consolidado e Fila Inicial

## Validação de Payload:
### Campos Obrigatórios para Chaveamento

A validação inicial do payload sempre deve conter o nome do sistema de origem e deve verificar a presença dos seguintes campos obrigatórios **nesta ordem de prioridade**:
1. **Documento**
2. **Telefone**
3. **Email**
4. **nome**

### Exemplo Estrutura
**Payload de entrada na API:**
```json
{
	"data": {
	"source": "E-commerce|Cigam|Omnichat|Zendesk...",
	"interaction_type": "Purchase|Exchange|Lead",
	"name": "Nome Completo",
	"document": "CPF/CNPJ",
	"phone": "DDD + número",
	"email": "endereço@email.com",
	"birthday": "2001/07/04",
	"gender": "M",
	"profession": "Painter",
	"address": "Av Parana, 370 - Centro, Lucas do Rio Verde, Mato Grosso, Brasil - 78455-000",
	// Outros campos adicionais...
  }
}
```

**Registro ordenado no banco:**
```json
{
	"_id": "691cbc55faa197f9263490a4", // Auto Gerado
	"key": "documento|telefone|email",
	"data": {
		"document": "CPF/CNPJ",
		"phone": "número",
		"email": "endereço@email.com",

		"additional_fields": {
			"name": "Nome Completo",
			"birthday": "2001/07/04",
			"gender": "M",
			"profession": "Painter",
			"address": "Av Parana, 370 - Centro, Lucas do Rio Verde, Mato Grosso, Brasil - 78455-000",
			// Outros campos adicionais...
		}
	},
	"metadata": {
		"source": "E-commerce|Cigam|Omnichat|Zendesk...",
		"interaction_type": "Purchase|Exchange|Lead",
		"interacted_at": "2025-11-01T11:47:21.000+00:00"
	},
}
```

#### Lógica de Validação.
- O payload deve conter pelo menos um dos campos obrigatórios que serão validados pela ordem listada acima, o primeiro campo válido encontrado definirá a **key** do registro.
- Caso nenhum dos campos obrigatórios seja encontrado, o payload será marcado como **inválido**.
- **campos adicionais** no payload podem ser livremente registrados na API de histórico do sistema, nessa etapa os dados entram "sujos".

#### Criação de Task para Workers
- Após a criação será automaticamente criada uma task direcionada aos workers do serviço
- A task tem como objetivo:
  - Validar todos os dados postados no payload
  - Consultar fontes externas quando necessário para **enriquecer** os dados recebidos
  - Garantir a qualidade e completude das informações processadas

# 2° Etapa - Fila de processamento

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
<!-- 
### Exemplo Ficha Consolidada
```json
{
  
  "data": {
    "nome": "Nome Completo",
    "documento": "CPF/CNPJ",
    "telefone": "DDD + número",
    "email": "endereço@email.com"
  },
  "historico_interacoes": [
    {
      "timestamp": "2025-11-20T19:11:33Z",
      "sistema_origem": "zendesk",
      "tipo_interacao": "suporte",
      "dados_especificos": {}
    }
  ],
  "ultima_atualizacao": "2025-11-20T19:11:33Z",
  "score_engajamento": 0.85
}
``` -->

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
- **Otimização Contínua**: Melhorias baseadas em métricas e feedback
