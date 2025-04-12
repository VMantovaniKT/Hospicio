# 🏥 Listagem de Internações em Apartamentos no MongoDB (*Usar o script no arquivo "Internacoes"*)

Este projeto contém um script em **MongoDB Aggregation Framework** que filtra e organiza as internações realizadas exclusivamente em apartamentos. Ele apresenta detalhes como as datas de entrada e saída, descrição do procedimento e número do quarto, ordenados por data de entrada.

---

## 📌 O que o script faz:

1. **Filtra internações em apartamentos**:
   - Seleciona apenas internações cujo campo `leito` seja igual a `"apartamento"`.

2. **Projeção de campos relevantes**:
   - Remove o campo `_id`.
   - Exibe as datas de entrada e saída.
   - Extrai o primeiro elemento do campo `descricao_internacao` como o procedimento.
   - Usa o identificador da internação (`_id_internacao`) como número do quarto.

3. **Ordenação**:
   - Organiza as internações pela data de entrada, em ordem crescente.

---

## 🚀 Script Principal:

```javascript
db.Internacoes.aggregate([
  {
    $match: {
      leito: "apartamento"
    }
  },
  {
    $project: {
      _id: 0,
      data_entrada: 1,
      data_saida: 1,
      procedimento: { $arrayElemAt: ["$descricao_internacao", 0] },
      numero_quarto: "$_id_internacao"
    }
  },
  {
    $sort: {
      data_entrada: 1
    }
  }
]);
