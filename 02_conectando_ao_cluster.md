# Capítulo 2: Conectando ao Cluster EMR

## Introdução

Após criar seu cluster EMR na AWS, o próximo passo é estabelecer uma conexão com ele para executar comandos, monitorar recursos e submeter trabalhos. Neste capítulo, você aprenderá como acessar o cluster via SSH a partir de diferentes sistemas operacionais e executar comandos básicos para verificar o estado do cluster.

## Pré-requisitos

Antes de começar, você precisará:

- Um cluster EMR em execução (status "Aguardando")
- O arquivo de chave privada (.pem) usado na criação do cluster
- O DNS público do nó master do cluster
- Um terminal (Linux/macOS) ou cliente SSH (Windows)

## Glossário para Iniciantes 📝

- **SSH**: Secure Shell, protocolo para acesso remoto seguro a servidores
- **Terminal**: Interface de linha de comando para executar comandos
- **Nó Master**: Servidor principal que coordena o trabalho no cluster EMR
- **YARN**: Yet Another Resource Negotiator, gerenciador de recursos do Hadoop
- **Spark-shell**: Interface interativa para testar comandos Spark
- **HDFS**: Hadoop Distributed File System, sistema de arquivos distribuído

## Conectando ao Cluster via SSH

### Para usuários Linux e macOS

1. Abra o terminal
2. Use o comando SSH com o arquivo de chave privada:

```bash
ssh -i /caminho/para/MeuParDeChaves.pem hadoop@ec2-xx-xx-xx-xx.compute-1.amazonaws.com
```


➡️ Digite o comando SSH substituindo o caminho da chave e o endereço DNS do nó master.

💡 **Dica**: Substitua `/caminho/para/MeuParDeChaves.pem` pelo caminho completo para o seu arquivo .pem e `ec2-xx-xx-xx-xx.compute-1.amazonaws.com` pelo DNS público do seu nó master.

3. Na primeira conexão, você receberá um aviso sobre a autenticidade do host. Digite "yes" para continuar.


➡️ Digite "yes" quando solicitado para adicionar o host à lista de hosts conhecidos.

4. Se a conexão for bem-sucedida, você verá o prompt do EMR:

```
       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-2/
```

### Para usuários Windows

#### Opção 1: Usando PuTTY

1. Converta o arquivo .pem para o formato .ppk usando PuTTYgen:
   - Abra PuTTYgen
   - Clique em "Load" e selecione seu arquivo .pem
   - Clique em "Save private key" para gerar o arquivo .ppk

➡️ Clique em "Load" para carregar o arquivo .pem e depois em "Save private key" para salvar como .ppk.

2. Configure a conexão no PuTTY:
   - No campo "Host Name", digite: `hadoop@ec2-xx-xx-xx-xx.compute-1.amazonaws.com`
   - No menu lateral, navegue até Connection > SSH > Auth
   - Em "Private key file for authentication", clique em "Browse" e selecione o arquivo .ppk

➡️ Configure o host e navegue até a seção de autenticação para selecionar o arquivo .ppk.

3. Clique em "Open" para iniciar a conexão
4. Clique em "Yes" no alerta de segurança

#### Opção 2: Usando Windows Subsystem for Linux (WSL)

Se você tiver o WSL instalado, pode seguir as mesmas instruções para Linux:

```bash
ssh -i /caminho/para/MeuParDeChaves.pem hadoop@ec2-xx-xx-xx-xx.compute-1.amazonaws.com
```

#### Opção 3: Usando Windows Terminal com OpenSSH

Windows 10 e 11 incluem o cliente OpenSSH:

```powershell
ssh -i C:\caminho\para\MeuParDeChaves.pem hadoop@ec2-xx-xx-xx-xx.compute-1.amazonaws.com
```

💡 **Dica**: No Windows, você pode precisar ajustar as permissões do arquivo .pem. No PowerShell, use:

```powershell
icacls.exe "C:\caminho\para\MeuParDeChaves.pem" /inheritance:r /grant:r "$($env:USERNAME):(R)"
```

## Comandos Úteis para Navegação e Verificação

Após conectar-se ao nó master, você pode executar diversos comandos para explorar o cluster:

### Verificar a versão do Hadoop

```bash
hadoop version
```

Exemplo de saída:
```
Hadoop 3.3.3
Source code repository https://github.com/apache/hadoop.git -r 6d7be4fb8b4c0c1d499c9f8e3c1503b58a6922d2
Compiled by ubuntu on 2022-08-02T07:01Z
Compiled with protoc 3.14.0
From source with checksum fb9dd8918a7b8a5b430d61af249f359
```

