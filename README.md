# docker-compose-laravel
A pretty simplified Docker Compose workflow that sets up a LEMP network of containers for local Laravel development. You can view the repo that inspired this repo [here](https://github.com/aschmelyun/docker-compose-laravel).

## Usage

To get started, make sure you have [Docker installed](https://docs.docker.com/docker-for-mac/install/) on your system, and then clone this repository.

Next, navigate in your terminal to the directory you cloned this, and spin up the containers for the web server by running `docker-compose up -d --build site`.

After that completes, follow the steps from the [src/README.md](src/README.md) file to get your Laravel project added in (or create a new blank one).

Bringing up the Docker Compose network with `site` instead of just using `up`, ensures that only our site's containers are brought up at the start, instead of all of the command containers as well. The following are built for our web server, with their exposed ports detailed:

- **nginx** - `:8081`
- **mysql** - `:3307`
- **php** - `:9000`
- **redis** - `:6380`
- **mailhog** - `:8026` 

Three additional containers are included that handle Composer, NPM, and Artisan commands *without* having to have these platforms installed on your local computer. Use the following command examples from your project root, modifying them to fit your particular use case.

- `docker-compose run --rm composer update`
- `docker-compose run --rm npm run dev`
- `docker-compose run --rm artisan migrate`

## Using BrowserSync with Laravel Mix

If you want to enable the hot-reloading that comes with Laravel Mix's BrowserSync option, you'll have to follow a few small steps. First, ensure that you're using the updated `docker-compose.yml` with the `:3000` and `:3001` ports open on the npm service. Then, add the following to the end of your Laravel project's `webpack.mix.js` file:

```javascript
.browserSync({
    proxy: 'site',
    open: false,
    port: 3000,
});
```

From your terminal window at the project root, run the following command to start watching for changes with the npm container and its mapped ports:

```bash
docker-compose run --rm --service-ports npm run watch
```

That should keep a small info pane open in your terminal (which you can exit with Ctrl + C). Visiting [localhost:3001](http://localhost:3001) in your browser should then load up your Laravel application with BrowserSync enabled and hot-reloading active.

## MailHog

The current version of Laravel (9 as of today) uses MailHog as the default application for testing email sending and general SMTP work during local development. Using the provided Docker Hub image, getting an instance set up and ready is simple and straight-forward. The service is included in the `docker-compose.yml` file, and spins up alongside the webserver and database services.

To see the dashboard and view any emails coming through the system, visit [localhost:8026](http://localhost:8026) after running `docker-compose up -d site`.

## Setup steps

```bash
# spin up the containers for the web server
$ docker-compose up -d --build site

# delete the readme file
$ cd ./src && rm README.md

# install a brand new Laravel project in ./src (you could clone here an existing project)
$ docker-compose run --rm composer create-project laravel/laravel .

# modify the database section in ./src/.env as follows
DB_CONNECTION=mysql
DB_HOST=db
DB_PORT=3306
DB_DATABASE=new_project-local-db
DB_USERNAME=new_project-local-user
DB_PASSWORD=password

# run the migrations
$ docker-compose run --rm artisan migrate --seed

# At this point you should see the site up and running in http://localhost:8081/

# the following steps are optional to generate a login / registration scaffolding 
# instal laravel/ui dependency
docker-compose run --rm composer require laravel/ui

# generate login / registration scaffolding in react, you could use vue instead
docker-compose run --rm artisan ui react --auth

# install packages
docker-compose run --rm npm install

# compile the frontend project
docker-compose run --rm npm run dev
# you may have to run the command above again after the "mix" error

# At this point you should have a login/register section in the navbar header in http://localhost:8081/
```

## Troubleshooting
- we use `node:13.7` because the newest versions throw permission errors.
- we use place the database volume at `~/.docker/data/new_project/db` otherwise we get some errors when placing it inside the project.