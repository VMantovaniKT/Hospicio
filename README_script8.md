# 🩺 Listagem de Internações em Enfermarias por Gastroenterologistas ( *Usar o script no arquivo "internacoes"* )

Este projeto contém um script em **MongoDB Aggregation Framework** que busca internações realizadas em enfermarias por médicos especializados em gastroenterologia. Ele combina informações das coleções `Internacoes`, `Medicos` e `Pacientes` para apresentar detalhes organizados de forma clara.

---

## 📌 O que o script faz:

1. **Junção com informações dos médicos**:
   - Realiza um `lookup` para obter dados dos médicos na coleção `Medicos` com base no campo `_id_medico`.

2. **Junção com informações dos pacientes**:
   - Realiza um segundo `lookup` para adicionar os dados dos pacientes da coleção `Pacientes` usando o campo `_id_paciente`.

3. **Filtragem**:
   - Seleciona apenas internações em que a especialidade do médico seja "Gastroenterologia" e o leito seja "enfermaria".

4. **Projeção de campos relevantes**:
   - Exibe o nome do paciente, nome do médico, data de entrada, procedimentos realizados, tipo de leito e especialidade do médico.

5. **Ordenação**:
   - Organiza os resultados por data de entrada em ordem crescente.

---

## 🚀 Script Principal:

```javascript
db.Internacoes.aggregate([
  {
    $lookup: {
      from: "Medicos",
      localField: "_id_medico",
      foreignField: "crm",
      as: "medico_info"
    }
  },
  {
    $unwind: "$medico_info"
  },
  {
    $lookup: {
      from: "Pacientes",
      localField: "_id_paciente",
      foreignField: "_id_paciente",
      as: "paciente_info"
    }
  },
  {
    $unwind: "$paciente_info"
  },
  {
    $match: {
      "medico_info.especialidade": "Gastroenterologia",
      "leito": "enfermaria"
    }
  },
  {
    $project: {
      _id: 0,
      nome_paciente: "$paciente_info.nome",
      nome_medico: "$medico_info.nome",
      data_internacao: "$data_entrada",
      procedimentos: "$descricao_internacao",
      leito: 1,
      especialidade_medico: "$medico_info.especialidade"
    }
  },
  {
    $sort: {
      data_internacao: 1
    }
  }
]);
