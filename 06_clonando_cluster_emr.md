# Capítulo 6: Clonando Cluster EMR

## Introdução

Quando trabalhamos com clusters EMR na AWS, frequentemente precisamos criar múltiplos clusters com configurações semelhantes para diferentes ambientes (desenvolvimento, teste, produção) ou para diferentes projetos. A clonagem de clusters é uma funcionalidade útil que permite replicar configurações existentes, economizando tempo e reduzindo erros de configuração manual. Neste capítulo, você aprenderá como clonar um cluster EMR existente e entenderá as melhores práticas para persistência de dados no S3.

## Pré-requisitos

Antes de começar, você precisará:

- Uma conta AWS ativa
- Um cluster EMR existente para clonar
- Permissões adequadas para criar recursos EMR
- Conhecimentos básicos de AWS Console

## Glossário para Iniciantes 📝

- **Clonagem de Cluster**: Processo de criar um novo cluster com as mesmas configurações de um existente
- **Persistência de Dados**: Armazenamento de dados de forma durável, independente do ciclo de vida do cluster
- **S3**: Amazon Simple Storage Service, serviço de armazenamento de objetos na AWS
- **HDFS**: Hadoop Distributed File System, sistema de arquivos distribuído do Hadoop
- **Configurações de Bootstrap**: Scripts executados durante a inicialização do cluster
- **Grupos de Segurança**: Controles de firewall virtual para recursos AWS

## Clonando uma Configuração de Cluster Existente

### Passo 1: Acessar o Console EMR

