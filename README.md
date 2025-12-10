public class StrategyLimitException extends RuntimeException {
    public StrategyLimitException(String message) {
        super(message);
    }
}


import java.util.List;

public interface Strategy {
    // Ordena e retorna uma nova lista ordenada (não altera a entrada)
    List<Integer> ordena(List<Integer> elementos) throws StrategyLimitException;

    // cada estratégia pode definir seu limite conceitual (padrão: Integer.MAX_VALUE)
    default int maxSize() {
        return Integer.MAX_VALUE;
    }
}

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class BubbleSort implements Strategy {

    private static final int MAX = 50; // exemplo de limite conceitual

    @Override
    public List<Integer> ordena(List<Integer> elementos) throws StrategyLimitException {
        if (elementos.size() > maxSize()) {
            throw new StrategyLimitException("BubbleSort: limite excedido");
        }
        // opera sobre uma cópia para garantir que a entrada não seja alterada
        List<Integer> copia = new ArrayList<>(elementos);
        Collections.sort(copia); // simula ordenação
        return copia;
    }

    @Override
    public int maxSize() {
        return MAX;
    }
}


import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class InsertionSort implements Strategy {

    private static final int MAX = 100; // exemplo

    @Override
    public List<Integer> ordena(List<Integer> elementos) throws StrategyLimitException {
        if (elementos.size() > maxSize()) {
            throw new StrategyLimitException("InsertionSort: limite excedido");
        }
        List<Integer> copia = new ArrayList<>(elementos);
        Collections.sort(copia);
        return copia;
    }

    @Override
    public int maxSize() {
        return MAX;
    }
}


import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class QuickSort implements Strategy {

    private static final int MAX = 200; // exemplo

    @Override
    public List<Integer> ordena(List<Integer> elementos) throws StrategyLimitException {
        if (elementos.size() > maxSize()) {
            throw new StrategyLimitException("QuickSort: limite excedido");
        }
        List<Integer> copia = new ArrayList<>(elementos);
        Collections.sort(copia);
        return copia;
    }

    @Override
    public int maxSize() {
        return MAX;
    }
}


import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class MergeSort implements Strategy {

    private static final int MAX = Integer.MAX_VALUE; // fallback: aceita qualquer tamanho

    @Override
    public List<Integer> ordena(List<Integer> elementos) {
        // Merge é fallback: aceitará e sempre ordenará
        List<Integer> copia = new ArrayList<>(elementos);
        Collections.sort(copia);
        return copia;
    }

    @Override
    public int maxSize() {
        return MAX;
    }
}

import java.util.ArrayList;
import java.util.List;

public class Cliente {

    private Strategy ordena;

    public Cliente(){
        this.ordena = new BubbleSort();
    }

    public void setBubble(){
        this.ordena = new BubbleSort();
    }

    public void setInsertion(){
        this.ordena = new InsertionSort();
    }

    public void setMerge(){
        this.ordena = new MergeSort();
    }

    public void setQuick(){
        this.ordena = new QuickSort();
    }

    public List<Integer> ordena(List<Integer> elementos){
        // cópia defensiva aqui (para garantir que a lista original NÃO seja alterada)
        List<Integer> copiaDefensiva = new ArrayList<>(elementos);

        try {
            // Strategy pode lançar StrategyLimitException quando recusa processar
            return this.ordena.ordena(copiaDefensiva);
        } catch (StrategyLimitException e) {
            // registrar substituição e usar MergeSort como fallback
            System.out.println("Estratégia " + this.ordena.getClass().getSimpleName()
                    + " recusou (limite). Usando MergeSort como fallback.");
            Strategy fallback = new MergeSort();
            return fallback.ordena(copiaDefensiva);
        }
    }
}

import java.util.ArrayList;
import java.util.List;

public class Atividade {

    public static void main(String[] args) {

        Cliente cliente = new Cliente();
        List<Integer> elementos = new ArrayList<>();
        elementos.add(10);
        elementos.add(0);
        elementos.add(3);

        // receber o resultado (não assumimos alteração da lista original)
        List<Integer> ordenados = cliente.ordena(elementos);

        for (Integer e : ordenados){
            System.out.println(e);
        }
    }
}














# compassUOL-Docker

# Desafio 02 - Implementação de WordPress com DevSecOps na AWS

Este repositório apresenta um projeto prático que tem como objetivo fortalecer habilidades em DevSecOps, fazendo uso do Docker e de diversos serviços da AWS para implantar uma aplicação WordPress completamente funcional e escalável.

