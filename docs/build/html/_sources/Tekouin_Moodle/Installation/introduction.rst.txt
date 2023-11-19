
Tekouin Moodle Installation
================================
This page explains how to install Moodle. If you are an expert and/or in a hurry, try Installation Quickstart.

If you just want to try Moodle on a standalone machine, there are 'one-click' installers for Windows (see Complete install packages for Windows) and for OSX (see Complete Install Packages for Mac OS X) or install on OS X. These are unsuitable for production servers.

If you want to avoid installing Moodle yourself completely, consider `MoodleCloud <https://moodle.com/moodlecloud/>`_.


Requirements
------------

Moodle is primarily developed in Linux using Apache, PostgreSQL/MySQL/MariaDB, and PHP (sometimes known as the LAMP platform). Typically, this is also how Moodle is run, although there are other options (Nginx/OpenLiteSpeed) as long as the software requirements of the release are met.

If you are installing Moodle on a Windows server, note that from php5.5 onwards, you will also need to have the Visual C++ Redistributable for Visual Studio 2012 installed from: `Visual C++ <http://www.microsoft.com/en-us/download/details.aspx?id=30679>`_ (x86 or x64).

The basic requirements for Moodle are as follows:

**Hardware**
- Disk space: 200MB for the Moodle code, plus as much as you need to store content. 5GB is probably a realistic minimum.
- Processor: 1 GHz (min), 2 GHz dual core or more recommended.
- Memory: 512MB (min), 1GB or more is recommended. 8GB plus is likely on a large production server.
- Consider separate servers for the web "front ends" and the database. It is much easier to "tune."
- All the above requirements will vary depending on specific hardware and software combinations as well as the type of use and load; busy sites may well require additional resources. Further guidance can be found under performance recommendations. Moodle scales easily by increasing hardware.

For very large sites, you are much better starting with a small pilot and gaining some experience and insight. A "what hardware do I need for 50,000 users?" style post in the forums is highly unlikely to get a useful answer.

**Software**
- See the release notes in the dev docs for software requirements.

Set up your server
------------------

Depending on the use case, a Moodle server may be anything from a Desktop PC (e.g., for testing and evaluating) to a rack-mounted or clustered solution to cloud VMs or other hosted solutions. As mentioned above, there are lots of possibilities for installing the basic server software, for details see:

- Installing AMP
- IIS
- Nginx
- Apache
- OpenLiteSpeed

It will help hugely, regardless of your deployment choices, if time is taken to understand how to configure the different parts of your software stack (HTTP daemon, database, PHP, etc.). Do not expect the standard server configuration to be optimal for Moodle. For example, the web server and database servers will almost certainly require tuning to get the best out of Moodle.

If you're unsure which packages to use (e.g., MariaDB vs. MySQL), then have a look at what the consensus on the internet is, but also have a look at the Performance recommendations page, as this may also help give you an idea of exactly how much work is involved/what your workflow may end up looking like and planning that aspect as well.

If a hosting provider is being used, ensure that all Moodle requirements (such as PHP version) are met by the hosting platform before attempting the installation. It will help to become familiar with changing settings within the hosting provider's platform (e.g., PHP file upload maximums) as the options and tools provided vary.

Download and copy files into place
------------------------------------

**IMPORTANT:** While there are now a number of places you can get the Moodle code (including host provided Moodle installers), you are strongly advised to only obtain Moodle from `moodle.org <http://moodle.org>`_. If you run into problems, it will be a great deal easier to support you.

You have two options:

1. Download your required version from `http://moodle.org/downloads <http://moodle.org/downloads>`_ and unzip/unpack...
    OR
2. Pull the code from the Git repository (recommended for developers and also makes upgrading very simple):
    .. code-block::

        $ git clone -b MOODLE_{{Version3}}_STABLE git://git.moodle.org/moodle.git

For a fuller discussion see `Git for Administrators <https://docs.moodle.org/dev/Git_for_Administrators>`_.

Either of the above should result in a directory called moodle, containing a number of files and folders.

You can typically place the whole folder in your web server documents directory, in which case the site will be located at http://yourwebserver.com/moodle, or you can copy all the contents straight into the main web server documents directory, in which case the site will be simply http://yourwebserver.com. See the documentation for your system and/or web server if you are unsure.

**Tip:** If you are downloading Moodle to your local computer and then uploading it to your hosted website, it is usually better to upload the compressed Moodle file and then decompress on your hosted website. If you decompress Moodle on your local computer, because Moodle is comprised of over 25,000 files, trying to upload over 25,000 files using an FTP client or your host's "file manager" can sometimes miss a file and cause errors.

