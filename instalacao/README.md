## Instalação e Configuração do PostgreSQL

Aqui será descrito como faço para instalar o PostreSQL a partir do código fonte, ou seja, compilado. Não irei abordar a instalação por pacotes do repositório, porque prefiro ter total controle sobre a instalação.

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
>> Esse passo pode demorar bastante, pois por padrão o make utilizará apenas um core de processamente. Para passar mais CPUs para o make utilize a flag ```-j``` com a quantidade deseja. Exemplo: ```make -j 2```.
>> Para saber quantos cores de processamente o seu servidore possui, basta executar o comando ```nproc```.

E o ```make install``` irá instalar o resultado do make, ou seja, criar os subdiretórios e copiar os arquivos para os seus respectivos lugares.
```bash
make install
```

Nesse tópico consultei a documentação oficial do postgres, no tópico [16.4. Installation Procedure](https://www.postgresql.org/docs/12/install-procedure.html).

Para entender melhor como funciona a instalação de um software a partir do código fonte consultei o artigo [install-software-from-source-code](https://itsfoss.com/install-software-from-source-code/), pois explica bem como funciona.

**03. Adicionando os binários do postgres no PATH do sistema operacional.**

Existe mais de uma forma de adicionar os binários no PATH do sistema operacional, mas prefiro fazer o seguinte.

Crie o arquivo ```/etc/profile.d/postgres.sh``` e insira o conteúdo abaixo.
```
PATH="$PATH:/usr/local/pgsql/12.4/bin"
export PATH
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
>> Nos casos onde você precisa ter mais de uma versão do PostgreSQL instalada, não recomendo executar esse passo, melhor executar os binários passando o PATH absoluto, porque dessa forma evita confusões relacionadas a versão.

Nesse tópico consultei a documentação oficial do postgres, no tópico [16.5.1. Shared Libraries](https://www.postgresql.org/docs/12/install-post.html#id-1.6.3.9.2).

**04. Criando o usuário postgres.**

Crie o usuário ```postgres``` na qual será o responsável pela execução dos processos e dono de todos os arquivos relacionados ao postgres.
```bash
useradd --user-group --no-create-home --shell /bin/bash postgres
```

Nesse tópico consultei a documentação oficial do postgres, no tópico [18.1. The PostgreSQL User Account](https://www.postgresql.org/docs/12/postgres-user.html).