1. Faça login no [Console AWS](https://console.aws.amazon.com/)
2. Navegue até o serviço EMR

🖼️ **Print da tela inicial do AWS Console**
➡️ Na barra de pesquisa superior, digite "EMR" e selecione o serviço "EMR" nos resultados.

### Passo 2: Localizar o Cluster a Ser Clonado

1. Na página inicial do EMR, você verá a lista de clusters
2. Identifique o cluster que deseja clonar

🖼️ **Print da lista de clusters no console EMR**
➡️ Localize o cluster que deseja clonar na lista de clusters.

### Passo 3: Iniciar o Processo de Clonagem

1. Selecione o cluster que deseja clonar clicando na caixa de seleção ao lado do nome
2. No menu "Ações", selecione "Clone"

🖼️ **Print do menu Ações com a opção Clone destacada**
➡️ Clique no menu "Ações" e selecione a opção "Clone" no menu suspenso.

Alternativamente, você pode usar o recurso "Re-executar cluster com as mesmas configurações":

1. Selecione o cluster que deseja clonar
2. No menu "Ações", selecione "Re-executar cluster com as mesmas configurações"

💡 **Dica**: A opção "Clone" permite que você revise e modifique as configurações antes de criar o novo cluster, enquanto "Re-executar" usa exatamente as mesmas configurações.

### Passo 4: Revisar e Ajustar Configurações

Após selecionar "Clone", você será direcionado para a página de criação de cluster com as configurações pré-preenchidas:

1. Revise o nome do cluster e modifique se necessário
2. Verifique a versão do EMR
3. Revise as aplicações selecionadas
4. Verifique as configurações de hardware (tipos de instância, contagem de nós)
5. Revise as configurações de segurança e rede

🖼️ **Print da tela de revisão de configurações**
➡️ Revise todas as configurações pré-preenchidas e faça os ajustes necessários.

### Passo 5: Ajustar Configurações Específicas

Algumas configurações que você pode querer ajustar:

1. **Nome do Cluster**: Dê um nome descritivo para o novo cluster
   ```
   Nome: MeuClusterSpark-Prod
   ```

2. **Configurações de Hardware**: Ajuste conforme necessário para o novo ambiente
   ```
   Tipo de instância do nó primário: r5.2xlarge
   Tipo de instância dos nós core: r5.2xlarge
   Contagem de nós core: 4
   ```

3. **Configurações de Segurança**: Verifique se os grupos de segurança e pares de chaves são apropriados
   ```
   Par de chaves EC2: MeuParDeChaves-Prod
   ```

4. **Configurações de Armazenamento**: Ajuste os caminhos do S3 para o novo cluster
   ```
   Bucket S3 para logs: s3://meu-bucket-logs-emr-prod/logs/
   ```

### Passo 6: Criar o Cluster Clonado

1. Após revisar e ajustar todas as configurações, clique em "Criar cluster"

🖼️ **Print do botão "Criar cluster" na parte inferior da página**
➡️ Clique no botão azul "Criar cluster" após revisar todas as configurações.

2. Você será redirecionado para a página de detalhes do novo cluster, onde poderá monitorar o progresso da criação

## Usando o Recurso "Re-executar cluster com as mesmas configurações"

O recurso "Re-executar" é uma forma mais rápida de clonar um cluster quando você não precisa fazer ajustes nas configurações:

### Passo 1: Selecionar o Cluster Original

1. Na lista de clusters, selecione o cluster que deseja clonar
2. No menu "Ações", selecione "Re-executar cluster com as mesmas configurações"

🖼️ **Print do menu Ações com a opção "Re-executar cluster" destacada**
➡️ Clique no menu "Ações" e selecione a opção "Re-executar cluster com as mesmas configurações".

### Passo 2: Confirmar a Re-execução

1. Uma janela de confirmação será exibida
2. Clique em "Re-executar" para confirmar

🖼️ **Print da janela de confirmação de re-execução**
➡️ Clique no botão "Re-executar" na janela de confirmação.

3. Um novo cluster será criado com as mesmas configurações do original

💡 **Dica**: Este método é ideal para recrear rapidamente um cluster que foi encerrado, mantendo exatamente as mesmas configurações.

## Explicação sobre Persistência em S3

Um aspecto fundamental ao trabalhar com clusters EMR é entender como os dados são armazenados e persistidos. Existem duas principais opções de armazenamento:

### HDFS vs S3

#### HDFS (Hadoop Distributed File System)

- **Temporário**: Os dados no HDFS existem apenas enquanto o cluster está em execução
- **Local**: Os dados são armazenados nos discos locais dos nós do cluster
- **Rápido**: Oferece alta performance para leitura/escrita dentro do cluster
- **Limitado**: Capacidade limitada ao tamanho do cluster

#### S3 (Simple Storage Service)

- **Persistente**: Os dados permanecem mesmo após o encerramento do cluster
- **Externo**: Os dados são armazenados fora do cluster, em um serviço gerenciado
- **Durável**: Oferece 99,999999999% (11 noves) de durabilidade
- **Escalável**: Capacidade praticamente ilimitada

🖼️ **Diagrama comparando HDFS e S3**
➡️ O HDFS armazena dados nos nós do cluster, enquanto o S3 armazena dados externamente, permitindo persistência além do ciclo de vida do cluster.

### Benefícios da Persistência em S3

1. **Separação de Computação e Armazenamento**: Você pode encerrar clusters quando não estão em uso, economizando custos, sem perder dados

2. **Compartilhamento de Dados**: Múltiplos clusters podem acessar os mesmos dados no S3

3. **Durabilidade**: Proteção contra falhas de hardware e perda de dados

4. **Custo-benefício**: Geralmente mais econômico para armazenamento de longo prazo

5. **Integração com outros Serviços AWS**: Fácil integração com serviços como Athena, Glue e Redshift

### Configurando Persistência em S3

Para garantir que seus dados sejam persistentes ao clonar clusters, siga estas práticas:

1. **Armazenar Dados de Entrada no S3**:
   ```
   s3://meu-bucket/dados/entrada/
   ```

2. **Configurar Saída para o S3**:
   ```
   s3://meu-bucket/dados/saida/
   ```

3. **Armazenar Scripts no S3**:
   ```
   s3://meu-bucket/scripts/
   ```

4. **Configurar Logs no S3**:
   ```
   s3://meu-bucket/logs/
   ```

💡 **Dica**: Organize seus buckets S3 com uma estrutura clara de diretórios para facilitar o gerenciamento:
```
s3://meu-bucket/
  ├── dados/
  │   ├── entrada/
  │   ├── saida/
  │   └── temp/
  ├── scripts/
  │   ├── python/
  │   └── shell/
  ├── logs/
  │   ├── cluster-1/
  │   └── cluster-2/
  └── wheels/
```

## Clonando Clusters via AWS CLI

Além da interface web, você também pode clonar clusters usando a AWS CLI, o que é útil para automação:

### Passo 1: Descrever o Cluster Existente

Primeiro, obtenha as configurações do cluster existente:

```bash
aws emr describe-cluster --cluster-id j-2AXXXXXXGAPLF > cluster-config.json
```

### Passo 2: Extrair Configurações Relevantes

Use jq ou outro processador JSON para extrair as configurações necessárias:

```bash
cat cluster-config.json | jq '.Cluster.Configurations'
```

### Passo 3: Criar um Arquivo de Configuração para o Novo Cluster

Crie um arquivo JSON com as configurações para o novo cluster:

```json
{
  "Name": "MeuClusterClonado",
  "ReleaseLabel": "emr-6.15.0",
  "Applications": [
    { "Name": "Spark" },
    { "Name": "Hadoop" }
  ],
  "Instances": {
    "InstanceGroups": [
      {
        "Name": "Master",
        "InstanceRole": "MASTER",
        "InstanceType": "m5.xlarge",
        "InstanceCount": 1
      },
      {
        "Name": "Core",
        "InstanceRole": "CORE",
        "InstanceType": "m5.xlarge",
        "InstanceCount": 2
      }
    ],
    "Ec2KeyName": "MeuParDeChaves",
    "KeepJobFlowAliveWhenNoSteps": true,
    "TerminationProtected": false
  },
  "LogUri": "s3://meu-bucket/logs/",
  "ServiceRole": "EMR_DefaultRole",
  "JobFlowRole": "EMR_EC2_DefaultRole"
}
```

### Passo 4: Criar o Novo Cluster

```bash
aws emr create-cluster --cli-input-json file://novo-cluster-config.json
```

## Boas Práticas para Clonagem de Clusters 🔥

1. **Nomenclatura Consistente**: Use um padrão de nomenclatura que indique o propósito e ambiente do cluster:
   ```
   [Projeto]-[Ambiente]-[Função]-[Data]
   Exemplo: DataLake-Prod-Processing-20230601
   ```

2. **Documentação**: Mantenha documentação sobre as configurações de cada cluster e suas finalidades.

3. **Automação**: Use AWS CloudFormation ou Terraform para automatizar a criação de clusters com configurações consistentes.

4. **Separação de Ambientes**: Mantenha clusters separados para desenvolvimento, teste e produção.

5. **Controle de Versão**: Mantenha as configurações de cluster em um sistema de controle de versão.

6. **Monitoramento**: Configure alertas e monitoramento para todos os clusters.

7. **Segurança**: Revise regularmente as configurações de segurança dos clusters.

8. **Otimização de Custos**: Configure clusters de desenvolvimento e teste para desligamento automático quando não estiverem em uso.

## Solução de Problemas Comuns

### Erro ao Clonar Cluster

Verifique:
- Se você tem permissões suficientes
- Se os recursos (tipos de instância) estão disponíveis na região
- Se os limites de serviço da sua conta permitem a criação de novos clusters

### Dados Perdidos Após Clonar

Verifique:
- Se os dados estavam armazenados no HDFS (temporário) ou no S3 (persistente)
- Se os caminhos do S3 estão configurados corretamente no novo cluster
- Se as permissões IAM permitem acesso aos buckets S3

### Cluster Clonado com Configurações Diferentes

Verifique:
- Se você usou "Clone" (que permite modificações) em vez de "Re-executar"
- Se houve alterações acidentais durante o processo de revisão
- Se há configurações específicas de região que podem ter sido alteradas

## Conclusão

Neste capítulo, você aprendeu como clonar um cluster EMR existente usando o console AWS e a AWS CLI, entendeu a importância da persistência de dados no S3 e conheceu as melhores práticas para gerenciar múltiplos clusters. A clonagem de clusters é uma funcionalidade poderosa que economiza tempo e reduz erros ao criar ambientes consistentes para diferentes propósitos. No próximo capítulo, aprenderemos como executar aplicações Spark usando arquivos JAR.

---

🔍 **Próximo Capítulo**: [Rodando com Arquivo JAR](07_rodando_com_arquivo_jar.md)
