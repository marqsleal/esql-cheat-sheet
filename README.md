# Elastic Search
Elastic Search Query Langague (ESQL) é um mecanismo NoSQL de busca extremamente escalável construído com base no Apache Lucene, utilizado em contextos onde é necessário uma busca de grandes volumes de dados, permitindo consultas rápidas e precisas.

## ELK Stack:
- Beats (Input Plugin);
- Logstash (Input >> Transform >> Stash);
- Elastic Search (ESQL);
- Kibana (UI Dashboard);

## Comparações (MySQL vs ESQL):
- Database - Index;
- Table - Pattern;
- Rows - Documents;
- Columns - Fields;

### Document:
- Documento (`document`) - unidade básica de dados dentro da Elastic Search Query Langague;
- Cada documento é um `json`, que armazena dados estruturados;

### Index:
- Índice (`index`) - parecido com uma tabela num SQL, mas é uma coleção de documentos relacionados;
- Cada índice tem um mapeamento, que define como seus campos são armazenados e indexados;
- Parecido como um banco de dados específico pra pesquisa;

#### Node:
- Nó (`node`) - é uma instância ES em runtime. Pode armazenar dados, ser responśavel por coordenação de consultas ou ambos.
- Vários nós podem formar um `cluster` para escalar horizontalmente;

#### Shards:
- Um índice é dividido em várias partições (`shards`), distribuídos em diferentes nós (`nodes`) para melhorar a escalabilidade e a performance;

#### Cluster:
- Um `cluster` é composto por vários nós (máquinas), que trabalham juntos para armazenar e processar os dados distribuídos;
- Cada nó tem um papel no cluster (Data Node, Master Node, etc);

## Operações Básicas
- Indexar Documento:
```bash
PUT /{index}/{_doc}/{id}
{
  "field": "value"
}
```
- Buscar Documento por ID:
```bash
GET /{index}/_doc/{id}
```

## Buscas Full Text:
Elastic Search usa o match e term para buscas textuais. O match é útil para textos completos.  
- Busca básica por texto:
```bash
GET /{index}/_search
{
  "query": {
    "match": {
      "field": "search text"
    }
  }
}
```
- Busca com operador booleano (AND/OR):
```bash
GET /{index}/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "field1": "text1" }},
        { "match": { "field2": "text2" }}
      ]
    }
  }
}
```

## Análises e Tokenização
ElasticSearch usa analyzers para processar o texto antes da busca. Isso envolve tokenização, remoção de stopwords, etc.  
- Usar o analyzer padrão:
```bash
GET /{index}/_analyze
{
  "analyzer": "standard",
  "text": "input text"
}
```
- Definir um analyzer customizado no mapeamento:
```bash
PUT /{index}
{
  "settings": {
    "analysis": {
      "analyzer": {
        "custom_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "stop"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "field": {
        "type": "text",
        "analyzer": "custom_analyzer"
      }
    }
  }
}
```

## Filtros e Agregações
- Filtros em queries:
```bash
GET /{index}/_search
{
  "query": {
    "bool": {
      "filter": {
        "term": { "field": "value" }
      }
    }
  }
}
```
- Agregações de texto (contagem de termos):
```bash
GET /{index}/_search
{
  "size": 0,
  "aggs": {
    "term_count": {
      "terms": { "field": "field.keyword" }
    }
  }
}
```

## Busca Fuzzy (aproximação de texto)
Para permitir erros de digitação ou aproximações:
- Query Fuzzy:
```bash
GET /{index}/_search
{
  "query": {
    "fuzzy": {
      "field": {
        "value": "aproximated text",
        "fuzziness": "AUTO"
      }
    }
  }
}
```

## Pesquisa por Frase
Para buscar por uma frase específica:
- Busca exata de frase:
```bash
GET /{index}/_search
{
  "query": {
    "match_phrase": {
      "field": "exact phrase"
    }
  }
}
```

## Paginação e Ordenação
- Paginação (from/size):
```bash
GET /{index}/_search
{
  "from": 0,
  "size": 10,
  "query": {
    "match_all": {}
  }
}
```
- Ordenação por um campo:
```bash
GET /{index}/_search
{
  "sort": [
    { "field": { "order": "desc" }}
  ]
}
```

