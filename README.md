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








Perfeito. Vou te entregar exatamente o que deve ir em cada arquivo, pronto para copiar no VSCode.
Isso segue o padrão Observer mostrado nos slides e funciona na estrutura que você mostrou (src/main/java).

Crie estes 8 arquivos dentro de src/main/java.

⸻

1️⃣ Bootstrap.java (classe principal)

public class Bootstrap {

    public static void main(String[] args) {

        Produto produto = new Produto("Notebook Gamer", 5000);

        Usuario usuario = new Usuario("Matheus", "matheus@email.com");

        AlertaDisponibilidade alertaDisponibilidade =
                new AlertaDisponibilidade(usuario);

        AlertaPromocao alertaPromocao =
                new AlertaPromocao(usuario);

        produto.addListener(alertaDisponibilidade);
        produto.addListener(alertaPromocao);

        System.out.println("Produto ficou disponível:");
        produto.tornarDisponivel();

        System.out.println("\nProduto entrou em promoção:");
        produto.entrarPromocao("PROMO20");
    }
}


⸻

2️⃣ Produto.java (SOURCE / SUBJECT)

import java.util.HashSet;
import java.util.Set;

public class Produto {

    private String nome;
    private double preco;
    private boolean disponivel;

    private Set<AlertaListener> listeners = new HashSet<>();

    public Produto(String nome, double preco) {
        this.nome = nome;
        this.preco = preco;
        this.disponivel = false;
    }

    public void addListener(AlertaListener listener) {
        listeners.add(listener);
    }

    public void removeListener(AlertaListener listener) {
        listeners.remove(listener);
    }

    public void tornarDisponivel() {

        this.disponivel = true;

        ProdutoEvent event = new ProdutoEvent(this);

        for (AlertaListener listener : listeners) {
            listener.produtoDisponivel(event);
        }
    }

    public void entrarPromocao(String codigoPromocional) {

        ProdutoEvent event = new ProdutoEvent(this, codigoPromocional);

        for (AlertaListener listener : listeners) {
            listener.produtoEmPromocao(event);
        }
    }

    public String getNome() {
        return nome;
    }

    public double getPreco() {
        return preco;
    }
}


⸻

3️⃣ ProdutoEvent.java (EVENT)

public class ProdutoEvent {

    private Produto produto;
    private String codigoPromocional;

    public ProdutoEvent(Produto produto) {
        this.produto = produto;
    }

    public ProdutoEvent(Produto produto, String codigoPromocional) {
        this.produto = produto;
        this.codigoPromocional = codigoPromocional;
    }

    public Produto getProduto() {
        return produto;
    }

    public String getCodigoPromocional() {
        return codigoPromocional;
    }
}


⸻

4️⃣ Usuario.java

public class Usuario {

    private String nome;
    private String email;

    public Usuario(String nome, String email) {
        this.nome = nome;
        this.email = email;
    }

    public String getNome() {
        return nome;
    }

    public String getEmail() {
        return email;
    }
}


⸻

5️⃣ AlertaListener.java (INTERFACE)

public interface AlertaListener {

    void produtoDisponivel(ProdutoEvent event);

    void produtoEmPromocao(ProdutoEvent event);

}


⸻

6️⃣ AlertaAdapter.java (ADAPTER)

public abstract class AlertaAdapter implements AlertaListener {

    @Override
    public void produtoDisponivel(ProdutoEvent event) {
        // não faz nada
    }

    @Override
    public void produtoEmPromocao(ProdutoEvent event) {
        // não faz nada
    }
}


⸻

7️⃣ AlertaDisponibilidade.java (LISTENER)

public class AlertaDisponibilidade extends AlertaAdapter {

    private Usuario usuario;

    public AlertaDisponibilidade(Usuario usuario) {
        this.usuario = usuario;
    }

    @Override
    public void produtoDisponivel(ProdutoEvent event) {

        Produto produto = event.getProduto();

        System.out.println(
                "Email enviado para " + usuario.getEmail()
        );

        System.out.println(
                "Produto disponível: " + produto.getNome()
        );

        System.out.println(
                "Link: /produto/" + produto.getNome()
        );
    }
}


⸻

8️⃣ AlertaPromocao.java (LISTENER)

public class AlertaPromocao extends AlertaAdapter {

    private Usuario usuario;

    public AlertaPromocao(Usuario usuario) {
        this.usuario = usuario;
    }

    @Override
    public void produtoEmPromocao(ProdutoEvent event) {

        Produto produto = event.getProduto();

        System.out.println(
                "Promoção enviada para " + usuario.getEmail()
        );

        System.out.println(
                "Produto: " + produto.getNome()
        );

        System.out.println(
                "Código promocional: " + event.getCodigoPromocional()
        );
    }
}


⸻

9️⃣ Estrutura final do projeto

Seu VSCode deve ficar assim:

src/main/java
│
├── Bootstrap.java
├── Produto.java
├── ProdutoEvent.java
├── Usuario.java
├── AlertaListener.java
├── AlertaAdapter.java
├── AlertaDisponibilidade.java
└── AlertaPromocao.java


⸻

10️⃣ Teste esperado

Quando rodar o programa:

Produto ficou disponível:
Email enviado para matheus@email.com
Produto disponível: Notebook Gamer
Link: /produto/Notebook Gamer

Produto entrou em promoção:
Promoção enviada para matheus@email.com
Produto: Notebook Gamer
Código promocional: PROMO20


⸻

✅ Com isso você já tem:
	•	implementação correta do Observer
	•	código compilando
	•	estrutura aceita pelo professor

⸻

Se quiser, eu também posso te mandar o diagrama UML ASCII perfeito para desenhar no papel, porque tem um detalhe de multiplicidade que muitos alunos erram nessa atividade.


