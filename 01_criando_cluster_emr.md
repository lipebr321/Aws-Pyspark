# Capítulo 1: Criando um Cluster EMR na AWS

## Introdução

O Amazon EMR (Elastic MapReduce) é um serviço gerenciado da AWS que facilita o processamento de grandes volumes de dados usando frameworks como Apache Hadoop, Apache Spark e outros. Neste capítulo, você aprenderá como criar um cluster EMR através do Console da AWS, com configurações otimizadas para executar aplicações Spark.

## Pré-requisitos

Antes de começar, você precisará:

- Uma conta AWS ativa
- Permissões adequadas para criar recursos EMR
- Conhecimento básico de AWS (conceitos de EC2, S3, IAM)

## Glossário para Iniciantes 📝

- **EMR**: Amazon Elastic MapReduce, serviço gerenciado para processamento de big data
- **Cluster**: Conjunto de servidores (nós) que trabalham juntos
- **Nó Master**: Servidor que coordena o trabalho no cluster
- **Nós Core**: Servidores que processam dados e armazenam no HDFS
- **Nós Task**: Servidores adicionais que apenas processam dados (sem armazenamento)
- **S3**: Amazon Simple Storage Service, armazenamento de objetos na nuvem
- **EC2**: Amazon Elastic Compute Cloud, serviço de computação virtual
- **IAM**: Identity and Access Management, gerenciamento de permissões na AWS

## Criando um Cluster EMR via Console AWS

### Passo 1: Acessar o Console EMR

