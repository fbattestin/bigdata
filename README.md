# bigdata4linux

FSimage - Arquivo com o estado atual do File System dos DATANODES. Localizado no NameNode.
EditLog - Semelhante ao Journal do Linux, com o log das alterações do File System. Normalmente é dedicado um host para armazenar esse arquivo que irá auxiliar o NameNode. Comumente chamado de SecondaryNameNode.


# NameNode : 
    É o processo mestre que mantém e gerencia os DataNodes;
    Registra os metadados de todos os blocos armazenados no cluster: localização dos blocos armazenados, tamanho dos arquivos,      permissões, hierarquia, etc;
    Registra todas e cada uma das alterações que ocorrem nos metadados do sistema de arquivos;
    Se um arquivo for excluído no HDFS, o NameNode registrará imediatamente isso no EditLog;
    Recebe regularmente um Heartbeat e um relatório de bloco de todos os
    DataNodes no cluster para garantir que os DataNodes estejam disponíveis;
    Mantém um registro de todos os blocos no HDFS e DataNode em que eles estão armazenados.
    É importante ressaltar que o NameNode em momento algum manipula os dados persistidos no HDFS.
    
    (*) Como é feito a replicação do NameNode?

# DataNode :
    É o processo executado em cada máquina slave;
    É o nó que armazena os dados de fato;
    É responsável por atender aos pedidos de leitura e gravação dos clientes;
    É também é responsável por excluir blocos, criar e replicar blocos baseadonas decisões tomadas pelo NameNode;
    Envia um Heartbeat ao NameNode periodicamente para relatar o estado geral do HDFS. Por padrão, a frequência é definida como 3 segundos.
    
# SecondaryNameNode (CheckpointNode) :
O SecondaryNameNode é o processo responsável por sincronizar os arquivos EditLogs com a imagem do FsImage, para gerar um novo FsImage mais atualizado. Essa atividade é executada pelo SecondaryNameNode de forma agendada, definindo-se um intervalo de tempo em segundos no arquivo core-site.xml. Também pode ser disparada sempre que o total de edições atingir um limite de
tamanho em bytes, predeterminado no arquivo de configuração.

    Processo responsável por ler constantemente todos os sistemas de arquivos e metadados da RAM do NameNode e o grava no disco rígido ou no sistema de arquivos;
    É responsável por combinar o EditLogs com FsImage do NameNode;
    Baixa os EditLogs do NameNode em intervalos regulares e aplica-se a FsImage;
    O novo FsImage é copiado de volta para o NameNode, que é usado sempre que o NameNode for iniciado na próxima vez.


# Blocagem :
    O bloco representa a menor localização contínua em seu disco rígido onde os dados são armazenados.
    O tamanho padrão de cada bloco é de 128 MB no Apache Hadoop 2.x (64 MB no Apache Hadoop 1.x) e que você pode configurar de acordo com sua exigência.
    Não é necessário que, no HDFS, cada arquivo seja armazenado em múltiplos exatos do tamanho de bloco configurado (128 MB,256 MB etc.)
    O HDFS usa um detector de erro conhecido como CRC-32C para verificar todo dado escrito e lido do sistema.
    Na escrita o último DataNode do pipeline é responsável por fazer essa verificação, se ele detectar um erro, o cliente é notificado e deve lidar com esse erro, repetindo a operação, por exemplo.


/* !!!!
        Como estão organizados os Datanodes nos RACKs (Fisicamente e Logicamente);
*/ 
# Comandos HDFS:
    
    prefixo: hdfs dfs 
    
    Listar
    -ls
    [maria_dev@sandbox-hdp ~]$ hdfs dfs -ls /
    
    Remover arquivos
    (*) Criar um arquivo vazio, ou altera a data de criação.
    [maria_dev@sandbox-hdp ~]$ hdfs dfs -touchz /tmp/README.txt
    [maria_dev@sandbox-hdp ~]$ hdfs dfs -rm /tmp/README.txt
    18/09/26 22:34:51 INFO fs.TrashPolicyDefault: Moved: 'hdfs://sandbox-hdp.hortonworks.com:8020/tmp/README.txt' to trash at:                  hdfs://sandbox-hdp.hortonworks.com:8020/user/maria_dev/.Trash/Current/tmp/README.txt

    Mover Arquivos
    -mv
    [maria_dev@sandbox-hdp ~]$ hdfs dfs -mv /tmp/README.txt /user/maria_dev/
    
    Copiar arquivos do sistema de arquivos LOCAL para o HDFS
    -copyFromLocal
    [maria_dev@sandbox-hdp ~]$ hdfs dfs -copyFromLocal /etc/fstab /tmp
    
    Visualizar arquivo
    -cat
    [maria_dev@sandbox-hdp ~]$ hdfs dfs -cat /tmp/fstab
    
    Mover arquivo
    -moveFromLocal
    [maria_dev@sandbox-hdp ~]$ hdfs dfs -moveFromLocal /tmp/passwd /user/maria_dev
    
    Copiar arquivos do sistema de arquivos HDFS para LOCAL
    -get
    [maria_dev@sandbox-hdp ~]$ hdfs dfs -get /user/maria_dev/README.txt /tmp
    
    Mesclar arquivos HDFS para LOCAL
    -getmerge
    [maria_dev@sandbox-hdp ~]$ hdfs dfs -getmerge /user/maria_dev /tmp/arquivos-hdfs

    Copiar os arquivos LOCAL para HDFS
    -put   
    [maria_dev@sandbox-hdp ~]$ hdfs dfs -put /etc/issue /tmp
    [maria_dev@sandbox-hdp ~]$ hdfs dfs -ls /tmp
    Found 1 items
    -rw-r--r--   1 maria_dev hdfs         47 2018-09-26 22:55 /tmp/issue

    Criar diretorio no HDFS
    -mkdir
    [maria_dev@sandbox-hdp ~]$ hdfs dfs -mkdir /tmp/arquivos/
    [maria_dev@sandbox-hdp ~]$ hdfs dfs -ls /tmp/
    Found 5 items
    drwxr-xr-x   - maria_dev hdfs          0 2018-09-26 22:57 /tmp/arquivos
    drwxr-xr-x   - hdfs      hdfs          0 2018-02-01 10:24 /tmp/entity-file-history
    -rw-r--r--   1 maria_dev hdfs        617 2018-09-26 22:41 /tmp/fstab
    drwx-wx-wx   - ambari-qa hdfs          0 2018-02-01 10:29 /tmp/hive
    -rw-r--r--   1 maria_dev hdfs         47 2018-09-26 22:55 /tmp/issue

    Listar HDFS de forma recursiva 
    -ls -R
    [maria_dev@sandbox-hdp ~]$ hdfs dfs -ls -R /tmp

    Remover diretorio HDFS
    -rmdir
    [maria_dev@sandbox-hdp ~]$ hdfs dfs -rmdir /tmp/arquivos/
    
    Copiar diretorio do LOCAL para HDFS
    -copyFromLocal
    [maria_dev@sandbox-hdp ~]$ hdfs dfs -copyFromLocal hadoop/ /tmp
    
    Remover de forma Recursiva
    -rm -R
    [maria_dev@sandbox-hdp ~]$ hdfs dfs -rm -R /tmp/hadoop

        

