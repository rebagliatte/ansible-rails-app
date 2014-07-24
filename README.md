# Multi stage environment setup for Rails

It installs:

* Ruby 2.1.2
* PostgreSQL
* Nginx

## Provision your servers with Ansible

1. Create a new server instance using Ubuntu 14.04 LTS on your favorite hosting provider (Linode, Digital Ocean, etc).

2. Create an SSH alias for your brand new server IP on your `~/.ssh/config` file.
  ```bash
  Host my-host-alias
    Hostname 192.0.2.0
    User root
    IdentityFile ~/.ssh/id_rsa
  ```
  Replace `my-host-alias` with the alias you'd like to use and `192.0.2.0` with your server's IP.

3. Add your SSH keys to the server.
  ```bash
  $ ssh-copy-id -i ~/.ssh/id_rsa.pub root@my-host-alias
  ```

4. Install Ansible on your local machine. If you are a mac user you might wanna use brew to do so.
  ```bash
  $ brew install ansible
  ```

5. Clone my ansible-rails repo <3
  ```bash
  $ git clone git@github.com:rebagliatte/ansible-rails-app.git
  $ cd ansible-rails-app
  ```

6. Copy the variable and inventory files for staging and production to the right locations and update them with your own settings.
  ```bash
  $ cp -a docs/group_vars/. group_vars
  $ cp -a docs/inventories/. .
  ```

7. Make sure Ansible can access staging properly.
  ```bash
  $ ansible -i staging -u root all -m ping
  ```

8. Run the playbook on staging.
  ```bash
  $ ansible-playbook -i staging site.yml -t ruby,user,postgresql,nginx
  ```
  The `-i` modifier specifies the inventory file you'll use. Possible values are `staging` and `production` for this repo.
  The `-t` modifier specifies the tags you'll execute and their order.

9. Your staging server is all set!!! Once you are ready you can run this over your production servers as well.
  ```bash
  $ ansible-playbook -i production site.yml -t ruby,user,postgresql,nginx
  ```

## Deploy your app with Capistrano

1. Create an app.
  ```bash
  $ rails new my-app
  ```

2. Add the rails capistrano gems to your application's Gemfile.
  ```ruby
  gem 'puma'

  group :development do
    gem 'capistrano', '~> 3.1.0', require: false
    gem 'capistrano-bundler', '1.1.1', require: false
    gem 'capistrano-rails', '1.1.0', require: false
    gem 'capistrano3-puma', require: false
  end
  ```

3. Install the gems and capify the project.
  ```bash
  $ bundle install
  $ bundle exec cap install
  ```

4. Alter your `Capfile` to require all the gems you just installed.
  ```ruby
  # Load DSL and Setup Up Stages
  require 'capistrano/setup'

  # Includes default deployment tasks
  require 'capistrano/deploy'

  # Includes rails goodies
  require 'capistrano/bundler'
  require 'capistrano/rails/assets'
  require 'capistrano/rails/migrations'
  require 'capistrano/puma'
  ```

5. Alter your `config/deploy.rb`.
  ```ruby
  # config valid only for Capistrano 3.1
  lock '3.2.1'

  set :application, 'my-app'
  set :repo_url, 'git@github.com:my-github-user/my-app.git'

  # Ask for a branch, default one is :master
  ask :branch, proc { `git rev-parse --abbrev-ref HEAD`.chomp }.call

  # Default deploy_to directory is /var/www/my_app
  set :deploy_to, '/home/deploy/apps/my-app'

  # Default value for :log_level is :debug
  set :log_level, :info

  # Default value for :linked_files is []
  set :linked_files, %w{config/database.yml}

  # Default value for linked_dirs is []
  set :linked_dirs, %w{bin log tmp/pids tmp/cache tmp/sockets vendor/bundle public/system}

  # Set ssh user
  set :user, 'deploy'
  set :ssh_options, {
    user: fetch(:user)
  }

  # For capistrano-puma
  set :puma_init_active_record, true

  # For capistrano-bundler
  set :bundle_path, -> { shared_path.join('vendor','bundle') }
  set :bundle_flags, '--deployment'
  ```
  Make sure to replace `my-github-user` and `my-app` with your own settings.

6. Alter `deploy/staging.rb`.
  ```ruby
  application_master = '192.0.2.0'

  role :app, application_master
  role :web, application_master
  role :db,  application_master

  set :rails_env, 'staging'
  set :host, 'my-app.com'
  set :keep_releases, 2
  ```
  Replace `192.0.2.0` and `my-app.com` with your server's IP and host, and change `:rails_env` to match the file you are altering.

  Do the same for your `deploy/production.rb` file.

7. Create a `config/environments/staging.rb` file based on `config/environments/production.rb` if you don't have one already in place.

8. Make sure your `database.yml` has its staging and production environments in place and the credentials match the ones you specified on your Ansible variables.
  ```yml
  development: &default
    adapter: postgresql
    pool: 5
    timeout: 5000
    host: localhost
    encoding: unicode
    database:

  test:
    <<: *default
    database:
    min_messages: warning

  production:
    <<: *default
    database:
    username:
    password:

  staging:
    <<: *default
    database:
    username:
    password:
  ```

9. Copy this file to your servers.
  ```bash
  $ scp config/database.yml deploy@my-host-alias:/home/deploy/apps/my-app/shared/config/database.yml
  ```

10. Let your servers access your repo by adding deploy keys for each one of them.
  ```bash
  ssh deploy@my-host-alias
  ssh-keygen -t rsa
  cat ~/.ssh/id_rsa.pub
  ```
  Copy your newly generated deploy keys (the output of the last command) and [add them as deploy keys on your Github repo](https://developer.github.com/guides/managing-deploy-keys/#setup-2).

11. Deploy staging
  ```bash
  $ cap staging deploy
  ```

12. All set!!! Once you are ready you can deploy production as well with `cap production deploy`.
