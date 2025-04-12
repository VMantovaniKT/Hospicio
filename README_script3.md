# 🩺 Identificar a primeira consulta registrada com receituário associado no MongoDB (*Usar o script no arquivo "Consultas"*)

Este projeto contém um script desenvolvido em **MongoDB Aggregation Framework**, que busca a **primeira consulta médica** registrada no banco de dados, onde existe um **receituário associado**. É ideal para cenários em que consultas com receituários precisam ser identificadas e analisadas.

---

## 📌 O que o script faz:

1. **Filtra consultas com receituário associado:**
   - Verifica se o campo `receita_medicamentos` contém pelo menos um item.

2. **Ordena pela data/hora:**
   - Organiza todas as consultas em ordem crescente com base no campo `data_hora`.

3. **Identifica a primeira consulta:**
   - Limita o resultado à primeira consulta com receituário, usando `$limit`.

4. **Retorna apenas os campos relevantes:**
   - Exibe os principais detalhes da consulta, como `crm_medico`, `id_paciente`, `data_hora`, `receita_medicamentos`, entre outros.

---

## 🚀 Script Principal

```javascript
db["Consultas"].aggregate([
  {
    $match: {
      "receita_medicamentos.0": { $exists: true } 
    }
  },
  {
    $sort: {
      data_hora: 1 
    }
  },
  {
    $limit: 1 
  },
  {
    $project: {
      _id: 1,
      crm_medico: 1,
      id_paciente: 1,
      especialidade_buscada: 1,
      data_hora: 1,
      descricao: 1,
      observacao: 1,
      receita_medicamentos: 1,
      convenio: 1,
      valor_consulta: 1
    }
  }
]);
