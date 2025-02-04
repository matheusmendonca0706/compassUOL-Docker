# compassUOL-Docker

# Desafio 02 - Implementação de WordPress com DevSecOps na AWS

Este repositório apresenta um projeto prático que tem como objetivo fortalecer habilidades em DevSecOps, fazendo uso do Docker e de diversos serviços da AWS para implantar uma aplicação WordPress completamente funcional e escalável.

## Sumário
- [Descrição Geral](#descrição-geral)
- [Tecnologias Utilizadas](#tecnologias-utilizadas)
- [Pré-requisitos](#pré-requisitos)
- [Etapas do Projeto](#etapas-do-projeto)
- [Materiais de Apoio](#materiais-de-apoio)
- [Melhorias Futuras](#melhorias-futuras)
- [Contribuições](#contribuições)
- [Licença](#licença)

## Descrição Geral

O projeto envolve a construção de uma infraestrutura na AWS que permita a implantação de um site WordPress, utilizando boas práticas de DevSecOps. As principais atividades incluem:

- Configuração de uma VPC personalizada com sub-redes públicas e privadas.
- Implementação de um Gateway NAT para gerenciar o tráfego de saída das sub-redes privadas.
- Criação de Security Groups específicos para controlar o acesso entre recursos.
- Configuração do Amazon EFS para armazenamento compartilhado de arquivos estáticos.
- Implantação do banco de dados MySQL usando o Amazon RDS, garantindo alta disponibilidade e segurança.
- Provisionamento de instâncias EC2 com Docker instalado para hospedar o WordPress.
- Configuração de um Load Balancer para distribuir o tráfego de rede de forma eficiente.
- Implementação do Auto Scaling para ajustar automaticamente a capacidade conforme a demanda.

## Tecnologias Utilizadas

### AWS Services:
- Amazon VPC
- Amazon EC2
- Amazon RDS (MySQL)
- Amazon EFS
- Elastic Load Balancing
- Auto Scaling
- Docker
- Linux (Amazon Linux 2)

## Pré-requisitos

Conta na AWS com permissões para:

- Gerenciar VPCs, sub-redes e tabelas de rotas.
- Criar e configurar Security Groups.
- Provisionar instâncias EC2 e gerenciar pares de chaves SSH.
- Configurar bancos de dados com o Amazon RDS.
- Criar sistemas de arquivos com o Amazon EFS.
- Configurar Load Balancers e grupos de Auto Scaling.

## Etapas do Projeto

### 1. Configuração da VPC
- Crie uma VPC personalizada.
- Defina sub-redes públicas e privadas em diferentes zonas de disponibilidade.
- Configure as tabelas de rotas apropriadas para cada sub-rede.

### 2. Implementação do Gateway NAT
- Implemente um Gateway NAT na sub-rede pública.
- Atualize as tabelas de rotas das sub-redes privadas para direcionar o tráfego de saída para o Gateway NAT.

### 3. Configuração dos Security Groups
**Para o EC2:**
- Controle o acesso SSH (22) a partir do seu IP ou de um Bastion Host.
- Permita tráfego de saída para o RDS (3306) e para o EFS (2049).

**Para o RDS:**
- Permita conexões somente das instâncias EC2.

**Para o EFS:**
- Autorize conexões NFS (2049) das instâncias EC2.

**Bastion Host (Opcional):**
- Configure um Bastion Host em uma sub-rede pública para acesso seguro às instâncias EC2 em sub-redes privadas.

### 4. Criação do Amazon EFS
- Configure um sistema de arquivos no Amazon EFS.
- Anote o endpoint fornecido, pois será usado nas instâncias EC2 para montar o sistema de arquivos.

### 5. Configuração do Amazon RDS (MySQL)
- Crie um banco de dados MySQL usando o Amazon RDS.
- Defina as credenciais de acesso e o nome do banco de dados inicial.
- Certifique-se de que o banco de dados não seja acessível publicamente e que os Security Groups estejam corretamente configurados.

### 6. Provisionamento das Instâncias EC2
**Criação de um Launch Template:**
- Utilize o Amazon Linux 2 como AMI.
- Selecione o tipo de instância adequado (e.g., t2.micro).
- Inclua um script de inicialização (`user_data.sh`) que:
  - Instala o Docker.
  - Configura o Docker para iniciar na inicialização.
  - Implementa o WordPress usando um Dockerfile ou Docker Compose.
  - Monta o EFS no diretório apropriado.
  - Configura variáveis de ambiente para conectar-se ao RDS.

### 7. Configuração do Load Balancer
- Crie um Classic Load Balancer.
- Configure as portas de escuta (e.g., HTTP na porta 80).
- Selecione as sub-redes públicas para o Load Balancer.
- Associe o Load Balancer às instâncias EC2.
- Configure verificações de saúde apontando para um arquivo específico (e.g., `/healthcheck.php`).

### 8. Implementação do Auto Scaling
- Crie um Auto Scaling Group usando o Launch Template criado.
- Defina a capacidade mínima, desejada e máxima (e.g., min=2, desired=2, max=4).
- Configure políticas de escalonamento baseadas em métricas (e.g., utilização de CPU).
- Associe o Auto Scaling Group ao Load Balancer para distribuição automática das novas instâncias.

### 9. Teste e Validação
- Acesse o DNS público fornecido pelo Load Balancer.
- Verifique se o WordPress está carregando corretamente.
- Certifique-se de que o acesso está sendo feito via HTTP (não HTTPS).
- Teste a escalabilidade aumentando artificialmente a carga e observando o Auto Scaling em ação.

## Materiais de Apoio
- [WordPress no Docker Hub](https://hub.docker.com/_/wordpress)
- Exemplo de Script no GitHub Gist
- Conexão a Instâncias EC2 Privadas sem Gateway NAT

### Tutoriais em Vídeo:
- Configuração Completa na AWS
- Deploy do WordPress com Docker e AWS
- Configuração de Auto Scaling Avançada

## Melhorias Futuras

### Automatização com CI/CD:
- Integrar o AWS CodeDeploy ou AWS CodePipeline para automatizar o processo de implantação.
- Implementar pipelines de Integração Contínua/Entrega Contínua (CI/CD).

### Segurança Avançada:
- Utilizar o AWS Certificate Manager para implementar certificados SSL.
- Configurar o Load Balancer para suportar HTTPS, aumentando a segurança das comunicações.

### Monitoramento e Logging:
- Configurar o Amazon CloudWatch para monitorar métricas de desempenho e saúde.
- Implementar logs centralizados com o Amazon CloudWatch Logs ou serviços como o ELK Stack.

### Backup e Recuperação:
- Configurar snapshots automáticos do RDS para recuperação em caso de falhas.
- Implementar estratégias de backup para os dados armazenados no EFS.

## Contribuições

Contribuições são bem-vindas! Sinta-se à vontade para abrir Issues ou Pull Requests com melhorias, correções ou sugestões.

## Licença

Este projeto está sob a licença MIT. Veja o arquivo LICENSE para mais detalhes.

---
Agradecemos por conferir este projeto. Esperamos que este guia seja útil e que você possa expandir e adaptar este ambiente para atender às suas necessidades específicas. **Boas implementações! 🚀**

