# Como atualizar

## Preparação 

## Instalar PHP na versão 7.0 ou >;

## Instalar MySql na versão 4.1 ou >;

### Clonar o repositorio pra sua máquina: 
#### via HTTPS: https://gitlab.ifpb.edu.br/celulas-cooperacao/editora-v3.git

#### via SSH: git@gitlab.ifpb.edu.br:celulas-cooperacao/editora-v3.git

#### Configurar o ambiente e executar versão desatualizada:

- Arquivos docker-compose e dockerfile atualizados estão aqui no repositório

- Executar o docker-compose up -d na raiz do projeto.

- Faça uma cópia do config.TEMPLATE.inc.php e apague o .TEMPLATE do nome do arquivo

- Acessar o arquivo config.inc.php

- Alterar "files_dir" para o caminho da pasta files

- Na pasta cache, criar as seguintes subpastas: t_cache, t_compile_, t_config, _db.

- Alterar General Settings de installed = On para installed = Off

- No terminal acessar a pasta tools e executar o seguinte comando para atualizar as tabelas do banco  de dados com a atualização do OMP PKP:
php upgrade.php upgrade

- Execute o comando: php -S localhost:8000

### Atualização

### Setar variáveis a fim de simplificar comandos no terminal: 

| VARIABLE        | Default             | Description                          |
| --------------- | ------------------- | ------------------------------------ |
| WEB_USER        | www-data            | Webserver user                       |
| WEB_GROUP       | www-data            | Webserver user’s group               |
| OMP_ROOT_PATH   | /var/www            | OMS root folder                      |
| OMP_WEB_PATH    | /var/www/html       | OMP web root folder                  |
| OMP_DB_HOST     | db                  | Database host’s name                 |
| OMP_DB_USER     | omp                 | Database user                        |
| OMP_DB_PASSWORD | omp              | Database password                    |
| OMP_DB_NAME     | omp                 | Database name                        |
| OMP_BACKUP_PATH | /srv/backup/ojs     | Folder to store your backups         |
| OMP_VERSION     | omp-3.1.1-1         | Version as in the OMP filename       |
| DATE            | YYYYMMDD-HH:MM:SS   | The current system date and time     |


### Rode a seguinte linha de código abaixo para setar as variáveis trocando os valores para os valores reais:

```bash

$ WEB_USER="www-data" && \
WEB_GROUP="www-data" && \
OMP_ROOT_PATH="/var/www" && \
OMP_WEB_PATH="/var/www/html" && \
OMP_DB_HOST="omp" && \
OMP_DB_USER="omp" && \
OMP_DB_PASSWORD="omp" && \
OMP_DB_NAME="omp" && \
OMP_BACKUP_PATH="/srv/backup/omp" && \
OMP_VERSION="omp-3.1.1-1" && \
OMP_PUBLIC_PATH="$OMP_WEB_PATH/public" && \
OMP_PRIVATE_PATH="$OMP_ROOT_PATH/files" && \
DATE=$(date "+%Y%m%d-%H:%M:%S")

```

### Criar Backup

- Backup do banco: 

```bash
$ mysqldump --host="$OMP_DB_HOST" -u $OMP_DB_USER -p $OMP_DB_PASSWORD $OMP_DB_NAME --result-file="$OMP_BACKUP_PATH/backupDB-$DATE.sql"
```

- Backup dos arquivos privados:

```bash
$ tar cvzf "$OMP_BACKUP_PATH/private-$DATE.tgz" "$OMP_PRIVATE_PATH"
```

- Backup dos arquivos publicos:

```bash
$ tar cvzf "$OMP_BACKUP_PATH/ompfiles-$DATE.tgz" "$OMP_WEB_PATH"
```

### Rodar scripts de atualização

- Baixar o pacote de atualização:

```bash
$ cd "$OMP_ROOT_PATH"
$ wget "https://pkp.sfu.ca/omp/download/$OMP_VERSION.tar.gz"
```
### Verificar requisitos do sistema

Verifique o arquivo README do tar.gz baixado e certifique-se de que seu sistema atenda aos seguintes requisitos.

- Versão Apache
- Versão MySQL ou PostgreSQL
- Versão PHP
- Bibliotecas de servidor e requisitos de módulo

Além disso, você desejará realizar as seguintes verificações: 

- Ajuste os tempos limite do PHP e os limites de memória para garantir que o processo de atualização seja concluído com êxito.
- Verifique as bibliotecas do servidor e os requisitos do módulo para quaisquer plug-ins que você adicionou (geralmente podem ser encontrados no arquivo README do plug-in).

### Instalar pacote de atualização

- Backup da aplicação

```bash
$ mv "$OMP_WEB_PATH" "$OMP_BACKUP_PATH"
```

- Extraia o pacote de atualização

```bash
$ mkdir "$OMP_WEB_PATH"
$ tar --strip-components=1 -xvzf "$OMP_VERSION.tar.gz" -C "$OMP_WEB_PATH"
```

- Restaure o arquivo config.inc.php

```bash
$ cp "$OMP_BACKUP_PATH/html/config.inc.php" "$OMP_WEB_PATH"
```

- Execute o seguinte comando para comparar seu arquivo de configuração com o modelo da nova versão. Adicione ou remova quaisquer opções de configuração conforme necessário.

```bash
$ diff "$OMP_WEB_PATH/config.inc.php" "$OMP_WEB_PATH/config.TEMPLATE.inc.php"
```

- Restaure os arquivos publicos

```bash
$ cp -r "$OMP_BACKUP_PATH/html/public/*" "$OMP_PUBLIC_PATH"
```

- Defina as permissões para os novos arquivos.

```bash
$ sudo chown -R $WEB_USER:$WEB_GROUP "$OMP_PUBLIC_PATH" "$OMP_WEB_PATH/cache/"

$ sudo chown -R $WEB_USER:$WEB_GROUP "$OMP_WEB_PATH/plugins/"
```

### Execute a atualização

```bash
$ php tools/upgrade.php check
```

### Finalmente, quando estiver pronto, execute o script de atualização, que pode levar várias horas para ser concluído. Especifique uma quantidade explícita de memória na linha de comando. Se o processo de atualização ficar sem memória, ele falhará e precisará começar novamente.

```bash
$ php -d memory_limit=2048M tools/upgrade.php upgrade
```