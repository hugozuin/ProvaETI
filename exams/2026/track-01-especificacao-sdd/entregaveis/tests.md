# Tests - Cenários de Teste

## Abrir bilhete
- T01: abrir com placa ABC1D23 retorna 201, status aberto e entrada terminando em -03:00.
- T02: abrir com entrada 2026-10-12T08:30:00-03:00 retorna 201 com essa mesma entrada.
- T03: placas abc1d23, ABC1D2, ABC-123 ou ausente retornam 422 placa_invalida.
- T04: entrada "ontem" retorna 422 entrada_invalida.
- T05: abrir a mesma placa duas vezes faz a segunda retornar 409 bilhete_em_aberto.
- T06: com a placa já aberta, uma nova abertura com entrada inválida retorna 422, porque o formato é validado antes do conflito.
- T07: depois de encerrar, a mesma placa abre de novo com 201.

## Encerrar e calcular valor
- T08: um bilhete de 2F minutos encerra com minutos igual a 2F e o valor de 2 frações.
- T09: um bilhete de 2F + 1 minutos cobra o valor de 3 frações.
- T10: um bilhete de exatamente TOL minutos cobra 0.
- T11: um bilhete de TOL + 1 minutos cobra as frações desde o minuto zero, com valor maior que 0.
- T12: um bilhete de 24 horas cobra exatamente TETO.
- T13: valor_centavos é inteiro, e a resposta não tem um campo chamado valor.
- T14: encerrar o mesmo bilhete duas vezes retorna 409 bilhete_ja_encerrado.
- T15: encerrar o id 999999 retorna 404 bilhete_nao_encontrado.

## Listar ativos
- T16: com dois bilhetes abertos e um deles encerrado, só o aberto aparece.
- T17: um bilhete aberto há 10 minutos aparece antes de um aberto há 60 minutos.

## Relatório diário
- T18: depois de encerrar 2 bilhetes hoje, o relatório de hoje mostra total_bilhetes 2 e o faturamento igual à soma dos valores.
- T19: com bilhetes de 30 e 31 minutos, tempo_medio_minutos é 31.
- T20: o relatório de 2000-01-01 retorna tudo zerado.
- T21: data=05-10-2026 retorna 422 data_invalida.

## Cancelar
- T22: cancelar um bilhete aberto retorna 200 com status cancelado e sem valor_centavos, e ele some dos ativos.
- T23 : cancelar duas vezes, ou cancelar um encerrado, retorna 409 bilhete_nao_aberto.

## Histórico por placa
- T24: uma placa com um bilhete encerrado, um cancelado e um aberto retorna os 3, do mais recente para o mais antigo.
- T25: uma placa que nunca estacionou retorna []; placa=abc retorna 422 placa_invalida.
