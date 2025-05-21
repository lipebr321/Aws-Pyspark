# Capítulo 5: Rodando Spark com Wheel no S3

## Introdução

Após criar e fazer upload do seu pacote wheel para o S3, o próximo passo é executá-lo no cluster EMR. Neste capítulo, você aprenderá como submeter jobs Spark que utilizam seu pacote wheel, monitorar a execução e validar os resultados. Esta abordagem permite que você execute código Python modular e bem organizado em um ambiente distribuído.

## Pré-requisitos

Antes de começar, você precisará:

- Um cluster EMR em execução
- Um arquivo wheel (.whl) armazenado no S3
- Acesso SSH ao nó master do cluster
- Conhecimentos básicos de PySpark e EMR

## Glossário para Iniciantes 📝

- **Wheel (.whl)**: Formato de pacote Python pré-compilado
- **py-files**: Parâmetro do spark-submit para incluir arquivos Python ou zips no PYTHONPATH
- **PYTHONPATH**: Variável de ambiente que define onde o Python procura módulos
- **Driver Program**: Processo principal que coordena a execução da aplicação Spark
- **Executor**: Processo que executa as tarefas da aplicação Spark
- **YARN**: Gerenciador de recursos do Hadoop que aloca recursos para aplicações
- **ApplicationMaster**: Processo YARN que gerencia a aplicação no cluster

## Executando o Wheel Diretamente no Cluster EMR

### Passo 1: Verificar se o Wheel está Disponível no S3

Antes de executar o job, verifique se o arquivo wheel está acessível no S3:

```bash
aws s3 ls s3://meu-bucket/wheels/meu_pacote_spark-0.1.0-py3-none-any.whl
```

🖼️ **Print do terminal mostrando o resultado do comando aws s3 ls**
➡️ Execute o comando para verificar se o arquivo wheel está disponível no S3.

### Passo 2: Preparar o Script Principal

Você pode criar um script simples que importa e utiliza as funções do seu pacote. Este script será o ponto de entrada para o spark-submit:

```python
# main.py
import argparse
import logging
from pyspark.sql import SparkSession

# Importar funções do nosso pacote wheel
from meu_pacote.utils import criar_spark_session, ler_dados_csv, salvar_como_parquet
from meu_pacote.transformacoes import transformar_colunas_texto, extrair_componentes_data
from meu_pacote.validacoes import validar_valores_nulos

def main():
    # Configurar argumentos da linha de comando
    parser = argparse.ArgumentParser(description='Processador de dados com wheel')
    parser.add_argument('--input', required=True, help='Caminho de entrada dos dados (S3)')
    parser.add_argument('--output', required=True, help='Caminho de saída para os resultados (S3)')
    args = parser.parse_args()
    
    # Configurar logging
    logging.basicConfig(level=logging.INFO, format='%(asctime)s [%(levelname)s] %(message)s')
    logger = logging.getLogger(__name__)
    
    logger.info(f"Iniciando processamento: entrada={args.input}, saída={args.output}")
    
    # Criar sessão Spark
    spark = criar_spark_session("Aplicação com Wheel")
    
    try:
        # Ler dados
        df = ler_dados_csv(spark, args.input)
        logger.info(f"Dados carregados: {df.count()} registros")
        
        # Aplicar transformações
        df = transformar_colunas_texto(df, ["produto", "categoria"])
        df = extrair_componentes_data(df, "data_venda")
        
        # Verificar valores nulos
        nulos = validar_valores_nulos(df, ["produto", "valor"])
        for coluna, contagem in nulos.items():
            logger.info(f"Coluna {coluna}: {contagem} valores nulos")
        
        # Salvar resultados
        salvar_como_parquet(df, args.output, particionar_por=["ano", "mes"])
        logger.info(f"Processamento concluído com sucesso!")
        
    except Exception as e:
        logger.error(f"Erro durante o processamento: {str(e)}")
        raise
    finally:
        spark.stop()

if __name__ == "__main__":
    main()
```

Faça upload deste script para o S3:

```bash
aws s3 cp main.py s3://meu-bucket/scripts/
```

### Passo 3: Executar o Job com Spark-Submit

