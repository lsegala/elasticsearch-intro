# elasticsearch (introdução)
    - https://www.elastic.co/kr/webinars/getting-started-elasticsearch

# 1. Criando um índice
```
PUT library
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  }
}
```

# 2. Inserindo dados (Bulk Insert)
    - https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html
```
POST library/books/_bulk
{"index":{"_id":1}}
{"title":"The quick brow fox","price":5,"colors":["red","green","blue"]}
{"index":{"_id":2}}
{"title":"The quick brow fox jumps over the lazy dog","price":15,"colors":["blue","yellow"]}
{"index":{"_id":3}}
{"title":"The quick brow fox jumps over the quick dog","price":8,"colors":["red","blue"]}
{"index":{"_id":4}}
{"title":"brow fox brown dog","price":2,"colors":["black","yellow","red","blue"]}
{"index":{"_id":5}}
{"title":"Lazy dog","price":9,"colors":["red","blue","green"]}
```

# 3. Busca
## 3.1. Pesquisar todos os documentos
```
GET library/_search
```

## 3.2. Procurar por documentos contendo _fox_
```
GET library/_search
{
  "query": {
    "match": {
      "title": "fox"
    }
  }
}
```

## 3.3. Procure por documentos contendo _quick_ ou _dog_ 
```
GET library/_search
{
  "query": {
    "match": {
      "title": "quick dog"
    }
  }
}
```

## 3.4. Procure documentos com a frase _quick dog_
```
GET library/_search
{
  "query": {
    "match_phrase": {
      "title": "quick dog"
    }
  }
}
```

## 3.5. Aplicar rankings para resultados de pesquisa usando algoritmo de "relevância" (_score)
    - https://www.elastic.co/guide/en/elasticsearch/guide/current/relevance-intro.html
```
GET library/_search
{
  "query": {
    "match": {
      "title": "quick"
    }
  }
}
```

# 4. Consulta composta

## 4.1. Procurar todos os documentos que contenham _rápido_ e _cão preguiçoso_
```
GET /library/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": "quick"
          }
        },
        {
          "match_phrase": {
            "title": {
              "query": "lazy dog"
            }
          }
        }
      ]
    }
  }
}
```

## 4.2. must_not: procurar por documentos que não contenham _rápido_ ou _cão preguiçoso_
```
GET /library/_search
{
  "query": {
    "bool": {
      "must_not": [
        {
          "match": {
            "title": "lazy"
          }
        },
        {
          "match_phrase": {
            "title": {
              "query": "quick dog"
            }
          }
        }
      ]
    }
  }
}
```

## 4.3. Aumentar a pontuação para consultas específicas
### 4.3.1. should: não corresponder necessariamente, mas dar preferência para o termo com boost
```
GET /library/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match_phrase": {
            "title": "quick dog"
          }
        },
        {
          "match_phrase": {
            "title": {
              "query": "lazy dog",
              "boost": 3
            }
          }
        }
      ]
    }
  }
}
```

### 4.3.2 must com should
```
GET /library/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "title": "lazy"
          }
        }
      ],
      "must": [
        {
          "match": {
            "title": "dog"
          }
        }
      ]
    }
  }
}
```

# 5. Highlight
```
GET /library/_search
{
  "query" : {
    "bool": {
      "should" : [
        {
          "match_phrase": { 
            "title": {
              "query" : "quick dog",
              "boost": 2
            } 
          }
        },
        {
          "match_phrase": { 
            "title": {
              "query" : "lazy dog"
            } 
          }
        }
      ]
    }
  },
  "highlight" : {
    "fields" : {
      "title": { }
    }
  }
}
```

# 6. Filtro
## 6.1. bool + filter
```
GET /library/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": "dog"
          }
        }
      ],
      "filter": {
        "range": {
          "price": {
            "gte": 5,
            "lte": 10
          }
        }
      }
    }
  }
}
```

## 6.2. Somente filter
```
GET /library/_search
{
  "query": {
    "bool": {
      "filter": {
        "range": {
          "price": {
            "gt": 5
          }
        }
      }
    }
  }
}
```

# 7. Análise
## 7.1. Frase dividida em termos
```
GET library/_analyze
{
  "tokenizer": "standard",
  "text": "Brown fox brown dog"
}
```

## 7.2. Tokens + Filtros
```
GET library/_analyze
{
  "tokenizer": "standard",
  "filter": [
    "lowercase"
  ],
  "text": "Brown fox brown dog"
}
```

### 7.3. Remover termos duplicados
```
GET library/_analyze
{
  "tokenizer": "standard",
  "filter": [
    "lowercase",
    "unique"
  ],
  "text": "Brown brown brown fox brown dog"
}
```

## 7.4. Analyzer
```
GET library/_analyze
{
  "analyzer": "standard",
  "text": "Brown fox brown dog"
}
```

