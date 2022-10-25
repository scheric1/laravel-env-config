# Laravel App Deployment
The following is instructions on how to configure the application using Litespeed Server Environment

## Installation

### Install Git Directory

Configure the Repo for pushing updates to

`mkdir /var/repo
cd /var/repo`

```
mkdir adminrcr.git
cd adminrcr.git
git init --bare
git symbolic-ref HEAD refs/heads/main
cd hooks/
```


### Post Receive

create a receiving hook
`vi post-receive`

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

`git remote add production ssh://root@server.com/var/repo/adminrcr.git`


PUSH to production!

`git push production master`


#### FIX FPM SECURITY FOR LARAVEL

`vi /etc/php/7.2/fpm/php.ini`

Make sure the following is uncommented with the value of 0

`cgi.fix_pathinfo=0`

Restart FPM

`service php7.2-fpm restart`


#### INSTALL COMPOSER

`cd ~ &&
curl -sS https://getcomposer.org/installer | php`

`mv composer.phar /usr/local/bin/composer`






#### INSTALL COMPOSER

`cd /var/www/html/admin.repaircostreferee.com
composer install --no-dev`

#### SET CORRECT PERMISSIONS

Ownership permissions

`chown -R :www-data /var/www/html/admin.repaircostreferee.com`

RWX Permissions

`chmod -R 775 /var/www/laravel/storage`

`chmod -R 775 /var/www/laravel/bootstrap/cache`

#### CONFIGURE .ENV

`cp .env.example .env`

Noted variables that should be updated

```
APP_ENV=production
APP_DEBUG=false

DB_DATABASE=rcr
DB_USERNAME=rcr
DB_PASSWORD=XXXXXXXXX

MAIL_DRIVER=smtp
MAIL_HOST=sendgrid.com
MAIL_PORT=2525
MAIL_USERNAME=XXXXXXXXXXX
MAIL_PASSWORD=XXXXXXXXXXX
MAIL_ENCRYPTION=null
```

#### GENERATE ENCRYPTION KEY

`php artisan key:generate`

#### CLEAR CACHE

`php artisan config:cache`

#### MIGRATE DATABASE

`php artisan migrate`

#### INITIALIZE FIRST USER

Start Tinker

`php artisan tinker`

Create User and Save

```
$user = new App\User;
$user->name = 'Admin';
$user->email = 'eric.schnell@repaircostreferee.com';
$user->password = Hash::make('password');
$user->save();

exit
```
