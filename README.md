# A small Rails server playbook for Ansible

It installs:

* Ruby 2.1.2
* PostgreSQL
* Nginx

Instructions:

1. Create a new server instance. Note this playbook was created to play nicely with Ubuntu 14.04 LTS.

2. Copy the variable and inventory files to the right locations and update them with your own settings.

  ```
  cp docs/all group_vars/all
  cp docs/production production
  cp docs/staging staging
  ```

3. Add your ssh keys to your servers.

  ```
  ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.0.2.0
  ```

  Where `192.0.2.0` is your server's IP. Repeat this step for each server you want to provision.

4. Make sure ansible can access staging properly.

  ```
  ansible -i staging all -m ping
  ```

5. Run your playbook on staging.

  ```
  ansible-playbook -i staging site.yml -t ruby,user,postgresql,nginx
  ```

  The `-i` modifier specifies the inventory file you'll use. Possible values are `staging` and `production` for this repo.
  The `-t` modifier specifies the tags you'll excecute and their order.

6. Deploy your application.

  ```
  cap staging deploy
  ```

7. After you are all set you can excecute the same steps on your production servers.

  ```
  ansible-playbook -i production site.yml -t ruby,user,postgresql,nginx
  cap production deploy
  ```
