<!-- # Instalando Zabbix Server v7.2 no Ubuntu 24.04 -->


<!-----/ Bloco de Código /-------- 
<pre><code class="language-bash">
echo "Hello World!"
</code></pre>
<br>
---------------------------------->

![banner](https://raw.githubusercontent.com/wevertonjlima/blog-assets/main/posts/zabbix-v7.2_ubuntu-v24.04/banner.jpg)

## INTRODUÇÃO

Este guia apresenta a instalação do **Zabbix Server 7.2** no Ubuntu Server 24.04 LTS.  
O conteúdo está dividido em quatro etapas principais:

* Pré-requisitos
* Instalação
* Configuração Web
* Conclusão

<br><br>


## 1 PRÉ-REQUISITOS
Antes de começar, é importante ter...

> - Ubuntu Server 24.04 LTS atualizado.
> - Acesso **`root`** ou usuário com privilégios **`sudo`** (local ou via ssh).
> - Conexão com a internet.

<br>

Torne-se **usuário root** e inicie uma nova sessão de shell.
<pre><code class="language-bash">
sudo -i
</code></pre>
<br>

Atualize o sistema e configure o fuso horário de acordo com a sua localidade.
<pre><code class="language-bash">
apt update -y
apt upgrade -y

timedatectl set-timezone America/Maceio
systemctl restart systemd-timedated
</code></pre>
<br>



## 2 INSTALAÇÃO

### **2.1 | Instalação do Banco de Dados**

Instale o banco de dados PostgreSQL.
<pre><code class="language-bash">
apt install postgresql -y
</code></pre>
<br>

Habilite e Inicialize o serviço do PostgreSQL.
<pre><code class="language-bash">
systemctl enable  postgresql --now
systemctl restart postgresql
systemctl status  postgresql
</code></pre>
<br>

Vamos criar a conta de usuário **`zabbix`** no PostgreSQL;  
o comando irá solicitar uma senha.  
> *Iremos utilizar neste exemplo, a senha* `H3lloW0rld!`  
> *utilize uma **senha diferente** em seus próprios projetos!*
<pre><code class="language-bash">
sudo -u postgres createuser --pwprompt zabbix
sudo -u postgres createdb -O zabbix zabbix

</code></pre>
<br>


### **2.2 | Instalando o Zabbix**

Adicione o repositório oficial do Zabbix ao sistema.

<pre><code class="language-bash">
wget https://repo.zabbix.com/zabbix/7.2/release/ubuntu/pool/main/z/zabbix-release/zabbix-release\_latest\_7.2+ubuntu24.04\_all.deb
dpkg -i zabbix-release\_latest\_7.2+ubuntu24.04\_all.deb
apt update -y
</code></pre>
<br>

Instale o Zabbix Server, Frontend, PhP, Apache e Agent2.
> *A versão do PHP pode variar conforme a release;* 
> *ajuste o pacote 'php8.3-pgsql' caso sua versão seja diferente.*

<pre><code class="language-bash">
apt install zabbix-server-pgsql zabbix-frontend-php php8.3-pgsql zabbix-apache-conf zabbix-sql-scripts zabbix-agent2 -y
</code></pre>
<br>


Instale o Zabbix Agent 2 plugins, de acordo com seu banco de dados, no nosso cenário, o PostgreSQL.

<pre><code class="language-bash">
apt install zabbix-agent2-plugin-postgresql -y
</code></pre>
<br>


#### **2.3 | Preparando o database do Zabbix e seu arquivo .conf**

Importe o esquema inicial do Zabbix para dentro do PostgreSQL.
> *Quando solicitado, utilize a senha* `H3lloW0rld!`

<pre><code class="language-bash">
zcat /usr/share/zabbix/sql-scripts/postgresql/server.sql.gz | sudo -u zabbix psql zabbix
</code></pre>
<br>

A seguir, edite o arquivo de configuração do Zabbix.
<pre><code class="language-bash">
nano /etc/zabbix/zabbix-server.conf
</code></pre>
<br>

Procure por este bloco de código no arquivo, e insira a senha do database do zabbiz:
```
### Option: DBPassword
#       Database password.
#       Comment this line if no password is used.
#
# Mandatory: no
# Default:
# DBPassword=
DBPassword=H3lloW0rld!

#
```
<br>


Inicialize os serviços do Zabbix / Apache2 / Agent2.
<pre><code class="language-bash">
systemctl enable  zabbix-server zabbix-agent2 apache2 --now
systemctl restart zabbix-server zabbix-agent2 apache2 --now
</code></pre>
<br>

Pronto! A parte de configuração via terminal foi concluída.
Vamos dar continuidade a configuração do Zabbix através do console web.



## 3 CONFIGURAÇÃO WEB

Esta é a etapa mais simples do processo de instalação/configuração de um servidor Zabbix.
Todo o processo será concluído via console Web, através da seguinte URL:

http://endereço-ip-do-servidor/zabbix
<br>

> **Dica adicional** - *Caso deseje que a página do zabbix carregue, inserindo apenas o endereço IP ou o hostname do computador (meuzabbix.meudominio.labs)  realiza o procedimento.  
Ative o modo `rewrite` do apache urilizando o comando abaixo:  
```sudo a2enmod rewrite```
>
> *A seguir edite o arquivo `nano /etc/apache2/sites-enabled/000-default.conf`,  
e insira os camandos abaixo. Salve e feche o editor.*
> 
> *Reinecie o serviço do apache usando o comando `sudo ay1systemctl restart apache2`.*


 <pre><code class="language-bash">
	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/html
      
        # Adicione estas linhas para o redirecionamento do ZABBIX
        # Lembre-se de antes ativar o seguinte serviço na linha de comando
        # 
        # sudo a2enmod rewrite
        #
        # E depois reiniciar o servico do Apache:
        # sudo ay1systemctl restart apache2

        RewriteEngine On
        RewriteCond %{REQUEST_URI} ^/$
        RewriteRule ^/$ /zabbix/index.php [R=301,L] 
</code></pre>
<br>




A página de instalação inicial será exibida.  
Click em [Next Step] :  

![Instalação do Zabbix 7.2](https://raw.githubusercontent.com/wevertonjlima/blog-assets/main/posts/zabbix-v7.2_ubuntu-v24.04/zabbix-7v2-web-installation-01.png)

<br>


Um tela de verificação de pré-requisitos surgirá.  Se tudo estiver OK, continue com o processo: 

![Instalação do Zabbix 7.2](https://raw.githubusercontent.com/wevertonjlima/blog-assets/main/posts/zabbix-v7.2_ubuntu-v24.04/zabbix-7v2-web-installation-02.png)



A proxima tela irá definir alguns parametros do banco de dados. Apenas lembre-se de inserir a mesma senha que foi utiulizada no inicio da instalação:  

![Instalação do Zabbix 7.2](https://raw.githubusercontent.com/wevertonjlima/blog-assets/main/posts/zabbix-v7.2_ubuntu-v24.04/zabbix-7v2-web-installation-03.png)



Se desejar, ajuste no nome do servidor, o fuso horarios e o tema visual do zabbix (light e ark mode).  

![Instalação do Zabbix 7.2](https://raw.githubusercontent.com/wevertonjlima/blog-assets/main/posts/zabbix-v7.2_ubuntu-v24.04/zabbix-7v2-web-installation-04.png)



A janela exibe um sumario com todas as opções escolhidas:  

![Instalação do Zabbix 7.2](https://raw.githubusercontent.com/wevertonjlima/blog-assets/main/posts/zabbix-v7.2_ubuntu-v24.04/zabbix-7v2-web-installation-05.png)



Parabéns! Você concluiu con sucesso a instalação do Zabbix Frontend. 

![Instalação do Zabbix 7.2](https://raw.githubusercontent.com/wevertonjlima/blog-assets/main/posts/zabbix-v7.2_ubuntu-v24.04/zabbix-7v2-web-installation-06.png)



Faça o login inicial **exatamente** como as credenciais abaixo:
> username: **Admin**  
> password: **zabbix**

![Instalação do Zabbix 7.2](https://raw.githubusercontent.com/wevertonjlima/blog-assets/main/posts/zabbix-v7.2_ubuntu-v24.04/zabbix-7v2-web-installation-07.png)



Finalmente a dashboard será apresentada concluindo totalmente a instalação do Zabbix Server:  

![Instalação do Zabbix 7.2](https://raw.githubusercontent.com/wevertonjlima/blog-assets/main/posts/zabbix-v7.2_ubuntu-v24.04/zabbix-7v2-web-installation-08.png)
<br>



---  
### 4 CONCLUSÃO

A instalação do Zabbix é um processo relativamente simples, precisando apenas de atenção nas configurações dos arquivos de configuração e no procedimento de ajuste do banco de dados. Agora que seu servidor está no ar, é hora de usa-lo de forma eficiente monitorando os principais itens dentro da sua rede de computadores.

Eis uma lista de próximas atividades, para começar a *"brincar"* com o Zabbix:


- Configuração de hosts e itens de monitoramento.
- Instalação do Zabbix Proxy (caso monitore redes remotas).
- Instalação dos Agentes 2.0.
- Configuração de redundância.
- Criação de triggers e alertas.
- Ajuste de templates.
- Dashboards e relatórios.
- Configuração do house-keeping.


Com esses passos, você estará aproveitando ao máximo as capacidades avançadas de monitoramento do Zabbix.

Espero que este tutorial tenha sido proveitoso e informativo para você!



>  Fontes deste artigo:

- [Site Oficial do Zabbix](https://www.zabbix.com/download?zabbix=7.2&os_distribution=ubuntu&os_version=24.04&components=server_frontend_agent_2&db=pgsql&ws=apache)

- [Zabbix 7.2 in 5 minutes with Debian](https://www.initmax.com/wiki/zabbix-7-2-instructions-for-installation-in-5-minutes/)

- [Youtube | How To Install Zabbix Server 7.2 - Dmitry Lambert](https://youtu.be/KmINqvB94c8?si=9HgLLoVJtCT0ngtF)


