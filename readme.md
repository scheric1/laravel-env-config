# Laravel App Deployment
The following is instructions on how to configure the application using Litespeed Server Environment

## Installation

### Install Git Directory

Configure the Repo for pushing updates to

`mkdir /var/repo
cd /var/repo`

Initialize Repo

```
mkdir projectname.git
cd projectname.git
git init --bare
```

Change HEAD to branch you intend to be pushing. Default is `master` but typically we want `main` now

`git symbolic-ref HEAD refs/heads/main`


### Post Receive

create a receiving hook
```
cd hooks/
vi post-receive
```

```
#!/bin/sh
git --work-tree=/home/domain.com --git-dir=/var/repo/domain.com.git checkout -f
chown -R username:username /home/domain.com/
cd /home/domain.com/ && npm run production
rm -R /home/domain.com/public/storage
cd /home/domain.com/ && php artisan storage:link
chown -R username:username /home/domain.com/
```

Make the script executable
`chmod +x post-receive`



Add production remote on your *LOCAL MACHINE* so you can push updates to production

`cd /local/dev/directory`

Add Remote

`git remote add production ssh://root@server.com/var/repo/projectname.git`


PUSH to production!

`git push production master`


#### INSTALL COMPOSER FOR USER

this installs composer command for the user. You will only need to do this on your first Laravel Project.

`cd ~ &&
curl -sS https://getcomposer.org/installer | php`

`mv composer.phar /usr/local/bin/composer`



#### INSTALL COMPOSER IN PROJECT

It is a security risk to install composer as root user. Therefore, always `su - username` first. In Cyberpanel, when you create a website, the site is added to the home directory. If you aren't sure about the username cyberpanel created, navigate to `/home` and type the `ls -l` command which will show you the user that owns each project.

```
su - username
composer install --no-dev
```

#### SET CORRECT PERMISSIONS

It is good to double check we have proper write permissions on a few directories.

`chmod -R 775 /var/www/laravel/storage`

`chmod -R 775 /var/www/laravel/bootstrap/cache`

#### CONFIGURE .ENV

If you introduced new env variables in your project, it is a good idea to commit them to your example env file. This way you can see them when you deploy a new project. Your regular .env file is an excluded file from git as a security measure. This way you don't accidently commit real credentials to your repo.

`cp .env.example .env`

Update necessary env variables.


#### GENERATE ENCRYPTION KEY

`php artisan key:generate`

#### CLEAR CACHE

`php artisan config:cache`

#### MIGRATE DATABASE

`php artisan migrate`

Note: If you've already populated your database in a test environment and want to maintain that same data, you can simply do a dump of the database and import it from cyberpanel using phpmyadmin instead of installing with a clean database. This is entirely dependent on your project needs.
