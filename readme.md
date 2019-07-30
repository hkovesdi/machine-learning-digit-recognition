Instructions to setup the project:

- cp .env.example .env
- git submodule init
- git submodule update
- cd docker-for-digit-recognition
- cp .env.example .env
- docker-compose up -d --build workspace nginx php-fpm python-flask
- docker-compose exec workspace bash
- composer install
- npm install
- php artisan key:generate
- npm run prod