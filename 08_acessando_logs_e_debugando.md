# Capítulo 8: Acessando Logs e Debugando

## Introdução

Quando executamos aplicações Spark no EMR, é comum enfrentarmos problemas que precisam ser diagnosticados e resolvidos. O acesso aos logs e a compreensão das técnicas de debugging são essenciais para identificar e corrigir erros, otimizar o desempenho e garantir o funcionamento correto das aplicações. Neste capítulo, você aprenderá onde encontrar os logs no EMR, como acessá-los e as melhores práticas para debugging de aplicações Spark.

## Pré-requisitos

Antes de começar, você precisará:

- Um cluster EMR em execução
- Aplicações Spark executadas no cluster
- Acesso SSH ao nó master do cluster
- Conhecimentos básicos de Spark e YARN

## Glossário para Iniciantes 📝

- **Log**: Registro cronológico de eventos ocorridos durante a execução de uma aplicação
- **YARN**: Yet Another Resource Negotiator, gerenciador de recursos do Hadoop
- **Container**: Unidade de execução no YARN que encapsula recursos (CPU, memória)
- **ApplicationMaster**: Processo YARN que gerencia a aplicação no cluster
- **Driver**: Processo principal que coordena a execução da aplicação Spark
- **Executor**: Processo que executa as tarefas da aplicação Spark
- **Spark History Server**: Interface web para visualizar logs de aplicações Spark concluídas
- **stderr/stdout**: Fluxos de saída padrão para erros e saída normal, respectivamente

## Onde Encontrar Logs no EMR

Os logs no EMR estão distribuídos em vários locais, dependendo do tipo de log e do componente relacionado:

### 1. Logs do YARN

O YARN é o gerenciador de recursos usado pelo Spark no EMR e mantém logs detalhados de todas as aplicações:

#### Localização no Sistema de Arquivos

```
/var/log/hadoop-yarn/containers/
/var/log/hadoop-yarn/apps/
```

#### Acesso via Linha de Comando

Para listar aplicações YARN:

```bash
yarn application -list
```

🖼️ **Print do terminal mostrando o resultado do comando yarn application -list**
➡️ Execute o comando para listar as aplicações YARN em execução ou concluídas recentemente.

Para ver os logs de uma aplicação específica:

```bash
yarn logs -applicationId application_1621234567890_0001
```

💡 **Dica**: Você pode filtrar os logs por tipo usando o parâmetro `-log_files`:

```bash
# Ver apenas logs do stdout
yarn logs -applicationId application_1621234567890_0001 -log_files stdout

# Ver apenas logs do stderr
yarn logs -applicationId application_1621234567890_0001 -log_files stderr
```

### 2. Logs do Spark

#### Localização no Sistema de Arquivos

Os logs específicos do Spark estão disponíveis em:

```
/var/log/spark/
```

Para aplicações em execução via YARN, os logs do Spark estão dentro dos diretórios de containers do YARN:

```
/var/log/hadoop-yarn/containers/container_1621234567890_0001_01_000001/
```

#### Acesso via Spark History Server

O Spark History Server fornece uma interface web para visualizar logs de aplicações concluídas:

```
http://MASTER_DNS:18080
```

Para acessar o Spark History Server, configure um túnel SSH:

```bash
ssh -i /caminho/para/MeuParDeChaves.pem -N -L 18080:localhost:18080 hadoop@ec2-xx-xx-xx-xx.compute-1.amazonaws.com
```

🖼️ **Print da interface web do Spark History Server**
➡️ Acesse http://localhost:18080 no seu navegador após configurar o túnel SSH.

### 3. Logs do EMR

O EMR também mantém logs específicos do serviço:

#### Localização no Sistema de Arquivos

```
/var/log/aws-emr-service-nanny/
/var/log/bootstrap-actions/
```

#### Logs no S3

Se você configurou o EMR para armazenar logs no S3 (recomendado), eles estarão disponíveis em:

```
s3://meu-bucket-logs-emr/logs/j-XXXXXXXXXXXXXXX/
```

