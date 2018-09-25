# bigdata4linux

FSimage - Arquivo com o estado atual do File System dos DATANODES. Localizado no NameNode.
EditLog - Semelhante ao Journal do Linux, com o log das alterações do File System. Normalmente é dedicado um host para armazenar esse arquivo que irá auxiliar o NameNode. Comumente chamado de SecondaryNameNode.


*NameNode: 
    É o processo mestre que mantém e gerencia os DataNodes;
    Registra os metadados de todos os blocos armazenados no cluster: localização dos blocos armazenados, tamanho dos arquivos,      permissões, hierarquia, etc;
    Registra todas e cada uma das alterações que ocorrem nos metadados do sistema de arquivos;
    Se um arquivo for excluído no HDFS, o NameNode registrará imediatamente isso no EditLog;
    Recebe regularmente um Heartbeat e um relatório de bloco de todos os
    DataNodes no cluster para garantir que os DataNodes estejam disponíveis;
    Mantém um registro de todos os blocos no HDFS e DataNode em que eles estão armazenados.
    É importante ressaltar que o NameNode em momento algum manipula os dados persistidos no HDFS.
    
    (*) Como é feito a replicação do NameNode?

DataNode :
    É o processo executado em cada máquina slave;
    É o nó que armazena os dados de fato;
    É responsável por atender aos pedidos de leitura e gravação dos clientes;
    É também é responsável por excluir blocos, criar e replicar blocos baseadonas decisões tomadas pelo NameNode;
    Envia um Heartbeat ao NameNode periodicamente para relatar o estado geral do HDFS. Por padrão, a frequência é definida como 3 segundos.
    
SecondaryNameNode (CheckpointNode) :
    É o processo responsável por ler constantemente todos os sistemas de arquivos e metadados da RAM do NameNode e o grava no disco rígido ou no sistema de arquivos;
    É responsável por combinar o EditLogs com FsImage do NameNode;
    Baixa os EditLogs do NameNode em intervalos regulares e aplica-se a FsImage;
    O novo FsImage é copiado de volta para o NameNode, que é usado sempre que o NameNode for iniciado na próxima vez.
O SecondaryNameNode é o processo responsável por sincronizar os arquivos EditLogs com a imagem do FsImage, para gerar um novo FsImage mais atualizado. Essa atividade é executada pelo SecondaryNameNode de forma agendada, definindo-se um intervalo de tempo em segundos no arquivo core-site.xml. Também pode ser disparada sempre que o total de edições atingir um limite de
tamanho em bytes, predeterminado no arquivo de configuração.

#Blocagem:
  


/* !!!!
        Como estão organizados os Datanodes nos RACKs (Fisicamente e Logicamente);
*/        