# Comandos PIG
As chamadas PIG podem ser realizadas via AMBARI (Hortonworks) ou via command line HDFS. 
Os scripts PIG são salvos com a extensão .pig conforme sintaxe abaixo:

> pig -x tez script.pig
       -x = executar
       tez = motor de execução do MAPREDUCE, com as opções TEZ, LOCAL e MAPREDUCE

    --CHAMDA DO PIG VIA SHELL:pig -x tez script.pig
    --ONDE TEZ É VARIÁVEL PARA ESCOLHA DO MOTOR DE EXECUÇÃO DO MAPREDURCE
    --TEZ É UM MOTOR DE GRAFO

    --CARREGANDO ARQUIVO PARA VARIAVEL "VIEW" DO PIG
    --lista_data = LOAD '/user/maria_dev/u.data' AS (usuarioID: int, filmeID: int, classificacao: int, data: int);
    --DUMP lista_data;

    --COMANDO LIMIT = SINTAXE SQL TOP
    --dados_limit = LIMIT lista_data 5;
    --DUMP dados_limit;


    --COMANDO FILTER = SINTAXE SQL WHERE
    --cla_filme = FILTER lista_data BY classificacao > 3.0;
    --DUMP cla_filme;

    --SINTAXE AND PARA O WHERE
    --teste = FILTER lista_data BY filmeID > 100 AND classificacao ==3;
    --DUMP teste;


    --CARREGANDO ARQUIVO DO HDFS PARA VARIAVEL lista_item ARQUIVO CONTEM CAMPOS TEXTO
    --E SAO CARRAGADOS NA VARIAVEL COM A TIPAGEM CHARARRAY.
    --lista_item = LOAD '/user/maria_dev/u.item' USING PigStorage ('|') AS (filmeID: int, filmeTitulo: chararray, dataLancamento: chararray, videoLancamento: chararray, link: chararray);

    --SINTAXE LIKE PARA CAMPOS CHAR/STRING
    --nome_filme = FILTER lista_item BY (filmeTitulo matches '.*Story.*');
    --DUMP nome_filme;

    --SINTAXE DISTINCT
    --USANDO O DATASET CARREGADO NA lista_data DO ARQUIVO u.data
    --data_unique = DISTINCT lista_data;
    --dump data_unique;

    --INNER JOIN
    --Carregando tabela com dados dos filmes.
    lista_data = load '/user/maria_dev/u.data' as (usuarioid: int, filmeid: int, classificacao: int, data: int);

    --Carregando tabela com itens dos filmes.
    lista_item = load '/user/maria_dev/u.item' USING PigStorage ('|') AS (filmeid: int, filmetitulo: chararray, datalancamento: chararray, videolancamento: chararray, link: chararray);

    --Carregando tabela com itens dos filmes.
    lista_item = load '/user/maria_dev/u.item' USING PigStorage ('|') AS (filmeid: int, filmetitulo: chararray, datalancamento: chararray, videolancamento: chararray, link: chararray);

    --INNER JOIN MOSTRANDO TODOS OS CAMPOS
    --inner_join = JOIN lista_data BY filmeid, lista_item BY filmeid;
    --DUMP inner_join;

    --INNER JOIN SELECIONANDO APENAS ALGUNS CAMPOS
    --nr = foreach inner_join generate lista_data::filmeid, lista_item::filmetitulo;
    --dump nr;

    --LEFT JOIN
    --left_join = JOIN lista_data BY filmeid LEFT OUTER, lista_item BY filmeid;
    --dump left_join;

    --RETORNANDO SOMENTE OS CAMPOS FILMEID E NOME DO FILME
    --nr = foreach left_join generate lista_data::filmeid, lista_item::filmetitulo;
    --dump nr;

    --SPLIT
    --Conceito: resultados definidos por uma condição são enviados para uma variável específicas.
    --Utilizando o dataset lista_data

    split lista_data into classificacao_menor if classificacao<3.0, classificacao_maior if classificacao>3.00;

    --inserindo os 5 primeiros (comando LIMIT) da lista em uma nova variavel
    lista_menor = limit classificacao_menor 5;
    lista_maior = limit classificacao_maior 5;

    dump lista_menor;
    dump lista_maior;