Onde `j-XXXXXXXXXXXXXXX` é o ID do cluster.

## Como Acessar Arquivos .stderr, .stdout, .log.gz

### Arquivos .stderr e .stdout

Estes arquivos contêm a saída de erro e a saída padrão dos processos, respectivamente:

#### Via SSH

```bash
# Acessar o nó master via SSH
ssh -i /caminho/para/MeuParDeChaves.pem hadoop@ec2-xx-xx-xx-xx.compute-1.amazonaws.com

# Navegar até o diretório de containers
cd /var/log/hadoop-yarn/containers/

# Listar os containers
ls -la

# Encontrar o container específico (geralmente o primeiro é o ApplicationMaster)
cd container_1621234567890_0001_01_000001/

# Ver o conteúdo dos arquivos de log
cat stdout
cat stderr
```

🖼️ **Print do terminal mostrando o conteúdo de um arquivo stderr**
➡️ Execute o comando cat stderr e observe a saída com mensagens de erro.

#### Via YARN CLI

```bash
yarn logs -applicationId application_1621234567890_0001 -log_files stdout > app_stdout.log
yarn logs -applicationId application_1621234567890_0001 -log_files stderr > app_stderr.log
```

### Arquivos .log.gz

Estes são arquivos de log compactados, comuns para logs mais antigos ou volumosos:

```bash
# Visualizar o conteúdo sem descompactar
zcat hadoop-yarn-resourcemanager-ip-172-31-25-107.log.gz | less

# Descompactar e salvar
gunzip -c hadoop-yarn-resourcemanager-ip-172-31-25-107.log.gz > resourcemanager.log
```

### Logs no S3

Para acessar logs armazenados no S3:

```bash
# Listar logs disponíveis
aws s3 ls s3://meu-bucket-logs-emr/logs/j-XXXXXXXXXXXXXXX/

# Copiar logs para o sistema de arquivos local
aws s3 cp s3://meu-bucket-logs-emr/logs/j-XXXXXXXXXXXXXXX/containers/application_1621234567890_0001/ ./logs/ --recursive

# Ver um arquivo específico
aws s3 cp s3://meu-bucket-logs-emr/logs/j-XXXXXXXXXXXXXXX/containers/application_1621234567890_0001/container_1621234567890_0001_01_000001/stderr - | less
```

## Boas Práticas para Debugging 🔥

### 1. Implementar Logging Adequado

Em aplicações Python:

```python
import logging

# Configurar logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s [%(levelname)s] %(message)s'
)
logger = logging.getLogger(__name__)

# Usar em diferentes níveis
logger.debug("Informação detalhada para debugging")
logger.info("Informação geral sobre o progresso")
logger.warning("Aviso sobre situações inesperadas")
logger.error("Erro que permite continuar a execução")
logger.critical("Erro crítico que impede a continuação")
```

Em aplicações Scala:

```scala
import org.apache.log4j.{Logger, Level}

// Configurar logger
val logger = Logger.getLogger(getClass.getName)

// Usar em diferentes níveis
logger.debug("Informação detalhada para debugging")
logger.info("Informação geral sobre o progresso")
logger.warn("Aviso sobre situações inesperadas")
logger.error("Erro que permite continuar a execução")
logger.fatal("Erro crítico que impede a continuação")
```

### 2. Monitorar o Progresso

Adicione logs para monitorar o progresso de operações longas:

```python
# Em Python
total_registros = df.count()
logger.info(f"Total de registros: {total_registros}")

# Processar em batches
for i, batch in enumerate(batches):
    logger.info(f"Processando batch {i+1}/{len(batches)}")
    # Processamento...
    logger.info(f"Batch {i+1} concluído")
```

### 3. Capturar e Registrar Exceções

```python
# Em Python
try:
    # Código que pode gerar exceção
    df = spark.read.csv(input_path)
    # Processamento...
except Exception as e:
    logger.error(f"Erro ao processar dados: {str(e)}")
    # Detalhes adicionais para debugging
    import traceback
    logger.error(traceback.format_exc())
    raise  # Re-lançar a exceção ou tratar adequadamente
```