**Secure the Moodle files:** It is vital that the files are not writable by the web server user. For example, on Unix/Linux (as root):
    .. code-block::

        chown -R root /path/to/moodle
        chmod -R 0755 /path/to/moodle
        (files are owned by the administrator/superuser and are only writable by them - readable by everyone else)

On test/dev sites, you may want to make the files writable to use the built-in plugin installer. This is discouraged for live sites (at least, revert to more secure settings if you do).

This link about `setting folder permissions on Linux Webservers <https://docs.moodle.org/311/en/Installation_FAQ#The_right_folder_permissions_for_a_website_on_a_Linux_server>`_ may assist as well.

Create an empty database
------------------------

Next, create a new, empty database for your installation. You need to find and make a note of the following information for use during the final installation stage:

- dbhost - the database server hostname. Probably localhost if the database and web server are the same machine, otherwise the name of the database server.
- dbname - the database name. Whatever you called it, e.g., moodle.
- dbuser - the username for the database. Whatever you assigned, e.g., moodleuser - do not use the root/superuser account. Create a proper account with the minimum permissions needed.
- dbpass - the password for the above user.

If your site is hosted, you should find a web-based administration page for databases as part of the control panel (or ask your administrator). For everyone else or for detailed instructions, see the page for your chosen database server:

- PostgreSQL
- MariaDB
- MySQL
- MSSQL
- Oracle (not recommended)

Create the (moodledata) data directory
---------------------------------------

