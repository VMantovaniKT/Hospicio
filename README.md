# 🩺 Consultas de Maior e Menor Valor no MongoDB (*Usar o script no arquivo "Consultas"*)

Este projeto contém um script desenvolvido em **MongoDB Aggregation Framework** que identifica as consultas de **maior valor** e **menor valor** no banco de dados. Este script não filtra por convênio, garantindo que todas as consultas sejam consideradas na análise.

---

## 🚀 Script Principal

```javascript
db["Consultas"].aggregate([
  {
    $facet: {
      maiorValor: [
        { $sort: { valor_consulta: -1 } },
        { $limit: 1 }
      ],
      menorValor: [
        { $sort: { valor_consulta: 1 } },
        { $limit: 1 }
      ]
    }
  },
  {
    $project: {
      consultaMaiorValor: { $arrayElemAt: ["$maiorValor", 0] },
      consultaMenorValor: { $arrayElemAt: ["$menorValor", 0] }
    }
  }
]);
