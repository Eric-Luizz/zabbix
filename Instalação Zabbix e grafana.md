# Guia de Instalação: Zabbix e Grafana no Ubuntu Server

## ⚠️ Importante

Ao utilizar esse tutorial, **execute cada comando individualmente** para garantir a instalação correta. **Não é recomendável executar todos os comandos de uma vez**, pois isso pode causar problemas, especialmente na criação de tabelas do MySQL.

---

## 1. Tornar-se Usuário Root

Para iniciar uma nova sessão de shell com privilégios de root, utilize o seguinte comando:

```bash
sudo -s
```

**Importante:** Utilize a senha padrão do usuário `user`, que foi utilizada na criação de todos os usuários, conforme indicado pelo supervisor da TI.

---

## 2. Atualizar o Sistema e Instalar Pacotes Necessários

Execute os seguintes comandos para atualizar o sistema e instalar pacotes essenciais:

```bash
apt update && apt upgrade -y
apt install net-tools -y
apt install mysql-server -y
```

---

## 3. Instalar Repositório Zabbix

Baixe e instale o repositório do Zabbix:

```bash
cd /root
wget https://repo.zabbix.com/zabbix/7.2/release/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.2+ubuntu24.04_all.deb 
dpkg -i zabbix-release_latest_7.2+ubuntu24.04_all.deb 
apt update
```

---

## 4. Instalar Servidor Zabbix, Frontend e Agente

Execute o seguinte comando para instalar os pacotes principais do Zabbix:

```bash
apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent -y
```

---

## 5. Criar Database no MySQL

Acesse o MySQL como root:

```bash
mysql -uroot -p
```

**Importante:** Utilize a senha padrão do usuário `user`, que foi utilizada na criação de todos os usuários, conforme indicado pelo supervisor da TI.

Digite a senha e crie o banco de dados e o usuário do Zabbix:

```sql
create database zabbix character set utf8mb4 collate utf8mb4_bin;
create user 'zabbix'@'localhost' identified by 'password';
grant all privileges on zabbix.* to 'zabbix'@'localhost';
set global log_bin_trust_function_creators = 1;
quit;
```

---

## 6. Importar Esquema e Dados Iniciais

Importe o esquema do banco de dados do Zabbix:

```bash
zcat /usr/share/zabbix/sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
```

Após inserir a senha corretamente, o processo ocorrerá em segundo plano.

---

## 7. Desabilitar Opção `log_bin_trust_function_creators`

Após a importação, desabilite a opção:

```bash
mysql -uroot -p
```

Digite a senha e execute:

```sql
set global log_bin_trust_function_creators = 0;
quit;
```

---

## 8. Configurar Banco de Dados no Zabbix

Edite o arquivo de configuração do Zabbix:

```bash
nano /etc/zabbix/zabbix_server.conf
```

Adicione ou edite a seguinte linha:

```bash
dbpassword=password
```

Salve e saia (`CTRL + S`, `CTRL + X`, `ENTER`).

---

## 9. Iniciar Serviços do Zabbix e Apache

Inicie os serviços e configure para iniciar automaticamente:

```bash
systemctl restart zabbix-server zabbix-agent apache2
systemctl enable zabbix-server zabbix-agent apache2
systemctl status zabbix-server
```

---

## 10. Acessar a Interface Web do Zabbix

Abra seu navegador e acesse:

```
http://host/zabbix
```

Substitua `host` pelo IP ou nome do seu servidor.

---

## Finalizado!

Agora seu Zabbix está instalado e pronto para ser configurado! 🚀

































# Instalação do Grafana 

## ⚠️ Importante

Antes de começar, certifique-se de que está **logado como root** utilizando o seguinte comando:

```bash
sudo -s
```

**Nota:** Utilize a senha padrão do usuário `user`, que foi utilizada na criação de todos os usuários, conforme indicado pelo supervisor da TI.

---

## 1. Adicionar Repositório Grafana APT

Para instalar o Grafana, primeiro precisamos adicionar seu repositório oficial ao sistema.

### Instalar pacotes necessários:

```bash
apt install -y gnupg2 curl software-properties-common
```

- **`gnupg2`**: Gerencia chaves de autenticação para a instalação de pacotes confiáveis.
- **`curl`**: Permite baixar arquivos de servidores web.
- **`software-properties-common`**: Fornece suporte a repositórios adicionais no Ubuntu.

### Adicionar chave GPG do repositório Grafana:

```bash
curl -fsSL https://packages.grafana.com/gpg.key | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/grafana.gpg
```

- **`curl -fsSL URL`**: Baixa a chave GPG do repositório oficial do Grafana.
- **`gpg --dearmor`**: Converte a chave para um formato compatível com o APT.
- **`-o /etc/apt/trusted.gpg.d/grafana.gpg`**: Armazena a chave no local correto.

### Adicionar repositório do Grafana ao sistema:

```bash
add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
```

- Esse comando adiciona o repositório do Grafana à lista de fontes do APT.

**Se o comando `add-apt-repository` não funcionar**, verifique se o pacote `software-properties-common` está instalado.

---

## 2. Instalar Grafana no Ubuntu

Agora, atualize os repositórios do APT e instale o Grafana:

```bash
apt update
apt -y install grafana
```

- **`apt update`**: Atualiza a lista de pacotes disponíveis.
- **`apt -y install grafana`**: Instala o Grafana automaticamente, sem perguntar confirmação.

---

## 3. Iniciar o Serviço Grafana

Agora que o Grafana foi instalado, inicie o serviço e configure-o para iniciar automaticamente com o sistema:

```bash
systemctl enable --now grafana-server
```

- **`systemctl enable grafana-server`**: Configura o serviço para iniciar automaticamente.
- **`--now`**: Inicia o serviço imediatamente após habilitá-lo.

---

## 4. Instalar o Plugin Zabbix para Grafana

Se você deseja usar o **Zabbix no Grafana**, instale o plugin oficial:

```bash
grafana-cli plugins install alexanderzobnin-zabbix-app
```

- **`grafana-cli plugins install [plugin]`**: Instala um plugin específico no Grafana.

Agora, reinicie o serviço do Grafana para carregar o plugin:

```bash
systemctl restart grafana-server
systemctl status grafana-server
```

- **`systemctl restart grafana-server`**: Reinicia o serviço para aplicar alterações.
- **`systemctl status grafana-server`**: Verifica se o serviço está rodando corretamente.

---

## 5. Acessar o Painel do Grafana

Agora você pode acessar o painel do Grafana pelo navegador, usando o IP do servidor e a porta `3000`:

```
http://SEU_IP:3000
```

**Credenciais padrão:**

```
Username: admin
Password: admin
```

Após o primeiro login, o sistema pedirá que você altere a senha.

---

## Finalizado!

Agora seu Grafana está instalado e pronto para uso! 🚀














