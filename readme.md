# About the project
Simple application that recognizes digits written on a canvas.

## Setup instructions

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