### Verificar a versão do Spark

```bash
spark-submit --version
```

Exemplo de saída:
```
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 3.3.1
      /_/
                        
Using Scala version 2.12.15, Java HotSpot(TM) 64-Bit Server VM, 1.8.0_332
```

### Listar os nós do cluster

```bash
yarn node -list
```

Exemplo de saída:
```
Total Nodes:3
         Node-Id             Node-State Node-Http-Address       Number-of-Running-Containers
ip-172-31-28-254.ec2.internal           RUNNING ip-172-31-28-254.ec2.internal:8042                                  0
ip-172-31-30-169.ec2.internal           RUNNING ip-172-31-30-169.ec2.internal:8042                                  0
ip-172-31-25-107.ec2.internal           RUNNING ip-172-31-25-107.ec2.internal:8042                                  0
```

### Verificar o status dos aplicativos YARN

```bash
yarn application -list
```

Exemplo de saída (quando não há aplicativos em execução):
```
Total number of applications (application-types: [] and states: [SUBMITTED, ACCEPTED, RUNNING]):0
```

### Verificar o espaço em disco

```bash
df -h
```

Exemplo de saída:
```
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        7.6G     0  7.6G   0% /dev
tmpfs           7.6G     0  7.6G   0% /dev/shm
tmpfs           7.6G  464K  7.6G   1% /run
tmpfs           7.6G     0  7.6G   0% /sys/fs/cgroup
/dev/xvda1       50G  8.4G   42G  17% /
/dev/xvdb      1014G   77M  964G   1% /mnt/s3
tmpfs           1.6G     0  1.6G   0% /run/user/1001
```

### Verificar a memória disponível

```bash
free -h
```

Exemplo de saída:
```
              total        used        free      shared  buff/cache   available
Mem:            15G        1.2G         12G        464K        1.8G         13G
Swap:            0B          0B          0B
```

### Verificar os processos em execução

```bash
ps aux | grep spark
```

### Navegar pelo sistema de arquivos HDFS

```bash
# Listar diretórios no HDFS
hdfs dfs -ls /

# Criar um diretório no HDFS
hdfs dfs -mkdir /user/hadoop/teste

# Verificar espaço disponível no HDFS
hdfs dfs -df -h
```

## Testando o Spark Localmente via Spark-Shell

O Spark-Shell é uma interface interativa que permite testar comandos Spark em tempo real. É uma excelente ferramenta para aprendizado e depuração.

### Iniciando o Spark-Shell

```bash
spark-shell
```


➡️ Execute o comando spark-shell e aguarde o carregamento da interface interativa.

Você verá uma interface Scala com o prompt `scala>`. Isso indica que o Spark-Shell está pronto para receber comandos.

### Exemplos Básicos no Spark-Shell

#### Criando um RDD (Resilient Distributed Dataset)

```scala
val data = Array(1, 2, 3, 4, 5)
val distData = sc.parallelize(data)
```

#### Contando elementos

```scala
distData.count()
```

Saída esperada:
```
res0: Long = 5
```

#### Somando elementos

```scala
distData.reduce((a, b) => a + b)
```

Saída esperada:
```
res1: Int = 15
```

#### Criando um DataFrame

```scala
val df = spark.createDataFrame(Seq(
  (1, "João", 25),
  (2, "Maria", 30),
  (3, "Pedro", 35)
)).toDF("id", "nome", "idade")

df.show()
```

Saída esperada:
```
+---+-----+-----+
| id| nome|idade|
+---+-----+-----+
|  1| João|   25|
|  2|Maria|   30|
|  3|Pedro|   35|
+---+-----+-----+
```

#### Filtrando dados

```scala
df.filter($"idade" > 30).show()
```

Saída esperada:
```
+---+-----+-----+
| id| nome|idade|
+---+-----+-----+
|  3|Pedro|   35|
+---+-----+-----+
```

#### Lendo um arquivo CSV (se disponível)

```scala
// Substitua pelo caminho real do seu arquivo
val csvDF = spark.read.option("header", "true").csv("s3://meu-bucket/dados/exemplo.csv")
csvDF.show(5)
```

### Saindo do Spark-Shell

Para sair do Spark-Shell, pressione `Ctrl+D` ou digite `:quit`.

## Usando PySpark Shell

Se você preferir Python ao invés de Scala, pode usar o PySpark Shell:

