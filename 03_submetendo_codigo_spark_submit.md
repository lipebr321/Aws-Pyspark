# Capítulo 3: Submetendo Código com Spark-Submit

## Introdução

Após conectar-se ao cluster EMR, o próximo passo é executar aplicações Spark para processar seus dados. O `spark-submit` é a ferramenta padrão para submeter aplicações Spark ao cluster, permitindo configurar recursos, dependências e parâmetros de execução. Neste capítulo, você aprenderá a estrutura básica do comando `spark-submit` e como executar aplicações Python que processam dados armazenados no S3.

## Pré-requisitos

Antes de começar, você precisará:

- Um cluster EMR em execução
- Acesso SSH ao nó master do cluster
- Conhecimentos básicos de Python e Spark
- Um bucket S3 para armazenar scripts e dados

## Glossário para Iniciantes 📝

- **Spark-submit**: Ferramenta de linha de comando para executar aplicações Spark
- **YARN**: Gerenciador de recursos do Hadoop que aloca recursos para aplicações
- **Deploy-mode**: Modo de implantação que define onde o driver da aplicação Spark será executado
- **Driver Program**: Processo principal que coordena a execução da aplicação Spark
- **Executor**: Processo que executa as tarefas da aplicação Spark
- **S3**: Amazon Simple Storage Service, serviço de armazenamento de objetos da AWS
- **Parquet**: Formato colunar de armazenamento otimizado para análise de dados

## Estrutura Básica de um Comando Spark-Submit

O comando `spark-submit` possui a seguinte estrutura básica:

```bash
spark-submit \
  --master <master-url> \
  --deploy-mode <deploy-mode> \
  --conf <key>=<value> \
  --class <main-class> \
  --jars <comma-separated-jars> \
  --py-files <comma-separated-python-files> \
  <application-jar-or-python-file> \
  [application-arguments]
```

### Parâmetros Principais

