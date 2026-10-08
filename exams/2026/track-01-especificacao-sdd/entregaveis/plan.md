# Plan - Decisões técnicas e arquitetura
A API expõe as rotas POST /bilhetes, POST /bilhetes/{id}/encerramento, POST /bilhetes/{id}/cancelamento, GET /bilhetes/ativos, GET /bilhetes?placa= e GET /relatorios/diario?data=, com campos em português, erros no formato {"erro": "codigo"} e serviço na porta PORTA.

## Stack
Python 3.11 com FastAPI e uvicorn, porque o contrato REST fica simples de expressar e o FastAPI já tem um TestClient para os testes. Os dados ficam num dicionário em memória: o enunciado não pede persistência, e assim o container sobe sem depender de banco. Os testes usam pytest com httpx.


## Contrato de Uso
| Método e rota | Descrição | Sucesso | Erro |
|---|---|---|---|
| `POST /bilhetes` | Abrir bilhete | 201 | 422 `placa_invalida` / `entrada_invalida`, 409 `bilhete_em_aberto` |
| `POST /bilhetes/{id}/encerramento` | Encerrar bilhete | 200 | 404 `bilhete_nao_encontrado`, 409 `bilhete_ja_encerrado` |
| `GET /bilhetes/ativos` | Listar abertos | 200 (array) | — |
| `GET /relatorios/diario?data=AAAA-MM-DD` | Relatório do dia | 200 | 422 `data_invalida` |
| `POST /bilhetes/{id}/cancelamento` | Cancelar bilhete | 200 | 404 `bilhete_nao_encontrado`, 409 `bilhete_nao_aberto` |
| `GET /bilhetes?placa=` | Histórico da placa | 200 (array) | 422 `placa_invalida` |

## Decisões
1. Dinheiro como inteiro em centavos em todo o cálculo. Ponto flutuante acumula erro, e o contrato exige valor_centavos inteiro.

2. Divisões arredondadas para cima feitas com inteiros. Assim o cálculo de frações e valor não passa por float. Isso importa quando o valor da fração não é exato, por exemplo uma tarifa de 450 dividida em 4 frações.

3. Minutos calculados como (saida - entrada) / 60. A suíte abre o bilhete no passado e encerra logo em seguida, se os milissegundos desse intervalo fossem arredondados para cima, uma fração exata viraria a fração seguinte.

4. Horário atual com fuso fixo, usando datetime.now, sem microssegundos. O container roda em UTC e o contrato exige -03:00.

5. O corpo do POST /bilhetes é lido como JSON bruto e validado manualmente, em vez de usar um modelo Pydantic. A validação automática do FastAPI responde {"detail": [...]}, e o contrato exige {"erro": "placa_invalida"} ou {"erro": "entrada_invalida"}.

6. O id das rotas é recebido como texto. Se não for número ou não existir, a resposta é 404. Isso evita o 422 automático que o FastAPI daria para um id inválido.

7. O tempo médio do relatório é calculado como (2 x soma + quantidade) / (2 × quantidade), que arredonda 0,5 para cima.

8. A rota GET /bilhetes/ativos é registrada antes das rotas com {id}, para que "ativos" não seja interpretado como id.

9. Uma função resetar_armazenamento() limpa os dados e reinicia o contador de ids. Ela é chamada antes de cada teste, para os testes não dependerem uns dos outros.