```bash
pyspark
```


➡️ Execute o comando pyspark e aguarde o carregamento da interface interativa Python.

### Exemplos Básicos no PySpark

#### Criando um RDD

```python
data = [1, 2, 3, 4, 5]
distData = sc.parallelize(data)
```

#### Contando elementos

```python
distData.count()
```

Saída esperada:
```
5
```

#### Somando elementos

```python
distData.reduce(lambda a, b: a + b)
```

Saída esperada:
```
15
```

#### Criando um DataFrame

```python
from pyspark.sql import Row
df = spark.createDataFrame([
    Row(id=1, nome="João", idade=25),
    Row(id=2, nome="Maria", idade=30),
    Row(id=3, nome="Pedro", idade=35)
])

df.show()
```

Saída esperada:
```
+---+-----+-----+
| id| nome|idade|
+---+-----+-----+
|  1| João|   25|
|  2|Maria|   30|
|  3|Pedro|   35|
+---+-----+-----+
```

## Transferindo Arquivos para o Cluster

### Do seu computador local para o cluster

Usando SCP (Secure Copy Protocol):

```bash
# Para Linux/macOS/Windows com OpenSSH
scp -i /caminho/para/MeuParDeChaves.pem arquivo.py hadoop@ec2-xx-xx-xx-xx.compute-1.amazonaws.com:~/

# Para Windows com PuTTY, use PSCP
pscp -i C:\caminho\para\MeuParDeChaves.ppk C:\caminho\para\arquivo.py hadoop@ec2-xx-xx-xx-xx.compute-1.amazonaws.com:~/
```

### Do cluster para o S3

```bash
# Copiar um arquivo local para o S3
aws s3 cp ~/arquivo.py s3://meu-bucket/scripts/

# Copiar um diretório inteiro para o S3
aws s3 cp ~/meu-diretorio/ s3://meu-bucket/scripts/ --recursive
```

## Boas Práticas para Conexão SSH 🔥

1. **Segurança**: Nunca compartilhe seu arquivo de chave privada (.pem).

2. **Timeout**: Para evitar desconexões por inatividade, configure o cliente SSH para enviar pacotes keep-alive:
   ```bash
   # Adicione ao seu arquivo ~/.ssh/config
   Host *.amazonaws.com
     ServerAliveInterval 60
   ```

3. **Túnel SSH**: Para acessar interfaces web do cluster (como o Spark History Server), configure um túnel SSH:
   ```bash
   ssh -i /caminho/para/MeuParDeChaves.pem -N -L 18080:localhost:18080 hadoop@ec2-xx-xx-xx-xx.compute-1.amazonaws.com
   ```
   Depois acesse `http://localhost:18080` no seu navegador.

4. **Aliases**: Crie aliases para comandos frequentes:
   ```bash
   # Adicione ao seu arquivo ~/.bashrc no nó master
   alias hdfs-ls='hdfs dfs -ls'
   alias yarn-apps='yarn application -list'
   ```

5. **Scripts de Inicialização**: Crie scripts para configurar seu ambiente automaticamente ao conectar:
   ```bash
   # Crie um arquivo ~/setup.sh no nó master
   #!/bin/bash
   echo "Configurando ambiente..."
   export SPARK_HOME=/usr/lib/spark
   export PATH=$PATH:$SPARK_HOME/bin
   ```

## Solução de Problemas Comuns

### Erro "Permission denied (publickey)"

Verifique:
- Se está usando o arquivo de chave correto
- Se as permissões do arquivo .pem estão corretas (chmod 400)
- Se está usando o nome de usuário correto (hadoop)

### Conexão SSH Lenta

Possíveis soluções:
- Desative a verificação DNS no cliente SSH
- Verifique a latência de rede para a região AWS
- Considere usar uma instância bastion na mesma região

### Erro "Host key verification failed"

Se você recriou o cluster com o mesmo DNS:
```bash
ssh-keygen -R ec2-xx-xx-xx-xx.compute-1.amazonaws.com
```

## Conclusão

Neste capítulo, você aprendeu como conectar ao seu cluster EMR via SSH a partir de diferentes sistemas operacionais, executar comandos básicos para verificar o estado do cluster e testar o Spark localmente usando o Spark-Shell. No próximo capítulo, aprenderemos como submeter código Spark para execução no cluster.

---

🔍 **Próximo Capítulo**: [Submetendo Código com Spark-Submit](03_submetendo_codigo_spark_submit.md)
