# Constitution — Zona Azul Digital

## 1. Objetivo

Estabelecer as regras e convenções obrigatórias para a implementação da API REST de bilhetes de estacionamento rotativo (Zona Azul Digital).

Todas as decisões de implementação devem respeitar as regras deste documento e o contrato definido no `ENUNCIADO.md`.

---

## 2. Parâmetros Fixos

Os seguintes valores são obrigatórios e não devem ser alterados:

| Parâmetro | Valor | Significado |
|---|---:|---|
| `TARIFA_HORA_CENTAVOS` | `400` | R$ 4,00 por hora |
| `FRACAO_MINUTOS` | `15` | Fração mínima de cobrança |
| `TETO_DIARIO_CENTAVOS` | `6000` | R$ 60,00 como valor máximo |
| `TOLERANCIA_MINUTOS` | `0` | Não existe período gratuito |
| `PORTA_SERVICO` | `8001` | Porta obrigatória da API |

A aplicação deve executar exclusivamente na porta `8001`.

---

## 3. Valores Monetários

- Todos os valores monetários devem ser representados exclusivamente em centavos.
- Valores monetários devem utilizar números inteiros.
- É proibido utilizar ponto flutuante para representar ou calcular valores monetários.
- A API nunca deve retornar valores monetários com casas decimais.
- Exemplo:
  - R$ 4,00 → `400`
  - R$ 15,00 → `1500`
  - R$ 60,00 → `6000`

### Regra de segurança

> [!WARNING]
> Nunca utilizar `float`, `double` ou equivalente para cálculos monetários. Todos os cálculos devem preservar valores inteiros em centavos.

---

## 4. Cálculo da Cobrança

A cobrança deve utilizar blocos de `15` minutos.

### Valor da fração

A tarifa de uma fração deve ser calculada a partir da tarifa horária:

`valor_fração = TARIFA_HORA_CENTAVOS / (60 / FRACAO_MINUTOS)`

Com os parâmetros atuais:

- Tarifa por hora: `400` centavos
- Fração: `15` minutos
- Quantidade de frações por hora: `4`
- Valor de cada fração: `100` centavos

Portanto:

- 15 minutos → 100 centavos
- 30 minutos → 200 centavos
- 45 minutos → 300 centavos
- 60 minutos → 400 centavos

---

## 5. Arredondamento do Tempo

O tempo deve sempre ser arredondado para cima para a próxima fração de `15` minutos.

Regras:

- Uma fração exata cobra exatamente uma fração.
- Qualquer excedente de 1 minuto inicia uma nova fração.
- Não existe tolerância.
- O cálculo deve considerar a duração total do estacionamento.

### Exemplos

| Tempo estacionado | Frações cobradas | Valor |
|---:|---:|---:|
| 1 min | 1 | 100 |
| 14 min | 1 | 100 |
| 15 min | 1 | 100 |
| 16 min | 2 | 200 |
| 29 min | 2 | 200 |
| 30 min | 2 | 200 |
| 31 min | 3 | 300 |
| 59 min | 4 | 400 |
| 60 min | 4 | 400 |
| 61 min | 5 | 500 |

---

## 6. Tolerância

A configuração atual possui:

`TOLERANCIA_MINUTOS = 0`

Portanto:

- Não existe período gratuito.
- Qualquer duração positiva deve gerar cobrança.
- 1 minuto já corresponde a 1 fração de cobrança.
- A tolerância não deve ser descontada da duração cobrada.

A implementação deve manter a regra parametrizada para respeitar `TOLERANCIA_MINUTOS`, mas o comportamento da variante atual é de tolerância zero.

---

## 7. Teto Diário

O valor cobrado por um bilhete nunca pode ultrapassar:

`TETO_DIARIO_CENTAVOS = 6000`

Ou seja, o valor máximo de qualquer bilhete é:

`R$ 60,00`

> [!WARNING]
> Mesmo que a duração calculada ultrapasse o valor de R$ 60,00, `valor_centavos` deve permanecer limitado a `6000`.

Exemplo:

| Valor calculado | Valor final |
|---:|---:|
| 5900 | 5900 |
| 6000 | 6000 |
| 6100 | 6000 |
| 10000 | 6000 |

---

## 8. Datas e Horários

- Datas e horários devem ser tratados em UTC internamente.
- A representação de data/hora deve utilizar ISO 8601.
- O formato esperado para armazenamento e processamento é:

`YYYY-MM-DDTHH:mm:ssZ`

- A entrada opcional `entrada` deve aceitar ISO-8601 com fuso horário conforme definido pelo contrato da API.
- Datas inválidas devem resultar em HTTP `422`.

A implementação não deve depender do horário local da máquina para realizar cálculos de duração.

---

## 9. Identificação de Placas

A placa deve:

- possuir exatamente 7 caracteres;
- conter somente caracteres alfanuméricos;
- utilizar letras maiúsculas;
- ser obrigatória na abertura do bilhete.

Placas inválidas devem retornar:

```json
{
  "erro": "placa_invalida"
}
