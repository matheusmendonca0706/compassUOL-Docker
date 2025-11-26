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

package test.psoft;

import org.junit.jupiter.api.Test;

import main.psoft.Biblioteca;
import main.psoft.Livro;

import static org.junit.jupiter.api.Assertions.*;

import java.util.List;

import org.junit.jupiter.api.BeforeEach;

public class BibliotecaTest {

    private Biblioteca biblioteca;
    private Livro livro;
    private Livro livroIndisponivel;

    @BeforeEach
    public void setUp() {
        this.biblioteca = new Biblioteca();
        this.livro = new Livro("Nome do livro");
        this.livroIndisponivel = new Livro("Livro indisponível");
        this.livroIndisponivel.setDisponibilidade(false);

        biblioteca.addUser("joao.pedro@gmail.com");
        biblioteca.addUser("coisa.peppa@gmail.com");

        biblioteca.addLivro("Senhor dos Anéis");
        biblioteca.addLivro(this.livro);
        biblioteca.addLivro(this.livroIndisponivel);
    }

    // ====================
    //        US1
    // ====================

    @Test
    public void testBuscaLivroExistentePeloTituloTotal() {
        assertEquals(this.livro, this.biblioteca.retornaLivro("Nome do livro"));
    }

    @Test
    public void testBuscaLivroExistentePeloTituloParcial() {
        assertEquals(this.livro, this.biblioteca.retornaLivro("Nome do"));
    }

    @Test
    public void testBuscaLivroInexistente() {
        assertNull(this.biblioteca.retornaLivro("não existo"));
    }

    // NOVOS TESTES US1
    @Test
    public void testBuscaTituloParcialNoMeio() {
        assertEquals(this.livro, this.biblioteca.retornaLivro("do li"));
    }

    @Test
    public void testBuscaTituloCaseInsensitive() {
        assertEquals(this.livro, this.biblioteca.retornaLivro("nOmE Do LiVrO"));
    }


    // ====================
    //        US2
    // ====================

    @Test
    public void testGetLivros() {
        List<String> livros = this.biblioteca.getLivrosCadastrados();
        assertNotNull(livros);
        assertTrue(livros.contains("Senhor dos Anéis"));
        assertTrue(livros.contains(this.livro.getNome()));
        assertTrue(livros.contains("Livro indisponível"));
    }

    @Test
    public void testGetLivrosDisponiveis() {
        List<String> livros = this.biblioteca.getLivrosDisponiveis();
        assertNotNull(livros);
        assertTrue(livros.contains("Senhor dos Anéis"));
        assertTrue(livros.contains(this.livro.getNome()));
        assertFalse(livros.contains("Livro indisponível"));
    }

    // NOVOS TESTES US2
    @Test
    public void testGetLivrosDisponiveisNenhumDisponivel() {
        for (String titulo : biblioteca.getLivrosCadastrados()) {
            biblioteca.retornaLivro(titulo).setDisponibilidade(false);
        }
        assertTrue(biblioteca.getLivrosDisponiveis().isEmpty());
    }

    @Test
    public void testGetLivrosQuandoNaoHaLivros() {
        Biblioteca b2 = new Biblioteca();
        assertTrue(b2.getLivrosCadastrados().isEmpty());
        assertTrue(b2.getLivrosDisponiveis().isEmpty());
    }

    @Test
    public void testGetLivrosDisponiveisApenasUmDisponivel() {
        this.livro.setDisponibilidade(false); // Nome do livro fica indisponível
        List<String> disponiveis = biblioteca.getLivrosDisponiveis();
        assertEquals(1, disponiveis.size());
        assertEquals("Senhor dos Anéis", disponiveis.get(0));
    }


    // ====================
    //        US3
    // ====================

    @Test
    public void testReservaLivroDisponivelUsuarioOK() {
        String result = this.biblioteca.reservaLivro("joao.pedro@gmail.com", "Senhor dos Anéis");
        assertEquals("Reserva bem sucedida do livro: Senhor dos Anéis", result);
    }

    @Test
    public void testReservaLivroDisponivelUsuarioNaoCadastrado() {
        RuntimeException ex = assertThrows(
            RuntimeException.class,
            () -> this.biblioteca.reservaLivro("naoCadastrado@gmail.com", "Senhor dos Anéis")
        );
        assertTrue(ex.getMessage().contains("Usuário não cadastrado."));
    }

    @Test
    public void testReservaLivroIndisponivel() {
        RuntimeException ex = assertThrows(
            RuntimeException.class,
            () -> this.biblioteca.reservaLivro("joao.pedro@gmail.com", "Livro indisponível")
        );
        assertTrue(ex.getMessage().contains("Livro indisponível."));
    }

    // NOVOS TESTES US3
    @Test
    public void testReservaLivroNaoCadastrado() {
        assertThrows(NullPointerException.class,
            () -> this.biblioteca.reservaLivro("joao.pedro@gmail.com", "LivroFantasma"));
    }

    @Test
    public void testReservaComTituloParcialNaoPode() {
        assertThrows(NullPointerException.class,
            () -> this.biblioteca.reservaLivro("joao.pedro@gmail.com", "Senhor"));
    }
}

