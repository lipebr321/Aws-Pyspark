# Capítulo 4: Criando e Subindo Wheel no VS Code

## Introdução

Quando trabalhamos com aplicações Spark mais complexas, é comum organizar o código em múltiplos arquivos e módulos Python. Para facilitar a distribuição e execução desse código no cluster EMR, uma prática recomendada é empacotar tudo em um arquivo wheel (.whl). Neste capítulo, você aprenderá como estruturar seu projeto no VS Code, criar um arquivo wheel e fazer upload para o S3, preparando-o para execução no EMR.

## Pré-requisitos

Antes de começar, você precisará:

- Visual Studio Code instalado
- Python 3.6+ instalado
- Conhecimentos básicos de Python e estrutura de pacotes
- Um bucket S3 para armazenar o arquivo wheel
- AWS CLI configurada (para upload para o S3)

## Glossário para Iniciantes 📝

- **Wheel (.whl)**: Formato de pacote Python pré-compilado que facilita a instalação
- **setup.py**: Arquivo que contém metadados e instruções para empacotar um projeto Python
- **VS Code**: Visual Studio Code, um editor de código leve e poderoso
- **Módulo**: Arquivo Python que contém definições e declarações
- **Pacote**: Diretório contendo módulos Python e um arquivo __init__.py
- **AWS CLI**: Interface de linha de comando da AWS para interagir com serviços como o S3
- **S3**: Amazon Simple Storage Service, serviço de armazenamento de objetos da AWS

## Estrutura de Pastas no VS Code

Uma estrutura organizada de projeto é fundamental para facilitar o desenvolvimento e a manutenção. Vamos criar uma estrutura típica para um projeto Spark:

```
meu_projeto_spark/
│
├── setup.py                  # Arquivo de configuração para criar o wheel
├── README.md                 # Documentação do projeto
├── requirements.txt          # Dependências do projeto
│
├── src/                      # Código-fonte do projeto
│   ├── meu_pacote/           # Pacote principal
│   │   ├── __init__.py       # Torna o diretório um pacote Python
│   │   ├── transformacoes.py # Módulo com transformações de dados
│   │   ├── validacoes.py     # Módulo com funções de validação
│   │   └── utils.py          # Módulo com funções utilitárias
│   │
│   └── scripts/              # Scripts executáveis
│       ├── __init__.py
│       └── processar_dados.py # Script principal para spark-submit
│
└── tests/                    # Testes unitários
    ├── __init__.py
    ├── test_transformacoes.py
    └── test_validacoes.py
```

🖼️ **Print da estrutura de pastas no VS Code**
➡️ Abra o VS Code e crie a estrutura de pastas conforme mostrado acima.

### Criando a Estrutura de Pastas

Você pode criar esta estrutura manualmente no VS Code ou usar comandos no terminal:

```bash
# Criar a estrutura de diretórios
mkdir -p meu_projeto_spark/src/meu_pacote
mkdir -p meu_projeto_spark/src/scripts
mkdir -p meu_projeto_spark/tests

# Criar arquivos __init__.py vazios
touch meu_projeto_spark/src/meu_pacote/__init__.py
touch meu_projeto_spark/src/scripts/__init__.py
touch meu_projeto_spark/tests/__init__.py

# Criar arquivos básicos
touch meu_projeto_spark/setup.py
touch meu_projeto_spark/README.md
touch meu_projeto_spark/requirements.txt
```

## Criando os Arquivos do Projeto

Vamos criar os arquivos principais do projeto:

### 1. setup.py

O arquivo `setup.py` é essencial para criar o pacote wheel. Ele contém metadados sobre o projeto e instruções para o empacotamento:

```python
# setup.py
from setuptools import setup, find_packages

setup(
    name="meu_pacote_spark",
    version="0.1.0",
    author="Seu Nome",
    author_email="seu.email@exemplo.com",
    description="Pacote para processamento de dados com Spark",
    long_description=open("README.md").read(),
    long_description_content_type="text/markdown",
    url="https://github.com/seu-usuario/meu-projeto-spark",
    packages=find_packages(where="src"),
    package_dir={"": "src"},
    classifiers=[
        "Programming Language :: Python :: 3",
        "License :: OSI Approved :: MIT License",
        "Operating System :: OS Independent",
    ],
    python_requires=">=3.6",
    install_requires=[
        "pyspark>=3.0.0",
        "pandas>=1.0.0",
    ],
)
```

🖼️ **Print do arquivo setup.py no VS Code**
➡️ Crie o arquivo setup.py com o conteúdo acima.

### 2. README.md