### 4. Usar Checkpoints para Debugging

Salve estados intermediários para facilitar o debugging:

```python
# Em Python
# Configurar diretório de checkpoint
spark.sparkContext.setCheckpointDir("s3://meu-bucket/checkpoints")

# Após transformações complexas
df_transformado = aplicar_transformacoes_complexas(df)
df_transformado.checkpoint()  # Materializa o DataFrame
df_transformado.cache()       # Mantém em memória para acesso rápido

# Verificar resultado intermediário
df_transformado.show(5)
```

### 5. Visualizar Plano de Execução

Analise o plano de execução para identificar gargalos:

```python
# Em Python
# Mostrar plano de execução lógico
df_final.explain()

# Mostrar plano de execução físico detalhado
df_final.explain(True)
```

🖼️ **Print do terminal mostrando o resultado do comando explain()**
➡️ Execute df_final.explain(True) e observe o plano de execução detalhado.

### 6. Monitorar Métricas de Execução

Use a interface web do Spark para monitorar métricas em tempo real:

- **Spark UI**: http://MASTER_DNS:4040 (para aplicações em execução)
- **YARN ResourceManager**: http://MASTER_DNS:8088
- **Spark History Server**: http://MASTER_DNS:18080 (para aplicações concluídas)

🖼️ **Print da interface web do Spark UI mostrando métricas de execução**
➡️ Acesse http://localhost:4040 no seu navegador após configurar o túnel SSH.

## Técnicas Avançadas de Debugging

### 1. Debugging Distribuído

Para depurar problemas específicos de nós:

```bash
# Verificar logs de um nó específico
yarn logs -applicationId application_1621234567890_0001 -nodeAddress ip-172-31-25-107.ec2.internal:8041
```

### 2. Análise de Skew de Dados

Identificar e corrigir skew (distribuição desigual) de dados:

```python
# Em Python
# Verificar distribuição de partições
partition_counts = df.rdd.glom().map(len).collect()
print(f"Contagem de registros por partição: {partition_counts}")
print(f"Min: {min(partition_counts)}, Max: {max(partition_counts)}, Avg: {sum(partition_counts)/len(partition_counts)}")

# Reparticionar para distribuir melhor os dados
df_rebalanced = df.repartition(100)
```

### 3. Debugging de Problemas de Memória

Para identificar e resolver problemas de memória:

```bash
# Verificar configurações de memória
yarn application -status application_1621234567890_0001 | grep memory

# Aumentar memória no spark-submit
--executor-memory 8g --driver-memory 4g
```

Ajustar configurações de memória no código:

```python
# Em Python
spark = SparkSession.builder \
    .appName("Minha Aplicação") \
    .config("spark.executor.memory", "8g") \
    .config("spark.driver.memory", "4g") \
    .config("spark.memory.fraction", "0.8") \
    .config("spark.memory.storageFraction", "0.3") \
    .getOrCreate()
```

### 4. Debugging de Problemas de Serialização

Para resolver problemas de serialização:

```python
# Em Python
# Usar broadcast para variáveis grandes
large_dict = {"key1": "value1", "key2": "value2", ...}
broadcast_dict = spark.sparkContext.broadcast(large_dict)

# Acessar no RDD/DataFrame
def process_with_dict(row):
    dict_value = broadcast_dict.value.get(row.key)
    # Processamento...
    return result

result_df = df.rdd.map(process_with_dict).toDF()
```

## Exemplos Práticos de Debugging

### Exemplo 1: Identificando e Corrigindo Erros de Transformação

Suponha que você tenha o seguinte código com erro:

```python
# Código com erro
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, to_date

spark = SparkSession.builder.appName("Debug Example").getOrCreate()

# Ler dados
df = spark.read.csv("s3://meu-bucket/dados/vendas.csv", header=True)

# Transformação com erro
df_processado = df.withColumn("data_formatada", to_date(col("data"), "yyyy-MM-dd"))
df_processado = df_processado.withColumn("valor_total", col("valor") * col("quantidade"))

# Salvar resultados
df_processado.write.parquet("s3://meu-bucket/resultados/vendas_processadas")
```

