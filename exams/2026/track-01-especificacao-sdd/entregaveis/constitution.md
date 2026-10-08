# Constitution - Regras Operacionais
Este projeto é uma API REST chamada Zona Azul Digital, que controla bilhetes de estacionamento rotativo. Ela abre, encerra e cancela bilhetes por placa, lista os bilhetes ativos, mostra o histórico de uma placa e gera um relatório diário. Só a API faz parte do escopo, sem tela nem back-office.

## Stack
Python 3.11 com FastAPI e uvicorn, com os dados guardados em memória (sem banco de dados). Os testes usam pytest e o TestClient do FastAPI, que depende do httpx.

## Contrato de Uso
| Método e rota | Descrição | Sucesso | Erro |
|---|---|---|---|
| `POST /bilhetes` | Abrir bilhete | 201 | 422 `placa_invalida` / `entrada_invalida`, 409 `bilhete_em_aberto` |
| `POST /bilhetes/{id}/encerramento` | Encerrar bilhete | 200 | 404 `bilhete_nao_encontrado`, 409 `bilhete_ja_encerrado` |
| `GET /bilhetes/ativos` | Listar abertos | 200 (array) | — |
| `GET /relatorios/diario?data=AAAA-MM-DD` | Relatório do dia | 200 | 422 `data_invalida` |
| `POST /bilhetes/{id}/cancelamento` | Cancelar bilhete | 200 | 404 `bilhete_nao_encontrado`, 409 `bilhete_nao_aberto` |
| `GET /bilhetes?placa=` | Histórico da placa | 200 (array) | 422 `placa_invalida` |

## Parâmetros da variante
Estes valores são fixos e ficam declarados uma única vez, como constantes no topo do código:

- TARIFA_HORA_CENTAVOS = TARIFA
- FRACAO_MINUTOS = FRACAO
- TETO_DIARIO_CENTAVOS = TETO
- TOLERANCIA_MINUTOS = TOLERANCIA
- PORTA_SERVICO = 9203

## Regras que valem para todo o código
1. Rotas, campos JSON e códigos de erro são escritos exatamente como no contrato, em português e com underscore: placa, entrada, saida, status, minutos, valor_centavos, total_bilhetes, faturamento_centavos, tempo_medio_minutos. Não traduzir para inglês nem usar camelCase.
2. Toda resposta de erro tem o corpo {"erro": "codigo_do_erro"}. Nunca usar o formato padrão do FastAPI, {"detail": ...}.
3. Erro de formato (placa, entrada ou data inválida) responde 422 e é sempre verificado antes das regras de negócio que respondem 409.
4. Valores em dinheiro são sempre inteiros em centavos. Nenhuma resposta pode ter número com casas decimais.
5. Datas e horários seguem a ISO-8601 com fuso -03:00 e sem microssegundos, por exemplo 2026-10-12T08:30:00-03:00. O "agora" do sistema é calculado no fuso fixo -03:00, independentemente do fuso do container.
6. Os IDs são inteiros sequenciais a partir de 1.
7. O enunciado traz um exemplo antigo de resposta, {"id": 7, "valor": 12.50}, que está errado de propósito. O contrato é o que vale.
