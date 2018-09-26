Role Name
=========

A brief description of the role goes here.

Role Variables
--------------
These are settable variables for this role and are pre-defined into `defaults/main.yml` with default values.
You might need to override them according to your need. Most of them **are not suitable** for a production environment.

Base variables
```yaml
passbolt_version: # Defines app release version
passbolt_archive: # Defines app archive full name
passbolt_dl_url: # Defines download url to retrieve and install app
```

Variables related to PHP-fpm configuration
```yaml
passbolt_php_fpm_includedir: # Defines php-fpm pools config directory
passbolt_php_fpm_pool: # Defines app pool config absolute path
passbolt_php_fpm_user: # Defines which user php-fpm will run the app from
passbolt_php_fpm_group: # Defines which group php-fpm will run the app from
passbolt_php_fpm_owner: # Defines the owner of php-fpm socket
passbolt_php_fpm_group: # Defines the group of php-fpm socket
passbolt_php_fpm_mode: # Defines the mode of php-fpm socket (mostly if using file-based socket)
passbolt_php_fpm_whitelist: # Defines a list of IP/hostname to allow to talk to php-fpm
passbolt_php_fpm_listen: # Defines php-fpm socket method (file-based, host:port, etc)
passbolt_php_fpm_listen_port: # Defines php-fpm socket port when not using `unix:`-based uri from passbolt_php_fpm_listen
```

Variable related to database setup
```yaml
passbolt_dbhost: # Defines database server hostname to conntect to
passbolt_dbuser: # Defines database user
passbolt_dbpass: # Defines database password
passbolt_dbname: # Defines database name to connect to
```

Variables related to SMTP setup
```yaml
passbolt_smtp_host: # Defines SMTP server hostname to connect to
passbolt_smtp_port: # Defines SMTP server port
passbolt_smtp_auth: # Defines SMTP authentication method (TLS, SSL). Leave it blank for none.
passbolt_smtp_user: # Defines SMTP user (if authentication is set)
passbolt_smtp_pass: # Defines SMTP password (if authentication is set)
passbolt_smtp_timeout: # Defines SMTP connection timeout allowed
passbolt_smtp_sender: # Defines name of the sender
passbolt_smtp_sender_email: # Defines email of the sender

passbolt_error_report: # Defines if app has to send out any catched SQL error
passbolt_error_report_email: # Defines recipient to send notification out
```
 
Dependencies
------------
* geerlingguy.repo-remi
When variable `passbolt_install_php` is true.

* jdauphant.nginx
When variable `passbolt_install_webserver` is true.

* geerlingguy.php
when variable `passbolt_install_php` is true.

* geerlingguy.mysql
When variable `passbolt_install_dbserver` is true.

Note that if you manage to use different roles than those above, consider setting those variable to `False` and make sure to run and deploy them first.

Also, you may replace nginx web server by apache or else if this is what your infrastructure is running for. This role doesn't come with a pre-defined web configuration. It's set through NGINX variables provided by this role (see `tests/test.yml` for more details).

Example Playbook
----------------

```yaml
    - hosts: vault-servers
      vars:
        nginx_sites:
            passbolt:
                - listen 443 ssl
                - server_name vault.domain.tld
                - ssl_certificate     /etc/ssl/domain.tld.cert
                - ssl_certificate_key /etc/ssl/domain.tld.key
                - server_tokens off
                - root "{{ passbolt_webroot }}"
                - location / { try_files $uri /index.php$is_args$args; }
                - location ~ \.php(/|$) {
                  fastcgi_pass {{ passbolt_php_fpm_listen }};
                  fastcgi_split_path_info ^(.+\.php)(/.*)$;
                  fastcgi_read_timeout 500;
                  include fastcgi_params;
                  fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
                  fastcgi_param SERVER_NAME $http_host;
                  fastcgi_param DOCUMENT_ROOT $realpath_root;
                  internal;
                  }
      roles:
         - {
            role: laxathom.passbolt
        }
```

Testing
---------
* Host requirements
    * docker engine. Make sure it's installed and running.

Set up ansible env
```bash
printf '[defaults]\nroles_path=../\nhost_key_checking = False' > ansible.cfg
```

Install tests requirement
```bash
% ansible-galaxy install -r tests/requirements.yml -p tests/roles
```
then run the playbook for deployment test
```bash
% sudo ansible-playbook -vv -i tests/inventory tests/test.yml
```
Once done, you should be able to hit the deployed & running application at http://localhost:8080/
