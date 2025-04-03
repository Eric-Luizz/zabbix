# Guia de Instalação: Zabbix e Grafana e Ubuntu Server

## Iniciar VMware com iso  Ubuntu Server 24.04 LTS

![Screenshot_1](https://github.com/user-attachments/assets/da1ee768-62bd-46dc-93df-8a00ae8d8282)


Seguir a Instalação inicial do Ubuntu server conforme as imagem abaixo :


![Screenshot_2](https://github.com/user-attachments/assets/3102621a-dcb8-42bb-83b0-4444c63c9f9d)

Concluído
![Screenshot_3](https://github.com/user-attachments/assets/f054e877-f2bd-4228-8e6d-439bb87d4702)

Concluído

Aqui deve configurar manualmente o ip do servidor ou DHCP liberar algum ip disponível. Deve optar conforme indicado pelo supervisor da TI.
![Screenshot_4](https://github.com/user-attachments/assets/5ab7c62c-dd34-451c-84cc-8dc1c99d1c89)


Segue a instalaçao conforme as imagem abaixo
Deve aguardar a conclusão dos teste de conexão de mirror antes de avançar

![Screenshot_5](https://github.com/user-attachments/assets/0cba6a2d-abb7-40e6-af45-bf14815df7cc)


Se não precisar fazer partiçoes de disco pode avançar para proxima etapa.

![Screenshot_6](https://github.com/user-attachments/assets/7b0711df-b001-440c-a87d-c90978c5c989)

Concluído

Configure essa etapa conforme a imagem abaixo e a senha deve ser a padrão do usuário `user`, que foi utilizada na criação de todos os usuários ou configure conforme indicado pelo supervisor da TI.
![Screenshot_7](https://github.com/user-attachments/assets/d4ec0ec9-fde1-45f3-a7ee-c0bbcb693b0f)

Concluído
![Screenshot_8](https://github.com/user-attachments/assets/8245ee66-601c-49a5-904c-47b14ffbbb28)

Concluído
![Screenshot_9](https://github.com/user-attachments/assets/b58d9d10-8bd0-4d6c-b73a-363493427a63)

Concluído

Não é necessário habilitar nada nessa etapa e avance próxima etapa

![Screenshot_10](https://github.com/user-attachments/assets/fc7390a1-33ca-49a8-9272-ba984680fee0)

Aguarde a instalação e dê Reboot Now
![Screenshot_11](https://github.com/user-attachments/assets/b7fed966-dde2-4d6d-8fa1-0804be9f0bd3)

Ignore o erro abaixo e tecle ENTER.


![Screenshot_12](https://github.com/user-attachments/assets/af380b18-0d5a-4387-9df6-62179d68eb46)


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


![Screenshot_13](https://github.com/user-attachments/assets/b7b6ab12-078f-4ac2-a41c-c96276aacc95) 

![Screenshot_14](https://github.com/user-attachments/assets/0e12f934-7dbb-45ca-bb14-a9bcc2722b6a)

![Screenshot_15](https://github.com/user-attachments/assets/ae1f46be-5dde-42e1-b857-2cbb836e95f4)

## 3. Instalar Repositório Zabbix

Baixe e instale o repositório do Zabbix:

```bash
cd /root
wget https://repo.zabbix.com/zabbix/7.2/release/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.2+ubuntu24.04_all.deb 
dpkg -i zabbix-release_latest_7.2+ubuntu24.04_all.deb 
apt update
```

---
![Screenshot_16](https://github.com/user-attachments/assets/56c663dc-da70-4466-900e-c0714029672d)

![Screenshot_17](https://github.com/user-attachments/assets/5ede79b5-a21d-46ca-af5d-471651d457da)

## 4. Instalar Servidor Zabbix, Frontend e Agente

Execute o seguinte comando para instalar os pacotes principais do Zabbix:

```bash
apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent -y
```

---
![Screenshot_18](https://github.com/user-attachments/assets/f8657614-0449-434d-ab6e-ec5cbd025dd1)
## 5. Criar Database no MySQL

Acesse o MySQL como root:

```bash
mysql -uroot -p
```
![Screenshot_19](https://github.com/user-attachments/assets/1abd5423-be33-4471-a6c4-e1aa2598d3ba)


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
![Screenshot_20](https://github.com/user-attachments/assets/58691d78-b270-4363-9450-626ab8a35264)

## 6. Importar Esquema e Dados Iniciais

Importe o esquema do banco de dados do Zabbix:

```bash
zcat /usr/share/zabbix/sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
```

Após inserir a senha corretamente, o processo ocorrerá em segundo plano.

Aguarde!

![Screenshot_22](https://github.com/user-attachments/assets/8103f4ad-a71d-447e-8cce-b79c49ca13db)

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
![Screenshot_23](https://github.com/user-attachments/assets/d3e8dc21-c726-4758-84f9-29e89a1634e7)

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
![Screenshot_24](https://github.com/user-attachments/assets/14a6cc62-e886-417a-82b3-3ddd2994b41b)

## 10. Acessar a Interface Web do Zabbix

Abra seu navegador e acesse:

```
http://host/zabbix
```
![Screenshot_25](https://github.com/user-attachments/assets/4925e236-91d5-40ae-a768-b067a23c87dd)

Substitua `host` pelo IP ou nome do seu servidor.
```
Coloque Idioma Português (pt_PT)
```
---

Nessa etapa verifique e se necessário faça ajuste para ficar igual na imagem e abaixo e na opção senha coloque mesma senha do zabbix'@'localhost' identified by 'password';

![Screenshot_26](https://github.com/user-attachments/assets/3386081b-1c3a-41a9-88f4-fe70fe62dd60)


Ajuste para fica igual a imagem abaixo:

![Screenshot_27](https://github.com/user-attachments/assets/01b762ac-8fbd-4d8c-abd8-b91d4fb34eb0)


## Finalizado!

Agora seu Zabbix está instalado e pronto para ser configurado! 🚀


![Screenshot_28](https://github.com/user-attachments/assets/30b692f2-838e-49a0-94d2-e7ace2812ca3)



![Screenshot_29](https://github.com/user-attachments/assets/ae47ef98-f718-4ac1-9599-f1785cf51323)

**Credenciais padrão:**

```
Username: Admin
Password: zabbix
```

Não esqueça de alterar a senha do Admin para a senha do Administrador que foi utilizada na criação de todos os usuários, conforme indicado pelo supervisor da TI.


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

![Screenshot_30](https://github.com/user-attachments/assets/dee7000c-d038-4f5e-9a03-effe5251fb8a)

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


![Screenshot_31](https://github.com/user-attachments/assets/2d95fe6e-5dd2-4db0-9e83-3a70cbbe4abc)

  


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
![Screenshot_32](https://github.com/user-attachments/assets/64f2de2c-7658-4a95-8150-b294a29e87b8)

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

![Screenshot_34](https://github.com/user-attachments/assets/2e105e02-2782-4189-beb7-409d32bad08e)

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

![image](https://github.com/user-attachments/assets/75de9e18-97c0-4239-9cfc-f329e2d89ff6)

**Credenciais padrão:**

```
Username: admin
Password: admin
```


Após o primeiro login, o sistema pedirá que você altere a senha para senha de administrador que foi utilizada na criação de todos os usuários, conforme indicado pelo supervisor da TI.


![Screenshot_33](https://github.com/user-attachments/assets/5fc5e7fb-27f4-4dfa-a294-53db00bfd82a)

---



### Para habilitar plugin do zabbix no grafana segue a imagens abaixo 


![Screenshot_35](https://github.com/user-attachments/assets/2b215117-160c-4ce5-b675-e280cce7a499)

![Screenshot_36](https://github.com/user-attachments/assets/fcb16016-7167-4ce9-8e1e-6e1aa4fe1d6e)

![Screenshot_37](https://github.com/user-attachments/assets/d6732761-7a35-4b3f-af0f-eba57b09e650)

![Screenshot_40](https://github.com/user-attachments/assets/98a77b73-4ad8-4cd3-8fdf-47f15132ff5b)

### Após dar enable na pluglin do zabbix, deve adicionar do Data sources de acordo com a imagem abaixo

![Screenshot_41](https://github.com/user-attachments/assets/60c40cc4-f495-4f7d-bdf2-db27781a3b07)

![Screenshot_42](https://github.com/user-attachments/assets/5149f12d-1cd5-4f4f-847c-35e3920ef4e5)


Na opção Connection coloque na URL ```http://localhost/zabbix/api_jsonrpc.php```
![Screenshot_43](https://github.com/user-attachments/assets/386c5314-8d6c-440d-8650-3ec228b44a8f)

![Screenshot_44](https://github.com/user-attachments/assets/330df0d0-fc5c-47e0-bcf6-cfd9a68aa1fc)

![Screenshot_45](https://github.com/user-attachments/assets/2a7e9da3-61dc-4ed0-8316-01b7a697c122)

## Finalizado!

Agora seu Grafana está instalado e pronto para uso! 🚀















