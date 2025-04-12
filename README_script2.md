# 📊 Filtrar internações com data de alta maior que a prevista no MongoDB (*usar o script no arquivo "Internacoes"*)

Este projeto contém um script desenvolvido em **MongoDB Aggregation Framework**, projetado para identificar internações hospitalares em que a **data de alta** excedeu a **data prevista para alta**. Esse script é útil para melhorar os processos hospitalares e gerenciar recursos com maior eficiência.

---

## 📌 O que o script faz:

1. **Converte datas armazenadas como string para o formato `Date`:**
   - Realiza essa conversão usando `$dateFromString`.

2. **Filtra internações atrasadas:**
   - Filtra registros onde `data_saida` é maior que `previsao_alta_data`.

3. **Agrupa dados relevantes:**
   - Calcula o total de internações atrasadas.
   - Retorna as descrições das internações filtradas.

4. **Ignora campos inválidos:**
   - Desconsidera registros com campos de data vazios ou inválidos.

---

## 🚀 Script Principal

```javascript
db["Internacoes"].aggregate([
  {
    $addFields: {
      data_saida_convertida: {
        $cond: {
          if: { $ne: ["$data_saida", ""] }, // Verifica se "data_saida" não está vazio
          then: { $dateFromString: { dateString: "$data_saida" } },
          else: null
        }
      },
      previsao_alta_convertida: {
        $cond: {
          if: { $ne: ["$previsao_alta_data", ""] }, // Verifica se "previsao_alta_data" não está vazio
          then: { $dateFromString: { dateString: "$previsao_alta_data" } },
          else: null
        }
      }
    }
  },
  {
    $match: {
      $expr: {
        $and: [
          { $gt: ["$data_saida_convertida", "$previsao_alta_convertida"] },
          { $ne: ["$data_saida_convertida", null] }, // Verifica conversão bem-sucedida
          { $ne: ["$previsao_alta_convertida", null] } // Verifica conversão bem-sucedida
        ]
      }
    }
  },
  {
    $project: {
      _id_internacao: 1,
      data_saida: 1,
      previsao_alta_data: 1,
      descricao_internacao: 1
    }
  }
]);
