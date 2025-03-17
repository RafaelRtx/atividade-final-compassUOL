# Projeto Final - Compass Uol

## Contexto
A **Fast Engineering S/A** enfrenta desafios de escalabilidade e disponibilidade para seu e-commerce. Atualmente, todo o sistema roda em um ambiente **on-premises**, mas o crescimento da demanda exige uma arquitetura mais flexível e resiliente.  

### Diante desse cenário, migraremos a aplicação para a AWS seguindo um processo em duas fases:
_ _ _ _ _
###  Fase 1 
 >**Lift-and-Shift (As-Is):**
 >>Migração rápida da infraestrutura atual para a AWS, garantindo continuidade operacional sem grandes mudanças.
### Fase 2
>**Modernização:**
>> Após a migração, otimizar a infraestrutura para Kubernetes e implementar melhores práticas em Cloud AWS.

## Diagrama da Arquitetura Atual (On-Premises):
![Imagem diagrama atual da infraestrutura da empresa](img/img-dg-atual.png)

- **database**: Servidor MySQL (500GB, 10GB RAM, 3 vCPUs)  
- **frontend**: Frontend em React (5GB, 2GB RAM, 1 vCPU)  
- **backend**: Backend com 3 APIs + Nginx (5GB, 4GB RAM, 2 vCPUs)  
- **Nginx** também atua como balanceador de carga e armazenamento de estáticos (imagens, arquivos, etc.).

# 1° Fase: Migração “As-Is” (ou Lift and Shift):
  O Objetivo dessa etapa é garantir que a aplicação seja replicada em outro servidor na nuvem exatamente do jeito que está.
  Por isso, utilizaremos os seguintes serviços para migração:

  **AWS MGN (AWS Application Migration Service)**.
   > Para replicação de servidores 

  **AWS DMS (Database Migration Service)**.
   > Para migração de banco de dados 

# 2. Escopo Detalhado da Migração As-Is

## 2.1. Atividades Necessárias
Para realizar a migração Lift-and-Shift, seguiremos estas etapas:

### Mapeamento da Infraestrutura Atual:
- **Servidor Frontend** (React)
- **Servidor Backend** (3 APIs, Ngnix)
- **Servidor de Banco de Dados** (MySQL)

### Configuração da AWS:
- Criar ambiente na AWS para replicação e testes.

- Configurar **AWS MGN** (Application Migration Service) para replicação contínua dos servidores, de maneira econômica.

 - Configurar **AWS DMS** para migração do banco de dados, minimizando o tempo de inatividade e garantindo a integridade dos dados durante a migração.

- Ajustar permissões e regras de segurança.

### Replicação de Dados e Testes:
- Instalar o **AWS MGN Agent** nos servidores locais para replicação.
- Utilizar o **AWS DMS** para migrar o banco de dados do ambiente local para a AWS, com a replicação contínua dos dados.
- Criar máquinas **EC2** espelhando a infraestrutura atual.
- Testar a aplicação na AWS antes da migração final.

### Cutover (Troca para Produção):
- Redirecionar tráfego para a infraestrutura na AWS.
- Desativar os servidores antigos após validação.

## Diagrama da migração (On-Premises):

![Imagem diagrama de migração da infraestrutura da empresa](img/img-dg-onpremise.jpeg)

---

# Fase 2: Modernização com EKS (Elastic Kubernetes Service)

## Serviços à serem utilizados: 

- **Amazon EKS** (Elastic Kubernetes Service): Orquestrador de contêineres, facilitando implantação de múltiplos microserviços.  
- **Amazon RDS** (MySQL) em Multi-AZ: Banco de dados gerenciado com backup e alta disponibilidade.  
- **Amazon S3**: Armazenamento de objetos estáticos (imagens, PDFs etc.) e/ou backups.  
- **Security**: AWS WAF, Security Groups, IAM Roles, Secrets Manager e KMS para criptografia.

---

### 1. Montar o Ambiente com os Containeres da aplicação

1. **Criando um Cluster EKS**
   - Usar **eksctl**, **Terraform**, ou **CloudFormation** para criar o cluster.  
   - Definir subnets privadas (worker nodes) e subnets públicas (para o Load Balancer).  
   - Habilitar **Cluster Autoscaler** para redimensionar nós conforme a demanda.

2. **Configure a VPC e Subnets**
   - As **subnets privadas** recebem os pods do Kubernetes (sem IP público).  
   - As **subnets públicas** hospedam o Application Load Balancer (ALB) ou NAT Gateways (se for preciso).  
   - **Internet Gateway (IGW)** conectado às subnets públicas para tráfego externo.

3. **Crie e Configure os Worker Nodes**
   - Crie um **Auto Scaling Group** para os nós do EKS (EC2).  
   - Defina o tipo de instância (ex.: `m5.large`), capacidade mínima e máxima (ex.: 2 a 6 nós).  
   - Vincule a **IAM Role** que permita operações do Kubernetes (por exemplo, criar logs em CloudWatch).

___

## 2. Transformação da Aplicação em Containers
- **Dockerizar a aplicação** para que ela seja executada em pods no **Amazon EKS**.

## 3. Configuração de Auto Scaling e Load Balancer
- Definir políticas de **Auto Scaling** para as instâncias EC2 dos nós do Kubernetes (EKS) com base na carga.
- Configuração do **Load Balancer** para distribuir o tráfego entre os pods de forma eficiente.

## 4. Ajustes nas Subnets
- Ajustar a configuração de subnets públicas para os nós EKS e subnets privadas para o banco de dados **RDS MySQL**.

## 5. Configuração de Rede e Segurança
- Configuração de **Security Groups** e **NACLs** para garantir uma comunicação segura entre os componentes da infraestrutura.
- Implementação de **AWS WAF** para proteger a aplicação contra ataques.

## 6. Monitoramento e Logs
- Implementação do **AWS CloudWatch** para monitoramento de logs, métricas e eventos da infraestrutura.

## 7. Configuração de Backup e Recuperação
- Implementação de backups regulares para o banco de dados **RDS MySQL** e arquivos estáticos em **S3**.

# Diagrama da Infraestrutura na AWS

Diagrama da infraestrutura modernizada na AWS com mais detalhes:

![Imagem diagrama modernizado da infraestrutura da empresa](img/img-dg-modernizado.png)

Com essa arquitetura, poderemos ter um sistema robusto e escalável com menor risco de falhas além de uma maior manutenibilidade.
