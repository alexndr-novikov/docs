# Getting started — Deployment

Deploying a Spiral Framework application can be a complex task, as it requires various steps to be performed in order to
ensure that the application is properly configured and ready for production use. In this guide, we will go over
several strategies for deploying an application, including basic file transfer, using source control, build scripts,
and Docker builds.

## Preparing the application

Deploying a Spiral Framework application on a production server requires several key configurations to ensure proper
operation and security.

> **Warning**
> Do not store `.env` file in your repository as it may contain sensitive information such as database credentials,
> API keys, and other secrets.

Here is a step-by-step guide on how to configure a Spiral application on a production server:

### Disable debug mode

Make sure that the `DEBUG` variable is set to `false` in your application's environment configuration. This will prevent
sensitive information from being displayed in case of an error.

```dotenv .env
DEBUG=false
```

### Set the environment to production

Change the `APP_ENV` variable in your application's environment configuration to `production`. This will prevent
accidental actions that should only be run on a development server.

```dotenv .env
APP_ENV=production
```

### Set the verbosity level

Set the `VERBOSITY_LEVEL` variable to a `basic` level to hide server errors from being displayed to the public.

```dotenv .env
VERBOSITY_LEVEL=basic
```

### Configure the logger

Set the desired `MONOLOG_DEFAULT_CHANNEL` in the application's environment configuration. This will determine where the
application's logs will be stored. Also set the `MONOLOG_DEFAULT_LEVEL` variable to `error` in your application's
environment configuration. This will prevent debug and info errors from being logged, helping to keep your logs clean
and easy to read.

```dotenv .env
MONOLOG_DEFAULT_CHANNEL=roadrunner
MONOLOG_DEFAULT_LEVEL=error
```

### Cycle ORM

If you are using the Cycle ORM, make sure that the `CYCLE_SCHEMA_CACHE` and `CYCLE_SCHEMA_WARMUP` variables are set to
`true` in your application's environment.

The `CYCLE_SCHEMA_CACHE` variable controls whether the ORM should cache the schema of the database tables. When set to
`true`, the ORM will cache the schema, which can improve the performance of the application by reducing the number of
database queries needed to retrieve the schema information.

The `CYCLE_SCHEMA_WARMUP` variable controls whether the ORM should warm up the schema cache when the application starts.
When set to `true`, the ORM will pre-populate the schema cache with the schema information, which can further improve
the performance of the application by reducing the time it takes to retrieve the schema information.

```dotenv .env
CYCLE_SCHEMA_CACHE=true
CYCLE_SCHEMA_WARMUP=true
```

### Composer

The following composer commands can be used to install vendor packages for a Spiral Framework application:

```terminal
composer install --optimize-autoloader --no-dev --no-scripts
```

This command installs the application's dependencies, generates the autoloader files for the application and optimizes
them for performance.

- `--no-dev` option is used to prevent any development dependencies from being installed ad
- `--no-scripts` option is used to prevent any `post-install` scripts from being executed.


## Nginx

If you want to use the Spiral Framework with RoadRunner and have Nginx server in front of it, you will need
to configure Nginx to act as a reverse proxy.

Here is an example of how to set up Nginx as a reverse proxy.

```nginx /etc/nginx/sites-enabled/roadrunner.conf
server {
    listen 80;

    server_name _;

    location / {
        proxy_pass http://127.0.0.1:8080;
    }
}
```

It listens on port `80` and directs all incoming requests to the IP address `127.0.0.1` on port `8080`, where
RoadRunner is running.

You can put this configuration in the `/etc/nginx/sites-available/` directory and link it to `/etc/nginx/sites-enabled/`
directory.

Here is an example of a configuration that uses a RoadRunner HTTP server:

```yaml .rr.yaml
http:
  address: 127.0.0.1:8080
```

> **Warning:**
> Do not use `address: 0.0.0.0:8080` in the RoadRunner configuration. This will prevent direct access to the RoadRunner
> HTTP server and only allow access through the Nginx reverse proxy.

## Deployer