Moodle requires a directory to store all of its files (all your site's uploaded files, temporary data, cache, session data, etc.). The web server needs to be able to write to this directory. On larger systems, consider how much free space you are going to use when allocating this directory.

Due to the default way Moodle caches data, you may have serious performance issues if you use relatively slow storage (e.g., NFS) for this directory. Read the Performance recommendations carefully and consider using (e.g.) redis or memcached for Caching.

**IMPORTANT:** This directory must NOT be accessible directly via the web. This would be a serious security hole. Do not try to place it inside your web root or inside your Moodle program files directory. Moodle will not install. It can go anywhere else convenient.

Here is an example (Unix/Linux) of creating the directory and setting the permissions for anyone on the server to write here. This is only appropriate for Moodle servers that are not shared. Discuss this with your server administrator for better permissions that just allow the web server user to access these files.
    .. code-block::
    
        # mkdir /path/to/moodledata
        # chmod 0777 /path/to/moodledata

Securing moodledata in a web directory
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you are using a hosted site and you have no option but to place 'moodledata' in a web-accessible directory. You may be able to secure it by creating an .htaccess file in the 'moodledata' directory. This does not work on all systems - see your host/administrator. Create a file called .htaccess containing only the following lines:

**Apache 2.2**
    .. code-block::

        order deny,allow
        deny from all

**Apache 2.4**
    .. code-block::

        Require all denied

Start Moodle install
--------------------

It's now time to run the installer to create the database tables and configure your new site. The recommended method is to use the command line installer. If you cannot do this for any reason (e.g., on a Windows server), the web-based installer is still available.

**Command line installer**

It's best to run the command line as your system's web user. You need to know what that is - see your system's documentation (e.g., Ubuntu/Debian is 'www-data', Centos is 'apache')

Example of using the command-line (as root - substitute 'www-data' for your web user):
    .. code-block::
        
        # chown root /path/to/moodle
        # cd /path/to/moodle/admin/cli
        # sudo -u www-data /usr/bin/php install.php
        # chown -R root /path/to/moodle

The chowns allow the script to write a new config.php file. More information about the options can be found using
    .. code-block::
        
        # php install.php --help

You will be asked for other settings that have not been discussed on this page - if unsure just accept the defaults. For a full discussion see `Administration via command line <https://docs.moodle.org/311/en/Installation_FAQ#Administration_via_command_line>`_.

**Web based installer**

For ease of use, you can install Moodle via the web. We recommend configuring your web server so that the page is not publicly accessible until the installation is complete.

To run the web installer script, just go to your Moodle's main URL using a web browser.

The installation process will take you through a number of pages. You should be asked to confirm the copyright, see the database tables being created, supply administrator account details, and supply the site details. The database creation can take some time - please be patient. You should eventually end up at the Moodle front page with an invitation to create a new course.

It is very likely that you will be asked to download the new config.php file and upload it to your Moodle installation - just follow the on-screen instructions.

Final configuration
--------------------

**Settings within Moodle**

There are a number of options within the Moodle Site Administration screens (accessible from the 'Site administration' tab in the 'Administration' block (Classic theme) or the Site administration button in the navigation bar (Boost). Here are a few of the more important ones that you will probably want to check:

- Administration > Site administration > Server > Email > Outgoing mail configuration: Set your SMTP server and authentication if required (so your Moodle site can send emails). You can also set a noreply email on this page.
- Administration > Site administration > Server > Server > Support contact. Set your support contact email.
- Administration > Site administration > Server > System paths: Set the paths to du, dot, and aspell binaries.
- Administration > Site administration > Server > HTTP: If you are behind a firewall you may need to set your proxy credentials in the 'Web proxy' section.
- Administration > Site administration > Location > Update timezones: Run this to make sure your timezone information is up to date. (more info `Location <https://docs.moodle.org/311/en/Location>`_)
- Set server's local timezone inside php.ini (should probably be inside /etc/php.ini or /etc/php.d/date.ini, depending on the underlying OS):
  .. code-block::

      [Date]
      ; Defines the default timezone used by the date functions 
      date.timezone = "YOUR LOCAL TIMEZONE"

**Remaining tasks**

- Configure Cron: Moodle's background tasks (e.g., sending out forum emails and performing course backups) are performed by a script which you can set to execute at specific times of the day. This is known as a cron script. Please refer to the `Cron instructions <https://docs.moodle.org/311/en/Cron>`_.
- Set up backups: See `Site backup and Automated course backup <https://docs.moodle.org/311/en/Site_backup_and_Automated_course_backup>`_.
- Secure your Moodle site: Read the `Security recommendations <https://docs.moodle.org/311/en/Security_recommendations>`_. Also, have a look at the `Security Checks Section under Site Administration -> Reports -> Security Checks <https://docs.moodle.org/311/en/Security_checks>`_.
- Increasing the maximum upload size: See `Installation FAQ Maximum upload file size - how to change it? <https://docs.moodle.org/311/en/Installation_FAQ#Maximum_upload_file_size_-_how_to_change_it.3F>`_.
- Check mail works: From Site administration > Server > Test outgoing mail configuration, use the link to send yourself a test email. Don't be tempted to skip this step.

**Installation is complete :)**

- Create a new course: You can now access Moodle through your web browser (

Moodle Kubernetes Operations Documentation
======================================================================

This documentation provides step-by-step instructions on how to apply, update, and scale the Kubernetes manifests for the Moodle application using `kubectl`.

Applying Manifests
------------------

**Applying Services, Deployments, and Ingress**

1. Open a terminal.

2. Navigate to the directory containing the YAML manifests.

3. Run the following command to apply the Service, Deployment, and Ingress:

    .. code-block::

        kubectl apply -f </path/to/moodlemanifest/>

   Replace `</path/to/moodlemanifest/>` with the actual filename if it differs.

4. Verify that the resources are created successfully:

    .. code-block::
    
        kubectl get services,deployments,ingress,pods -n <namespace>
    

Updating Manifests
-------------------

**Updating Deployment**

1. Make necessary changes to the `Deployment` manifest (`</path/to/moodlemanifest/>`).

2. Run the following command to apply the changes:

    .. code-block::

        kubectl apply -f </path/to/moodlemanifest/>
    

   This command updates the existing deployment with the new configuration.

3. Verify the update by checking the status of the deployment:

    .. code-block::

        kubectl get deployments -n <namespace>
    

Scaling Deployments
-------------------

**Scaling the Number of Replicas**

1. To scale the number of replicas in the `Deployment`, run the following command:

    .. code-block::

        kubectl scale deployment moodle -n <namespace> --replicas=3
    

   Replace `3` with the desired number of replicas.

2. Verify the scaling operation:

    .. code-block::

        kubectl get deployments -n <namespace>

   Ensure that the `AVAILABLE` count reflects the desired number of replicas.

Deleting Resources
------------------

**Deleting Services, Deployments, Ingress, and PersistentVolumeClaims**

1. To delete the resources, run the following command:

    .. code-block::

        kubectl delete -f </path/to/moodlemanifest/>

   This command deletes all resources defined in the manifests.

2. Verify that the resources are deleted:

    .. code-block::
    
        kubectl get services,deployments,ingress,persistentvolumeclaims,pods -n <namespace>

   Ensure that no resources are listed.

Conclusion
-----------

This documentation provides essential operations for managing the Kubernetes manifests for the Moodle application. Ensure to follow these steps carefully, and adapt them to your specific deployment requirements. If you encounter issues or have further questions, please reach out to the support team.


