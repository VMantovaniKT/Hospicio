# 🏥 Análise de Internações no MongoDB(*Usar o script no arquivo "Internacoes"*)

Este projeto contém um script desenvolvido em **MongoDB Aggregation Framework**, que realiza uma análise detalhada das internações hospitalares. Ele calcula o valor total das internações, fornece estatísticas gerais e apresenta informações agrupadas por tipo de leito.

---

## 📌 O que o script faz:

1. **Cálculo do total de internação**:
   - Extrai o valor numérico da diária.
   - Calcula a diferença em dias entre a entrada e a saída.
   - Multiplica o valor da diária pelo número de dias de internação para calcular o total.

2. **Análise estatística**:
   - Calcula o total arrecadado, média de diárias, média de dias de internação, maior e menor valor de internação.

3. **Agrupamento por tipo de leito**:
   - Analisa internações por leito, incluindo total arrecadado, número de internações e média de dias.

4. **Retorna as internações completas**:
   - Apresenta todos os dados das internações com os cálculos realizados.

---

## 🚀 Script Principal:

```javascript
db["Internacoes"].aggregate([
  {
    $project: {
      _id_internacao: 1,
      _id_paciente: 1,
      leito: 1,
      data_entrada: 1,
      data_saida: 1,
      valor_diaria: 1,
      valor_diaria_numerico: {
        $toDouble: {
          $replaceAll: {
            input: { $substrCP: ["$valor_diaria", 2, { $strLenCP: "$valor_diaria" }] },
            find: ",",
            replacement: "."
          }
        }
      },
      dias_internacao: {
        $ceil: {
          $divide: [
            { $subtract: [ { $toDate: "$data_saida" }, { $toDate: "$data_entrada" } ] },
            1000 * 60 * 60 * 24
          ]
        }
      }
    }
  },
  {
    $addFields: {
      total_internacao: {
        $multiply: ["$valor_diaria_numerico", "$dias_internacao"]
      }
    }
  },
  {
    $facet: {
      todas_internacoes: [
        {
          $project: {
            _id_internacao: 1,
            _id_paciente: 1,
            leito: 1,
            data_entrada: 1,
            data_saida: 1,
            valor_diaria: 1,
            dias_internacao: 1,
            total_internacao: {
              $concat: [
                "R$",
                {
                  $toString: {
                    $round: ["$total_internacao", 2]
                  }
                }
              ]
            }
          }
        }
      ],
      estatisticas: [
        {
          $group: {
            _id: null,
            total_arrecadado: { $sum: "$total_internacao" },
            media_diaria: { $avg: "$valor_diaria_numerico" },
            media_dias: { $avg: "$dias_internacao" },
            maior_internacao: { $max: "$total_internacao" },
            menor_internacao: { $min: "$total_internacao" }
          }
        },
        {
          $project: {
            _id: 0,
            total_arrecadado: {
              $concat: [
                "R$",
                {
                  $toString: {
                    $round: ["$total_arrecadado", 2]
                  }
                }
              ]
            },
            media_diaria: {
              $concat: [
                "R$",
                {
                  $toString: {
                    $round: ["$media_diaria", 2]
                  }
                }
              ]
            },
            media_dias: { $round: ["$media_dias", 1] },
            maior_internacao: {
              $concat: [
                "R$",
                {
                  $toString: {
                    $round: ["$maior_internacao", 2]
                  }
                }
              ]
            },
            menor_internacao: {
              $concat: [
                "R$",
                {
                  $toString: {
                    $round: ["$menor_internacao", 2]
                  }
                }
              ]
            }
          }
        }
      ],
      por_leito: [
        {
          $group: {
            _id: "$leito",
            total: { $sum: "$total_internacao" },
            count: { $sum: 1 },
            media_dias: { $avg: "$dias_internacao" }
          }
        },
        {
          $project: {
            leito: "$_id",
            _id: 0,
            total: {
              $concat: [
                "R$",
                {
                  $toString: {
                    $round: ["$total", 2]
                  }
                }
              ]
            },
            quantidade: "$count",
            media_dias: { $round: ["$media_dias", 1] }
          }
        }
      ]
    }
  },
  {
    $project: {
      internacoes: "$todas_internacoes",
      estatisticas: { $arrayElemAt: ["$estatisticas", 0] },
      por_tipo_leito: "$por_leito"
    }
  }
]);