Ao executar, você recebe um erro. Vamos depurar:

1. **Verificar o erro nos logs**:

```bash
yarn logs -applicationId application_1621234567890_0001 -log_files stderr
```

Saída (exemplo):
```
Py4JJavaError: An error occurred while calling o111.withColumn.
: org.apache.spark.sql.AnalysisException: cannot resolve 'quantidade' given input columns: [id, produto, categoria, valor, data];
```

2. **Corrigir o código**:

```python
# Verificar schema
df.printSchema()

# Verificar se a coluna existe
print("Colunas disponíveis:", df.columns)

# Corrigir o código
if "quantidade" not in df.columns:
    # Adicionar coluna padrão ou ajustar a lógica
    df = df.withColumn("quantidade", lit(1))

# Continuar com a transformação
df_processado = df.withColumn("valor_total", col("valor") * col("quantidade"))
```

### Exemplo 2: Debugging de Problemas de Performance

Suponha que você tenha uma aplicação que está demorando muito para executar:

1. **Analisar o plano de execução**:

```python
df_final.explain(True)
```

2. **Identificar operações custosas** (como shuffles, joins, etc.)

3. **Otimizar o código**:

```python
# Antes: Join custoso
df_joined = df1.join(df2, df1.id == df2.id)

# Depois: Join otimizado com broadcast
from pyspark.sql.functions import broadcast
df_joined = df1.join(broadcast(df2), df1.id == df2.id)

# Persistir DataFrames intermediários usados múltiplas vezes
df_intermediario = transformacao_complexa(df)
df_intermediario.cache()

# Reparticionar antes de operações de agrupamento
df_reparticionado = df.repartitionByRange(100, "chave_distribuicao")
```

## Solução de Problemas Comuns

### 1. "Container killed by YARN"

**Sintomas**: Aplicação falha com mensagem "Container killed by YARN for exceeding memory limits"

**Soluções**:
- Aumentar a memória do executor: `--executor-memory 8g`
- Reduzir o paralelismo: `--conf spark.sql.shuffle.partitions=100`
- Otimizar transformações que consomem muita memória
- Usar persistência em disco em vez de memória: `.persist(StorageLevel.DISK_ONLY)`

### 2. "Task not serializable"

**Sintomas**: Erro "Task not serializable: java.io.NotSerializableException"

**Soluções**:
- Garantir que todas as classes usadas em funções lambda sejam serializáveis
- Usar `broadcast` para variáveis não serializáveis
- Mover a criação de objetos não serializáveis para dentro das funções lambda

### 3. "No space left on device"

**Sintomas**: Erro "No space left on device" nos logs

**Soluções**:
- Limpar diretórios temporários: `/tmp`, `/mnt/tmp`
- Configurar o Spark para usar o S3 para shuffle: `spark.shuffle.storage.s3.path`
- Aumentar o tamanho do volume EBS do nó master

### 4. Aplicação Lenta

**Sintomas**: A aplicação executa, mas é muito mais lenta do que o esperado

**Soluções**:
- Verificar o plano de execução com `explain(True)`
- Identificar e corrigir skew de dados
- Otimizar joins com `broadcast` para tabelas pequenas
- Ajustar o número de partições
- Usar formatos de arquivo otimizados (Parquet, ORC)
- Aplicar filtros o mais cedo possível no pipeline

## Conclusão

Neste capítulo, você aprendeu onde encontrar logs no EMR, como acessar arquivos de log em diferentes formatos, e as melhores práticas para debugging de aplicações Spark. A capacidade de diagnosticar e resolver problemas é essencial para o desenvolvimento e manutenção de aplicações Spark eficientes e confiáveis. No próximo capítulo, aprenderemos como verificar os resultados do processamento no S3.

---

🔍 **Próximo Capítulo**: [Verificando Resultado no S3](09_verificando_resultado_no_s3.md)
