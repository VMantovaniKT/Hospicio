# 🩺 Relatório de Internações por Enfermeiros no MongoDB ( *Usar o script no arquivo "Internacoes"* )

Este projeto contém um script em **MongoDB Aggregation Framework** que identifica enfermeiros com mais de uma internação associada. Ele busca informações detalhadas sobre os enfermeiros, incluindo nome, COREN e o número total de internações realizadas, apresentando os resultados ordenados pelo total de internações em ordem decrescente.

---

## 📌 O que o script faz:

1. **Desagregação dos enfermeiros**:
   - Usa `$unwind` para transformar o array de IDs de enfermeiros (`_id_enfermeiro`) em documentos individuais.

2. **Agrupamento por ID de enfermeiro**:
   - Conta o número total de internações associadas a cada enfermeiro.

3. **Filtragem**:
   - Mantém apenas enfermeiros com mais de uma internação (`total_internacoes > 1`).

4. **Junção com informações dos enfermeiros**:
   - Usa `$lookup` para buscar dados adicionais dos enfermeiros na coleção `Enfermeiros`.

5. **Formatação e projeção**:
   - Exibe apenas os campos relevantes: nome, COREN e total de internações.

6. **Ordenação dos resultados**:
   - Organiza os enfermeiros pelo total de internações em ordem decrescente.

---

## 🚀 Script Principal:

```javascript
db.Internacoes.aggregate([
  {
    $unwind: "$_id_enfermeiro"
  },
  {
    $group: {
      _id: "$_id_enfermeiro",
      total_internacoes: { $sum: 1 }
    }
  },
  {
    $match: {
      total_internacoes: { $gt: 1 }
    }
  },
  {
    $lookup: {
      from: "Enfermeiros",
      localField: "_id",
      foreignField: "_id_enfermeiro",
      as: "dados_enfermeiro"
    }
  },
  {
    $unwind: "$dados_enfermeiro"
  },
  {
    $project: {
      _id: 0,
      nome: "$dados_enfermeiro.nome",
      coren: "$dados_enfermeiro.coren",
      total_internacoes: 1
    }
  },
  {
    $sort: {
      total_internacoes: -1
    }
  }
]);
