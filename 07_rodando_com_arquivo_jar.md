# Capítulo 7: Rodando com Arquivo JAR

## Introdução

Além de executar código Python no Spark, muitas vezes precisamos trabalhar com aplicações escritas em Java ou Scala, que são compiladas em arquivos JAR. O Apache Spark foi originalmente desenvolvido em Scala e oferece suporte nativo para essas linguagens. Neste capítulo, você aprenderá como compilar um arquivo JAR para o Spark e executá-lo no cluster EMR, além de entender as diferenças entre aplicações Python e JAR no contexto do Spark.

## Pré-requisitos

Antes de começar, você precisará:

- Um cluster EMR em execução
- Conhecimentos básicos de Java ou Scala
- JDK (Java Development Kit) instalado
- Maven ou SBT para compilação
- Acesso SSH ao nó master do cluster

## Glossário para Iniciantes 📝

- **JAR**: Java ARchive, formato de pacote que contém classes Java compiladas e recursos
- **Scala**: Linguagem de programação que combina programação orientada a objetos e funcional
- **Maven**: Ferramenta de automação de compilação para projetos Java
- **SBT**: Scala Build Tool, ferramenta de compilação para projetos Scala
- **JVM**: Java Virtual Machine, ambiente de execução para bytecode Java
- **Bytecode**: Código intermediário gerado pela compilação de código Java ou Scala
- **Classe Principal**: Ponto de entrada de uma aplicação Java/Scala (contém método main)

## Como Compilar um .jar em Java/Scala para o Spark

### Estrutura de um Projeto Scala para Spark

Vamos começar com a estrutura básica de um projeto Scala para Spark usando SBT:

```
meu-projeto-spark/
├── build.sbt                 # Arquivo de configuração do SBT
├── project/
│   └── build.properties      # Versão do SBT
└── src/
    └── main/
        └── scala/
            └── com/
                └── exemplo/
                    └── SparkApp.scala  # Código-fonte Scala
```

### Arquivo build.sbt

O arquivo `build.sbt` define as dependências e configurações do projeto:

```scala
name := "MeuProjetoSpark"
version := "1.0"
scalaVersion := "2.12.15"

// Dependências
libraryDependencies ++= Seq(
  "org.apache.spark" %% "spark-core" % "3.3.1" % "provided",
  "org.apache.spark" %% "spark-sql" % "3.3.1" % "provided"
)

// Configurações para criar um "fat JAR" (com todas as dependências)
assembly / assemblyJarName := "meu-projeto-spark-assembly-1.0.jar"

// Resolver conflitos de arquivos durante o assembly
assembly / assemblyMergeStrategy := {
  case PathList("META-INF", xs @ _*) => MergeStrategy.discard
  case "application.conf" => MergeStrategy.concat
  case x => MergeStrategy.first
}
```
➡️ Crie o arquivo build.sbt com o conteúdo acima.

💡 **Dica**: O modificador `% "provided"` indica que as dependências do Spark não serão incluídas no JAR final, pois elas já estão disponíveis no ambiente de execução do EMR.

### Arquivo project/build.properties

```
sbt.version=1.8.0
```

### Exemplo de Código Scala

Vamos criar um exemplo simples de aplicação Spark em Scala:

```scala
// src/main/scala/com/exemplo/SparkApp.scala
package com.exemplo

import org.apache.spark.sql.{SparkSession, DataFrame}
import org.apache.spark.sql.functions._

object SparkApp {
  def main(args: Array[String]): Unit = {
    if (args.length < 2) {
      System.err.println("Uso: SparkApp <input-path> <output-path>")
      System.exit(1)
    }
    
    val inputPath = args(0)
    val outputPath = args(1)
    
    // Criar sessão Spark
    val spark = SparkSession.builder()
      .appName("Exemplo Spark em Scala")
      .getOrCreate()
    
    try {
      println(s"Processando dados de $inputPath para $outputPath")
      
      // Ler dados
      val df = spark.read
        .option("header", "true")
        .option("inferSchema", "true")
        .csv(inputPath)
      
      println(s"Schema dos dados:")
      df.printSchema()
      
      // Realizar transformações
      val resultDF = processarDados(df)
      
      // Mostrar amostra dos resultados
      println("Amostra dos dados processados:")
      resultDF.show(5)
      
      // Salvar resultados
      resultDF.write
        .mode("overwrite")
        .parquet(outputPath)
      
      println(s"Processamento concluído com sucesso. Resultados salvos em $outputPath")
    } finally {
      spark.stop()
    }
  }
  
  def processarDados(df: DataFrame): DataFrame = {
    // Exemplo de transformações
    df.withColumn("valor_com_imposto", col("valor") * 1.1)
      .withColumn("ano", year(col("data")))
      .withColumn("mes", month(col("data")))
      .withColumn("categoria_upper", upper(col("categoria")))
  }
}
```


➡️ Crie o arquivo SparkApp.scala com o conteúdo acima.

### Compilando o Projeto com SBT

