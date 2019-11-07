ansible-pretalx
===============

.. image:: https://travis-ci.org/pretalx/ansible-pretalx.svg?branch=master
   :target: https://travis-ci.org/pretalx/ansible-pretalx

`ansible-pretalx` is an ansible role to install pretalx_ in a production environment.

To use it in your ansible playbook, either clone this git repository next to your other roles or install it with ansible-galaxy_.

You can find the variables you can set here_, along with their default values. Please refer
to the `pretalx configuration documentation`_ for further details.

.. _pretalx: https://pretalx.com
.. _ansible-galaxy: https://galaxy.ansible.com/
.. _here: https://github.com/pretalx/ansible-pretalx/blob/master/defaults/main.yml
.. _pretalx configuration documentation: https://docs.pretalx.org/en/latest/administrator/configure.html

Quick Start
-----------

You will need an Ansible playbook that uses this role, a database for Pretalx to connect to, and Nginx to front the Python server. 

You can configure variables in that playbook, overriding anything in the defaults_.

The following steps were tested on Debian 10:

0. First, make sure `Ansible is installed`_
1. Clone this repo somewhere, such as your home directory
2. Install a role to manage the database. For this quick start we'll use `ANXS' Postgres role in Ansible Galaxy`_::

    $ ansible-galaxy install anxs.postgresql

3. Write a playbook that uses both ``ansible-pretalx`` and ``anxs.postgresql``, installs Nginx, and configures everything by setting variables.

  Give this file any name, such as ``pretalx.yml``. Save this file in the parent dir containing ``ansible-pretalx/``. That is, if you cloned this repo in your homedir, put ``pretalx.yml`` in your homedir. 

  Please don't use this verbatim in production. The secret key should be a random string, and the Postgres user password should be a different random string. ::

    ---
    - hosts: localhost
      become: true
      tasks:
        - name: Install dependencies
          package:
            name: 
              - acl
              - nginx
              - python3-pip
            state: latest
        - name: Remove nginx default site
          file:
            name: "/etc/nginx/sites-enabled/default"
            state: absent
    - hosts: localhost
      become: true
      roles:
        - anxs.postgresql
        - ansible-pretalx
      vars:
        postgresql_databases:
          - name: "{{ pretalx_database_name }}"
            owner: "{{ pretalx_database_user }}"
        postgresql_users:
          - name: "{{ pretalx_database_user }}"
            pass: "{{ pretalx_database_password }}"
        postgresql_locale: "C.UTF-8"
        postgresql_ctype: "C.UTF-8"
        pretalx_database_password: "secret1"
        pretalx_secret_key: "secret2"
        pretalx_redis_location: false
        pretalx_nginx_conf_dir: "/etc/nginx/sites-enabled/"
        pretalx_tls: false

4. Run Ansible! ::

    $ ansible-playbook pretalx.yml

  If anything goes wrong, check:

  * Ansible's output
  * ``journalctl -u 'pretalx*'``
  * ``/usr/share/webapps/pretalxevent/data/logs/pretalx.log``

5. Finally, set up your Pretalx instance::

   $ sudo -u pretalx_event python3 -m pretalx init

.. _defaults: https://github.com/pretalx/ansible-pretalx/blob/master/defaults/main.yml
.. _Ansible is installed: https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html
.. _ANXS' Postgres role in Ansible Galaxy: https://galaxy.ansible.com/ANXS/postgresql
