# PHP 7 and MySQL for Bitbucket Pipelines

Bitbucket Pipelines require everything to be in a single image meaning you can't do it *The Docker Way™* and split all of your services up into individual containers.

The image contains most things you need to test a PHP application.

## Example

Here's an example installing a [Laravel](https://laravel.com/) application, running the tests with a Composer locked version of [PHPUnit](https://phpunit.de) and pushing code coverage information to [Codecov](https://codecov.io):

    image: punkstar/bitbucket-pipelines-php7-mysql
    pipelines:
      default:
        - step:
            script:
              - php -r "file_exists('.env') || copy('.env.ci', '.env');"
              - service mysql start
              - mysql -h localhost -u root -proot -e "CREATE DATABASE test;"
              - mv /etc/php5/cli/conf.d/20-xdebug.ini /etc/php5/cli/conf.d/20-xdebug.ini.disabled
              - composer config -g github-oauth.github.com lalalalalala
              - composer install --no-interaction --no-progress --prefer-dist
              - php artisan key:generate
              - php artisan migrate
              - mv /etc/php5/cli/conf.d/20-xdebug.ini.disabled /etc/php5/cli/conf.d/20-xdebug.ini
              - vendor/bin/phpunit --coverage-clover=coverage.xml
              - if [ $? -eq 0 ]; then bash <(curl -s https://codecov.io/bash); fi