## Sumário
- [Descrição Geral](#descrição-geral)
- [Tecnologias Utilizadas](#tecnologias-utilizadas)
- [Pré-requisitos](#pré-requisitos)
- [Etapas do Projeto](#etapas-do-projeto)
- [Melhorias Futuras](#melhorias-futuras)

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
- Linux

## Pré-requisitos

- Gerenciar VPCs, sub-redes e tabelas de rotas.
- Criar e configurar Security Groups.
- Provisionar instâncias EC2 e gerenciar pares de chaves SSH.
- Configurar bancos de dados com o Amazon RDS.
- Criar sistemas de arquivos com o Amazon EFS.
- Configurar Load Balancers e grupos de Auto Scaling.

## Etapas do Projeto

É válido ponderar que a AWS constantemente atualiza o seu layout, portanto, resolvi nao utilizar imagens das telas da AWS nesse projeto, tendo em vista que voces poderão se deparar com imagens diferente das minhas.

### 1. Configuração da VPC
- Crie uma VPC personalizada.
- Defina sub-redes públicas e privadas em diferentes zonas de disponibilidade.
- Configure as tabelas de rotas apropriadas para cada sub-rede.

### 2. Implementação do NAT Gateway 
- Implemente um NAT Gateway na sub-rede pública.
- Atualize as tabelas de rotas das sub-redes privadas para direcionar o tráfego de saída para o Gateway NAT.

### 3. Configuração dos Security Groups
**Para o EC2:**

| Tipo | Protocolo | Intervalo de portas | Destino |
|---|---|---|---|
| HTTP | TCP | 80 |Grupo de Segurança do Load balancers|
| SSH | TCP | 22 |0.0.0.0/0 |
| MYSQL/AURORA | TCP | 3306 | Grupo de Segurança do RDS|
| NFS | TCP | 2049 | Grupo de Segurança do EFS|

- Controle o acesso SSH (22) a partir do seu IP ou de um Bastion Host.
- Permita tráfego de saída para o RDS (3306) e para o EFS (2049).

**Para o RDS:**

| Tipo | Protocolo | Intervalo de portas | Destino |
|---|---|---|---|
| MYSQL/AURORA | TCP | 3306 | Grupo de Segurança das Instâncias|

- Permita conexões somente das instâncias EC2.

**Para o EFS:**

| Tipo | Protocolo | Intervalo de portas | Destino |
|---|---|---|---|
| NFS | TCP | 2049 |Grupo de Segurança das Instâncias|

- Autorize conexões NFS (2049) das instâncias EC2.

### 4. Criação do Amazon EFS
- Configure um sistema de arquivos no Amazon EFS.
- Altere apenas o grupo de segurança para o grupo criado para o serviço
- Anote o endpoint fornecido, pois será usado nas instâncias EC2 para montar o sistema de arquivos.

### 5. Configuração do Amazon RDS (MySQL)
- Crie um banco de dados MySQL usando o Amazon RDS.
- Escolha a VPC e suas subredes criadas anteriormente e mantenha o acesso privado.
- Selecione o grupo de segurança criado para o RDS e mantenha a Zona de disponibilidade sem preferência.
- Defina as credenciais de acesso e o nome do banco de dados inicial.
- Certifique-se de que o banco de dados não seja acessível publicamente e que os Security Groups estejam corretamente configurados.

### 6. Provisionamento das Instâncias EC2
**Criação de um Launch Template:**
- Utilize o Amazon Linux 2 como AMI.
- Selecione o tipo de instância adequado (e.g., t2.micro).
- Inclua um script de inicialização `user_data.sh`(disponivel nesse repositório) que:
  - Instala o Docker.
  - Configura o Docker para iniciar na inicialização.
  - Implementa o WordPress usando um Dockerfile ou Docker Compose.
  - Monta o EFS no diretório apropriado.
  - Configura variáveis de ambiente para conectar-se ao RDS.

### 7. Configuração do Load Balancer
- Crie um Classic Load Balancer.
- Selecione a VPC criada anteriormente
- Selecione o Grupo de Segurança já criado para o serviço
- Configure as portas de escuta (e.g., HTTP na porta 80).
- Selecione as sub-redes públicas para o Load Balancer.
- Associe o Load Balancer às instâncias EC2.

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

## Melhorias Futuras

### Automatização com CI/CD:
- Integrar o AWS CodeDeploy ou AWS CodePipeline para automatizar o processo de implantação.
- Implementar pipelines de Integração Contínua/Entrega Contínua (CI/CD).

### Monitoramento e Logging:
- Configurar o Amazon CloudWatch para monitorar métricas de desempenho e saúde.
- Implementar logs centralizados com o Amazon CloudWatch Logs ou serviços como o ELK Stack.

### Backup e Recuperação:
- Configurar snapshots automáticos do RDS para recuperação em caso de falhas.
- Implementar estratégias de backup para os dados armazenados no EFS.





import java.util.ArrayList;
import java.util.List;

public class Atividade {

    public static void main(String[] args) {

        Cliente cliente = new Cliente();
        List<Integer> elementos = new ArrayList<>();
        elementos.add(10);
        elementos.add(0);
        elementos.add(3);

        cliente.ordena(elementos);
        for (Integer e : elementos){
            System.out.println(e);
        }
    }
}




import java.util.Collections;
import java.util.List;

public class BubbleSort implements Strategy{

    @Override
    public List<Integer> ordena(List<Integer> elementos) {
        Collections.sort(elementos);
        return elementos;
    }
    
}





import java.util.List;

public class Cliente {

    private Strategy ordena;

    public Cliente(){
        this.ordena = new BubbleSort();
    }

    public void setBubble(){
        this.ordena = new BubbleSort();
    }

    public void setInsertion(){
        this.ordena = new InsertionSort();
    }

    public void setMerge(){
        this.ordena = new MergeSort();
    }

    public void setQuick(){
        this.ordena = new QuickSort();
    }

    public List<Integer> ordena(List<Integer> elementos){
        return this.ordena.ordena(elementos);
    }

}





import java.util.Collections;
import java.util.List;

public class InsertionSort implements Strategy{
    
    @Override
    public List<Integer> ordena(List<Integer> elementos) {
        Collections.sort(elementos);
        return elementos;
    }

}




import java.util.Collections;
import java.util.List;

public class MergeSort implements Strategy {
    
    @Override
    public List<Integer> ordena(List<Integer> elementos) {
        Collections.sort(elementos);
        return elementos;
    }

}




import java.util.Collections;
import java.util.List;

public class QuickSort implements Strategy {
    
    @Override
    public List<Integer> ordena(List<Integer> elementos) {
        Collections.sort(elementos);
        return elementos;
    }

}



import java.util.List;

public interface Strategy {
        List<Integer> ordena(List<Integer> elementos);
}
