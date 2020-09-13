## Instalação e Configuração do PostgreSQL

Aqui será descrito como faço para instalar o PostgreSQL a partir do código fonte, ou seja, compilado. Não irei abordar a instalação por pacotes do repositório, porque prefiro ter total controle sobre a instalação.

É fato que a instalação de forma compilada dificulta um pouco a parte de automação do ambiente, mas irei mostrar mais na frente que é possível e não muito complicado, mas é preciso entender bem os passos abaixo.

**01. Instalação das dependências.**

Para compilar o código fonte do postgres é necessário instalar algumas dependências. Instale os pacotes abaixo.
```bash
yum install epel-release -y
yum clean metadata
yum install bison bison-devel readline-devel zlib-devel openssl-devel flex gcc make systemd-devel -y
```

Nesse tópico consultei a documentação oficial do postgres, no tópico [16.2. Requirements](https://www.postgresql.org/docs/12/install-requirements.html).

Consultei também a wiki do PostgreSQL no tópico [Compile and Install from source code](https://wiki.postgresql.org/wiki/Compile_and_Install_from_source_code).

**02. Obtendo e compilando o código fonte.**

Baixe o código fonte do postgres a partir do [site oficial](https://www.postgresql.org/ftp/source/). Irei baixar a versão **12.4**.
```bash
cd /tmp
wget https://ftp.postgresql.org/pub/source/v12.4/postgresql-12.4.tar.gz
```

Agora descompacte o arquivo ```postgresql-12.4.tar.gz``` enviando para o diretório ```/usr/src```.
```bash
tar zxvf postgresql-12.4.tar.gz -C /usr/src
```

Para compilar, acesse o diretório ```/usr/src/postgresql-12.4``` e execute os comandos de compilação.

O ```configure``` irá validar se todos os requisitos foram atendidos.
```bash
bash configure --prefix=/usr/local/pgsql/12.4 --with-systemd --with-openssl
```

O ```make``` irá de fato compilar o código fonte.
```bash
make
```
> Esse passo pode demorar bastante, pois por padrão o make utilizará apenas um core de processamento. Para passar mais CPUs ao make utilize a flag ```-j``` com a quantidade deseja. Exemplo: ```make -j 2```.
> Para saber quantos cores de processamente o seu servidor possui, basta executar o comando ```nproc```.

O ```make install``` irá instalar o resultado do make, ou seja, criar os subdiretórios e copiar os arquivos para os seus respectivos lugares.
```bash
make install
```

E o ```make install-docs``` apenas instala a documentação, permitindo, por exemplo, o uso do ```man psql``` e estudar pelo terminal mesmo. É um passo opcional.
```bash
make install-docs
```

Nesse tópico consultei a documentação oficial do postgres, no tópico [16.4. Installation Procedure](https://www.postgresql.org/docs/12/install-procedure.html).

Para entender melhor como funciona a instalação de um software a partir do código fonte consultei o artigo [install-software-from-source-code](https://itsfoss.com/install-software-from-source-code/), pois explica bem como funciona.

**03. Adicionando os binários do postgres no PATH do sistema operacional.**

Existe mais de uma forma de adicionar os binários no PATH do sistema operacional, mas prefiro fazer o seguinte.

Crie o arquivo ```/etc/profile.d/postgres.sh``` e insira o conteúdo abaixo.
```
PATH="$PATH:/usr/local/pgsql/12.4/bin"
export PATH

MANPATH="/usr/local/pgsql/12.4/share/man"
export MANPATH
```

Para inicializar a variável imediatamente sem precisar reiniciar a sessão no bash execute:
```bash
source /etc/profile.d/postgres.sh
```

Se você consultar o valor da variável $PATH verá algo semelhante com o resultado abaixo.
```
/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/usr/local/pgsql/12.4/bin:/home/vagrant/.local/bin:/home/vagrant/bin
```

Para validar, execute ```psql --version```.
```
psql (PostgreSQL) 12.4
```
> Nos casos onde você precisa ter mais de uma versão do PostgreSQL instalada, não recomendo executar esse passo, melhor executar os binários passando o PATH absoluto, porque dessa forma evita confusões relacionadas a versão.

Nesse tópico consultei a documentação oficial do postgres, no tópico [16.5.1. Shared Libraries](https://www.postgresql.org/docs/12/install-post.html#id-1.6.3.9.2).

**04. Criando o usuário postgres.**

Crie o usuário ```postgres``` na qual será o responsável pela execução dos processos e dono de todos os arquivos relacionados ao postgres.
```bash
useradd --user-group --no-create-home --shell /bin/bash postgres
```

Nesse tópico consultei a documentação oficial do postgres, no tópico [18.1. The PostgreSQL User Account](https://www.postgresql.org/docs/12/postgres-user.html).

**05. Inicializando o cluster de banco de dados.**

No PostgreSQL o conceito de ```cluster``` define uma coleção de banco de dados gerenciado por uma única instância de um servidor.

Seguindo uma boa prática, aqui irei inicializar o cluster numa partição secundária, provisionada exclusivamente para receber os dados do postgres.

Outra boa prática a seguir é não inicializar o cluster diretamente no ponto de montagem, mas criar um subdiretório, ou seja, não inicializar o cluster diretamente no ```/data```, mas no ```/data/pgdata```.

Essa prática evita problemas com permissões e protege os dados contra possíveis falhas no ponto de montagem.

O ```Vagrantfile``` disponível na raíz deste repositório provisiona automaticamente um segundo disco e o monta no ```/data```. Ficou fácil, você precisa agora criar o subdiretório chamado ```pgdata```.
```bash
mkdir -p /data/pgdta
```

Mude o dono do diretório para o usuário ```postgres```.
```bash
chown postgres:postgres /data/pgdata
```

Ajuste a permissão para ```0700```.
```bash
chmod 0700 /data/pgdata
```

Agora inicialize o cluster com o comando abaixo.
```bash
su --command "/usr/local/pgsql/12.4/bin/initdb --encoding UTF-8 --locale pt_BR.UTF-8 --pgdata /data/pgdata" --shell /bin/bash postgres
```

O resultado será semelhante ao mostrado abaixo.
```
The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.

The database cluster will be initialized with locale "pt_BR.UTF-8".
The default text search configuration will be set to "portuguese".

Data page checksums are disabled.

fixing permissions on existing directory /data/pgdata ... ok
creating subdirectories ... ok
selecting dynamic shared memory implementation ... posix
selecting default max_connections ... 100
selecting default shared_buffers ... 128MB
selecting default time zone ... America/Recife
creating configuration files ... ok
running bootstrap script ... ok
performing post-bootstrap initialization ... ok
syncing data to disk ... ok

initdb: warning: enabling "trust" authentication for local connections
You can change this by editing pg_hba.conf or using the option -A, or
--auth-local and --auth-host, the next time you run initdb.

Success. You can now start the database server using:

    /usr/local/pgsql/12.4/bin/pg_ctl -D /data/pgdata -l logfile start
```

Listando a estrutura de diretórios criada temos o seguinte:
```
[root@primary ~]# ls -l /data/pgdata
total 52
drwx------. 5 postgres postgres    41 Sep 12 23:03 base
drwx------. 2 postgres postgres  4096 Sep 12 23:03 global
drwx------. 2 postgres postgres     6 Sep 12 23:02 pg_commit_ts
drwx------. 2 postgres postgres     6 Sep 12 23:02 pg_dynshmem
-rw-------. 1 postgres postgres  4760 Sep 12 23:02 pg_hba.conf
-rw-------. 1 postgres postgres  1636 Sep 12 23:02 pg_ident.conf
drwx------. 4 postgres postgres    68 Sep 12 23:03 pg_logical
drwx------. 4 postgres postgres    36 Sep 12 23:02 pg_multixact
drwx------. 2 postgres postgres    18 Sep 12 23:02 pg_notify
drwx------. 2 postgres postgres     6 Sep 12 23:02 pg_replslot
drwx------. 2 postgres postgres     6 Sep 12 23:02 pg_serial
drwx------. 2 postgres postgres     6 Sep 12 23:02 pg_snapshots
drwx------. 2 postgres postgres     6 Sep 12 23:02 pg_stat
drwx------. 2 postgres postgres     6 Sep 12 23:02 pg_stat_tmp
drwx------. 2 postgres postgres    18 Sep 12 23:02 pg_subtrans
drwx------. 2 postgres postgres     6 Sep 12 23:02 pg_tblspc
drwx------. 2 postgres postgres     6 Sep 12 23:02 pg_twophase
-rw-------. 1 postgres postgres     3 Sep 12 23:02 PG_VERSION
drwx------. 3 postgres postgres    60 Sep 12 23:02 pg_wal
drwx------. 2 postgres postgres    18 Sep 12 23:02 pg_xact
-rw-------. 1 postgres postgres    88 Sep 12 23:02 postgresql.auto.conf
-rw-------. 1 postgres postgres 26640 Sep 12 23:02 postgresql.conf
```

Nesse tópico consultei a documentação oficial do postgres, no tópico [18.2. Creating a Database Cluster](https://www.postgresql.org/docs/12/creating-cluster.html).