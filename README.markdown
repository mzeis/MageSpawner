# MageSpawner

This is a little handy shell script that I am using for quite a while now.
The goal is to quickly create new (and remove) Magento installations for testing purposes. Testing as in "I want to try something", not as in Unit Testing.

*Before you start, let me state something:*

**Do not use this on a production machine. Ideally, use it on a machine where you don't care about losing everything.** The remove script uses a "rm -rf" command, so if something goes wrong, it could go awfully wrong. I solely use this on a VM where everything important is backed up regularily.

## Requirements

  * Linux/Unix (tested on Debian)
  * bash
  * You may need to grant the Linux user `sudo` permissions for `chown`, `chmod` and `rm` (see the "To sudo or not to sudo?" section of this document) 
  * These commands must be available:
     * basename
     * command
     * mysql
     * php
     * sed
     * tar
     * wget OR curl

## Installation

  * Get the files from Github.
  * Copy config.conf.sample to config.conf and adjust the settings as needed.
  * Make the files ./install and ./remove executable.
  
## To sudo or not to sudo?

MageSpawner wants to help you by setting the correct permissions and ownerships for files and directories. It also wants to help you cleaning up when you remove a test installation.

For this to work, MageSpawner needs to execute `chmod`, `chown` and `rm` on the files. Up to version 0.3.5, MageSpawner always used `sudo` if the user executing the script wasn't a superuser.

Starting with version 0.4.0 there is a new configuration variable `LINUX_USE_SUDO` (defaulting to `true`). If your setup allows to `chmod`, `chown` and `rm` without `sudo` you can set this to `false`. A typical example would be running PHP with FastCGI.

## Getting started

Run `./install` and follow the instructions. If you didn't specify otherwise, sudo will be used for the commands `chown` and `chmod`. If you don't specify all parameters required for an install, you will be asked for the details in an interactive mode.

Furthermore MageSpawner will save downloaded archive files so you don't have to download the file every time. 

This is how it looks like in my VM:

	user@server:/path/to/MageSpawner $ ./install
	Welcome to MageSpawner.

	Which version do you want to install?
	1) None, abort installation
	2) CE 1.5.0.1
	3) CE 1.5.1.0
	4) CE 1.6.0.0
	5) CE 1.6.1.0
	6) CE 1.6.2.0
	7) CE 1.7.0.2
	8) CE 1.8.0.0
	9) CE 1.8.1.0
	Enter the number (e.g. 1): 9
	CE 1.8.1.0 selected.

	Specify the shop name. It may be used for the URL, directory and database name, so don't use spaces, special characters or the like.
	Shop name (default: shop): ce1810
	Thanks, the required information was provided. The installation will now begin.

	Unpacking package and setting permissions.
	Package unpacked and permissions set.

	Creating database...
	Database created.

	Executing Magento setup script...
	SUCCESS: aedeb37b468f920658bb1af6bca10617
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

	Do you want to use modman? (y/n) y
	Initialized Module Manager at /path/to/htdocs/ce1810.magetests.vm

	Final permission settings...
	Done.

	If you didn't see any error messages, everything went fine.
	Set up your vhost config and host entries (if needed) and you are ready to go!
	The frontend shop URL is http://ce1810.magetests.vm.


After the script has finished, you may be two steps away from using the new Magento installation:

  1. **Add an entry two your hosts file**. Your browser has to know which server the domain (e.g. 1702.magentoshops.vm) belongs to. If you use a real domain and have set an wildcard DNS record, then you are good to go. Otherwise, open your hosts file and a line like `192.168.1.2 1702.magentoshops.vm` (don't forget to change this to the correct IP and domain).

  2. **Add an entry to your server configuration**. Your web server has to know how to handle the request. Normally, you would have to add a [Virtual Host](http://httpd.apache.org/docs/2.2/vhosts/name-based.html) every single time you setup a new Magento installation.

     As programmers don't like to repeat themselves and I'm a programmer, I'm using the [Apache Module mod_vhost_alias](http://httpd.apache.org/docs/2.0/mod/mod_vhost_alias.html) and the nifty `VirtualDocumentRoot` directive.

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

## Automatic installation via command line

You may want to do a fully automated installation. This feature was added in v0.3.

Have a look at this example:
    
    ./install --mageversion ce1810 --magename ce1810 --magedomain ce1810.magentoshops.vm --modman y
    
For an explanation of the parameters, call `./install --help`:

	MageSpawner (v0.4.0)
	
	MageSpawner is a tool for quick setup of multiple Magento test installations.
	THIS IS NOT MEANT TO BE USED ON PRODUCTION ENVIORNMENTS!
	
	Usage:
	./install
	Then follow the instructions on the screen.
	
	Consult the README for further information.
	
	Options:
	
	  --help , -h, -?           display this help message
	  --magedomain              Specify the shop domain (e.g. 1702.magentoshops.vm).
	                            The subdomain has to be the same string as --magename
	  --magename                The shop name which gets used for the URL, directory and database name.
	  --mageversion             Version of Magento to be used. Has to be one of
	                            [ce1501|ce1510|ce1600|ce1610|ce1620|ce1702|ce1800|ce1810]
	  --modman                  Whether you want modman to be initialized.
	                            [y|n]
	  --usage                   display this help message
	  --version                 display the version of this script
	
	Get more information at
	https://github.com/mzeis/MageSpawner


The parameters `--magedomain`, `--magename`, `--mageversion` and `--modman` are required for a fully automatic installation. If parameters are omitted, you will be asked for the missing information.

Please note that you may be asked for your `sudo` password. 

## Removing a test installation

If you want to remove a test installation, call `./remove`. Again, the script will ask for your sudo password if you didn't specify otherwise.

This is how it looks on my VM:

	user@server:/path/to/MageSpawner $ ./remove
	Welcome to the Magento uninstall script.
	
	Which shop do you want to uninstall? Provide only the subdomain part of the domains provided.
	ce1800.magetests.vm
	ce1810.magetests.vm
	Shop code: ce1810
	
	Deleting files...
	Files deleted.
	
	Deleting database...
	Database deleted.
	
	Shop was deleted successfully. Please delete vhost entries and host config as needed.

If you know the shopcode, you also can call `./remove --mageshopcode [code]`, e.g. `./remove --mageshopcode ce1810`. 

## Changelog

### v0.5.0
* Added Magento CE 1.9.0.0

### v0.4.0
* Added configuration variable LINUX_USE_SUDO for disabling sudo

### v0.3.5
* Fixed version number in script

### v0.3.4
* Added Magento CE 1.8.1.0
* Fixed bug with wrong set currency "EURO".

### v0.3.3
* Added Option --mageshopcode to remove

### v0.3.2
* Added Magento CE 1.8.0.0

### v0.3.1
* Cache downloaded archive files for faster creation of a new shop (directory configurable via MAGE_DOWNLOAD_DIR)

### v0.3
* Added command line parameters for automatic installation

### v0.2.1
* Merged pull request by Nils Preuß:
  "Added fix for 'PHP Extensions "0" must be loaded , while installing magento-1.7.x' that occured in at least some installations"
  Thanks for this!
* Script doesn't have to be called with sudo anymore because the script itself will ask for sudo if needed.
* New configuration parameters for file and folder permissions

### v0.2
* Script can now both use wget or cURL to download Magento. wget is preferred. (thanks Rouven for the input)
* modman can be initialized from install script
* Minor changes / fixes

### v0.1
* Initial commit   

## License

   Copyright 2011-2014 Matthias Zeis

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.
