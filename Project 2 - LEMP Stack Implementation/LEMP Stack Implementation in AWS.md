# Project-2: LEMP Stack Implementation in AWS

## Step 0 - Creating the virtual server with Ubuntu Server OS 
1. Launch a new EC2 instance of t2.micro family with Ubuntu Server 20.04 LTS (HVM) in the preferred region
2. Download and save your private key (.pem file) securely and do not share it with anyone!
   - If you lose it, you will not be able to connect to your server ever again!
3. Launching the GitBash and runnning the following command: <br>
   `ssh -i <Your-private-key.pem> ubuntu@<EC2-Public-IP-address>` <br>
![Alt text](<Screenshot/1. SSH into Web Server.jpg>)


## Step 1 - Installing the Nginx Web Server
Use **`apt`** package manager to install the package:

 **`sudo apt update -y`**

Use the following command to get Nginx package installed:

 **`sudo apt install nginx -y`**

 Verify Nginx is Up and running.

 **`sudo systemctl status nginx`**

![Alt text](<Screenshot/2. Nginx Status.jpg>)

 If it is green and running, then you did everything correctly - you have just launched your first Web Server in the Clouds!

 Before we can receive any traffic by our Web Server, we need to open TCP port 80 which is the default port that web browsers use to access web pages on the Internet.

 Verify the web server is reachable from the localhost

```
 curl http://localhost:80

or

 curl http://127.0.0.1:80
 
```
**The output of the above command is shown below:**

![Alt text](<Screenshot/3. Verifying Web Server is Accessed from Localhost.jpg>)

![Public IP](<Screenshot/4. Public IP Address.jpg>)

![Access from Internet](<Screenshot/5. Access from Internet.jpg>)



## Step 2 - Installing MySQL

Use below command to acquire and installed the required software:

```
sudo apt install mysql-server
```
When the installation is done, login to the MySQL console by executing the following commands: 

```
sudo mysql
```
![Login Into MySQL Console](<Screenshot/6. Login to MySQL Console.jpg>)

It’s recommended that you run a security script that comes pre-installed with MySQL. This script will remove some insecure default settings and lock down access to your database system. Before running the script you will set a password for the root user, using mysql_native_password as default authentication method. We’re defining this user’s password as `PassWord.1`. 

```
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';
```

Exit the MySQL shell with 
```
exit 
```

Start the interactive script by running 
```
 sudo mysql_secure_installation
```
This will ask if you want to configure the `VALIDATE PASSWORD PLUGIN`

When finished test if you're able to log in to the MySQL console by typing:

```
sudo mysql -p
```

![Alt text](<Screenshot/7. Login Test.jpg>)

## Step 3 - Installing PHP
Now, Nginx is installed to serve the content and MySQL is installed to store and manage data. It's time to install **`PHP`** to process code and generate dynamic content for the web server. 

Install `php-fpm` and tell Nginx to pass PHP requests to this software for processing. Additionally, install `php-mysql` module that allows PHP to communicate with MySQL-based databases, by running the following command:


```
sudo apt install php-fpm php-mysql -y
```

## Step 4 - Configuring Nginx to Use PHP Processor
Create the root web directory `projectLEMP` as follows:

```
sudo mkdir /var/www/projectLEMP
```
Assign the ownership of the directory with the `$USER` emvornment variable, which will reference your current system user:
```
sudo chown -R $USER:$USER /var/www/projectLEMP
```

Open a new configuration file in Nginx’s sites-available directory

```
sudo nano /etc/nginx/sites-available/projectLEMP
```
This will create a new blank file and paste the below configuration:
```
#/etc/nginx/sites-available/projectLEMP

server {
    listen 80;
    server_name projectLEMP www.projectLEMP;
    root /var/www/projectLEMP;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }

}
```
 
 Activate the configuration by linking the config file from Nginx’s sites-enabled directory:
 ```
 sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/
 ```

 Test the configuration for sytax errors by typing:
 ```
 sudo nginx -t
 ```
![Alt text](<Screenshot/8. Testing.jpg>)

Disable default Nginx host that is currently configured to listen on port 80 and reload the nginx to apply the changes by following below commands:

```
sudo unlink /etc/nginx/sites-enabled/default

sudo systemctl reload nginx
```

Test the new server by creating the `index.html` file under `/var/www/projectLEMP` directory.

```
sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html
```
Access the browser by typing the pubic IP of the webserver: `http://<Public-IP-Address>:80
`

![Alt text](<Screenshot/9. Test new Web server.jpg>)

Next step, we’ll create a PHP script to test that Nginx is in fact able to handle .php files within our newly configured website.


## Step 5 - Testing PHP with Nginx
Now, create the test PHP file by executing the following command:
```
sudo nano /var/www/projectLEMP/info.php
```
Paste following lines into the new file:
```
<?php
phpinfo();
```
Access the page in the web browser by visiting the `http://<Public-IP-Address>` followed by `/info.php`

```
http://<Public-IP-Address>/info.php
```
![Alt text](<Screenshot/13. Info Php Page.jpg>)

## Step 6 - Retrieving data from MySQL database with PHP
Create the database named `kn_database` and user named `kn_user` 

```
CREATE DATABASE kn_database;

CREATE USER 'kn_user'@'%' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';

```
![Alt text](<Screenshot/10. Create DB.jpg>)

Give user the persion to manage the database.

```
GRANT ALL ON kn_database.* TO 'kn_user'@'%';
```

After creating `kn_user` , granting permission and login to our `kn_database` with the new `kn_user` account created.

```
mysql -u kn_user -p kn_database
```
![Alt text](<Screenshot/11. Test new user.jpg>)


Create the `todo_list` table with the following statement:
```
 CREATE TABLE kn_database.todo_list (item_id INT AUTO_INCREMENT,content VARCHAR(255),PRIMARY KEY(item_id));
```

Insert fews rows of content in the test table:
```
INSERT INTO kn_database.todo_list (content) VALUES ("My first database item");
INSERT INTO kn_database.todo_list (content) VALUES ("My second important item");
```

See the content of the table:
```
select * from kn_database.todo_list;
```
![Alt text](<Screenshot/12. Data in Table.jpg>)

Now create a PHP script that will connect to MySQL and query the content. Let's create a new PHP file in the custom web root directory.

```
vi /var/www/projectLEMP/todo_list.php
```
Paste the following configuration it to the PHP file:
```
<?php
$user = "kn_user";
$password = "PassWord.1";
$database = "kn_database";
$table = "todo_list";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h2>TODO</h2><ol>";
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}
```
Access the page in the web browser by visiting the `http://<Public-IP-Address>` followed by `/todo_list.php`

```
http://<Public-IP-Address>/todo_list.php
```
![Alt text](<Screenshot/14. Database Item.jpg>)