Para compilar o projeto e criar o arquivo JAR:

```bash
# Navegar até o diretório do projeto
cd meu-projeto-spark

# Compilar e criar o JAR
sbt clean assembly
```


➡️ Execute o comando e observe a saída mostrando o processo de compilação.

O arquivo JAR será gerado em `target/scala-2.12/meu-projeto-spark-assembly-1.0.jar`.

### Compilando o Projeto com Maven

Alternativamente, você pode usar Maven para projetos Java. Aqui está um exemplo de `pom.xml`:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.exemplo</groupId>
    <artifactId>meu-projeto-spark</artifactId>
    <version>1.0</version>

    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <encoding>UTF-8</encoding>
        <scala.version>2.12.15</scala.version>
        <scala.compat.version>2.12</scala.compat.version>
        <spark.version>3.3.1</spark.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_${scala.compat.version}</artifactId>
            <version>${spark.version}</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql_${scala.compat.version}</artifactId>
            <version>${spark.version}</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>net.alchim31.maven</groupId>
                <artifactId>scala-maven-plugin</artifactId>
                <version>4.5.6</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>compile</goal>
                            <goal>testCompile</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.2.4</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <transformers>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass>com.exemplo.SparkApp</mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

Para compilar com Maven:

```bash
mvn clean package
```

O arquivo JAR será gerado em `target/meu-projeto-spark-1.0.jar`.

## Exemplo de Spark-Submit com .jar

Após compilar o JAR, você precisa fazer upload para o S3 e executá-lo no cluster EMR:

### Passo 1: Fazer Upload do JAR para o S3

```bash
aws s3 cp target/scala-2.12/meu-projeto-spark-assembly-1.0.jar s3://meu-bucket/jars/
```


➡️ Execute o comando aws s3 cp e observe a saída confirmando o upload.

### Passo 2: Executar o JAR com Spark-Submit

```bash
spark-submit \
  --master yarn \
  --deploy-mode cluster \
  --class com.exemplo.SparkApp \
  s3://meu-bucket/jars/meu-projeto-spark-assembly-1.0.jar \
  s3://meu-bucket/dados/vendas.csv \
  s3://meu-bucket/resultados/vendas_processadas_scala
```


➡️ Execute o comando spark-submit conforme mostrado acima, substituindo os caminhos do S3 pelos seus.

### Parâmetros Específicos para JARs

- `--class`: Especifica a classe principal (com o método main) que será executada
- `--jars`: Lista de JARs adicionais necessários (separados por vírgula)
- `--packages`: Dependências Maven a serem baixadas automaticamente

### Exemplo Avançado com Configurações Adicionais

```bash
spark-submit \
  --master yarn \
  --deploy-mode cluster \
  --executor-memory 4g \
  --executor-cores 2 \
  --num-executors 10 \
  --conf spark.yarn.submit.waitAppCompletion=true \
  --conf spark.sql.shuffle.partitions=200 \
  --conf spark.serializer=org.apache.spark.serializer.KryoSerializer \
  --class com.exemplo.SparkApp \
  --jars s3://meu-bucket/jars/biblioteca-auxiliar.jar \
  s3://meu-bucket/jars/meu-projeto-spark-assembly-1.0.jar \
  s3://meu-bucket/dados/vendas.csv \
  s3://meu-bucket/resultados/vendas_processadas_scala
```

## Diferença entre .py e .jar no Contexto do Spark

### Arquitetura de Execução

#### Python (.py)

1. **Processo de Execução**:
   - O código Python é interpretado pelo PySpark
   - PySpark se comunica com o Spark Core (JVM) através do Py4J
   - Há uma camada adicional de comunicação entre Python e JVM

2. **Desempenho**:
   - Geralmente mais lento devido à serialização/desserialização entre Python e JVM
   - Overhead de comunicação entre processos

3. **Uso de Memória**:
   - Maior consumo de memória devido à duplicação de dados entre JVM e Python


➡️ O PySpark utiliza Py4J para comunicação entre o código Python e o Spark Core na JVM.

#### Java/Scala (.jar)

1. **Processo de Execução**:
   - O código Java/Scala é executado diretamente na JVM
   - Comunicação direta com o Spark Core
   - Sem camadas adicionais de comunicação

2. **Desempenho**:
   - Geralmente mais rápido, especialmente para operações complexas
   - Sem overhead de serialização/desserialização entre linguagens

3. **Uso de Memória**:
   - Mais eficiente, sem duplicação de dados entre ambientes


➡️ Aplicações Java/Scala são executadas diretamente na JVM, sem camadas intermediárias.

### Quando Usar Cada Um

#### Python (.py)

**Vantagens**:
- Desenvolvimento mais rápido
- Sintaxe mais simples e concisa
- Ecossistema rico para ciência de dados (pandas, numpy, scikit-learn)
- Melhor para prototipagem e exploração de dados

