# Plan - Decisões técnicas e arquitetura

## Contrato de Uso
| Método e rota | Descrição | Sucesso | Erro |
|---|---|---|---|
| `POST /bilhetes` | Abrir bilhete | 201 | 422 `placa_invalida` / `entrada_invalida`, 409 `bilhete_em_aberto` |
| `POST /bilhetes/{id}/encerramento` | Encerrar bilhete | 200 | 404 `bilhete_nao_encontrado`, 409 `bilhete_ja_encerrado` |
| `GET /bilhetes/ativos` | Listar abertos | 200 (array) | — |
| `GET /relatorios/diario?data=AAAA-MM-DD` | Relatório do dia | 200 | 422 `data_invalida` |
| `POST /bilhetes/{id}/cancelamento` | Cancelar bilhete | 200 | 404 `bilhete_nao_encontrado`, 409 `bilhete_nao_aberto` |
| `GET /bilhetes?placa=` | Histórico da placa | 200 (array) | 422 `placa_invalida` |