```markdown
# Meu Pacote Spark

Pacote Python para processamento de dados com Apache Spark no EMR.

## Instalação

```bash
pip install meu_pacote_spark-0.1.0-py3-none-any.whl
```

## Uso

```python
from meu_pacote.transformacoes import transformar_dados
from meu_pacote.validacoes import validar_schema

# Seu código aqui
```
```

### 3. requirements.txt

```
pyspark>=3.0.0
pandas>=1.0.0
pytest>=6.0.0
```

### 4. src/meu_pacote/transformacoes.py

```python
# src/meu_pacote/transformacoes.py
from pyspark.sql import DataFrame
from pyspark.sql.functions import col, upper, to_date, year, month, dayofmonth

def transformar_colunas_texto(df: DataFrame, colunas: list) -> DataFrame:
    """
    Converte colunas de texto para maiúsculas.
    
    Args:
        df: DataFrame do Spark
        colunas: Lista de nomes de colunas para transformar
        
    Returns:
        DataFrame com as colunas transformadas
    """
    for coluna in colunas:
        df = df.withColumn(coluna, upper(col(coluna)))
    return df

def extrair_componentes_data(df: DataFrame, coluna_data: str) -> DataFrame:
    """
    Extrai ano, mês e dia de uma coluna de data.
    
    Args:
        df: DataFrame do Spark
        coluna_data: Nome da coluna de data
        
    Returns:
        DataFrame com colunas adicionais para ano, mês e dia
    """
    return df.withColumn("data_formatada", to_date(col(coluna_data))) \
             .withColumn("ano", year(col("data_formatada"))) \
             .withColumn("mes", month(col("data_formatada"))) \
             .withColumn("dia", dayofmonth(col("data_formatada")))

def calcular_metricas(df: DataFrame, coluna_valor: str) -> DataFrame:
    """
    Calcula métricas adicionais com base em uma coluna de valor.
    
    Args:
        df: DataFrame do Spark
        coluna_valor: Nome da coluna com valores numéricos
        
    Returns:
        DataFrame com colunas adicionais de métricas
    """
    return df.withColumn("valor_com_imposto", col(coluna_valor) * 1.1) \
             .withColumn("valor_com_desconto", col(coluna_valor) * 0.9) \
             .withColumn("valor_arredondado", col(coluna_valor).cast("integer"))
```

### 5. src/meu_pacote/validacoes.py

```python
# src/meu_pacote/validacoes.py
from pyspark.sql import DataFrame
from pyspark.sql.types import StructType

def validar_schema(df: DataFrame, schema_esperado: StructType) -> bool:
    """
    Valida se o DataFrame possui o schema esperado.
    
    Args:
        df: DataFrame do Spark
        schema_esperado: Schema esperado
        
    Returns:
        True se o schema for válido, False caso contrário
    """
    return df.schema == schema_esperado

def validar_valores_nulos(df: DataFrame, colunas: list) -> dict:
    """
    Verifica se existem valores nulos nas colunas especificadas.
    
    Args:
        df: DataFrame do Spark
        colunas: Lista de nomes de colunas para verificar
        
    Returns:
        Dicionário com contagem de nulos por coluna
    """
    resultado = {}
    for coluna in colunas:
        contagem = df.filter(df[coluna].isNull()).count()
        resultado[coluna] = contagem
    return resultado

def validar_intervalo_valores(df: DataFrame, coluna: str, min_valor: float, max_valor: float) -> bool:
    """
    Verifica se todos os valores de uma coluna estão dentro de um intervalo.
    
    Args:
        df: DataFrame do Spark
        coluna: Nome da coluna para verificar
        min_valor: Valor mínimo aceitável
        max_valor: Valor máximo aceitável
        
    Returns:
        True se todos os valores estiverem no intervalo, False caso contrário
    """
    fora_intervalo = df.filter((col(coluna) < min_valor) | (col(coluna) > max_valor)).count()
    return fora_intervalo == 0
```

### 6. src/meu_pacote/utils.py