**Ideal para**:
- Cientistas de dados e analistas
- Projetos que requerem integração com bibliotecas Python
- Prototipagem rápida e análise exploratória
- Equipes com conhecimento predominante em Python

#### Java/Scala (.jar)

**Vantagens**:
- Melhor desempenho para operações complexas
- Tipagem estática (menos erros em tempo de execução)
- Acesso direto a todas as funcionalidades do Spark
- Melhor para aplicações de produção de alto desempenho

**Ideal para**:
- Engenheiros de dados e desenvolvedores
- Aplicações de produção com requisitos de desempenho
- Processamento de grandes volumes de dados
- Equipes com conhecimento em Java/Scala

### Comparação de Código

Vamos comparar o mesmo processamento em Python e Scala:

#### Python (PySpark)

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, upper, year, month

# Criar sessão Spark
spark = SparkSession.builder \
    .appName("Exemplo PySpark") \
    .getOrCreate()

# Ler dados
df = spark.read.option("header", "true") \
               .option("inferSchema", "true") \
               .csv("s3://meu-bucket/dados/vendas.csv")

# Transformar dados
df_processado = df.withColumn("valor_com_imposto", col("valor") * 1.1) \
                  .withColumn("ano", year(col("data"))) \
                  .withColumn("mes", month(col("data"))) \
                  .withColumn("categoria_upper", upper(col("categoria")))

# Salvar resultados
df_processado.write.mode("overwrite") \
                   .parquet("s3://meu-bucket/resultados/vendas_processadas")

# Encerrar sessão
spark.stop()
```

#### Scala

```scala
import org.apache.spark.sql.SparkSession
import org.apache.spark.sql.functions._

// Criar sessão Spark
val spark = SparkSession.builder()
  .appName("Exemplo Scala")
  .getOrCreate()

// Ler dados
val df = spark.read
  .option("header", "true")
  .option("inferSchema", "true")
  .csv("s3://meu-bucket/dados/vendas.csv")

// Transformar dados
val dfProcessado = df.withColumn("valor_com_imposto", col("valor") * 1.1)
  .withColumn("ano", year(col("data")))
  .withColumn("mes", month(col("data")))
  .withColumn("categoria_upper", upper(col("categoria")))

// Salvar resultados
dfProcessado.write
  .mode("overwrite")
  .parquet("s3://meu-bucket/resultados/vendas_processadas")

// Encerrar sessão
spark.stop()
```

Como você pode ver, a lógica e a estrutura são muito semelhantes, com diferenças principalmente na sintaxe.

## Boas Práticas para Desenvolvimento com Java/Scala 🔥

1. **Gerenciamento de Dependências**: Use Maven ou SBT para gerenciar dependências de forma consistente.

2. **Serialização**: Implemente `Serializable` em classes personalizadas para evitar erros de serialização.

3. **Kryo Serializer**: Use o Kryo Serializer para melhor desempenho:
   ```bash
   --conf spark.serializer=org.apache.spark.serializer.KryoSerializer
   ```

4. **Tipagem**: Aproveite o sistema de tipos estáticos para detectar erros em tempo de compilação.

5. **Datasets vs DataFrames**: Use Datasets para operações que se beneficiam de tipagem estática.

6. **Logging**: Implemente logging adequado para facilitar a depuração:
   ```scala
   import org.apache.log4j.{Logger, Level}
   val logger = Logger.getLogger(getClass.getName)
   logger.info("Processando dados...")
   ```

7. **Testes Unitários**: Escreva testes unitários com frameworks como ScalaTest ou JUnit.

8. **Documentação**: Use Scaladoc/Javadoc para documentar seu código.

## Solução de Problemas Comuns

### Erro "ClassNotFoundException"

Verifique:
- Se o parâmetro `--class` está correto e corresponde ao nome completo da classe (incluindo o pacote)
- Se o JAR contém a classe especificada (use `jar tvf meu-jar.jar | grep ClasseName`)
- Se há conflitos de versão entre as dependências

### Erro "NoSuchMethodError"

Possíveis causas:
- Incompatibilidade de versões entre bibliotecas
- Método não encontrado na versão da biblioteca disponível no cluster

Solução:
- Verifique as versões das dependências
- Use `--packages` para garantir a versão correta das dependências

### Erro "OutOfMemoryError"

Possíveis soluções:
- Aumente a memória do driver e executores
- Otimize as transformações para reduzir o uso de memória
- Ajuste as configurações de garbage collection

## Conclusão

Neste capítulo, você aprendeu como compilar um arquivo JAR para o Spark usando Java ou Scala, como executá-lo no cluster EMR e as diferenças entre aplicações Python e JAR no contexto do Spark. Cada abordagem tem suas vantagens e desvantagens, e a escolha entre elas depende dos requisitos do projeto, das habilidades da equipe e dos objetivos de desempenho. No próximo capítulo, aprenderemos como acessar e analisar logs para depurar aplicações Spark no EMR.

---

🔍 **Próximo Capítulo**: [Acessando Logs e Debugando](08_acessando_logs_e_debugando.md)
