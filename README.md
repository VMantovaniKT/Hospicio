# colocar o comando da string no shell das "Consultas" 📊

 

  
Este projeto contém um script desenvolvido para realizar operações de agregação no banco de dados MongoDB, com o objetivo de filtrar e calcular valores específicos a partir de uma coleção de consultas médicas. Foi usado o cenário de um hospital para gerenciar dados médicos e extrair métricas valiosas.

# 📌 O que o script faz
Converte o campo data_hora para o formato de data (caso esteja armazenado como string).

Filtra consultas realizadas no ano de 2020.

Filtra apenas consultas feitas sob convênio.

Calcula a média do valor das consultas realizadas e o total de consultas no ano.

## 🚀 Script Principal

```javascript
db["Consultas"].aggregate([
  {
    $addFields: {
      data_hora_convertida: { $dateFromString: { dateString: "$data_hora" } }
    }
  },
  {
    $match: {
      data_hora_convertida: {
        $gte: new Date("2020-01-01T00:00:00Z"),
        $lte: new Date("2020-12-31T23:59:59Z")
      }
    }
  },
  {
    $match: {
      "convenio.nome": { $exists: true }
    }
  },
  {
    $group: {
      _id: null,
      valorMedio: { $avg: "$valor_consulta" },
      totalConsultas: { $sum: 1 }
    }
  }
]);

Exemplo de saída:
json
{
  "_id": null,
  "valorMedio": 340,
  "totalConsultas": 3
}


