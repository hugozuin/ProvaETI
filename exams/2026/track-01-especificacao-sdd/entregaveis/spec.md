# Spec - Zona Azul Digital

> 

## Contrato de Uso
| Método e rota | Descrição | Sucesso | Erro |
|---|---|---|---|
| `POST /bilhetes` | Abrir bilhete | 201 | 422 `placa_invalida` / `entrada_invalida`, 409 `bilhete_em_aberto` |
| `POST /bilhetes/{id}/encerramento` | Encerrar bilhete | 200 | 404 `bilhete_nao_encontrado`, 409 `bilhete_ja_encerrado` |
| `GET /bilhetes/ativos` | Listar abertos | 200 (array) | — |
| `GET /relatorios/diario?data=AAAA-MM-DD` | Relatório do dia | 200 | 422 `data_invalida` |
| `POST /bilhetes/{id}/cancelamento` | Cancelar bilhete | 200 | 404 `bilhete_nao_encontrado`, 409 `bilhete_nao_aberto` |
| `GET /bilhetes?placa=` | Histórico da placa | 200 (array) | 422 `placa_invalida` |


## Casos de Uso

### UC1 - Abrir bilhete

O corpo recebe a placa e, opcionalmente, o horário de entrada, por exemplo {"placa": "ABC1D23", "entrada": "2026-10-12T08:30:00-03:00"}.

A placa é obrigatória e precisa ter exatamente 7 caracteres, só letras maiúsculas de A a Z e números. Placa com letra minúscula, hífen ou outro tamanho é inválida e não deve ser corrigida automaticamente. A entrada é opcional: se não vier, o bilhete abre com o horário atual. Se vier, precisa ser uma data ISO-8601 válida.

As validações acontecem nesta ordem: primeiro a placa, depois a entrada e, por último, a regra de uma placa aberta por vez.

Critérios de aceite:
- {"placa": "ABC1D23"} retorna 201 com {"id": 1, "placa": "ABC1D23", "entrada": "...-03:00", "status": "aberto"}.
- Se a entrada for enviada, o bilhete abre exatamente naquele horário.
- Placa ausente ou fora do formato (abc1d23, ABC1D2, ABC-123) retorna 422 {"erro": "placa_invalida"}.
- Entrada que não é ISO-8601, como "ontem", retorna 422 {"erro": "entrada_invalida"}.
- Abrir uma placa que já tem bilhete aberto retorna 409 {"erro": "bilhete_em_aberto"}. Se a mesma requisição também tiver erro de formato, responde 422.
- Depois de encerrar ou cancelar, a mesma placa pode abrir outro bilhete (201).

### UC2 - Encerrar bilhete

Não recebe corpo. A saída é o horário atual e o valor é calculado assim:

1. Os minutos são os minutos completos entre a entrada e a saída, ou seja, os segundos divididos por 60 com arredondamento para baixo.
2. Se os minutos forem menores ou iguais à tolerância, o valor é 0. Se passarem da tolerância, nem que seja por 1 minuto, o tempo é cobrado inteiro desde o minuto zero, sem descontar a tolerância.
3. O tempo é cobrado em frações de FRACAO_MINUTOS, sempre arredondando para cima: frações = teto(minutos / FRACAO_MINUTOS). Uma fração exata cobra 1 fração; 1 minuto a mais já cobra a próxima.
4. O valor é teto(frações × TARIFA_HORA_CENTAVOS × FRACAO_MINUTOS / 60), calculado com inteiros.
5. Por fim, aplica-se o teto: o valor nunca passa de TETO_DIARIO_CENTAVOS.

Critérios de aceite:
- Encerrar um bilhete aberto retorna 200 com id, placa, entrada, saida, minutos, valor_centavos e status "encerrado".
- Um bilhete de 2 × FRACAO_MINUTOS minutos cobra 2 frações; com 1 minuto a mais, cobra 3.
- Um bilhete de 24 horas cobra exatamente TETO_DIARIO_CENTAVOS.
- valor_centavos é sempre um número inteiro.
- Id inexistente ou não numérico retorna 404 {"erro": "bilhete_nao_encontrado"}.
- Encerrar um bilhete já encerrado retorna 409 {"erro": "bilhete_ja_encerrado"}; encerrar um cancelado retorna 409 {"erro": "bilhete_nao_aberto"}.

### UC3 - Listar ativos

Retorna só os bilhetes com status aberto, do mais recente para o mais antigo. A ordem é pela entrada, e, quando a entrada empata, o maior id vem primeiro.

Critérios de aceite:
- Bilhetes encerrados ou cancelados não aparecem.
- Entre um bilhete aberto há 60 minutos e outro aberto há 10 minutos, o de 10 minutos vem primeiro.
- Sem bilhetes abertos, retorna 200 com [].

### UC4 - Relatório diário

Considera apenas os bilhetes encerrados cuja saída aconteceu na data pedida. Abertos e cancelados ficam de fora. A resposta tem o formato {"data": "2026-10-05", "total_bilhetes": 12, "faturamento_centavos": 8400, "tempo_medio_minutos": 47}.

O tempo médio arredonda 0,5 para cima. Por isso não deve usar o round() do Python, que arredonda 0,5 para o número par.

Critérios de aceite:
- total_bilhetes é a quantidade de encerrados no dia, e faturamento_centavos é a soma dos valores, como inteiro.
- Com dois bilhetes de 30 e 31 minutos, o tempo médio é 31.
- Um dia sem encerramentos retorna 0 em total_bilhetes, faturamento_centavos e tempo_medio_minutos.
- Data ausente ou inválida (05-10-2026, 2026-13-45) retorna 422 {"erro": "data_invalida"}.

### UC5 - Cancelar bilhete

Só um bilhete aberto pode ser cancelado, e o cancelamento não gera cobrança.

Critérios de aceite:
- Cancelar um bilhete aberto retorna 200 com status "cancelado", sem saida, minutos e valor_centavos, e ele deixa de aparecer nos ativos.
- Cancelar um bilhete já cancelado ou encerrado retorna 409 {"erro": "bilhete_nao_aberto"}.
- Id inexistente retorna 404 {"erro": "bilhete_nao_encontrado"}.

### UC6 - Histórico por placa

Retorna todos os bilhetes da placa, em qualquer status, do mais recente para o mais antigo, na mesma ordem do UC3.

Critérios de aceite:
- Uma placa com um bilhete encerrado, um cancelado e um aberto retorna os 3.
- Uma placa válida que nunca estacionou retorna 200 com [].
- Placa ausente ou inválida retorna 422 {"erro": "placa_invalida"}.
