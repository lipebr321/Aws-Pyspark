# Capítulo 9: Verificando Resultado no S3

## Introdução

Após executar suas aplicações Spark no EMR e processar os dados, é fundamental verificar os resultados gerados no S3. A validação dos dados processados é uma etapa crítica para garantir a qualidade e a integridade dos resultados. Neste capítulo, você aprenderá como acessar e verificar arquivos Parquet e outros formatos no S3, navegar pelo console da AWS e utilizar ferramentas como o Amazon Athena para consultar e validar os dados processados.

## Pré-requisitos

Antes de começar, você precisará:

- Uma conta AWS ativa
- Dados processados armazenados no S3
- Permissões adequadas para acessar o S3
- Conhecimentos básicos de SQL (para uso do Athena)

## Glossário para Iniciantes 📝

- **S3**: Amazon Simple Storage Service, serviço de armazenamento de objetos da AWS
- **Bucket**: Container principal para armazenamento de objetos no S3
- **Parquet**: Formato colunar de armazenamento otimizado para análise de dados
- **Partição**: Divisão dos dados em diretórios baseados em valores de colunas específicas
- **Athena**: Serviço de consulta interativa da AWS que permite analisar dados no S3 usando SQL
- **Glue**: Serviço de catalogação e ETL da AWS que integra com S3 e Athena
- **Prefixo**: Caminho dentro de um bucket S3, similar a um diretório em sistemas de arquivos

## Como Acessar Arquivos Parquet Gerados

Os arquivos Parquet são um formato colunar otimizado para análise de dados, amplamente utilizado em aplicações Spark. Vamos explorar diferentes maneiras de acessar e verificar esses arquivos.

### Usando o Console S3 da AWS

#### Passo 1: Acessar o Console S3