Using automation tools like [Deployer](https://deployer.org/) can automate the deployment process and handle tasks such
as database migrations, cache clearing and more. They also allow for faster deployment and rollback processes, and can
be integrated with other tools like load balancers and monitoring systems. This can make the deployment process more
efficient and less prone to errors, and makes it easier to keep track of which version of the code is currently on the
production server, making it hard to rollback in case of errors.

Deployer is a popular deployment tool that can be used to automate the deployment process for a Spiral Framework
application. One of the advantages of using Deployer is that it can provide "zero downtime" deployment.

Zero downtime deployment is a technique that allows you to update your application without interrupting the service to
your users. This is achieved by deploying the new version of the application alongside the old version, and then
swapping them out once the new version is ready. This allows for a seamless transition, as users are not affected by any
downtime during the deployment process.

Deployer provides the capability of Zero Downtime deployment by creating a new release of the application in a different
directory and switching the symlink to the new release.

### Installing

To install Deployer, run next command in your project dir:

```terminal
composer require --dev deployer/deployer
```

### Configuration

To initialize deployer in your project run:

```terminal
./vendor/bin/dep init
```

Deployer will ask you a few questions and after finishing you will have a `deploy.php` or `deploy.yaml` file. This is
your deployment recipe. It contains hosts, tasks and requires other recipes.

Here is an example of a `deploy.php` file:

```php deploy.php
namespace Deployer;

require 'recipe/spiral.php';

set('repository', 'https://github.com/xxx/my-app');

add('shared_files', []);
add('shared_dirs', []);
add('writable_dirs', []);

host('example.org')
    ->set('remote_user', 'deployer')
    ->set('deploy_path', '/var/www/my-app');

after('deploy:failed', 'deploy:unlock');

desc('Deploys your project');
task('deploy', [
    'deploy:prepare',
    'deploy:environment',
    'deploy:vendors',
    'spiral:encrypt-key',
    'spiral:configure',
    'deploy:download-rr',
    'deploy:publish',
    'deploy:restart-rr'
]);
```

To connect to the remote host we need to specify an identity key or private key. We can add our identity key directly
into the host definition, but it's better to put it in the `~/.ssh/config` file:

```bash ~/.ssh/config
Host example.org
  IdentityFile ~/.ssh/id_rsa
```

Now let's provision our server. As our host doesn't have user deployer, we are going to override `remote_user` for
provision via `-o remote_user=root`.

```terminal
dep provision -o remote_user=root
```

Deployer will ask you a few questions during provisioning: php version, database type, etc. Next Deployer will configure
our server and create the deployer user. Provision takes around 5 minutes and will install everything we need to run a
website. A new website will be configured at `deploy_path`.

### Deploying

To deploy your application, run the following command:

```terminal
./vendor/bin/dep deploy
```

> **Note**
> If deploy failed, Deployer will print an error message and which command was unsuccessful. Most likely we need to
> configure the correct database credentials in `.env` file or similar.

#### CI/CD

Deployer can be used in CI/CD pipelines. 


> **See more**
> Read more about it in [Deployer CI/CD](https://deployer.org/docs/7.x/ci-cd) section.

<hr>

## Docker

Docker allows for easy deployment, scaling, and management of application containers. It also makes it easy to replicate
the production environment and test applications locally, which makes it easier to find and fix issues.

This method involves building a Docker image of the application, and then running the image on the production server.

### Dockerfile

Here is an example of a `Dockerfile` that can be used to build a Docker image for a Spiral Framework application:

```dockerfile 
# Build rr binary
FROM spiralscout/roadrunner:latest as rr

# Clone the project
FROM alpine/git as git

ARG REPOSITORY=https://github.com/xxx/my-app
ARG BRANCH=master
RUN git clone -b $BRANCH $REPOSITORY /app

WORKDIR /app/bin

# Configure PHP project
FROM php:8.1.3-cli-alpine3.15 as backend

RUN apk add --no-cache \
        curl libcurl wget libzip-dev libmcrypt-dev libxslt-dev libxml2-dev icu-dev zip

RUN docker-php-ext-install \
        opcache zip xsl dom exif intl pcntl bcmath sockets

COPY --from=rr /usr/bin/rr /app

ARG APP_VERSION=v1.0
ENV COMPOSER_ALLOW_SUPERUSER=1

WORKDIR /app

RUN curl -s https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin/ --filename=composer

RUN composer config --no-plugins allow-plugins.spiral/composer-publish-plugin false
RUN composer install --optimize-autoloader --no-dev

RUN docker-php-source delete && apk del ${BUILD_DEPENDS}

EXPOSE 8080/tcp

CMD ./rr serve -c .rr.yaml
```

You can build the image by running the following command:

```terminal
docker build . -t my-application
```

After the image is built, you need to push it to a Docker registry or hub. You can also tag the image with a version
number, such as `my-application:1.0`.

```terminal
docker push my-application:1.0
```

### Docker Compose

One of the advantages of using Docker is that you can manage environment variables for your application through the
`docker-compose` file and external `.env `files, rather than having to create `.env` files inside the container.

Here is an example of a `docker-compose.yml` file that can be used to start the application:

```yaml docker-compose.yaml
version: '3'
services:
  app:
    image: my-application:1.0
    ports:
      - "8080:8080"
    environment:
      - DEBUG=false
      - APP_ENV=production
      - ...
...
```

### Starting and Stopping the Application

You can start the application by running the following command:

```terminal
docker-compose up -d
```

and stop it by running

```terminal
docker-compose down
```

<hr>

## Source Control

A popular way to deploy an application is by using source control. This involves connecting your production server to a
remote repository, such as Git or SVN, and then deploying the application by pulling in the latest changes from the
repository.

### Example

1. Connect to the production server via SSH
2. Navigate to the root directory of your application on the server
3. Run the command `git pull origin master` to pull in the latest changes from the remote repository
4. Once the changes are pulled in, run `composer install --optimize-autoloader --no-dev` to install the dependencies
5. Run database migrations and other necessary console commands
6. Restart the RoadRunner server workers `./rr reset`

> **Warning:**
> Using this strategy without automation can make the deployment process more time-consuming and prone to errors. Every
> time a deployment is needed, developers would need to manually pull the code from the repository and run any necessary
> console commands in a specific order. This increases the risk of human error, such as running the commands in the
> wrong order or forgetting to run a command, which can lead to issues with the application.

<hr>

## File Transfer

One of the simplest ways to deploy a Spiral Framework application is through basic file transfer. This can be done using
FTP, or SCP. The process involves uploading the application files from your local machine to the production server.

### Example

1. Connect to the production server using FTP client (FileZilla, WinSCP)
2. Navigate to the root directory of your application on the server
3. Select all of the application files on your local machine and upload them to the server
4. Once the upload is complete, run `composer install --optimize-autoloader --no-dev` to install the dependencies
5. Run database migrations and other necessary console commands
6. Restart the RoadRunner server workers `./rr reset`

> **Warning:**
> This strategy is considered less efficient and less secure than other methods. It can take a long time to upload all
> the files manually, and there is a risk of data loss or corruption if the transfer is interrupted. Additionally, it
> can be difficult to keep track of which version of the code is currently on the production server, making it hard to
> rollback in case of errors.