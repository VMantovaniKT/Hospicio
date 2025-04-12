# 🩺 Filtragem de Consultas de Menores de Idade fora da Pediatria( *Usar o script no arquivo "Consultas"* )

Este projeto contém um script em **MongoDB Aggregation Framework** que busca consultas realizadas por pacientes menores de idade em especialidades médicas que não sejam pediatria. Ele calcula a idade do paciente na data da consulta e exibe detalhes organizados.

---

## 📌 O que o script faz:

1. **Enriquecimento com dados do paciente**:
   - Realiza uma junção (`$lookup`) entre a coleção `Consultas` e a coleção `Pacientes` com base no campo `id_paciente`.
   - Adiciona as informações dos pacientes na consulta por meio do campo `paciente_info`.

2. **Cálculo de Idade**:
   - Converte as datas de nascimento e consulta para o formato de data.
   - Calcula a idade do paciente na data da consulta com base na diferença entre as datas.

3. **Filtragem**:
   - Seleciona apenas pacientes com idade inferior a 18 anos.
   - Exclui consultas cuja especialidade buscada seja "Pediatria", independentemente de maiúsculas ou minúsculas.

4. **Projeção de Campos**:
   - Apresenta o nome do paciente, data da consulta, especialidade e idade na data da consulta, removendo os outros campos.

5. **Ordenação**:
   - Organiza os resultados por data de consulta em ordem crescente.

---

## 🚀 Script Principal:

```javascript
db.Consultas.aggregate([
  {
    $lookup: {
      from: "Pacientes",
      localField: "id_paciente",
      foreignField: "_id_paciente",
      as: "paciente_info"
    }
  },
  {
    $unwind: "$paciente_info"
  },
  {
    $addFields: {
      data_nascimento: {
        $dateFromString: {
          dateString: "$paciente_info.data_nascimento",
          format: "%Y-%m-%d"
        }
      },
      data_consulta: {
        $dateFromString: {
          dateString: "$data_hora",
          format: "%Y-%m-%d %H:%M:%S"
        }
      }
    }
  },
  {
    $addFields: {
      idade_na_consulta: {
        $floor: {
          $divide: [
            { $subtract: ["$data_consulta", "$data_nascimento"] },
            31557600000
          ]
        }
      }
    }
  },
  {
    $match: {
      "idade_na_consulta": { $lt: 18 },
      "especialidade_buscada": { 
        $not: /^pediatria$/i
      }
    }
  },
  {
    $project: {
      _id: 0,
      nome_paciente: "$paciente_info.nome",
      data_consulta: 1,
      especialidade: "$especialidade_buscada",
      idade_na_consulta: 1
    }
  },
  {
    $sort: {
      data_consulta: 1
    }
  }
]);