## Sugestões de Termo
Elastic Search pode sugerir correções de palavras:
- Sugerir correções de palavras:
```bash
GET /{index}/_search
{
  "suggest": {
    "text": "misspeled text",
    "simple_phrase": {
      "phrase": {
        "field": "field",
        "size": 1
      }
    }
  }
}
```

## Exemplos

### Busca Full-Text com Filtros e Ordenação  
Este exemplo faz uma busca por um termo específico, aplica filtros e ordena os resultados:
```bash
GET /livros/_search
{
  "query": {
    "bool": {
      "must": {
        "match": {
          "conteudo": "aprendizado profundo"
        }
      },
      "filter": [
        { "term": { "autor.keyword": "John Doe" }},
        { "range": { "data_publicacao": { "gte": "2020-01-01" }}}
      ]
    }
  },
  "sort": [
    { "data_publicacao": { "order": "desc" }}
  ],
  "from": 0,
  "size": 10
}
```

- **Match**: Busca por "aprendizado profundo" no campo `conteudo`;
- **Filter**: Limita a busca a livros do autor "John Doe" publicados após 2020;
- **Sort**: Ordena por data de publicação mais recente;
- **Paginação**: Retorna os 10 primeiros resultados;

### Agregação de Termos com Fuzzy Search
Busca por termos semelhantes e conta a frequência de uma categoria específica:  
```bash
GET /produtos/_search
{
  "query": {
    "fuzzy": {
      "nome": {
        "value": "laptop",
        "fuzziness": "AUTO"
      }
    }
  },
  "aggs": {
    "categorias_populares": {
      "terms": {
        "field": "categoria.keyword",
        "size": 5
      }
    }
  }
}
```

- **Fuzzy Search**: Busca produtos cujo nome seja "laptop" ou variações semelhantes (considera erros de digitação);
- **Aggregation**: Conta as 5 categorias mais comuns em que os produtos "laptop" aparecem;

### Pesquisa de Frase com Sugerir Correções
Busca uma frase específica no campo de descrição e sugere correções de termos com erros: 
```bash
GET /artigos/_search
{
  "query": {
    "match_phrase": {
      "descricao": "inteligência artificial"
    }
  },
  "suggest": {
    "desc_sugestoes": {
      "text": "inteligência artficial",
      "term": {
        "field": "descricao",
        "suggest_mode": "always"
      }
    }
  }
}
```

- **Match Phrase**: Busca exata por "inteligência artificial" no campo descricao;
- **Suggest**: Sugere correções para a frase "inteligência artficial" (corrige a palavra "artficial");

### Agregações de Data com Filtros por Termos
Busca documentos e faz agregações por data, com filtro por termos específicos:  
```bash
GET /vendas/_search
{
  "query": {
    "bool": {
      "filter": {
        "term": {
          "produto.keyword": "Celular"
        }
      }
    }
  },
  "aggs": {
    "vendas_por_mes": {
      "date_histogram": {
        "field": "data_venda",
        "calendar_interval": "month"
      }
    }
  }
}
```  

- **Bool Filter**: Filtra por vendas de "Celular".
- **Date Histogram Aggregation**: Conta as vendas agregadas por mês (`data_venda`).

### Busca Multi-Match com Agregação e Filtro de Range
Este exemplo usa uma busca que combina múltiplos campos e também faz uma agregação e filtro por faixa de valor.
```bash
GET /imoveis/_search
{
  "query": {
    "bool": {
      "must": {
        "multi_match": {
          "query": "apartamento 3 quartos",
          "fields": ["titulo", "descricao"]
        }
      },
      "filter": {
        "range": {
          "preco": { "gte": 300000, "lte": 600000 }
        }
      }
    }
  },
  "aggs": {
    "media_preco_por_bairro": {
      "terms": {
        "field": "bairro.keyword"
      },
      "aggs": {
        "media_preco": {
          "avg": { "field": "preco" }
        }
      }
    }
  }
}
```  

- **Multi-Match**: Busca por "apartamento 3 quartos" nos campos `titulo` e `descricao`;
- **Range Filter**: Filtra os resultados com preço entre R$300.000 e R$600.000;
- **Terms Aggregation**: Faz agregação dos imóveis por bairro;
- **Avg Aggregation**: Calcula o preço médio por bairro;