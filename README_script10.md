# 🩺 Busca de Médicos com Nome "Gabriel" no MongoDB ( *Usar o script no arquivo "Medicos"*  )

Este projeto contém um script desenvolvido em **MongoDB Aggregation Framework** que realiza uma busca case-insensitive por médicos cujo nome contenha "Gabriel". Ele extrai informações detalhadas, como CRM, especialidades, tipo, status, contato e data de nascimento, apresentando os resultados ordenados pelo nome completo.

---

## 📌 O que o script faz:

1. **Busca por nome**:
   - Utiliza `$regex` para realizar uma busca no campo `nome` por médicos cujo nome contenha "Gabriel", independentemente de maiúsculas ou minúsculas.

2. **Projeção de informações**:
   - Exibe detalhes como CRM, nome completo, especialidades, tipo, status e data de nascimento formatada.
   - Inclui o primeiro email e telefone disponíveis no campo `contato`.

3. **Formatação de status**:
   - Converte o status médico (`1` para "Ativo" e `0` para "Inativo").

4. **Ordenação dos resultados**:
   - Organiza os médicos pelo nome completo em ordem crescente.

---

## 🚀 Script Principal:

```javascript
db.Medicos.aggregate([
  {
    $match: {
      nome: { $regex: /Gabriel/i }
    }
  },
  {
    $project: {
      _id: 0,
      crm: "$documentos.crm",
      nome_completo: "$nome",
      especialidades: 1,
      tipo: 1,
      status: {
        $cond: {
          if: { $eq: ["$status_medico", 1] },
          then: "Ativo",
          else: "Inativo"
        }
      },
      contato: {
        email: { $arrayElemAt: ["$contato.email", 0] },
        telefone: { $arrayElemAt: ["$contato.telefone", 0] }
      },
      data_nascimento: {
        $dateToString: {
          format: "%d/%m/%Y",
          date: { $toDate: "$data_nascimento" }
        }
      }
    }
  },
  {
    $sort: {
      nome_completo: 1
    }
  }
]);
