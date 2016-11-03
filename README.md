## PHP Mentoring App

[![Build Status](https://travis-ci.org/phpmentoring/webapp.svg?branch=master)](https://travis-ci.org/phpmentoring/webapp)
[![Codeship Status for phpmentoring/testapp](https://codeship.com/projects/a10984a0-df3f-0132-12f0-767a4e17443c/status?branch=master)](https://codeship.com/projects/80502)
[![Code Climate](https://codeclimate.com/github/phpmentoring/webapp/badges/gpa.svg)](https://codeclimate.com/github/phpmentoring/webapp)

### Requirements

You should have [Composer](http://getcomposer.org) installed and available. If you need to install composer, go to their _[Getting Started](https://getcomposer.org/doc/00-intro.md)_ docs. If you will be using [Vagrant](http://vagrantup.com) for local development, you'll need to have it installed as well.

### Setup

#### Using Vagrant

1. Clone this repo.
2. Go into the folder, and run `vagrant up` to start the VM.
3. Add the following entry to your hosts file: 

    192.168.56.101    www.mentoring.dev mentoring.dev

4. Run `vagrant ssh` to go into the VM.
5. Change directory to `/var/www` and run `composer install`

    If you get an error like the following:

        Could not fetch https://api.github.com/repos/vlucas/phpdotenv/zipball/732d2adb7d916c9593b9d58c3b0d9ebefead07aa, please create a GitHub OAuth token to go over the API rate limit
        Head to https://github.com/settings/tokens/new?scopes=repo&description=Composer+on+packer-virtualbox-iso-1422588891+2015-05-22+0002 to retrieve a token.
        It will be stored in "/home/vagrant/.composer/auth.json" for future use by Composer.

    You will need to navigate to the URL in a browser, authenticate with GitHub, and generate the access token.
    
6. From the same directory, run `vendor/bin/phinx init` to generate a `phinx.yml` file.
7. Edit `/var/www/phinx.yml`:

    In the `paths` section, set the `migrations` value to:

    ```{.yaml}
    migrations: %%PHINX_CONFIG_DIR%%/data/migrations
    ```

    In the `development` section, set the following MySQL credentials:

    ```{.yaml}
    name: scotchbox
    user: root
    pass: root
    ```

9. Run `vendor/bin/phinx migrate` to run the database migrations.
10. Copy `.env.example` to `.env`
11. Create a new Github Application at <https://github.com/settings/applications/new>
    - Set the **Homepage** and **Authorization callback URL** to <http://mentoring.dev>
12. Update your `.env` file and set the `GITHUB_API_KEY` and `GITHUB_API_SECRET` to the Client ID and Client Secret for your app.
13. Generate the public directory output with:

    `rm -rf public/; vendor/bin/sculpin generate; mv output_dev public; cp source/.htaccess public/;`

14. Visit <http://mentoring.dev> in your browser!

#### Running Without Vagrant

If you want to help out and cannot download or run Vagrant, you can run the application locally using PHP's built in dev server. You will need the following PHP extensions:

* php-curl
* php-intl

Steps:

1. Clone this repo.
2. Add the following entry to your hosts file. On unix-like systems, its usually found at `/etc/hosts`: 

    127.0.0.1    www.mentoring.dev mentoring.dev


3. Change directory to the directory where you cloned the project in step 1 and run `composer install`
4. Copy `.env.example` to `.env`
5. From the same directory, run `vendor/bin/phinx init` to generate a `phinx.yml` file.
6. Edit `./phinx.yml`'s development section (lines 19-21) with the following values for MySQL.

    ```{.yaml}
    name: mentoring
    user: mentoring
    pass: vagrant
    ```

    To use sqlite, in `./phinx.yml` change change the adapter to `sqlite` (line 17) the name (line 19) to `data/mentoring.db`. In `.env` change `DB_DRIVER` to `pdo_sqlite` on line 1.

7. Edit `./phinx.yml`'s `paths.migrations` value (line 2) to:

    ```
    %%PHINX_CONFIG_DIR%%/data/migrations
    ```

8. Run `vendor/bin/phinx migrate` to run the database migrations.
9. Create a new Github Application at <https://github.com/settings/applications/new>
    - Set the **Homepage** and **Authorization callback URL** to <http://mentoring.dev:8080>
10. Update your `.env` file and set the `GITHUB_API_KEY` and `GITHUB_API_SECRET` to the Client ID and Client Secret for your app.
12. Generate the public directory output with:

   `rm -rf public/; vendor/bin/sculpin generate; mv output_dev public; cp source/.htaccess public/;`

13. To run using PHP's built-in server, navigate to the root of the project and run:
   
    php -S mentoring.dev:8080 -t public public/index.php
   
14. Visit <http://mentoring.dev:8080> in your browser!


### Email in Development

Before you start the VM, you need to change your .env file. To use mailcatcher, you'll want the following config:

MAIL_HOST=0.0.0.0
MAIL_PORT=1025

Mailcatcher is installed in the VM, but to use it you need to ssh into the VM and execute the following command to start the mail server:

`mailcatcher --ip=0.0.0.0`

You can then view all emails being sent out by the app in your host machine's browser at the following address:

`http://192.168.56.101:1080`
