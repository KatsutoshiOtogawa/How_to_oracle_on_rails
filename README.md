# How_to_oracle_on_rails


```shell
# rails dependency packages.
dnf -y install rubygem-rails sqlite-devel libsass node yarnpkg 
yarn global add webpack

# oracle dependency
# oracle database client library for using rails.
# need to build oci8

# add LD_LIBRARY_PATH. This variable need to build ruby-oci8.
# *export LD_LIBRARY_PATH=$ORACLE_HOME/lib is always need to install and update ruby-oci8*. 
export LD_LIBRARY_PATH=$ORACLE_HOME/lib
gem install ruby-oci8

# create rails project
rails new *you_project_name*
```

modify Gemfile and config/database.yml
```shell
# Gemfile add.
# Use Oracle Databaase as the database for Active Record
gem 'activerecord-oracle_enhanced-adapter', '~> 6.1.0'
gem 'ruby-oci8', '~> 2.2', '>= 2.2.9'
```

```shell
# config/database.yml
# adapter use oracle_enhanced.

default: &default
  adapter: oracle_enhanced
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  timeout: 5000

development:
  <<: *default
  database: //localhost:1521/XEPDB1
  username: railsdev
  password: railsdev
```

install rails dependency gem.
```shell
# I reccomend not use --path vendor/bundle. this option make system complex.
# if you use --path vendor/bundle and want to use bundle install and update, you always add path LD_LIBRARY_PATH=$ORACLE_HOME/lib.
bundle install
```

## oracle settings
### oracle install
If you want to know how to install oracle, check [How_to_install_oracle_in_fedora](https://github.com/KatsutoshiOtogawa/How_to_install_oracle_in_fedora). This project explain and note practical source code.
### oracle settings for rails
```shell
# create table space for using rails user and application.
sqlplus system/$ORACLE_PASSWORD@XEPDB1 << EOF
    CREATE TABLESPACE testdevtbs
    DATAFILE '/opt/oracle/oradata/XE/XEPDB1/testdevtbs001.dbf' 
    SIZE 1G
    ;
EOF

# create user for using rails.
sqlplus system/$ORACLE_PASSWORD@XEPDB1 << EOF
    CREATE USER railsdev IDENTIFIED BY railsdev
    DEFAULT TABLESPACE
        testdevtbs
    TEMPORARY TABLESPACE
        TEMP
    PROFILE
        DEFAULT
    ;
    -- CREATE SESSION need to login for oracle database.
    -- CREATE TABLE need to execute command rails db:migrate.
    -- CREATE SEQUENCE need to execute command rails db:migrate.
    GRANT CREATE SESSION,CREATE TABLE,CREATE SEQUENCE to railsdev;
    ALTER USER railsdev ACCOUNT UNLOCK;
    -- assinged table space for rails user
    ALTER USER railsdev QUOTA 10M ON testdevtbs;
EOF

# If you connect oracle database from external host, you need to port forwarding oracle port.
# 1521 is oracle database default port.
# In rhel and fedora compatible os firewalld is default firewall application and daemon.
firewall-cmd --add-port=1521/tcp --zone=public --permanent
firewall-cmd reload
```
