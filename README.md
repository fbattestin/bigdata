# bigdata

#Security
Multitenant, um único (e grande) cluster corporativo com a capacidade de separar logicamente o acesso aos dados por usuários ou por grupos.
Isso é importante para controlar quem possui acesso aos dados e a quais dados devem ou não serem acessados.

Decisões Arquiteturais e Tradeoffs
  1 - Autenticacao e Controle de Contas de Usuários;
    Um cluster Hadoop requer um grande número de contas Kerberos, isso significa que;
      - Cada usuário da plataforma possui uma conta Kerberos;
      - Cada serviço do Hadoop usa uma conta Kerberos por máquina do Cluster;
    A criação segura de uma conta de serviço do Hadoop e a distribuição de keytabs é crucial para a operação segura do cluster.
    
    USUÁRIO -> (autentica) -> KDC (Key Distribution Center) <- (autentica) <- Hadoop Services
    
  
    
  2 - Autorização e Controle de Acesso;
  3 - Interface de Aplicações Analíticas com o Data Lake;
