# 🩺 Relatório de Consultas por Médico no MongoDB ( *Usar script no arquivo "Consultas"* )

Este projeto contém um script em **MongoDB Aggregation Framework** que gera um relatório detalhado de consultas realizadas por médicos, agrupando dados e juntando informações adicionais, como especialidades e status do médico.

---

## 📌 O que o script faz:

1. **Agrupamento por CRM do médico**:
   - Conta o número total de consultas realizadas por cada médico.
   - Identifica a data da primeira e última consulta realizadas.

2. **Junção com dados dos médicos**:
   - Enriquece as informações de cada médico por meio de uma junção com a coleção `Medicos`.
   - Recupera detalhes como nome, especialidades, tipo e status do médico.

3. **Normalização dos dados do médico**:
   - Garante que médicos sem registro sejam marcados como "Médico não cadastrado" com o CRM correspondente.

4. **Formatação do resultado**:
   - Apresenta os dados do médico, como especialidades, status, total de consultas e período de atividade.

5. **Ordenação e Limite**:
   - Ordena os médicos pelo número total de consultas (descendente) e pelo nome (ascendente).
   - Limita o número de resultados para até 100 registros.

---

## 🚀 Script Principal:

```javascript
db.Consultas.aggregate([
  {
    $group: {
      _id: "$crm_medico",
      total_consultas: { $sum: 1 },
      primeira_consulta: { $min: "$data_hora" },
      ultima_consulta: { $max: "$data_hora" }
    }
  },
  {
    $lookup: {
      from: "Medicos",
      let: { crm_busca: "$_id" },
      pipeline: [
        { 
          $match: { 
            $expr: { $eq: ["$documentos.crm", "$$crm_busca"] } 
          } 
        },
        { 
          $project: { 
            _id: 0,
            nome: 1,
            especialidades: 1,
            tipo: 1,
            status: "$status_medico"
          }
        }
      ],
      as: "medico"
    }
  },
  {
    $unwind: {
      path: "$medico",
      preserveNullAndEmptyArrays: true
    }
  },
  {
    $project: {
      _id: 0,
      crm: "$_id",
      nome_medico: {
        $ifNull: ["$medico.nome", "Médico não cadastrado (CRM: " + { $toString: "$_id" } + ")"]
      },
      especialidades: {
        $ifNull: ["$medico.especialidades", ["Especialidade não informada"]]
      },
      tipo_medico: "$medico.tipo",
      status: {
        $switch: {
          branches: [
            { case: { $eq: ["$medico.status", 1] }, then: "Ativo" },
            { case: { $eq: ["$medico.status", 0] }, then: "Inativo" }
          ],
          default: "Status desconhecido"
        }
      },
      total_consultas: 1,
      periodo_atividade: {
        primeira_consulta: 1,
        ultima_consulta: 1
      },
      crm_valido: { $cond: [{ $gt: ["$medico", null] }, true, false] }
    }
  },
  {
    $sort: {
      "total_consultas": -1,
      "nome_medico": 1
    }
  },
  {
    $limit: 100
  }
]);
