# Resumo do Projeto - Serviço de Audiências

## Visão Geral
Sistema para gerenciamento de audiências (listas de contatos) com interface de painel administrativo e API REST.

## Funcionalidades Principais

### 📊 Painel Administrativo

**Listagem de Audiências** (`/audiences`)
- Exibe tabela com metadados: Nome, Contatos, Data de Criação e Atualização
- Implementa soft-delete (registros não são removidos permanentemente)
- ⚠️ Lógica de limpeza no MongoDB precisa de revisão

**Detalhes da Audiência** (`/audiences/{id}`)
- Visualização individual de audiências
- Funcionalidade de renomeação (planejada)

**Reciclagem** (`/audiences/recycle`)
- ⚠️ Limitação atual: trava ao excluir mais de 10 mil linhas

## API
**Apenas audiências criadas via API podem ser alteradas via os comandos abaixo.**

### /list `GET`
- [ ] Mostra o resumo da tabela `public.audiences`, analogo ao painel:
<br><sup> Mostra todo o conteúdo da tabela sem filtros.
	<br> `id`, `name`, `contacts`, `source_id`, `collection`, `created_at`, `updated_at`, `deleted_at`

### /list/{audience_id}?page=1 `GET`
- [ ] Lista o conteúdo da audiencia de forma paginada.

### /create `POST`
- [ ] Cria uma nova audiencia com nome customizado.
	```json
	'payload': {
		"name": "Lista - clientes",
		"data": [
			{
				"document": "123.456.789-00",
				"name": "Nome do Contato",
				"phone": "5511999999999",
				"email": "email@example.example"
			}
		],
	},
	```

### /insert `POST`
- [ ] insere um novo contato dentro da audiencia.
	<br><sup> contatos inseridos são unicos pelo documento, a audiencia manterá apenas ultimo registro do mesmo documento inserido afim de não incorrer em duplicatas.
	```json
	'payload': {
			"snowflake_id": "104629547441512448",
			"data": [
					{
							"document": "000.000.000-00",
							"name": "Nomeado",
							"phone": "5511999999992",
							"email": "email@example.example"
					}
			],
	}
	```

### /delete `DELETE`
- [ ] Realiza a limpeza de todos os contatos dentro de uma audiencia.
	```json
	'payload': {
		"snowflake_id": "104174327926243328",
	}
	```

### /drop `DELETE`
- [ ] Realiza o soft-delete da audiência para ela não aparecer na visão do painel.
	```json
	'payload': {
		"snowflake_id": "104174327926243328",
	}
	```

### /restore `POST`
- [ ] Remove a flag deleted_at de uma lista, permitindo utilizar ela novamente.
	```json
	'payload': {
		"snowflake_id": "104174327926243328",
	}
	```