Agora, vamos executar o job usando o comando `spark-submit` com o parâmetro `--py-files` para incluir o wheel:

```bash
spark-submit \
  --master yarn \
  --deploy-mode cluster \
  --py-files s3://meu-bucket/wheels/meu_pacote_spark-0.1.0-py3-none-any.whl \
  s3://meu-bucket/scripts/main.py \
  --input s3://meu-bucket/dados/vendas.csv \
  --output s3://meu-bucket/resultados/vendas_processadas
```

🖼️ **Print do terminal executando o comando spark-submit**
➡️ Execute o comando spark-submit conforme mostrado acima, substituindo os caminhos do S3 pelos seus.

💡 **Dica**: O parâmetro `--py-files` pode receber múltiplos arquivos separados por vírgula, incluindo arquivos .py, .zip ou .whl.

## Código Pronto para Spark-Submit Utilizando --py-files

Vamos ver alguns exemplos mais avançados de como utilizar o parâmetro `--py-files` com diferentes configurações:

### Exemplo 1: Configuração Básica

```bash
spark-submit \
  --master yarn \
  --deploy-mode cluster \
  --py-files s3://meu-bucket/wheels/meu_pacote_spark-0.1.0-py3-none-any.whl \
  s3://meu-bucket/scripts/main.py \
  --input s3://meu-bucket/dados/vendas.csv \
  --output s3://meu-bucket/resultados/vendas_processadas
```

### Exemplo 2: Com Configurações de Recursos

```bash
spark-submit \
  --master yarn \
  --deploy-mode cluster \
  --executor-memory 4g \
  --executor-cores 2 \
  --num-executors 10 \
  --driver-memory 2g \
  --conf spark.yarn.submit.waitAppCompletion=true \
  --py-files s3://meu-bucket/wheels/meu_pacote_spark-0.1.0-py3-none-any.whl \
  s3://meu-bucket/scripts/main.py \
  --input s3://meu-bucket/dados/vendas.csv \
  --output s3://meu-bucket/resultados/vendas_processadas
```

### Exemplo 3: Com Múltiplos Arquivos Python

```bash
spark-submit \
  --master yarn \
  --deploy-mode cluster \
  --py-files s3://meu-bucket/wheels/meu_pacote_spark-0.1.0-py3-none-any.whl,s3://meu-bucket/scripts/utils_adicionais.py \
  s3://meu-bucket/scripts/main.py \
  --input s3://meu-bucket/dados/vendas.csv \
  --output s3://meu-bucket/resultados/vendas_processadas
```

### Exemplo 4: Com Configurações de Log e Checkpoint

```bash
spark-submit \
  --master yarn \
  --deploy-mode cluster \
  --conf spark.eventLog.enabled=true \
  --conf spark.eventLog.dir=s3://meu-bucket/spark-logs \
  --conf spark.yarn.maxAppAttempts=2 \
  --py-files s3://meu-bucket/wheels/meu_pacote_spark-0.1.0-py3-none-any.whl \
  s3://meu-bucket/scripts/main.py \
  --input s3://meu-bucket/dados/vendas.csv \
  --output s3://meu-bucket/resultados/vendas_processadas
```

## Monitorando o Progresso da Tarefa

### Via Linha de Comando

Após submeter o job, você pode monitorar seu progresso usando comandos YARN:

```bash
# Listar aplicações em execução
yarn application -list

# Verificar status de uma aplicação específica
yarn application -status application_1621234567890_0001

# Ver logs da aplicação
yarn logs -applicationId application_1621234567890_0001
```

🖼️ **Print do terminal mostrando o resultado do comando yarn application -list**
➡️ Execute o comando para listar as aplicações em execução no YARN.

### Via Interface Web

Você também pode monitorar o progresso através das interfaces web do EMR:

1. **ResourceManager UI**: http://MASTER_DNS:8088
2. **Spark History Server**: http://MASTER_DNS:18080

Para acessar essas interfaces, configure um túnel SSH:

```bash
ssh -i /caminho/para/MeuParDeChaves.pem -N -L 8088:localhost:8088 -L 18080:localhost:18080 hadoop@ec2-xx-xx-xx-xx.compute-1.amazonaws.com
```

