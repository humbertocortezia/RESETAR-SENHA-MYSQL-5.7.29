# RESETAR-SENHA-MYSQL-5.7.29
TUTORIAL PARA RESETAR A SENHA DO MYSQL SERVE NO LINUX VERSÃO 5.7.29


Reset root password no MySQL 5.7.*
Publicado em 17 de setembro de 2016
Estava muito tempo sem postar, mas hoje eu passei uma dor de cabeça muito grande até resolver este problema, então eu tinha que postar essa solução.

Eu estava com o Ubuntu 14.10 e resolvi atualizar ele para a última versão hoje (17/09/2016), Ubuntu 16.04.

Tudo estava correndo bem, mesmo tendo que reinstalar/reconfigurar algumas bibliotecas/aplicativos.

O último passo da verificação foi tentar rodar meus aplicativos web que usam MySQL. A surpresa inicial foi que o MySQL havia sido desinstalado. Verifiquei se os arquivos de dados, com todos os meus bancos de dados, estavam salvos. Graças a minha sorte tudo estava a salvo.

Primeiramente eu apenas fiz a instalação dos pacotes do MySQL, onde a versão mais nova era a 5.7:

$ sudo apt-get install mysql-server-5.7 mysql-client-5.7

Após finalizar a instalação, eu tentei logar. O problema ocorreu justamente nesse momento, quando eu tentei logar no MySQL:

$ mysql -u root

ERROR 1045: Access denied for user: 'root'@'localhost'

Antes de fazer a atualização do Ubuntu meu MySQL não tinha senha de root. Ela era vazia. Eu apenas fazia mysql -u root e o acesso seguia com sucesso.

Então o que eu deveria fazer? Bem, provavelmente a senha de root foi alterada ou qualquer outro problema que eu não sei o que foi. A solução então era alterar a senha de root. Eu passe mais de 5 horas tentando alterar a senha, lendo vários sites, blogs, jornais, revistas, contato extra-terrestre, …, inclusive o site oficial do MySQL, e tudo repetia as mesmas coisas. Até que uma luz veio e me levou até uma postagem no StackOverflow com um erro não tão igual, mas com a mesma solução.

Todos os sites diziam a mesma coisa, mas eu acabei encontrando o que faltava. Segue abaixo:

# 1. Primeiro devemos parar o serviço do MySQL:

# $ sudo service mysql stop

2. Em seguida iniciamos o MySQL em modo seguro, sem as opções de permissões. Isso vai servir para você alterar a senha. Mas o segredo vem depois:

# $ sudo mysqld_safe --skip-grant-tables &

Poderá ocorrer o seguinte erro:

mysqld_safe Logging to '/var/log/mysql/error.log'.
mysqld_safe Logging to '/var/log/mysql/error.log'.
mysqld_safe Directory '/var/run/mysqld' for UNIX socket file don't exists.

Caso ocorra, execute o seguinte comando para criar o diretório onde ficará o socket do mysql:

# $ sudo mkdir -p /var/run/mysqld
# $ sudo chown mysql:mysql /var/run/mysqld

3. Após iniciar o servidor você vai acessar o cliente MySQL com usuário root, sem senha:

# $ mysql -u root

4. Agora vamos alterar a senha do usuário root.

Aqui é que está o SEGREDO. No MySQL 5.7 foi alterado algumas coisas no banco de dados de sistema mysql. A tabela user não possui mais a coluna password, o nome agora se chama authentication_string. Então faça a alteração da senha da seguinte maneira:

# mysql> use mysql;
# mysql> update user set authentication_string=PASSWORD("SuaSenha") where User='root';

Bom, a maioria dos sites paravam aí, o que não resolvia o problema. O que faltava era um pequeno detalhes, devemos alterar o campo plugin setando o mecanismo padrão de verificação do password. Então segue abaixo:

# mysql> update user set plugin="mysql_native_password";
# mysql> flush privileges;
# mysql> quit;

Feito isso, a senha do root foi alterada e está pronto para ser acessado.

5. Dê um stop no MySQL e depois inicie normalmente o serviço:

# $ sudo mysqladmin shutdown # essa foi a forma mais fácil que eu achei para matar o mysqld_safe

Caso não dê certo, você também pode tentar matar o MySQL localizando o id do processo e dando um kill nele:

# $ ps -ef | grep mysql
# $ sudo kill -9 [id dos processos que você achou no comando anterior] 
Após parar o serviço, inicie ele normalmente:

# $ sudo service mysql start
Pronto, agora é só conectar e correr para o abraço:

# $ mysql -u root -p

Então era isso galera. Queria apenas registrar este detalhe para ajudar mais pessoas, já que a maioria dos sites possui informações bem antigas.

Qualquer dúvida é só perguntar nos comentários.

Abraço!

# FONTE https://www.leandrosales.com.br/2016/09/17/reset-root-password-no-mysql-5-7/

ESSE CARA ME SALVOU!!!!