1. Faça login no [Console AWS](https://console.aws.amazon.com/)
2. Navegue até o serviço S3


➡️ Na barra de pesquisa superior, digite "S3" e selecione o serviço "S3" nos resultados.

#### Passo 2: Navegar até o Bucket

1. Na lista de buckets, clique no bucket onde seus dados foram salvos
2. Navegue pelos prefixos (diretórios) até encontrar seus dados processados


➡️ Clique no nome do bucket que contém seus dados processados.

#### Passo 3: Explorar a Estrutura de Particionamento

Se seus dados foram particionados (por exemplo, por ano e mês), você verá uma estrutura de diretórios:

```
meu-bucket/
└── resultados/
    └── vendas_processadas/
        ├── ano=2022/
        │   ├── mes=1/
        │   │   ├── part-00000-xxxx.parquet
        │   │   └── part-00001-xxxx.parquet
        │   └── mes=2/
        │       ├── part-00000-xxxx.parquet
        │       └── part-00001-xxxx.parquet
        └── ano=2023/
            └── mes=1/
                ├── part-00000-xxxx.parquet
                └── part-00001-xxxx.parquet
```


➡️ Navegue pelos diretórios particionados para visualizar a estrutura.

#### Passo 4: Verificar Metadados dos Arquivos

1. Clique em um arquivo Parquet
2. Na aba "Visão geral", você verá metadados como tamanho, data de modificação, classe de armazenamento, etc.

➡️ Clique em um arquivo Parquet e observe os metadados disponíveis.

### Usando a AWS CLI

A AWS CLI oferece uma maneira eficiente de interagir com o S3 via linha de comando:

#### Listar Objetos no S3

```bash
# Listar o conteúdo de um prefixo
aws s3 ls s3://meu-bucket/resultados/vendas_processadas/

# Listar recursivamente (incluindo subdiretórios)
aws s3 ls s3://meu-bucket/resultados/vendas_processadas/ --recursive

# Listar apenas arquivos Parquet
aws s3 ls s3://meu-bucket/resultados/vendas_processadas/ --recursive | grep .parquet
```


➡️ Execute o comando aws s3 ls e observe a lista de objetos no S3.

#### Verificar Tamanho Total dos Dados

```bash
# Calcular o tamanho total dos dados
aws s3 ls s3://meu-bucket/resultados/vendas_processadas/ --recursive --summarize
```

Exemplo de saída:
```
Total Objects: 42
Total Size: 1.2 GB
```

#### Baixar Arquivos para Inspeção Local

```bash
# Baixar um arquivo específico
aws s3 cp s3://meu-bucket/resultados/vendas_processadas/ano=2023/mes=1/part-00000-xxxx.parquet ./

# Baixar um diretório inteiro
aws s3 cp s3://meu-bucket/resultados/vendas_processadas/ano=2023/mes=1/ ./ --recursive
```

### Usando PySpark para Verificar os Dados

PySpark é uma excelente ferramenta para verificar e validar os dados Parquet:

```python
from pyspark.sql import SparkSession

# Inicializar a sessão Spark
spark = SparkSession.builder \
    .appName("Verificação de Dados") \
    .getOrCreate()

# Ler os dados Parquet
df = spark.read.parquet("s3://meu-bucket/resultados/vendas_processadas/")

# Verificar o schema
print("Schema dos dados:")
df.printSchema()

# Contar registros
total_registros = df.count()
print(f"Total de registros: {total_registros}")

# Mostrar amostra dos dados
print("Amostra dos dados:")
df.show(5)

# Verificar estatísticas básicas
print("Estatísticas básicas:")
df.describe().show()

# Verificar distribuição por partição
print("Distribuição por ano e mês:")
df.groupBy("ano", "mes").count().orderBy("ano", "mes").show()

# Verificar valores nulos
print("Contagem de valores nulos por coluna:")
from pyspark.sql.functions import col, count, when
df.select([count(when(col(c).isNull(), c)).alias(c) for c in df.columns]).show()

# Encerrar a sessão Spark
spark.stop()
```

Salve este código em um arquivo `verificar_dados.py` e execute-o no cluster EMR:

```bash
spark-submit verificar_dados.py
```


➡️ Execute o script e observe os resultados da verificação dos dados.

## Visualizando o Conteúdo com Athena (Bônus)

O Amazon Athena é um serviço de consulta interativa que permite analisar dados diretamente no S3 usando SQL padrão, sem necessidade de carregar os dados em um banco de dados.

### Passo 1: Acessar o Console Athena

1. Faça login no [Console AWS](https://console.aws.amazon.com/)
2. Navegue até o serviço Athena


➡️ Na barra de pesquisa superior, digite "Athena" e selecione o serviço "Athena" nos resultados.

### Passo 2: Configurar o Local de Resultados

Antes de usar o Athena, você precisa configurar um local no S3 para armazenar os resultados das consultas:

1. Clique em "Configurações"
2. Em "Local dos resultados de consulta", insira um caminho S3, por exemplo:
   ```
   s3://meu-bucket/athena-results/
   ```
3. Clique em "Salvar"


➡️ Configure o local dos resultados de consulta no S3.

### Passo 3: Criar um Banco de Dados

1. No painel "Editor de consultas", clique no menu suspenso "Database" (Banco de dados)
2. Selecione "Create database" (Criar banco de dados)
3. Digite um nome para o banco de dados, por exemplo: `dados_processados`
4. Clique em "Create database" (Criar banco de dados)


➡️ Crie um novo banco de dados no Athena.

### Passo 4: Criar uma Tabela

Você pode criar uma tabela que aponta para seus dados Parquet no S3:

```sql
CREATE EXTERNAL TABLE vendas_processadas (
  id STRING,
  produto STRING,
  categoria STRING,
  valor DOUBLE,
  valor_com_imposto DOUBLE,
  data_formatada DATE,
  cliente STRING
)
PARTITIONED BY (ano INT, mes INT)
STORED AS PARQUET
LOCATION 's3://meu-bucket/resultados/vendas_processadas/';
```


➡️ Execute o comando SQL para criar a tabela externa.

### Passo 5: Carregar Partições

Para que o Athena reconheça as partições existentes:

```sql
MSCK REPAIR TABLE vendas_processadas;
```

Alternativamente, você pode adicionar partições manualmente:

```sql
ALTER TABLE vendas_processadas ADD
PARTITION (ano=2022, mes=1) LOCATION 's3://meu-bucket/resultados/vendas_processadas/ano=2022/mes=1/'
PARTITION (ano=2022, mes=2) LOCATION 's3://meu-bucket/resultados/vendas_processadas/ano=2022/mes=2/'
PARTITION (ano=2023, mes=1) LOCATION 's3://meu-bucket/resultados/vendas_processadas/ano=2023/mes=1/';
```

### Passo 6: Consultar os Dados

Agora você pode executar consultas SQL para analisar seus dados:

```sql
-- Contar registros
SELECT COUNT(*) AS total_registros FROM vendas_processadas;

-- Verificar distribuição por partição
SELECT ano, mes, COUNT(*) AS contagem
FROM vendas_processadas
GROUP BY ano, mes
ORDER BY ano, mes;

-- Calcular estatísticas por categoria
SELECT 
  categoria,
  COUNT(*) AS contagem,
  AVG(valor) AS valor_medio,
  MIN(valor) AS valor_minimo,
  MAX(valor) AS valor_maximo,
  SUM(valor) AS valor_total
FROM vendas_processadas
GROUP BY categoria
ORDER BY valor_total DESC;

-- Filtrar dados específicos
SELECT *
FROM vendas_processadas
WHERE ano = 2023 AND mes = 1 AND valor > 1000
LIMIT 10;
```


➡️ Execute uma consulta SQL e observe os resultados.

### Passo 7: Exportar Resultados

Você pode baixar os resultados das consultas em vários formatos:

1. Após executar uma consulta, clique em "Download results" (Baixar resultados)
2. Selecione o formato desejado (CSV, JSON, etc.)

➡️ Clique no botão de download e selecione o formato desejado.

## Verificando a Integridade dos Dados

Além de visualizar os dados, é importante verificar sua integridade:

### 1. Verificar Contagem de Registros

Compare a contagem de registros de entrada e saída:

```python
# Em PySpark
entrada_count = spark.read.csv("s3://meu-bucket/dados/vendas.csv", header=True).count()
saida_count = spark.read.parquet("s3://meu-bucket/resultados/vendas_processadas/").count()

print(f"Registros de entrada: {entrada_count}")
print(f"Registros de saída: {saida_count}")
print(f"Diferença: {abs(entrada_count - saida_count)}")
```

### 2. Verificar Consistência dos Dados

Verifique se as transformações foram aplicadas corretamente:

```python
# Em PySpark
# Ler dados originais
df_original = spark.read.csv("s3://meu-bucket/dados/vendas.csv", header=True)

# Ler dados processados
df_processado = spark.read.parquet("s3://meu-bucket/resultados/vendas_processadas/")

# Verificar se valor_com_imposto = valor * 1.1
from pyspark.sql.functions import col, round
df_check = df_processado.withColumn(
    "check_imposto", 
    round(col("valor") * 1.1, 2) == round(col("valor_com_imposto"), 2)
)

# Contar registros inconsistentes
inconsistentes = df_check.filter(~col("check_imposto")).count()
print(f"Registros com cálculo de imposto inconsistente: {inconsistentes}")
```

### 3. Verificar Particionamento

Verifique se o particionamento foi aplicado corretamente:

```python
# Em PySpark
from pyspark.sql.functions import year, month, to_date

# Verificar particionamento por ano e mês
df_original = spark.read.csv("s3://meu-bucket/dados/vendas.csv", header=True)
df_original = df_original.withColumn("data_formatada", to_date(col("data")))
df_original = df_original.withColumn("ano_calculado", year(col("data_formatada")))
df_original = df_original.withColumn("mes_calculado", month(col("data_formatada")))

df_processado = spark.read.parquet("s3://meu-bucket/resultados/vendas_processadas/")

# Verificar se os valores de partição correspondem aos valores calculados
df_check = df_processado.withColumn(
    "check_ano", 
    col("ano") == col("ano_calculado")
).withColumn(
    "check_mes",
    col("mes") == col("mes_calculado")
)

# Contar registros com particionamento inconsistente
inconsistentes_ano = df_check.filter(~col("check_ano")).count()
inconsistentes_mes = df_check.filter(~col("check_mes")).count()

print(f"Registros com ano inconsistente: {inconsistentes_ano}")
print(f"Registros com mês inconsistente: {inconsistentes_mes}")
```

## Boas Práticas para Verificação de Dados 🔥

1. **Validação Automática**: Crie scripts de validação que podem ser executados automaticamente após o processamento.

2. **Testes de Integridade**: Verifique se todos os registros foram processados e se não houve perda de dados.

3. **Verificação de Schema**: Confirme se o schema dos dados processados está conforme esperado.

4. **Validação de Regras de Negócio**: Verifique se as transformações aplicadas estão de acordo com as regras de negócio.

5. **Monitoramento de Qualidade**: Implemente métricas de qualidade de dados (completude, precisão, consistência).

6. **Documentação**: Mantenha documentação atualizada sobre a estrutura e o significado dos dados processados.

7. **Versionamento**: Considere versionar seus dados processados para facilitar a rastreabilidade.

8. **Backup**: Mantenha backups dos dados originais antes de aplicar transformações irreversíveis.

## Ferramentas Adicionais para Análise de Dados

Além do Athena, existem outras ferramentas que podem ser úteis para verificar e analisar dados no S3:

### 1. AWS Glue Data Catalog

O AWS Glue pode rastrear automaticamente seus dados no S3 e criar um catálogo de metadados:

1. No console AWS, navegue até o serviço Glue
2. Crie um crawler para rastrear seus dados no S3
3. Execute o crawler para popular o catálogo de dados
4. Use o catálogo com Athena, Redshift Spectrum ou EMR


➡️ Configure um crawler para rastrear seus dados no S3.

### 2. Amazon QuickSight

O QuickSight é um serviço de visualização de dados que pode se conectar diretamente ao Athena:

1. No console AWS, navegue até o serviço QuickSight
2. Crie uma nova análise
3. Selecione Athena como fonte de dados
4. Escolha o banco de dados e a tabela criados anteriormente
5. Crie visualizações interativas dos seus dados

➡️ Crie uma visualização dos seus dados processados.

### 3. Jupyter Notebooks no EMR

Você pode usar Jupyter Notebooks no EMR para análise interativa:

1. Ao criar o cluster EMR, selecione a aplicação "Jupyter" ou "JupyterHub"
2. Conecte-se à interface web do Jupyter
3. Crie um novo notebook com kernel PySpark
4. Use o código PySpark mostrado anteriormente para analisar seus dados


➡️ Use o Jupyter Notebook para análise interativa dos dados.

## Solução de Problemas Comuns

### 1. Arquivos Parquet Não Podem Ser Lidos

**Sintomas**: Erro ao tentar ler arquivos Parquet

**Possíveis causas e soluções**:
- **Incompatibilidade de versão**: Verifique se a versão do Parquet é compatível com a ferramenta de leitura
- **Arquivos corrompidos**: Verifique se o job Spark foi concluído com sucesso
- **Permissões insuficientes**: Verifique as permissões IAM para acessar os objetos no S3

### 2. Partições Não Aparecem no Athena

**Sintomas**: Algumas partições não aparecem nas consultas do Athena

**Possíveis causas e soluções**:
- **Partições não registradas**: Execute `MSCK REPAIR TABLE` para detectar novas partições
- **Formato de partição inconsistente**: Verifique se todas as partições seguem o mesmo padrão
- **Problemas de permissão**: Verifique se o Athena tem permissão para acessar todas as partições

### 3. Contagem de Registros Inconsistente

**Sintomas**: A contagem de registros de saída não corresponde à contagem de entrada

**Possíveis causas e soluções**:
- **Filtros aplicados**: Verifique se algum filtro foi aplicado durante o processamento
- **Duplicatas**: Verifique se há registros duplicados nos dados de entrada ou saída
- **Erros durante o processamento**: Verifique os logs do Spark para erros

## Conclusão

Neste capítulo, você aprendeu como acessar e verificar arquivos Parquet e outros formatos no S3, navegar pelo console da AWS e utilizar ferramentas como o Amazon Athena para consultar e validar os dados processados. A verificação dos resultados é uma etapa crucial no pipeline de processamento de dados, garantindo a qualidade e a integridade dos dados processados. No próximo capítulo, concluiremos a apostila com um resumo dos aprendizados e referências úteis para aprofundamento.

---

🔍 **Próximo Capítulo**: [Conclusão e Referências](10_conclusao_referencias.md)