🖼️ **Print da interface web do YARN ResourceManager**
➡️ Acesse http://localhost:8088 no seu navegador após configurar o túnel SSH.

## Validando o Resultado dos Dados Processados

Após a conclusão do job, é importante validar se os dados foram processados corretamente:

### Passo 1: Verificar se os Arquivos Foram Criados no S3

```bash
aws s3 ls s3://meu-bucket/resultados/vendas_processadas/
```

### Passo 2: Verificar a Estrutura de Particionamento

```bash
aws s3 ls s3://meu-bucket/resultados/vendas_processadas/ano=2023/mes=1/
```

### Passo 3: Examinar os Dados com Spark

Você pode criar um script simples para ler e validar os dados processados:

```python
# validar_resultados.py
from pyspark.sql import SparkSession

# Inicializar a sessão Spark
spark = SparkSession.builder \
    .appName("Validação de Resultados") \
    .getOrCreate()

# Ler os dados processados
df = spark.read.parquet("s3://meu-bucket/resultados/vendas_processadas/")

# Mostrar o schema
print("Schema dos dados processados:")
df.printSchema()

# Contar registros
total_registros = df.count()
print(f"Total de registros: {total_registros}")

# Verificar partições
print("Distribuição por ano e mês:")
df.groupBy("ano", "mes").count().orderBy("ano", "mes").show()

# Verificar estatísticas básicas
print("Estatísticas de valores:")
df.select("valor", "valor_com_imposto").describe().show()

# Encerrar a sessão Spark
spark.stop()
```

Execute este script localmente no nó master:

```bash
spark-submit validar_resultados.py
```

## Boas Práticas para Execução de Jobs com Wheels 🔥

1. **Versionamento**: Inclua a versão no nome do arquivo wheel para facilitar o controle de versões.

2. **Logs Detalhados**: Implemente logging detalhado em seu código para facilitar a depuração.

3. **Tratamento de Erros**: Implemente tratamento de exceções robusto para falhas previsíveis.

4. **Checkpointing**: Para jobs longos, use checkpointing para permitir recuperação em caso de falhas:
   ```python
   spark.sparkContext.setCheckpointDir("s3://meu-bucket/checkpoints")
   df.checkpoint()
   ```

5. **Configuração de Recursos**: Ajuste os parâmetros de memória e cores com base no tamanho dos dados:
   ```bash
   --executor-memory 4g --executor-cores 2 --num-executors 10
   ```

6. **Monitoramento**: Configure alertas e monitoramento para jobs críticos.

7. **Testes**: Teste seu código com conjuntos de dados menores antes de executar em produção.

8. **Documentação**: Mantenha documentação atualizada sobre como executar seus jobs.

## Solução de Problemas Comuns

### Erro "No module named 'meu_pacote'"

Verifique:
- Se o caminho do wheel no parâmetro `--py-files` está correto
- Se o wheel foi criado corretamente
- Se o wheel está acessível no S3

### Erro "Container killed by YARN"

Possíveis soluções:
- Aumente a memória do executor (`--executor-memory`)
- Reduza o paralelismo ou o número de partições
- Verifique se há transformações que consomem muita memória

### Job Lento ou Travado

Verifique:
- Número de partições (muito poucas ou muitas)
- Skew nos dados (distribuição desigual)
- Operações de shuffle excessivas
- Configurações de memória e cores inadequadas

### Erro ao Acessar Arquivos no S3

Verifique:
- Permissões IAM do cluster
- Se o bucket e os objetos existem
- Se o formato do caminho S3 está correto

## Conclusão

Neste capítulo, você aprendeu como executar aplicações Spark que utilizam pacotes wheel armazenados no S3, monitorar o progresso da execução e validar os resultados. Esta abordagem permite que você desenvolva código Python modular e bem organizado, facilitando a manutenção e reutilização. No próximo capítulo, aprenderemos como clonar um cluster EMR para reutilizar configurações.

---

🔍 **Próximo Capítulo**: [Clonando Cluster EMR](06_clonando_cluster_emr.md)
