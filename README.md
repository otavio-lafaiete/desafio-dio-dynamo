# desafio-dio-dynamo
Repositório contendo comandos utilizados para criar um dynamo no desafio.

Link do repositório original do curso: https://github.com/cassianobrexbit/dio-live-dynamodb

- Criando a tabela:

```
aws dynamodb create-table \
    --table-name Guitar \
    --attribute-definitions \
        AttributeName=Brand,AttributeType=S \
        AttributeName=Shape,AttributeType=S \
    --key-schema \
        AttributeName=Brand,KeyType=HASH \
        AttributeName=Shape,KeyType=RANGE \
    --provisioned-throughput \
        ReadCapacityUnits=10,WriteCapacityUnits=5
```

- Inserir um item

```
aws dynamodb put-item \
    --table-name Guitar \
    --item file://itemguitar.json \
```

- Inserir múltiplos itens

```
aws dynamodb batch-write-item \
    --request-items file://batchguitar.json
```

- Criar um index global secundário baeado no preço

```
aws dynamodb update-table \
    --table-name Guitar \
    --attribute-definitions AttributeName=Price,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"Price-index\",\"KeySchema\":[{\"AttributeName\":\"Price\",\"KeyType\":\"HASH\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```

- Criar um index global secundário baseado no nome da marca e no preço

```
aws dynamodb update-table \
    --table-name Guitar \
    --attribute-definitions\
        AttributeName=Brand,AttributeType=S \
        AttributeName=Price,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"BrandPrice-index\",\"KeySchema\":[{\"AttributeName\":\"Brand\",\"KeyType\":\"HASH\"}, {\"AttributeName\":\"Price\",\"KeyType\":\"RANGE\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```

- Criar um index global secundário baseado no estilo da guitarra e no ano

```
aws dynamodb update-table \
    --table-name Guitar \
    --attribute-definitions\
        AttributeName=Shape,AttributeType=S \
        AttributeName=GuitarYear,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"ShapeGuitarYear-index\",\"KeySchema\":[{\"AttributeName\":\"Shape\",\"KeyType\":\"HASH\"}, {\"AttributeName\":\"GuitarYear\",\"KeyType\":\"RANGE\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```

- Pesquisar item por marca

```
aws dynamodb query \
    --table-name Guitar \
    --key-condition-expression "Brand = :brand" \
    --expression-attribute-values  '{":brand":{"S":"Tagima"}}'
```
- Pesquisar item por marca e preço da guitarra

```
aws dynamodb query \
    --table-name Guitar \
    --index-name BrandPrice-index \
    --key-condition-expression "Brand = :brand and Price = :price" \
    --expression-attribute-values file://keyconditions.json
```

- Pesquisa pelo index secundário baseado no preço da guitarra

```
aws dynamodb query \
    --table-name Guitar \
    --index-name Price-index \
    --key-condition-expression "Price = :price" \
    --expression-attribute-values  '{":price":{"S":"1500.00"}}'
```

- Pesquisa pelo index secundário baseado no nome da marca e no preço

```
aws dynamodb query \
    --table-name Guitar \
    --index-name BrandPrice-index \
    --key-condition-expression "Brand = :v_brand and Price = :v_price" \
    --expression-attribute-values  '{":v_brand":{"S":"Epiphone"},":v_price":{"S":"1500.00"} }'
```

- Pesquisa pelo index secundário baseado no estilo da guitarra e no ano

```
aws dynamodb query \
    --table-name Guitar \
    --index-name ShapeGuitarYear-index \
    --key-condition-expression "Shape = :v_theShape and GuitarYear = :v_GuitarYear" \
    --expression-attribute-values  '{":v_theShape":{"S":"Stratocaster"},":v_GuitarYear":{"S":"2023"} }'
```