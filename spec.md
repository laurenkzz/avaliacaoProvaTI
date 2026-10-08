 Base URL: http://localhost:{PORTA_SERVICO}

## UC1 — Abrir bilhete

`POST /bilhetes` — body `{"placa": "ABC1D23"}` (7 caracteres alfanuméricos, maiúsculos) → `201`

```json`
{"id": 1, "placa": "ABC1D23", "entrada": "<ISO-8601 com fuso -03:00>", "status": "aberto"}

O body aceita entrada opcional (ISO-8601 com fuso). Quando informada, o bilhete abre naquele instante em vez de "agora", permitindo testes determinísticos de fração e teto. Formato inválido → 422 {"erro": "entrada_invalida"}.

UC2 — Encerrar bilhete

POST /bilhetes/{id}/encerramento → 200

Bilhetes já encerrados ou que não estejam abertos não podem ser encerrados → 409.

{"id": 1, "placa": "ABC1D23", "entrada": "...", "saida": "...", "minutos": 95, "valor_centavos": 1250}

Bilhete inexistente → 404.

UC3 — Listar ativos

GET /bilhetes/ativos → 200

Retorna somente bilhetes com status "aberto".

UC4 — Relatório diário

GET /relatorios/diario?data=AAAA-MM-DD → 200

Data fora do padrão → 422.

{"data": "2026-10-05", "total_bilhetes": 12, "faturamento_centavos": 8400, "tempo_medio_minutos": 47}
UC5 — Cancelar bilhete

POST /bilhetes/{id}/cancelamento → 200

Altera o status para "cancelado".

Bilhetes encerrados ou que não estejam abertos não podem ser cancelados → 409.

Bilhete inexistente → 404.

UC6 — Histórico por placa

GET /bilhetes?placa=ABC1D23 → 200

Retorna o histórico de bilhetes da placa.

Placa ausente → 422.

Placa inválida → 422.

UC8 — Uma vaga por placa

POST /bilhetes para placa que já possui bilhete aberto → 409 {"erro": "bilhete_em_aberto"}.

Após encerrar ou cancelar o bilhete, a placa volta a poder abrir um novo bilhete.
