# Terraform BigQuery

O exemplo a seguir mostra o uso do terraform-google-bigquery para implantar um único conjunto de dados e duas tabelas com esquemas básicos. Este exemplo é uma boa referência para entender e testar o uso do módulo.

## Inputs
| Nome Descrição Tipo | Padrão | Requerido |
| ------ | ------------- |: ----: |: -----: |: -----: |
| conjunto de dados | _labels | Um mapeamento de rótulos para atribuir à tabela. | mapa (string) | n / a | sim |
| default | _table | _expiration | _ms | TTL padrão de tabelas usando o conjunto de dados em MS. | string | `" null "` | não
| delete | _contents | _on | _destroy | (Opcional) Se definido como verdadeiro, exclui todas as tabelas no conjunto de dados ao destruir o recurso; caso contrário, a destruição do recurso falhará se houver tabelas. | bool | `" null "` | não
| projeto | _id | Projeto onde o conjunto de dados e a tabela são criados. | string | n / a | sim |
| tabelas | Uma lista de mapas que inclui table_id, schema, clustering, time_partitioning, expiration_time, rótulos em cada elemento. | objeto | `<list>` | não
