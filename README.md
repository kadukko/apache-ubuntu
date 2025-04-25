# apache-ubuntu

Esse é um tutorial de como instalar o apache no Linux Ubuntu, normalmente faço essa configuração no WSL e utilizo o VSCode Server para aplicações em PHP.

### ✅ 1. Instalar o Apache e PHP 7.x

```bash
sudo apt update
sudo apt install apache2 php7.4 libapache2-mod-php7.4
```

Substitua `php7.4` por outra versão (ex: `php7.3`, `php7.2`) se quiser.

---

### ✅ 2. Testar se o PHP está funcionando

Crie um arquivo chamado `info.php`:

```bash
sudo nano /var/www/html/info.php
```

E coloque o seguinte conteúdo:

```php
<?php phpinfo(); ?>
```

Depois, abra no navegador: `http://localhost/info.php`
Se aparecer a tela com informações do PHP, está tudo certo.

---

### ✅ 3. Clone o repositório

```bash
cd /var/www
sudo git clone https://github.com/seuusuario/seurepo.git
```

Agora você tem algo tipo:  
`/var/www/seurepo`

---

### ✅ 4. Mude o dono dos arquivos

O Apache geralmente roda como o usuário `www-data`, então é uma boa garantir que o Apache tenha acesso completo. Você também pode manter seu usuário como dono se for você quem atualiza o repositório.

#### Opção 1: dar tudo ao Apache (se for só pra produção)

```bash
sudo chown -R www-data:www-data /var/www/seurepo
```

#### Opção 2: seu usuário + permissões para o Apache (mais comum em desenvolvimento)

```bash
sudo chown -R $USER:www-data /var/www/seurepo
sudo find /var/www/seurepo -type d -exec chmod 775 {} \;
sudo find /var/www/seurepo -type f -exec chmod 664 {} \;
```

Isso deixa:

- Você com controle total
- Apache com acesso de leitura e execução (e gravação, se necessário)

---

### ✅ 6. Usar `setgid` no diretório

Se quiser garantir que todos os arquivos futuros pertençam ao grupo `www-data`, mesmo com `git pull`, você pode aplicar isso:

```bash
sudo chmod g+s /var/www/seurepo
```

Isso força os arquivos criados no futuro a herdar o grupo do diretório.


---

### ✅ 7. Criar um Virtual Host

Aqui você vai configurar um domínio local para não ficar usando o `localhost`, normalmente eu prefiro criar um subdominio do `localhost`, por exemplo: `meusite.localhost`

```bash
sudo nano /etc/apache2/sites-available/meusite.localhost.conf
```

Conteúdo básico:

```apache
<VirtualHost *:80>
    ServerAdmin webmaster@meusite.local
    ServerName meusite.local
    DocumentRoot /var/www/seurepo

    <Directory /var/www/seurepo>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/meusite_error.log
    CustomLog ${APACHE_LOG_DIR}/meusite_access.log combined
</VirtualHost>
```

Para manter alguns arquivos fora acesso público, você pode configurar regras **FileMatch** no `.htaccess`, ou se preferir manter os arquivos publicos dentro de uma subpasta **public_html** no seu repositório.

Depois você vai alterar o **Directory** da configuração acima, ficará assim:

```
    <Directory /var/www/seurepo/public_html>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
```

Assim você pode manter alguns arquivos de configurações seguros com o seu `.env`

#### d. Ative o site e reinicie o Apache

```bash
sudo a2ensite meusite.localhost.conf
sudo systemctl reload apache2
```

#### e. (IMPORTANTE) Adicione o domínio ao seu arquivo `hosts`

```bash
sudo nano /etc/hosts
```

Adicione no final:

```
127.0.0.1    meusite.localhost
```

Se você utiliza o WSL, faça a mesma configuração no Windows alterando o arquivo `C:\Windows\System32\drivers\etc\hosts`

---

### ✅ 8. Testar

Coloque um `index.php` dentro do `/var/www/seurepo` com algo como:

```php
<?php echo "Meu site está no ar com PHP 7!"; ?>
```

Depois, acesse no navegador:  
📡 `http://meusite.localhost`

---

### ✅ 9. Resolvendo problemas


Pode ser que o seu servidor não esteja configurado para usar **RewriteEngine** do `.htaccess`, caso isso ocorra vai aparece a mensagem:

```
Service Unavailable

The server is temporarily unable to service your request due to maintenance downtime or capacity problems. 
Please try again later.
```

Solução: 

1. **Ative o módulo `mod_rewrite`**:
   ```bash
   sudo a2enmod rewrite
   ```

2. **Garanta que seu vhost permita o uso de `.htaccess`** com `AllowOverride All`. Seu VirtualHost deve ter algo assim:

   ```apache
   <VirtualHost *:80>
       ServerName alertasaude.localhost
       DocumentRoot /var/www/alertasaude

       <Directory /var/www/alertasaude>
           AllowOverride All
           Require all granted
       </Directory>
   </VirtualHost>
   ```

3. **Reinicie o Apache**:
   ```bash
   sudo systemctl restart apache2
   ```
