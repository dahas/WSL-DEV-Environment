# Set up a LEMP Stack on Windows and initialise a PHP project from scratch

**Environment:**
- Windows 10/11
- Microsoft VS Code
- WSL2
- Ubuntu 22.04
- NodeJS
- PHP 8.1
- MySQL 8
- Composer
- PHPUnit
- Nginx
- XDebug
- Git

<hr>

## Install WSL

Open Powershell as Administrator and run:
```
PS wsl --install
```
<hr>

## Install Ubuntu:

Open Microsoft Store and search for "Ubuntu". Choose 22.04 (or higher). Download it, then click open. Follow the instructions in the terminal.

To enable SystemD create `wsl.conf`:
```
~$ sudo nano /etc/wsl.conf 
```
Enter:
```
[boot]
systemd=true
```
Exit an save file.

### Update Packages
```
~$ sudo apt update
~$ sudo apt full-upgrade
```
<hr>

## Install NodeJS with Node Version Manager "nvm"
```
~$ sudo apt install curl 
~$ curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash
~$ source ~/.profile
~$ nvm install node --lts
```
You can use nvm to switch between different NodeJS versions. Use `nvm ls` to list all available versions and `nvm use x` to select a specific version (where x is the major part of the version number: x.y.z). 
<hr>

## Install PHP 8.1
```
~$ sudo apt install --no-install-recommends php8.1
~$ sudo apt-get install -y php8.1-cli php8.1-common php8.1-mysql php8.1-zip php8.1-gd php8.1-mbstring php8.1-curl php8.1-xml php8.1-bcmath
```
<hr>

## Download and install Composer
Download installer and verify hash:
```
~$ curl -sS https://getcomposer.org/installer -o /tmp/composer-setup.php
~$ HASH=`curl -sS https://composer.github.io/installer.sig`
~$ echo $HASH
~$ php -r "if (hash_file('SHA384', '/tmp/composer-setup.php') === '$HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
~$ sudo php /tmp/composer-setup.php --install-dir=/usr/local/bin --filename=composer
```
<hr>

## Install MySQL
```
~$ sudo apt-get install -y mysql-server php-mysql
```
Important: Alter the root user first:
```
~$ sudo mysql
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
mysql> exit
```
Then run secure installation:
```
~$ mysql_secure_installation
    > Validate Password component? No
    > Change Root Password? No
    > Remove anonymous users? Yes		
    > Disallow Remote Login? Yes
    > Remove test database? Yes
    > Reload Provileg Tables? Yes
```
<hr>

## Install PHPUnit
```
~$ wget -O phpunit https://phar.phpunit.de/phpunit-9.phar
~$ chmod +x phpunit
~$ ./phpunit --version
```
<hr>

## Nginx
```
~$  sudo apt install nginx
~$  sudo service nginx start
```

Test in web browser: `http://localhost`

Additinally we need a "bridge" between PHP and Nginx. Therefor install the "FastCGI Process Manager" for PHP:
```
~$ sudo apt install php8.1-fpm
```

Now you can create your project in `/var/www`:
```
~$ sudo mkdir /var/www/your_domain
~$ sudo chown -R $USER:$USER /var/www/your_domain
~$ cd /var/www/your_domain
~$ git clone https://github.com/dahas/PHPSkeleton.git .
```

Create the server configuration for your project:
```
$ sudo nano /etc/nginx/sites-available/your_domain
```
Copy and paste the following:
```
server {
    listen 2400; 
    server_name localhost;
    root /var/www/your_domain/public;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ /index.php$is_args$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
     }
}
```

Enable your site with a sim link:
```
$ sudo ln -s /etc/nginx/sites-available/your_domain /etc/nginx/sites-enabled/
```

Test your configuration:
```
$ sudo nginx -t
```

Restart Nginx:
```
$ sudo systemctl reload nginx
```

Test in web browser: `http://localhost:2400`
<hr>

## Unit Testing
Put files you want to be tested into the `tests` folder of your project. Add the suffix "*Test.php" to the file and "*Test" to the class name. 

- PHPUnit tutorial: https://startutorial.com/view/phpunit-beginner-part-1-get-started 
- PHPUnit docs: https://phpunit.de/documentation.html

### Enable PHPUnit in VSCode
In the root folder of your project create a a file named **phpunit.xml** and paste the following content into it:
```
// phpunit.xml

<?xml version="1.0" encoding="UTF-8"?>
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="https://schema.phpunit.de/9.5/phpunit.xsd" bootstrap="vendor/autoload.php">
  <testsuites>
    <testsuite name="Default">
      <directory>tests</directory>
    </testsuite>
  </testsuites>
</phpunit>
```
Next, open the VSCode settings, search for "phpunit" and add the path to the phar file:

**Path to the PHPUnit binary: "~/phpunit"**

### Running the tests 
Now you can execute your tests either by entering the following commands in the terminal ...
```
$ phpunit tests/FooTest.php --testdox  // Test a specific file
$ phpunit tests --testdox  // Test all files in tests folder
```
or by using the **VSCode Panel** to the left of your screen.
<hr>

## Debugging

### Install XDebug
- Open `localhost:2400/info.php` in Web browser.
- Take a note of the path to the PHP.ini file. You´ll need it later.
- Copy everything and paste it here: https://xdebug.org/wizard
- Follow the instructions.

### Set up VSCode for Debugging
Create a configuration file for the debugger in VSCode and replace its content with the following:
```
// .vscode/launch.json

{
    "version": "0.2.0",
    "configurations": [

        //^^ Debug currently open PHP file
        {
            "name": "PHP File",
            "type": "php",
            "request": "launch",
            "program": "${file}",
            "args": [],
            "externalConsole": false,
            "port": 9003
        },

        //^^ Debug PHP App
        {
            "name": "PHP App",
            "type": "php",
            "request": "launch",
            "program": "${workspaceFolder}/public/index.php",
            "args": [],
            "externalConsole": false,
            "port": 9003
        },

        //^^ Debug currently open JS file
        {
            "name": "JS File",
            "type": "node",
            "request": "launch",
            "program": "${file}",
            "skipFiles": [
                "<node_internals>/**"
            ]
        },

        //^^ Debug JS App
        {
            "name": "JS App",
            "type": "node",
            "request": "launch",
            "program": "${workspaceFolder}/public/index.js",
            "skipFiles": [
                "<node_internals>/**"
            ]
        }

    ]
}
```
<hr>

## GIT
Git is pre-installed on Ubuntu-22.04. But you probably will be asked to provide a name and an email address of your Git account:
```
~$ git config --global user.name "your_name"
~$ git config --global user.email "your_email"
```
<hr>

## Backup
### Create a Backup
To backup the whole stack you have to export the WSL distribution to a safe place in your file system.

Quit all WSL instances:
```
PS wsl --shutdown
```

To get a full list of installed distros run the following command in Powershell:
```
PS wsl -l -v
```

Find yours and use the exact name. E. g.:
```
PS wsl --export Ubuntu-22.04 F:\backups\ubuntu2204.tar
```

### Restore a Backup
To restore the backup you have to run a command, that matches the following pattern:

wsl --import \<ImageName\> \<TargetDirectory\> \<SourceDirectory\>

In your case:
```
PS --import Ubuntu-22.04 C:\YourFolder\Ubuntu-22.04 F:\backups\ubuntu2204.tar
```

Verify the import was successfull:
```
PS wsl --list
```
<hr>

## Update composer autoloader
When ever you add or modify a path in `composer.json` you have to update the autoloader:
```
~$ cd application
~$ composer update
~$ cd ..
```