# 8. Compreendendo o processo de análise
## 8.1. Caso 1
```
GET library/_analyze
{
  "tokenizer": "standard",
  "filter": [
    "lowercase"
  ],
  "text": "THE quick.brown_FOx jumped! $19.95 @ 3.0"
}
```

## 8.2. Caso 2
```
GET library/_analyze
{
  "tokenizer": "letter",
  "filter": [
    "lowercase"
  ],
  "text": "THE quick.brown_FOx jumped! $19.95 @ 3.0"
}
```

## 8.3. Problemas com emails ou URLs
```
GET library/_analyze
{
  "tokenizer": "standard",
  "text": "elastic@example.com website: https://www.elastic.co"
}
```

## 8.4. Solução do problema de emails e URLs
```
GET library/_analyze
{
  "tokenizer": "uax_url_email",
  "text": "elastic@example.com website: https://www.elastic.co"
}
```

# 9. Agregação
## 9.1. Simples
```
GET library/_search
{
  "size": 0,
  "aggs": {
    "popular-colors": {
      "terms": {
        "field": "colors.keyword"
      }
    }
  }
}
```

## 9.2. Consulta + Agg
```
GET library/_search
{
  "query": {
    "match": {
      "title": "dog"
    }
  },
  "aggs": {
    "popular-colors": {
      "terms": {
        "field": "colors.keyword"
      }
    }
  }
}
```

## 9.3. Subagregação
```
GET library/_search
{
  "size": 0, 
  "aggs": {
    "price-statistics": {
      "stats": {
        "field": "price"
      }
    },
    "popular-colors": {
      "terms": {
        "field": "colors.keyword"
      },
      "aggs": {
        "avg-price-per-color": {
          "avg": {
            "field": "price"
          }
        }
      }
    }
  }
}
```

# 10. Recuperar um documento específico
```
GET library/books/1
```

## 10.1. Criar um documento
```
POST library/books/1
{
  "title": "The quick brow fox",
  "price": 10,
  "colors": ["red","green","blue"]
}
```

## 10.2. Atualizar um documento
```
POST library/books/1/_update
{
  "doc": {
    "title": "The quick fantastic fox"
  }
}
```

# 11. Mapeamento
## 11.1. Auto mapeamento
```
GET library/_mapping
```

## 11.2. Criar um índice especificando o mapeamento
```
PUT famous-librarians
{
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 0,
    "analysis": {
      "analyzer": {
        "my-desc-analyzer": {
          "type": "custom",
          "tokenizer": "uax_url_email",
          "filter": [
            "lowercase"
          ]
        }
      }
    }
  },
  "mappings": {
    "librarian": {
      "properties": {
        "name": {
          "type": "text"
        },
        "favourite-colors": {
          "type": "keyword"
        },
        "birth-date": {
          "type": "date",
          "format": "year_month_day"
        },
        "hometown": {
          "type": "geo_point"
        },
        "description": {
          "type": "text",
          "analyzer": "my-desc-analyzer"
        }
      }
    }
  }
}
```

## 11.3. Exemplos
### 11.3.1. Exemplo 1
```
PUT famous-librarians/librarian/1
{
  "name": "Sarah Byrd Askew",
  "favourite-colors": [
    "Yellow",
    "light-grey"
  ],
  "birth-date": "1877-02-15",
  "hometown": {
    "lat": 32.349722,
    "lon": -87.641111
  },
  "description": "An American public librarian who pioneered the establishment of county libraries in the United States - https://en.wikipedia.org/wiki/Sarah_Byrd_Askew"
}
```

### 11.3.2. Exemplo 2
```
PUT famous-librarians/librarian/2
{
  "name": "John J. Beckley",
  "favourite-colors": [
    "Red",
    "off-white"
  ],
  "birth-date": "1757-08-07",
  "hometown": {
    "lat": 51.507222,
    "lon": -0.1275
  },
  "description": "An American political campaign manager and the first Librarian of the United States Congress, - https://en.wikipedia.org/wiki/John_J._Beckley"
}
```

## 11.4. Pesquisas
### 11.4.1. Consulta alternativa
```
GET famous-librarians/_search
{
  "query": {
    "query_string": {
      "fields": [
        "favourite-colors"
      ],
      "query": "yellow OR off-white"
    }
  }
}
```

### 11.4.2. Consulta por data
```
GET famous-librarians/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match_all": {}
        }
      ],
      "filter": {
        "range": {
          "birth-date": {
            "gte": "now-200y",
            "lte": "2000-01-01"
          }
        }
      }
    }
  }
}
```

### 11.4.3. Consulta por raio a partir de uma coordenada
```
GET famous-librarians/_search
{
  "query": {
    "bool": {
      "filter": {
        "geo_distance": {
          "distance": "100km",
          "hometown": {
            "lat": 32.41,
            "lon": -86.92
          }
        }
      }
    }
  }
}
```