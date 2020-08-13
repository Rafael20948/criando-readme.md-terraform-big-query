# terraform-google-bigquery

O exemplo a seguir mostra o uso do terraform-google-bigquery para implantar um único conjunto de dados e duas tabelas com esquemas básicos. Este exemplo é uma boa referência para entender e testar o uso do módulo.

## Compatibilidade

Este módulo deve ser usado com o Terraform 0.12. Se você não tem
[atualizado] (https://www.terraform.io/upgrade-guides/0-12.html) e precisa de uma versão compatível com Terraform 0.11.x deste módulo,
a última versão lançada destinada ao Terraform 0.11.x é [1.0.0] (https://registry.terraform.io/modules/terraform-google-modules/bigquery/google/1.0.0).

## Atualizando

A versão atual é 4.X. Os guias a seguir estão disponíveis para auxiliar nas atualizações:

- [3.0 -> 4.0](./docs/upgrading_to_bigquery_v4.0.md)
- [2.0 -> 3.0](./docs/upgrading_to_bigquery_v3.0.md)
- [1.0 -> 2.0](./docs/upgrading_to_bigquery_v2.0.md)
- [0.1 -> 1.0](./docs/upgrading_to_bigquery_v1.0.md)

## Uso

O uso básico deste módulo é o seguinte:

```hcl
module "bigquery" {
  source  = "terraform-google-modules/bigquery/google"
  version = "~> 4.3"

  dataset_id                  = "foo"
  dataset_name                = "foo"
  description                 = "some description"
  project_id                  = "<PROJECT ID>"
  location                    = "US"
  default_table_expiration_ms = 3600000

  tables = [
  {
    table_id          = "foo",
    schema            =  "<PATH TO THE SCHEMA JSON FILE>",
    time_partitioning = {
      type                     = "DAY",
      field                    = null,
      require_partition_filter = false,
      expiration_ms            = null,
    },
    expiration_time = null,
    clustering      = ["fullVisitorId", "visitId"],
    labels          = {
      env      = "dev"
      billable = "true"
      owner    = "joedoe"
    },
  },
  {
    table_id          = "bar",
    schema            =  "<PATH TO THE SCHEMA JSON FILE>",
    time_partitioning = null,
    expiration_time   = 2524604400000, # 2050/01/01
    clustering        = [],
    labels = {
      env      = "devops"
      billable = "true"
      owner    = "joedoe"
    },
  }
  ],

  views = [
    {
      view_id    = "barview",
      use_legacy_sql = false,
      query          = <<EOF
      SELECT
       column_a,
       column_b,
      FROM
        `project_id.dataset_id.table_id`
      WHERE
        approved_user = SESSION_USER
      EOF,
      labels = {
        env      = "devops"
        billable = "true"
        owner    = "joedoe"
      }
    }
  ]
  dataset_labels = {
    env      = "dev"
    billable = "true"
  }
}
```

### Descrição detalhada das `tabelas` das variáveis

A variável `tables` deve ser fornecida como uma lista de objetos com as seguintes chaves:
```
{	
  table_id = "some_id"                        	 ID exclusivo da tabela (será usado como ID e nome exclusivo para a tabela).
  schema = "path/to/schema.json"              	 Caminho para o arquivo schema json.
  time_partitioning = {                       	 Defina como `null` para omitir a configuração de particionamento para a tabela.
        type                     = "DAY",     	 O único tipo suportado é DAY, que irá gerar uma partição por dia com base no tempo de carregamento dos dados.
        field                    = null,      	 O campo usado para determinar como criar uma partição baseada em tempo. Se o particionamento baseado em tempo for habilitado sem                                                  esse valor, a tabela será particionada com base no tempo de carregamento. Defina-o como `null` para omitir a configuração.
        require_partition_filter = false,     	 Se definido como verdadeiro, as consultas sobre esta tabela requerem a especificação de um filtro de partição que pode ser usado                                                  para a eliminação da partição. Defina-o como `null` para omitir a configuração.
        expiration_ms            = null,      	 Número de milissegundos para manter o armazenamento de uma partição.
      },	
  clustering = ["fullVisitorId", "visitId"]   	 Especifica os nomes das colunas a serem usados para armazenamento em cluster de dados. São permitidas até quatro colunas de                                                      nível superior e devem ser especificadas em ordem de prioridade decrescente. O particionamento deve ser configurado para usar                                                    clustering.
  expiration_time = 2524604400000             	 A hora em que esta tabela expira, em milissegundos desde a época. Se definido como `null`, a tabela irá persistir          indefinidamente.
  labels = {                                  	 Um mapeamento de rótulos para atribuir à tabela.
      env      = "dev"	
      billable = "true"	
    }	
}	

```

### Descrição detalhada da variável `views`

A variável `views` deve ser fornecida como uma lista de objetos com as seguintes chaves:
```	
{	
  view_id = "some_id"                                                	  ID de visualização exclusivo. será definido como um nome amigável também
  query = "Select user_id, name from `project_id.dataset_id.table`"  	  a consulta Selecionar que criará a visualização. As tabelas devem ser criadas antes.
  use_legacy_sql = false                                             	  se deve usar sql legado ou sql padrão
  labels = {                                                         	  Um mapeamento de rótulos para atribuir à exibição.
      env      = "dev"	
      billable = "true"	
  }	
}

```

## Inputs

| Nome | Descrição | Tipo | Default | Requerido |
|------|-------------|:----:|:-----:|:-----:|
| acesso | Uma matriz de objetos que define o acesso ao conjunto de dados para uma ou mais entidades. | any | `<list>` | não |
| dataset\_id | ID exclusivo para o conjunto de dados que está sendo provisionado. | string | n/a | sim |
| dataset\_labels | Pares de valores-chave em um mapa para rótulos de conjuntos de dados | map(string) | `<map>` | não |
| dataset\_name | Nome amigável para o conjunto de dados que está sendo provisionado. | string | `"null"` | não |
| default\_table\_expiration\_ms | TTL de tabelas usando o conjunto de dados em MS | number | `"null"` | não |
| delete\_contents\_on\_destroy | (Opcional) Se definido como verdadeiro, exclui todas as tabelas no conjunto de dados ao destruir o recurso; caso contrário, a destruição do recurso falhará se houver tabelas. | bool | `"null"` | não |
| description | descrição do Dataset | string | `"null"` | não |
| location | A localização regional do conjunto de dados, apenas EUA e UE são permitidos no módulo | string | `"US"` | não |
| project\_id | Projeto onde o conjunto de dados e a tabela são criados | string | n/a | sim |
| tables | Uma lista de objetos que incluem table_id, schema, clustering, time_partitioning, expiration_time e rótulos. | object | `<list>` | não |
| views | Uma lista de objetos que incluem table_id, que é view id e view query | object | `<list>` | não |

## Outputs

| Nome | Descrição |
|------|-------------|
| bigquery\_dataset | Recurso de conjunto de dados do Bigquery. |
| bigquery\_tables | Mapa de recursos da tabela bigquery sendo provisionados. |


<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->

## Requisitos

Estas seções descrevem os requisitos para usar este módulo.

### Software

As seguintes dependências devem estar disponíveis: 

- [Terraform][terraform] v0.12
- [Terraform Provider for GCP][terraform-provider-gcp] plugin v3

### Conta de Serviço

Uma conta de serviço com as funções a seguir deve ser usada para provisionar os recursos deste módulo:

- BigQuery Proprietário dos Dados: `roles/bigquery.dataOwner`

O [módulo de fábrica do projeto] [módulo de fábrica do projeto] e o 
[Módulo IAM] [módulo iam] pode ser usado em combinação para provisionar 
um conta de serviço com as funções necessárias aplicadas.

