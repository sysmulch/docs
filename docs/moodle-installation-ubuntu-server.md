---
layout: default
title: Installing the Moodle LMS on an Ubuntu Server
parent: Installation Guides
nav_order: 2
---

# Installing Moodle on an Ubuntu Server

This installation guide is meant to help you install the Moodle application on a server running Ubuntu. If you have not used Moodle before, explore [these ways to try Moodle](https://moodle.com/news/try-moodle/).

The steps in this guide have been tested on Moodle version 4.0. 

## System requirements
A dedicated server or a virtual private server
- running an Ubuntu LTS release
- with a RAM of at least 1 GB to get a basic Moodle site up and running

## Before you begin
You should

- have basic Ubuntu command line skills
- know how to use a text editor in Ubuntu (such as vi or nano)
- have root access to the Ubuntu server where you want to install Moodle
- set aside at least one hour for this procedure

{: .note } 
If you experience difficulties during the installation, seek help on the [Moodle Community Forums](https://moodle.org/course/view.php?id=5).


## Step 1: Install Apache, MySQL, PHP, and additional software

1. Add the PPA (personal package archive) for PHP.

    ```
   sudo add-apt-repository ppa:ondrej/php sudo apt update
    ```

1. Install Apache, MySQL, and PHP 7.4 (or another PHP version up to 8).

   ```
   sudo apt install apache2 mysql-client mysql-server php7.4 libapache2-mod-php
   ```

1. Secure the MySQL installation. When prompted to set a root password, use your password manager to create a strong password.

   ```
   sudo mysql_secure_installation
   ```

   {: .important }
   Save the MySQL root password.

1. Install additional software and dependencies.

    ```
    sudo apt install graphviz aspell ghostscript clamav php7.4-pspell php7.4-curl php7.4-gd php7.4-intl php7.4-mysql php7.4-xml php7.4-xmlrpc php7.4-ldap php7.4-zip php7.4-soap php7.4-mbstring
    ```

1. Restart Apache.

    ```
    sudo service apache2 restart
    ```

1. Install Git.

    ```
   sudo apt install git
    ```

## Step 2: Download Moodle

1. Go to the opt directory.

   ```
   cd /opt
   ```

1. Download the Moodle code.

    ```
    sudo git clone git://git.moodle.org/moodle.git
    ```
    
1. Go to the downloaded Moodle folder.

    ```
    cd moodle
    ```

1. Retrieve a list of Moodle branches available.

    ```
    sudo git branch -a
    ```

1. Locate the branch for git to track. To install Moodle 4.0:

    ```
    sudo git branch --track MOODLE_400_STABLE origin/MOODLE_400_STABLE
    ```

1. Check out Moodle.

    ```
    sudo git checkout MOODLE_400_STABLE
    ```

## Step 3: Create web directory for Moodle

1. Copy Moodle files into the web directory.

    ```
    sudo cp -R /opt/moodle /var/www/html/
    ```

1. Create the moodledata directory.

    ```
    sudo mkdir /var/moodledata
    ```

1. Make www-data the owner of the moodledata directory.

    ```
    sudo chown -R www-data /var/moodledata
    ```

1. Change the permissions of the moodledata directory.

    ```
    sudo chmod -R 777 /var/moodledata
    ```

1. Change the permissions of the Moodle web directory.

   ```
   sudo chmod -R 0755 /var/www/html/moodle
   ```
    
## Step 4: Set up MySQL server

1. Open the MySQL configuration file with a text editor (for example, vi).

    ```
    sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
    ```

1. Scroll down to the `[mysqld]` section in the file. At the end of Basic Settings, add the following lines:

   ```
    default_storage_engine = innodb
    innodb_file_per_table = 1
    innodb_file_format = Barracuda
    ```

    Save the file to return to the terminal.

1. Restart the MySQL server.

    ```
    sudo service mysql restart
    ```

1. Sign into the MySQL server.

    ```
    sudo mysql -u root -p
    ```

   A password prompt appears. 

1. Enter the password you set earlier. The MySQL prompt appears. Type the below commands at the prompt.

    - Create the Moodle database in the MySQL server.
    
      ```
      CREATE DATABASE moodle DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
      ```

    - Create a user for the Moodle database. Replace `moodlegeek` and `moodlegeekpassword` with a username and a strong password of your choice.
  
      ```
      CREATE USER 'moodlegeek'@'localhost' IDENTIFIED BY 'moodlegeekpassword';
      ```

    - Allow the user privileges to use the Moodle database. Replace `moodlegeek` and `moodlegeekpassword` with the username and password you set in the previous step.

        ```markdown
        GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,CREATE TEMPORARY TABLES,DROP,INDEX,ALTER ON moodle.* TO 'moodlegeek'@'localhost';
        ```

    - Exit the MySQL server.

        ```markdown
        quit;
        ```

## Step 5: Complete the installation

1. Make the Moodle web directory writable.

    ```markdown
    sudo chmod -R 777 /var/www/html/moodle
    ```
1. Open a browser on your computer. Go to `http://IP.ADDRESS.OF.YOUR.SERVER/moodle`. Replace `IP.ADDRESS.OF.YOUR.SERVER` with your server IP address.

1. Follow the prompts to install Moodle. Make the following changes during the installation:

    - Change the path for `moodledata` to `/var/moodledata`
    - For Database Type, choose `mysqli`
    - Under Database Settings:
        - Host server: `localhost`
        - Database: `moodle`
        - User: the username you created earlier
        - Password: the password you created earlier
    - For Tables Prefix, choose `mdl_`

1. At the Environment Checks stage, check if any elements required for Moodle are not installed. Follow the instructions to install them.

1. Follow the prompts and confirm the installation.

1. At the Site Administrator account stage, create the username and password for the site administrator account.

    {: .important }
    Save the site administrator account password.

1. When the installation is complete, return to the Ubuntu terminal. Remove write permissions on the Moodle web directory.

    ```markdown
    sudo chmod -R 0755 /var/www/html/moodle
    ```
1. You can now starting using your Moodle site at `http://IP.ADDRESS.OF.YOUR.SERVER/moodle`.

### Verify the installation

If installation is successful, the dashboard of your Moodle site appears. You can access the Administration options from the top navigation bar.

## Step 6: Complete post-installation tasks

### Check system paths

Set the system paths for better system performance. Go to `Site Administration > Server > System Paths` and enter the following: 

- Path to du: `/usr/bin/du`
- Path to apsell: `/usr/bin/aspell`
- Path to dot: `/usr/bin/dot`

If you do not have an anti-virus application on your server:

1. Create the Quarantine Directory.

    ```markdown
    sudo mkdir /var/quarantine
    ```

2. Change Ownership

    ```markdown
    sudo chown -R www-data /var/quarantine
    ```

3. Navigate to `Site Administration > Plugins > Antivirus plugins > Manage antivirus plugins`. Enable ClamAV Antivirus.

### Check configuration options

The main configuration file for Moodle is `config.php`. It is in `/var/www/html/moodle`.

*Example of configuration:*

To make Moodle accessible from `http://IP.ADDRESS.OF.YOUR.SERVER/` instead of `http://IP.ADDRESS.OF.YOUR.SERVER/moodle`, make this change in `$CFG->wwwroot` in `config.php`.

### Understand upgrade options

To upgrade Moodle to a higher version, see [this page](https://docs.moodle.org/402/en/Upgrading).

It is not possible to downgrade Moodle to a lower version.

### Understand performance recommendations

Read the [performance recommendations](https://docs.moodle.org/402/en/Performance_recommendations).

<hr>

### Documentation credit

This guide was written by Ravi Murugesan using [the installation guide template from The Good Docs Project](https://gitlab.com/tgdp/templates/-/blob/main/installation-guide/template-installation-guide.md). 
