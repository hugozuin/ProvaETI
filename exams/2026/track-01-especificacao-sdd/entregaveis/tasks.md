# Tasks - Decomposição
As tarefas devem ser feitas nesta ordem, seguindo as regras do constitution.md. Em cada uma, primeiro escreva os testes indicados do tests.md e depois o código, até eles passarem.

1. Base do projeto: criar app/main.py com as constantes da variante, o fuso -03:00, o armazenamento em memória, a função resetar_armazenamento() e uma função que monta as respostas de erro no formato {"erro": ...}. Criar app/test_main.py com o TestClient e o reset antes de cada teste.

2. Abrir bilhete (UC1): validar a placa e a entrada e impedir dois bilhetes abertos para a mesma placa, sempre verificando o formato antes do conflito. Testes T01 a T07.

3. Encerrar bilhete (UC2): calcular os minutos, aplicar a tolerância, as frações, o valor inteiro e o teto, e tratar os erros 404 e 409. Testes T08 a T15.

4. Cancelar bilhete (UC5): permitir cancelar só bilhetes abertos, sem gerar cobrança. Testes T22 e T23.

5. Listagens (UC3 e UC6): implementar os ativos e o histórico por placa, do mais recente para o mais antigo. A rota de ativos deve ser registrada antes das rotas com {id}. Testes T16, T17, T24 e T25.

6. Relatório diário (UC4): somar os encerrados do dia e calcular o tempo médio arredondando 0,5 para cima. Testes T18 a T21.

7. Verificação final: rodar o pytest com os 25 testes passando e conferir que nenhuma resposta tem número decimal nem o formato {"detail"}.
