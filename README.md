# A small Rails server playbook for Ansible

It installs:

* Ruby 2.1
* PostgreSQL
* nginx
* Puma (jungle)

Instructions:

1. Copy configuration files to the right locations and update them with your own settings

  ```
  cp docs/defaults.yml.example vars/defaults.yml
  cp docs/hosts.example hosts
  ```

2. Provision your servers

  ```
  ansible-playbook -i hosts ruby-webapp.yml -t ruby,deploy,postgresql,nginx
  ```

3. Deploy your application

  ```
  cap deploy
  ```

4. Start puma

  ```
  ansible-playbook -i hosts ruby-webapp.yml -t puma
  ```