```python
# src/meu_pacote/utils.py
from pyspark.sql import SparkSession, DataFrame
import logging

def criar_spark_session(app_name: str) -> SparkSession:
    """
    Cria uma sessão Spark com configurações otimizadas.
    
    Args:
        app_name: Nome da aplicação
        
    Returns:
        Sessão Spark configurada
    """
    return SparkSession.builder \
        .appName(app_name) \
        .config("spark.sql.adaptive.enabled", "true") \
        .config("spark.sql.adaptive.coalescePartitions.enabled", "true") \
        .config("spark.sql.shuffle.partitions", "200") \
        .getOrCreate()

def ler_dados_csv(spark: SparkSession, caminho: str, tem_cabecalho: bool = True) -> DataFrame:
    """
    Lê dados de um arquivo CSV.
    
    Args:
        spark: Sessão Spark
        caminho: Caminho para o arquivo CSV
        tem_cabecalho: Indica se o CSV tem cabeçalho
        
    Returns:
        DataFrame com os dados do CSV
    """
    return spark.read \
        .option("header", str(tem_cabecalho).lower()) \
        .option("inferSchema", "true") \
        .csv(caminho)

def salvar_como_parquet(df: DataFrame, caminho: str, modo: str = "overwrite", particionar_por: list = None) -> None:
    """
    Salva um DataFrame como arquivo Parquet.
    
    Args:
        df: DataFrame do Spark
        caminho: Caminho para salvar o arquivo
        modo: Modo de escrita (overwrite, append, etc.)
        particionar_por: Lista de colunas para particionar os dados
    """
    writer = df.write.mode(modo)
    
    if particionar_por:
        writer = writer.partitionBy(*particionar_por)
    
    writer.parquet(caminho)
    logging.info(f"Dados salvos com sucesso em: {caminho}")
```

### 7. src/scripts/processar_dados.py

```python
# src/scripts/processar_dados.py
from pyspark.sql.types import StructType, StructField, StringType, DoubleType, DateType
import argparse
import logging
import sys

# Importar módulos do nosso pacote
from meu_pacote.utils import criar_spark_session, ler_dados_csv, salvar_como_parquet
from meu_pacote.transformacoes import transformar_colunas_texto, extrair_componentes_data, calcular_metricas
from meu_pacote.validacoes import validar_schema, validar_valores_nulos

def configurar_logging():
    """Configura o sistema de logging."""
    logging.basicConfig(
        level=logging.INFO,
        format='%(asctime)s [%(levelname)s] %(message)s',
        handlers=[logging.StreamHandler(sys.stdout)]
    )

def definir_schema_vendas():
    """Define o schema esperado para os dados de vendas."""
    return StructType([
        StructField("id_venda", StringType(), False),
        StructField("produto", StringType(), True),
        StructField("categoria", StringType(), True),
        StructField("valor", DoubleType(), True),
        StructField("data_venda", StringType(), True),
        StructField("cliente", StringType(), True)
    ])

def processar_dados(input_path, output_path):
    """
    Função principal para processar os dados.
    
    Args:
        input_path: Caminho para os dados de entrada no S3
        output_path: Caminho para salvar os resultados no S3
    """
    logging.info(f"Iniciando processamento de dados")
    logging.info(f"Entrada: {input_path}")
    logging.info(f"Saída: {output_path}")
    
    # Criar sessão Spark
    spark = criar_spark_session("Processamento de Vendas")
    
    try:
        # Ler dados
        df = ler_dados_csv(spark, input_path)
        logging.info(f"Dados carregados com sucesso. Total de registros: {df.count()}")
        
        # Validar schema
        schema_esperado = definir_schema_vendas()
        if not validar_schema(df, schema_esperado):
            logging.warning("O schema dos dados não corresponde ao esperado!")
            logging.info("Schema atual:")
            df.printSchema()
        
        # Verificar valores nulos
        colunas_importantes = ["produto", "valor", "data_venda"]
        nulos = validar_valores_nulos(df, colunas_importantes)
        for coluna, contagem in nulos.items():
            if contagem > 0:
                logging.warning(f"Coluna {coluna} contém {contagem} valores nulos")
        
        # Aplicar transformações
        df_transformado = df
        
        # Transformar texto para maiúsculas
        df_transformado = transformar_colunas_texto(df_transformado, ["produto", "categoria", "cliente"])
        
        # Extrair componentes de data
        df_transformado = extrair_componentes_data(df_transformado, "data_venda")
        
        # Calcular métricas adicionais
        df_transformado = calcular_metricas(df_transformado, "valor")
        
        # Mostrar amostra dos dados processados
        logging.info("Amostra dos dados processados:")
        df_transformado.show(5, truncate=False)
        
        # Salvar resultados
        salvar_como_parquet(
            df_transformado, 
            output_path, 
            particionar_por=["ano", "mes"]
        )
        
        logging.info(f"Processamento concluído com sucesso!")
        
    except Exception as e:
        logging.error(f"Erro durante o processamento: {str(e)}")
        raise
    finally:
        spark.stop()

def main():
    """Função principal que processa argumentos da linha de comando."""
    parser = argparse.ArgumentParser(description='Processador de dados de vendas')
    parser.add_argument('--input', required=True, help='Caminho de entrada dos dados (S3)')
    parser.add_argument('--output', required=True, help='Caminho de saída para os resultados (S3)')
    
    args = parser.parse_args()
    
    configurar_logging()
    processar_dados(args.input, args.output)

if __name__ == "__main__":
    main()
```