- `--master`: Define o gerenciador de recursos (yarn, local, spark://host:port)
- `--deploy-mode`: Define o modo de implantação (client ou cluster)
- `--conf`: Define configurações específicas do Spark
- `--class`: Define a classe principal para aplicações Java/Scala
- `--jars`: Lista de JARs adicionais para incluir no classpath
- `--py-files`: Lista de arquivos Python adicionais para incluir no PYTHONPATH
- `<application-jar-or-python-file>`: Caminho para o arquivo JAR ou Python da aplicação
- `[application-arguments]`: Argumentos para a aplicação

## Exemplo Funcional com Python

Vamos criar um exemplo simples de aplicação PySpark que lê dados de um arquivo CSV no S3, realiza algumas transformações e salva o resultado em formato Parquet no S3.

### Passo 1: Criar o Script Python

Primeiro, vamos criar um script Python básico para processar dados:

```python
# processador_dados.py
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, year, month, dayofmonth

def main():
    # Inicializar a sessão Spark
    spark = SparkSession.builder \
        .appName("Processador de Dados S3") \
        .getOrCreate()
    
    # Definir caminhos de entrada e saída
    input_path = "s3://meu-bucket/dados/vendas.csv"
    output_path = "s3://meu-bucket/resultados/vendas_processadas"
    
    # Ler dados do CSV
    print("Lendo dados do S3...")
    df = spark.read.option("header", "true") \
                   .option("inferSchema", "true") \
                   .csv(input_path)
    
    print(f"Schema do DataFrame original:")
    df.printSchema()
    
    print(f"Total de registros: {df.count()}")
    
    # Realizar transformações
    print("Realizando transformações...")
    df_processado = df.withColumn("valor_com_imposto", col("valor") * 1.1) \
                      .withColumn("ano", year(col("data"))) \
                      .withColumn("mes", month(col("data"))) \
                      .withColumn("dia", dayofmonth(col("data")))
    
    # Mostrar alguns resultados
    print("Amostra dos dados processados:")
    df_processado.show(5)
    
    # Salvar resultados em formato Parquet
    print(f"Salvando resultados em {output_path}")
    df_processado.write.mode("overwrite") \
                       .partitionBy("ano", "mes") \
                       .parquet(output_path)
    
    print("Processamento concluído com sucesso!")
    
    # Encerrar a sessão Spark
    spark.stop()

if __name__ == "__main__":
    main()
```

![image](https://github.com/user-attachments/assets/56e3dc65-52d2-48aa-a2b6-8b2b0b5afb43)

➡️ Crie o arquivo processador_dados.py com o código acima usando um editor como vscode ou vim.

### Passo 2: Fazer Upload do Script para o S3

Você pode fazer upload do script diretamente para o S3:

```bash
# Primeiro, crie o arquivo localmente no nó master
nano processador_dados.py
# Cole o código e salve (Ctrl+O, Enter, Ctrl+X)

# Depois, faça upload para o S3
aws s3 cp processador_dados.py s3://meu-bucket/scripts/
```

### Passo 3: Submeter a Aplicação com Spark-Submit

Agora, vamos submeter a aplicação ao cluster usando o comando `spark-submit`:

```bash
spark-submit \
  --master yarn \
  --deploy-mode cluster \
  --conf spark.yarn.submit.waitAppCompletion=true \
  --conf spark.sql.parquet.compression.codec=snappy \
  s3://meu-bucket/scripts/processador_dados.py
```


➡️ Execute o comando spark-submit conforme mostrado acima, substituindo o caminho do S3 pelo seu bucket.

💡 **Dica**: Use o parâmetro `--conf spark.yarn.submit.waitAppCompletion=true` para que o comando `spark-submit` aguarde a conclusão da aplicação antes de retornar.

## Explicação Detalhada dos Parâmetros

### `--master yarn`

Este parâmetro especifica que o YARN será usado como gerenciador de recursos. No EMR, o YARN é o gerenciador padrão e recomendado.

### `--deploy-mode cluster`

No modo `cluster`, o driver da aplicação Spark é executado dentro do cluster YARN, em um dos nós. Isso é recomendado para aplicações de produção, pois:

1. Se a conexão SSH cair, a aplicação continua rodando
2. Os logs do driver ficam no cluster, não no terminal local
3. O driver utiliza recursos do cluster, não da máquina que submete o job

No modo `client` (padrão), o driver é executado na máquina que submete o job, o que é útil para desenvolvimento e debugging.


➡️ No modo client, o driver roda na máquina que submete o job. No modo cluster, o driver roda dentro do YARN.

### Configurações Adicionais Úteis

Você pode adicionar várias configurações usando o parâmetro `--conf`:

```bash
spark-submit \
  --master yarn \
  --deploy-mode cluster \
  --conf spark.executor.memory=4g \
  --conf spark.executor.cores=2 \
  --conf spark.executor.instances=10 \
  --conf spark.driver.memory=2g \
  --conf spark.sql.shuffle.partitions=200 \
  s3://meu-bucket/scripts/processador_dados.py
```

## Como Verificar Logs no EMR

Quando você executa uma aplicação Spark no modo cluster, os logs não são exibidos diretamente no terminal. Você precisa acessá-los através do YARN ou do Spark History Server.

### Verificando Logs via YARN

1. Primeiro, liste as aplicações YARN para obter o ID da aplicação:

```bash
yarn application -list
```

Exemplo de saída:
```
Total number of applications (application-types: [] and states: [SUBMITTED, ACCEPTED, RUNNING]):1
                Application-Id      Application-Name        Application-Type          User           State         Final-State         Progress                       Tracking-URL
application_1621234567890_0001     Processador de Dados S3           SPARK        hadoop           RUNNING           UNDEFINED             10%                  http://ip-172-31-25-107.ec2.internal:8088/proxy/application_1621234567890_0001/
```

2. Visualize os logs da aplicação:

```bash
yarn logs -applicationId application_1621234567890_0001
```

💡 **Dica**: Para ver apenas os logs do driver:

```bash
yarn logs -applicationId application_1621234567890_0001 -log_files stdout
```

### Verificando Logs via Interface Web

O EMR fornece interfaces web para monitorar aplicações:

1. **ResourceManager UI**: http://MASTER_DNS:8088
2. **Spark History Server**: http://MASTER_DNS:18080

Para acessar essas interfaces, você precisa configurar um túnel SSH:

```bash
ssh -i /caminho/para/MeuParDeChaves.pem -N -L 8088:localhost:8088 -L 18080:localhost:18080 hadoop@ec2-xx-xx-xx-xx.compute-1.amazonaws.com
```


➡️ Acesse http://localhost:8088 no seu navegador após configurar o túnel SSH.

## Monitorando o Progresso da Aplicação

### Via Linha de Comando

```bash
# Verificar status da aplicação
yarn application -status application_1621234567890_0001

# Listar contêineres da aplicação
yarn container -list application_1621234567890_0001
```

### Via Interface Web

Na interface do ResourceManager (http://localhost:8088), você pode:

1. Ver o status de todas as aplicações
2. Acessar logs detalhados
3. Monitorar o uso de recursos
4. Matar aplicações se necessário


➡️ Clique no ID da aplicação na interface do ResourceManager para ver detalhes.

## Exemplos Avançados de Spark-Submit

### Exemplo com Argumentos de Linha de Comando

```python
# processador_com_args.py
from pyspark.sql import SparkSession
import sys

def main():
    if len(sys.argv) != 3:
        print("Uso: processador_com_args.py <input_path> <output_path>")
        sys.exit(1)
        
    input_path = sys.argv[1]
    output_path = sys.argv[2]
    
    spark = SparkSession.builder \
        .appName("Processador com Argumentos") \
        .getOrCreate()
    
    # Resto do código...
    
if __name__ == "__main__":
    main()
```

Comando para executar:

```bash
spark-submit \
  --master yarn \
  --deploy-mode cluster \
  s3://meu-bucket/scripts/processador_com_args.py \
  s3://meu-bucket/dados/entrada.csv \
  s3://meu-bucket/resultados/saida
```

### Exemplo com Configuração de Memória e Cores

```bash
spark-submit \
  --master yarn \
  --deploy-mode cluster \
  --driver-memory 4g \
  --executor-memory 8g \
  --executor-cores 4 \
  --num-executors 5 \
  s3://meu-bucket/scripts/processador_dados.py
```

### Exemplo com Arquivos Python Adicionais

```bash
spark-submit \
  --master yarn \
  --deploy-mode cluster \
  --py-files s3://meu-bucket/scripts/utils.py,s3://meu-bucket/scripts/transformacoes.py \
  s3://meu-bucket/scripts/processador_dados.py
```

## Boas Práticas para Spark-Submit 🔥

1. **Modo de Implantação**: Use `--deploy-mode cluster` para aplicações de produção.

2. **Gerenciamento de Recursos**: Ajuste `--executor-memory`, `--executor-cores` e `--num-executors` com base no tamanho do cluster e dos dados.

3. **Particionamento**: Configure `spark.sql.shuffle.partitions` para otimizar o desempenho (regra geral: 2-3x o número total de cores no cluster).

4. **Compressão**: Use `spark.sql.parquet.compression.codec=snappy` para um bom equilíbrio entre compressão e velocidade.

5. **Logs**: Ative logs detalhados com `spark.eventLog.enabled=true` para melhor monitoramento.

6. **Tratamento de Erros**: Implemente tratamento de exceções robusto em seus scripts Python.

7. **Checkpointing**: Para jobs longos, considere usar checkpointing para recuperação de falhas:
   ```python
   spark.sparkContext.setCheckpointDir("s3://meu-bucket/checkpoints")
   df.checkpoint()
   ```

## Solução de Problemas Comuns

### Erro "No such file or directory"

Verifique:
- Se o caminho do S3 está correto
- Se as permissões IAM permitem acesso ao bucket
- Se o script foi carregado corretamente para o S3

### Erro "Container killed by YARN"

Possíveis soluções:
- Aumente a memória do executor (`--executor-memory`)
- Reduza o paralelismo ou o número de partições
- Verifique se há transformações que consomem muita memória (como `groupBy` ou `join`)

### Aplicação Lenta

Verifique:
- Número de partições (muito poucas ou muitas)
- Skew nos dados (distribuição desigual)
- Operações de shuffle excessivas
- Configurações de memória e cores inadequadas

## Conclusão

Neste capítulo, você aprendeu a estrutura básica do comando `spark-submit`, como executar aplicações Python que processam dados no S3 e como monitorar o progresso e verificar logs no EMR. No próximo capítulo, aprenderemos como empacotar seu código Python em um arquivo wheel (.whl) para facilitar a distribuição e execução no cluster.

---

🔍 **Próximo Capítulo**: [Criando e Subindo Wheel no VS Code](04_criando_e_subindo_wheel_vs_code.md)