1. Faça login no [Console AWS](https://console.aws.amazon.com/)
2. Navegue até o serviço EMR

![image](https://github.com/user-attachments/assets/f615c837-8f0c-4acc-a92c-56a704e45818)

➡️ Na barra de pesquisa superior, digite "EMR" e selecione o serviço "EMR" nos resultados.

### Passo 2: Iniciar a Criação do Cluster

1. Na página inicial do EMR, clique no botão "Criar cluster"

![image](https://github.com/user-attachments/assets/42e8797b-91c2-496f-a876-dcfb0ab94d7c)

➡️ Localize o botão amarelo "Criar cluster" no canto superior direito da tela e clique nele.

![image](https://github.com/user-attachments/assets/0db822a5-c1b0-46e6-8c10-9d8e17e40798)


### Passo 3: Configurações Básicas do Cluster

1. Na seção "Nome e aplicações", configure:

```
Nome do cluster: MeuClusterSpark
```

2. Em "Aplicações", selecione a opção "Spark"

![image](https://github.com/user-attachments/assets/c295fb3f-9a4f-457a-ad64-bf3555734f14)

➡️ Selecione a versão emr-6.15.0, role até a seção "Applications" e marque Spark.

💡 **Dica**: Escolha a versão mais recente do EMR disponível para ter acesso às últimas funcionalidades e correções de bugs do Spark.

### Passo 4: Configurações de Hardware

1. Na seção "Configurações de hardware", configure:

```
Tipo de instância do nó primário: m5.xlarge
Tipo de instância dos nós core: m5.xlarge
Contagem de nós core: 2
```

![image](https://github.com/user-attachments/assets/92dfd151-281f-44d5-8f66-2f865258e877)

➡️ Selecione os tipos de instância na lista suspensa e defina a quantidade de nós.

💡 **Dica**: Para ambientes de teste, m5.xlarge oferece um bom equilíbrio entre custo e desempenho. Para cargas de trabalho de produção, considere instâncias com mais memória (série r5) ou otimizadas para computação (série c5).

2. Para ambientes de teste, você pode desativar o auto-scaling:

![image](https://github.com/user-attachments/assets/786662ed-e8ae-46e6-b3f7-65625da6d948)

➡️ Desmarque a opção "Ativar auto-scaling" se estiver apenas testando.

### Passo 5: Configurações Gerais do Cluster

1. Na seção "Configurações gerais do cluster", configure:

```
Registro em log: Ativado
Bucket S3: s3://meu-bucket-logs-emr/logs/
```

![image](https://github.com/user-attachments/assets/2f82518c-bd01-447f-aabf-b1e8b3a54402)

➡️ Ative o registro em log e especifique um bucket S3 para armazenar os logs.

💡 **Dica**: Sempre mantenha os logs ativados para facilitar a depuração de problemas.

![image](https://github.com/user-attachments/assets/1aa281f2-c02c-438c-9aef-fae19d53e215)


### Passo 6: Configurações de Segurança

1. Na seção "Segurança", configure:

```
Par de chaves EC2: MeuParDeChaves
```

![image](https://github.com/user-attachments/assets/266c9142-f97c-4f55-8a62-b4e96560e848)

➡️ Selecione um par de chaves existente ou crie um novo.

💡 **Dica**: Se você não tiver um par de chaves, clique em "Criar par de chaves" e siga as instruções para criar e baixar um novo par.

2. Configure os grupos de segurança:

```
Usar grupos de segurança padrão
```

🛠️ **Configuração avançada de segurança**:

Para permitir acesso SSH ao cluster, você precisa configurar os grupos de segurança:

```bash
# Verificar o ID do grupo de segurança do nó master
aws ec2 describe-security-groups --filters Name=group-name,Values=ElasticMapReduce-master

# Adicionar regra para permitir SSH (porta 22)
aws ec2 authorize-security-group-ingress \
    --group-id sg-xxxxxxxxxxxxxxxxx \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0
```

⚠️ **Atenção**: Em ambientes de produção, restrinja o acesso SSH apenas aos IPs necessários, substituindo 0.0.0.0/0 pelo seu IP ou range de IPs corporativos.

### Passo 7: Configurações de Permissões

1. Na seção "Permissões", configure:

```
Perfil de serviço do EMR: EMR_DefaultRole
Perfil de instância do EC2: EMR_EC2_DefaultRole
```

![image](https://github.com/user-attachments/assets/f6f53a87-06d5-4bdf-a6a3-c9d0895a7c47)

➡️ Selecione os perfis padrão ou crie perfis personalizados se necessário.

💡 **Dica**: Para operações básicas, os perfis padrão são suficientes. Para acesso a recursos específicos da AWS, você precisará personalizar as políticas IAM.

### Passo 8: Revisão e Criação

1. Revise todas as configurações
2. Clique em "Criar cluster"


➡️ Verifique todas as configurações e clique no botão azul "Criar cluster" no final da página.

## Monitorando a Criação do Cluster

Após iniciar a criação, você será redirecionado para a página de detalhes do cluster:


➡️ Observe o status do cluster, que inicialmente será "Iniciando".

O processo de criação do cluster pode levar de 5 a 20 minutos, dependendo das configurações escolhidas.

💡 **Dica**: Você pode acompanhar o progresso na seção "Status" da página de detalhes do cluster.

## Habilitando Acesso SSH ao Cluster

Para acessar o cluster via SSH, você precisa garantir que:

1. O grupo de segurança do nó master permita tráfego na porta 22
2. Você tenha o arquivo .pem do par de chaves usado na criação do cluster

### Verificando o DNS Público do Nó Master

Na página de detalhes do cluster, localize a seção "Resumo":


➡️ Anote o "DNS público primário" listado nesta seção.

### Configurando Permissões do Arquivo de Chave

Antes de usar o arquivo .pem para SSH, ajuste suas permissões:

```bash
# Ajustar permissões do arquivo de chave
chmod 400 MeuParDeChaves.pem
```

## Configurações Recomendadas para Diferentes Cenários

### Para Ambiente de Desenvolvimento/Teste

```
Versão EMR: emr-6.15.0
Aplicações: Spark
Tipo de instância do nó primário: m5.xlarge
Tipo de instância dos nós core: m5.xlarge
Contagem de nós core: 2
Auto-scaling: Desativado
```

### Para Ambiente de Produção com Cargas Médias

```
Versão EMR: emr-6.15.0
Aplicações: Spark
Tipo de instância do nó primário: r5.2xlarge
Tipo de instância dos nós core: r5.2xlarge
Contagem de nós core: 4
Auto-scaling: Ativado (mínimo 4, máximo 10)
```

### Para Processamento de Big Data Intensivo

```
Versão EMR: emr-6.15.0
Aplicações: Spark
Tipo de instância do nó primário: r5.4xlarge
Tipo de instância dos nós core: r5.4xlarge
Contagem de nós core: 8
Tipo de instância dos nós task: c5.4xlarge
Contagem de nós task: 4
Auto-scaling: Ativado (mínimo 8, máximo 20)
```

## Boas Práticas para Clusters EMR 🔥

1. **Armazenamento**: Prefira armazenar dados no S3 em vez do HDFS para maior durabilidade e separação entre armazenamento e computação.

2. **Custo**: Utilize instâncias spot para nós task para reduzir custos (até 90% mais baratas que instâncias sob demanda).

3. **Segurança**: Restrinja o acesso SSH apenas aos IPs necessários e use VPC privadas quando possível.

4. **Monitoramento**: Ative o CloudWatch para monitorar métricas do cluster e configurar alarmes.

5. **Logs**: Sempre configure um bucket S3 para armazenar logs, facilitando a depuração.

6. **Ciclo de Vida**: Para clusters temporários, configure ações de término automático após a conclusão das tarefas.

## Solução de Problemas Comuns

### Cluster Falha ao Iniciar

Verifique:
- Limites de serviço da sua conta AWS
- Permissões IAM dos perfis utilizados
- Disponibilidade da zona/região escolhida

### Erro de Acesso SSH

Verifique:
- Configurações do grupo de segurança
- Permissões do arquivo .pem
- Se o cluster está totalmente provisionado (status "Aguardando")

## Conclusão

Neste capítulo, você aprendeu como criar um cluster EMR na AWS via Console, configurar os tipos de instâncias, segurança e permissões. No próximo capítulo, aprenderemos como conectar ao cluster via SSH e executar comandos básicos.

---

🔍 **Próximo Capítulo**: [Conectando ao Cluster](02_conectando_ao_cluster.md)