## Criando o Arquivo Wheel

Agora que temos a estrutura do projeto e os arquivos necessários, vamos criar o arquivo wheel:

### Passo 1: Instalar as Ferramentas Necessárias

Primeiro, precisamos instalar as ferramentas para criar o wheel:

```bash
pip install wheel setuptools
```

### Passo 2: Executar o Comando para Criar o Wheel

Navegue até o diretório raiz do projeto e execute:

```bash
python setup.py bdist_wheel
```

🖼️ **Print do terminal executando o comando python setup.py bdist_wheel**
➡️ Execute o comando no terminal integrado do VS Code e observe a saída.

Este comando criará um diretório `dist` contendo o arquivo wheel, com um nome similar a:

```
dist/meu_pacote_spark-0.1.0-py3-none-any.whl
```

💡 **Dica**: O nome do arquivo wheel segue o formato `{nome_pacote}-{versão}-{python_tag}-{abi_tag}-{plataforma_tag}.whl`.

## Subindo o Wheel para o S3

Agora que temos o arquivo wheel, vamos fazer upload para o S3 para que possamos usá-lo no EMR:

### Passo 1: Verificar o Arquivo Wheel Gerado

```bash
ls -la dist/
```

### Passo 2: Fazer Upload para o S3 usando AWS CLI

```bash
aws s3 cp dist/meu_pacote_spark-0.1.0-py3-none-any.whl s3://meu-bucket/wheels/
```

🖼️ **Print do terminal executando o comando aws s3 cp**
➡️ Execute o comando aws s3 cp e observe a saída confirmando o upload.

💡 **Dica**: Certifique-se de que o usuário que está executando o comando tem permissões para escrever no bucket S3.

### Passo 3: Verificar se o Upload foi Bem-sucedido

```bash
aws s3 ls s3://meu-bucket/wheels/
```

## Boas Práticas para Organização de Código 🔥

1. **Estrutura Modular**: Divida seu código em módulos com responsabilidades bem definidas.

2. **Documentação**: Adicione docstrings a todas as funções e classes para facilitar o entendimento.

3. **Tipagem**: Use type hints para melhorar a legibilidade e permitir verificação estática de tipos.

4. **Testes**: Inclua testes unitários para validar a funcionalidade do código.

5. **Logging**: Implemente logging adequado para facilitar a depuração.

6. **Tratamento de Erros**: Use blocos try/except para lidar com exceções de forma elegante.

7. **Configuração Centralizada**: Mantenha configurações em um local centralizado para fácil manutenção.

8. **Versionamento**: Use controle de versão (Git) e siga o versionamento semântico para seus pacotes.

## Dicas para Desenvolvimento no VS Code 💡

1. **Extensões Úteis**:
   - Python (Microsoft)
   - Pylance (Microsoft)
   - Python Docstring Generator
   - GitLens
   - Remote - SSH (para desenvolvimento remoto)

2. **Configuração do Linter**:
   Crie um arquivo `.vscode/settings.json` com:
   ```json
   {
     "python.linting.enabled": true,
     "python.linting.pylintEnabled": true,
     "python.linting.flake8Enabled": true,
     "python.formatting.provider": "black"
   }
   ```

3. **Depuração**:
   Configure o arquivo `.vscode/launch.json` para depurar seu código:
   ```json
   {
     "version": "0.2.0",
     "configurations": [
       {
         "name": "Python: Current File",
         "type": "python",
         "request": "launch",
         "program": "${file}",
         "console": "integratedTerminal"
       }
     ]
   }
   ```

4. **Terminal Integrado**: Use o terminal integrado do VS Code para executar comandos sem sair do editor.

## Solução de Problemas Comuns

### Erro "No module named 'setuptools'"

Solução:
```bash
pip install setuptools
```

### Erro ao Criar o Wheel

Verifique:
- Se o arquivo setup.py está correto
- Se todos os arquivos referenciados existem
- Se as dependências estão instaladas

### Erro ao Fazer Upload para o S3

Verifique:
- Suas credenciais AWS
- Permissões do bucket
- Conexão com a internet

## Conclusão

Neste capítulo, você aprendeu como estruturar um projeto Python no VS Code, criar um arquivo wheel e fazer upload para o S3. Esta abordagem facilita a organização, manutenção e distribuição do seu código para execução no EMR. No próximo capítulo, aprenderemos como executar este wheel diretamente no cluster EMR usando o spark-submit.

---

🔍 **Próximo Capítulo**: [Rodando Spark com Wheel no S3](05_rodando_spark_com_wheel_no_s3.md)
