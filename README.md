# A small Rails server playbook for Ansible

It installs:

* Ruby 2.1.2
* PostgreSQL
* Nginx
* Puma

Instructions:

1. Copy configuration files to the right locations and update them with your own settings

  ```
  cp docs/defaults.yml.example vars/defaults.yml
  cp docs/hosts.example hosts
  ```

2. Provision your servers

  ```
  ansible-playbook -i hosts ruby-webapp.yml -t ruby,user,postgresql,nginx
  ```

3. Create a shared directory and copy sensitive files to the server like database.yml through scp

4. Deploy your application

  ```
  cap deploy
  ```
