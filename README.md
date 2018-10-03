# bigdata4linux

FSimage - Arquivo com o estado atual do File System dos DATANODES. Localizado no NameNode.
EditLog - Semelhante ao Journal do Linux, com o log das alterações do File System. Normalmente é dedicado um host para armazenar esse arquivo que irá auxiliar o NameNode. Comumente chamado de SecondaryNameNode.

Para exercicios:
    Appliance: HDP_2.6.4_virtualbox_01_02_2018_1428.ova
    git clone https://github.com/4linux/hadoop.git
    hdfs dfs -put /home/maria_dev/hadoop/ml-100k/* /home/maria_dev/
    hdfs dfs -put /home/maria_dev/hadoop/dados/* /user/maria_dev/


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

# Simulado - Prova HortonWroks [HDPCD] 
    #Task1 - Importando Arquivos para HDFS
    -----------------------------------------
    [maria_dev@sandbox-hdp ~]$ cd ~
    [maria_dev@sandbox-hdp ~]$ git clone https://github.com/rashidaligee/HDPCD-Certification.git 
    Initialized empty Git repository in /home/maria_dev/HDPCD-Certification/.git/
    remote: Enumerating objects: 197, done.
    remote: Counting objects: 100% (197/197), done.
    remote: Compressing objects: 100% (96/96), done.
    remote: Total 197 (delta 93), reused 197 (delta 93), pack-reused 0
    Receiving objects: 100% (197/197), 995.31 KiB | 742 KiB/s, done.
    Resolving deltas: 100% (93/93), done.
    [maria_dev@sandbox-hdp ~]$ 

    [maria_dev@sandbox-hdp flightdelays]$ hdfs dfs -mkdir /user/maria_dev/flightdelays/
    [maria_dev@sandbox-hdp flightdelays]$ hdfs dfs -put flight* /user/maria_dev/flightdelays    
    [maria_dev@sandbox-hdp flightdelays]$ hdfs dfs -ls /user/maria_dev/flightdelays
    Found 3 items
    -rw-r--r--   1 maria_dev hdfs     956486 2018-10-01 22:20 /user/maria_dev/flightdelays/flight_delays1.csv
    -rw-r--r--   1 maria_dev hdfs     961831 2018-10-01 22:20 /user/maria_dev/flightdelays/flight_delays2.csv
    -rw-r--r--   1 maria_dev hdfs     972860 2018-10-01 22:20 /user/maria_dev/flightdelays/flight_delays3.csv
    --------------------------------
    Task 2 - Cleanup using PIG
    --------------------------------
    vi filghtdelays_cleanup.pig
    
    --Criando variável para armarezar dataset em memória
    stage0 = LOAD 'flightdelays' USING PigStorage (',');
    --DUMP stage0;
    --Criando variável com filtro no campo deppTime (ou no posicional do dataset $4) <> de 'NA'
    stage1 = FILTER stage0 BY (chararray)$4 != 'NA';
    --Gerando arquivo com resultset reduzido a partir do filtro descrito acima.;
    stage2 = FOREACH stage1 GENERATE $0 AS year:int,$1 as month:int,$2 as dayofmonth:int, $4 as deptime:int, $8 as uniquecarrier:chararray, $9 as flightnum:int, $14 as arrdelay:int, $16 as origin:chararray, $17 as dest:chararray;
    STORE stage2 INTO 'flightdelays_clean' USING PigStorage (',');

    ---------------------------------
    Task 3 
    ---------------------------------
    Task 3.1
    --------
        stage0 = LOAD 'flightdelays_clean' USING PigStorage (',') as (year:int, month:int, dayofmonth:int, deptime:int, uniquecarrier:chararray, flightnum:int, arrdelay:int, origin:chararray, dest:chararray);
    stage1 = GROUP stage0 ALL;
    stage2 = FOREACH stage1 GENERATE COUNT(stage0);
    STORE stage2 INTO 'fligthdelays_cleand_total';
    --dump stage2;
    
    Task 3.2
    --------
    stage0 = LOAD 'flightdelays_clean' USING PigStorage (',');
    stage1 = FILTER stage0 BY (chararray)$8 == 'DEN';
    stage2 = GROUP stage1 ALL;
    stage3 = FOREACH stage2 GENERATE COUNT(stage1);
    STORE stage3 INTO 'denver_total';
    --dump stage3;

    Task 3.3
    --------
    stage0 = LOAD 'flightdelays_clean' USING PigStorage (',');
    stage1 = FILTER stage0 BY (chararray)$8 == 'DEN' AND (int)$6 > 60;
    --dump stage1;
    stage2 = GROUP stage1 ALL;
    stage3 = FOREACH stage2 GENERATE COUNT(stage1);
    STORE stage3 INTO 'denver_late';

    ----------------------------------------------
    Task 4 - Criar tabela externa no HIVE (HQL)
    ----------------------------------------------
    --Iniciar shell do HIVE
    >hive
    >CREATE EXTERNAL TABLE flightdelays_ (year int,month int,dayofmonth int, deptime int, uniquecarrier string, flightnum int, arrdelay int, origin string, dest string) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' LOCATION '/user/maria_dev/flightdelays_clean';

    --ou via script de comando.
    hcat -f hive_external.hql
    
    ----------------------------------------------
    Task 5 - LOAD HDFS File into HIVE Table (Pig)
    ----------------------------------------------
    
    stage0 = LOAD 'flightdelays' USING org.apache.hive.hcatalog.pig.HCatLoader();
    stage1 = FILTER stage0 BY arrdelay > 0;
    stage2 = ORDER stage1 BY arrdelay DESC PARALLEL 3;
    STORE stage2 INTO 'flightdelays_nonzero' USING PigStorage (',');
    
    Task 6
    CREATE TABLE sfo_weather_txt (station_name STRING, year INT, month INT, dayofmonth INT, precipitation INT, temperature_max INT, temperature_MIN INT) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' STORED AS TEXTFILE;

    LOAD DATA LOCAL INPATH '....' OVERWRITE INTO TABLE

# HBase
    >hbase shell
    --Listar tabelas no hbase
    >list
    --exibir status do sistema
    >status 'detailed'
    --criar tabelas
    create 'clientes','contato','pagamento'
    create 'vendas','info_produto','info_venda'
    --informações da tabela
    desc 'vendas'
    desc 'clientes'
    --inserir registros 
    put 'clientes','1','contato:Nome', 'Fabio Battestin'
    put 'clientes','1','contato:Endereco','Av. Domingos de Morais'
    put 'clientes','1','pagamento:Tipo','Cartao de Credito'
    put 'clientes','1','pagamento:Valor','1300'
    put 'clientes','1','pagamento:ID_Pagamento','001'
    put 'vendas','1','info_produto:Produto','TV'
    put 'vendas','1','info_produto:Marca','LG'
    put 'vendas','1','info_produto:Departamento','TV e Video'
    put 'vendas','1','info_venda:Vendedor','Pauro'
    put 'vendas','1','info_venda:ID_Pag','001'
    
    --select na tabela
    scan 'clientes'
    quit
    
# Projeto02
Carga via PIG para HBase
pig -useHCatalog
    
    lista_data = LOAD '/user/maria_dev/u.data' AS (usuarioID: int, filmeID: int, classificacao: int, data: int);
    dump lista_data;
    lista_item = LOAD '/user/admin/u.item' using PigStorage('|') AS (filmeID: int, filmeTitulo: chararray, dataLancamento: chararray, videoLancamento: chararray, link: chararray);

    classifcacao_por_filme = GROUP lista_data BY filmeID;

    avgRatings = FOREACH classifcacao_por_filme GENERATE group AS filmeID,
        AVG(lista_data.classificacao) AS avgRating;

    cincoestrelas = FILTER avgRatings BY avgRating > 4.0;

    ---dump cincoestrelas;

    STORE cincoestrelas INTO 'hbase://cincoestrelas' USING org.apache.pig.backend.hadoop.hbase.HBaseStorage('filmeID:filmeID avgRating:avgRating');
    
 
# Projeto03

    lista_data = LOAD '/user/maria_dev/u.data' AS (usuarioid: int, filmeid: int, classificacao: int, data: int);
    lista_item = LOAD '/user/maria_dev/u.item' using PigStorage('|') AS (filmeid: int, filmetitulo: chararray, datalancamento: chararray, videolancamento: chararray, link: chararray);

    classifcacao_por_filme = GROUP lista_data BY filmeid;

    avgratings = FOREACH classifcacao_por_filme GENERATE group AS filmeid,
        AVG(lista_data.classificacao) AS avgrating;

    cincoestrelas = FILTER avgratings BY avgrating > 4.0;
    --dump cincoestrelas;
    STORE cincoestrelas INTO 'cincoestrelas' using org.apache.hive.hcatalog.pig.HCatLoader();
    ~                                                                                                   
    
