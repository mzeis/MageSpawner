# MageSpawner

This is a little handy shell script that I am using for quite a while now.
The goal is to quickly create new (and remove) Magento installations for testing purposes. Testing as in "I want to try something", not as in Unit Testing.

*Before you start, let me state something:*

**Do not use this on a production machine. Ideally, use it on a machine where you don't care about losing everything.** The remove script uses a "rm -rf" command, so if something goes wrong, it could go awfully wrong. I solely use this on a VM where everything important is backed up regularily.

## Requirements

  * Linux/Unix (tested on Debian)
  * bash
  * Needs to be run via sudo / as root (chown and chmod are used) 
  * These commands must be available:
     * basename
     * mysql
     * php
     * sed
     * tar
     * wget
     * which

## Installation

  * Get the files from Github.
  * Copy config.conf.sample to config.conf and adjust the settings as needed.
  * Make the files ./install and ./remove executable.

## Getting started

Run `sudo ./install` and follow the instructions. You have to be a superuser because chown and chmod are used. At the moment, you have to enter the details in an interactive mode (= no command line automatisation).

This is how it looks like on my VM:

    Welcome to MageSpawner.
    
    Which version do you want to install?
    1) None, abort installation
    2) CE 1.5.0.1
    3) CE 1.5.1.0
    4) CE 1.6.0.0
    5) CE 1.6.1.0
    6) CE 1.6.2.0
    7) CE 1.7.0.2
    Enter the number (e.g. 1): 7
    CE 1.7.0.2 selected.
    
    Specify the shop name. It may be used for the URL, directory and database name, so don't use spaces, special characters or the like.
    Shop name (default: shop): 1702
    Thanks, the required information was provided. The installation will now begin.
    
    Downloading Magento package...
    --2012-03-04 12:32:13--  http://www.magentocommerce.com/downloads/assets/1.7.0.2/magento-1.7.0.2.tar.gz
    Auflösen des Hostnamen www.magentocommerce.com... 209.15.239.51
    Verbindungsaufbau zu www.magentocommerce.com|209.15.239.51|:80... verbunden.
    HTTP-Anforderung gesendet, warte auf Antwort... 200 OK
    Länge: 17653068 (17M) [application/x-gzip]
    In »magento-1.7.0.2.tar.gz« speichern.
    
    100%[===================================================================================================================>] 17.653.068  1,75M/s   in 15s
    
    2012-03-04 12:32:28 (1,13 MB/s) - »magento-1.7.0.2.tar.gz« gespeichert [17653068/17653068]
    
    Package downloaded.
    
    Unpacking package and setting permissions.
    Package unpacked and permssions set.
    
    Creating database...
    Database created.
    
    Executing Magento setup script...
    SUCCESS: 9a96247823ea7c2cd8f3ebff7db0a544
    Setup script executed. Please write down the encryption key provided above.
    
    Reindexing Magento indexes...
    Product Attributes index was rebuilt successfully
    Product Prices index was rebuilt successfully
    Catalog URL Rewrites index was rebuilt successfully
    Product Flat Data index was rebuilt successfully
    Category Flat Data index was rebuilt successfully
    Category Products index was rebuilt successfully
    Catalog Search Index index was rebuilt successfully
    Stock Status index was rebuilt successfully
    Tag Aggregation Data index was rebuilt successfully
    Indexes reindexed.
    
    Editing .htaccess for VirtualDocumentRoot configuration (setting RewriteBase)...
    .htaccess edited.
    
    Final permission settings...
    Done.
    
    If you didn't see any error messages, everything went fine.
    Set up your vhost config and host entries (if needed) and you are ready to go!
    The frontend shop URL is http://1702.magentoshops.vm.


After the script has finished, you may be two steps away from using the new Magento installation:

  1. **Add an entry two your hosts file**. Your browser has to know which server the domain (e.g. 1702.magentoshops.vm) belongs to. If you use a real domain and have set an wildcard DNS record, then you are good to go. Otherwise, open your hosts file and a line like `192.168.1.2 1702.magentoshops.vm` (don't forget to change this to the correct IP and domain).

  2. **Add an entry to your server configuration**. Your web server has to know how to handle the request. Normally, you would have to add a [Virtual Host](http://httpd.apache.org/docs/2.2/vhosts/name-based.html) every single time you setup a new Magento installation.

     As programmers don't like to repeat themselves an I'm a programmer, I'm using the [Apache Module mod_vhost_alias](http://httpd.apache.org/docs/2.0/mod/mod_vhost_alias.html) and the nifty `VirtualDocumentRoot` directive.

     This is what my configuration looks like:
  
         <VirtualHost *:80>
             ServerAdmin my@email.com
             ServerName magentoshops.vm
             ServerAlias *.magentoshops.vm
             UseCanonicalName Off
             VirtualDocumentRoot /var/www/magento/shops/%0/
             # Some more settings
             # Magento development settings
             SetEnv MAGE_IS_DEVELOPER_MODE 1
         </VirtualHost>

     The VirtualDocumentRoot path /var/www/magento/shops/ has to match the MAGE_BASE_DIR setting in config.conf.
     The script automatically edits Magentos .htaccess so that the RewriteBase is adjusted for this setup.  

## License

   Copyright 2011-2012 Matthias Zeis

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.