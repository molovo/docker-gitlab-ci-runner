# Docker GitlabCI Runner images

Gitlab CI runner docker base images with ssh-key sharing.

## Table of contents

- [Base Image](#base-image)
  - [Build](#build)
  - [Usage](#usage)
- [PHP Images](#php-images)
  - [Usage](#usage-1)
  - [Custom PHP configuration](#custom-php-configuration)
  - [Development](#development)
- [NodeJS Image](#nodejs-image)
  - [Usage](#usage-2)
  - [Development](#development-1)
- [Gitlab CI setup](#gitlab-ci-setup)
- [Contributors](#contributors)

## Base Image

This docker image is based on [gitlabhq/gitlab-ci-runner](https://github.com/gitlabhq/gitlab-ci-runner) image and provide a way to pass it an ssh-key or automatically
generate a new one.
This is a base image you can extend with your own stack.

### Build

In order to build it, you need to execute the following commands:

```
docker build -t gitlabhq/gitlab-ci-runner github.com/gitlabhq/gitlab-ci-runner
docker build -t molovo/docker-gitlab-ci-runner github.com/molovo/docker-gitlab-ci-runner
```

### Usage

```
docker pull molovo/docker-gitlab-ci-runner
```

Then, you can run as many runners as you want by executing:

```
docker run \
  -e CI_SERVER_URL=https://ci.example.com \
  -e REGISTRATION_TOKEN=replaceme \
  -e HOME=/root \
  -e GITLAB_SERVER_FQDN=gitlab.example.com \
  molovo/docker-gitlab-ci-runner
```

If you need to pass an ssh key to the runner (a deploy key for example), use the following command:

```
docker run \
  -e CI_SERVER_URL=https://ci.example.com \
  -e REGISTRATION_TOKEN=replaceme \
  -e HOME=/root \
  -e GITLAB_SERVER_FQDN=gitlab.example.com \
  -v /absolute/path/to/your/home/.ssh/id_rsa:/root/.ssh/id_rsa:ro \
  molovo/docker-gitlab-ci-runner
```

If you don't mount this optional volume, an ssh-key will be automatically generated and the public key will be displayed
on standard output.

If you need to start a bash inside your container, use the following command:

```
docker run \
  -e CI_SERVER_URL=https://ci.example.com \
  -e REGISTRATION_TOKEN=replaceme \
  -e HOME=/root --rm -it \
  molovo/docker-gitlab-ci-runner:latest /bin/bash
```

## PHP Images

We provide docker gitlab-ci runner images for php 5.4, 5.5 and 5.6 containing the following stack:

- PHP 5.x
- Git
- Composer

### Usage

You can run as many runners as you want by executing:

```
docker run \
  -e CI_SERVER_URL=https://ci.example.com \
  -e REGISTRATION_TOKEN=replaceme \
  -e HOME=/root \
  -e GITLAB_SERVER_FQDN=gitlab.example.com \
  -v /absolute/path/to/your/home/.ssh/id_rsa:/root/.ssh/id_rsa:ro \
  molovo/docker-gitlab-ci-runner-php5.6
```

In your GitlabCI, a basic phpunit job could looks like this:

```
composer install
php vendor/phpunit/phpunit/phpunit --coverage-text
```

By displaying code coverage as text, you can easily extract code coverage metrics. In your project settings, under
"Test coverage parsing", just input the following regex: `  Lines:\s+(\d+.\d+\%)`.

### Custom PHP configuration

All PHP images comes with the following extensions pre-installed but not enabled by default with the exception of
`xdebug`:

- memcache.so
- mongo.so
- redis.so
- ssh2.so
- xdebug.so

If you need to customize the php configuration, you can add your own settings to the `ci-runner.ini` file.
For example, the following command in one of your gitlab-ci jobs will enable `mongo.so` extension:

```
echo 'extension="mongo.so"' >> ~/.phpenv/versions/$(phpenv version-name)/etc/conf.d/ci-runner.ini
```

You can of course customize any other parameter of the `php.ini` configuration file. Below command will set `Europe/Paris` as the default timezone:

```
echo 'date.timezone="Europe/Paris"' >> ~/.phpenv/versions/$(phpenv version-name)/etc/conf.d/ci-runner.ini
```

### Composer and Github API rate limit

All PHP images comes with Composer pre-installed and ready to be used. But, as you might already know, Github API rate limit is quite often reached when building projects in CI. See what [Composer doc say about it](https://getcomposer.org/doc/articles/troubleshooting.md#api-rate-limit-and-oauth-tokens).

One way to handle this problem is to create an `auth.json` file and share it with your gitlab-ci-runners via a volume:

```json
{
    "http-basic": {},
    "github-oauth": {
        "github.com": "GITHUB_GENERATED_TOKEN"
    }
}
```

Then, start your runner with an extra `-v` option:

```
docker run \
  -e ...
  -v /absolute/path/to/composer-auth.json:/root/.composer/auth.json:ro \
  molovo/docker-gitlab-ci-runner-php5.6
```

### Development

This docker image is based on molovo/docker-gitlab-ci-runner image. In order to build it, you need to execute the following
command:

```
docker build -t molovo/docker-gitlab-ci-runner-php5.6 github.com/molovo/docker-gitlab-ci-runner/php/5.6
```

## NodeJS Image

A docker gitlab-ci runner containing the following stack:

- NodeJS
- NPM

### Usage

You can run as many runners as you want by executing:

```
docker run \
  -e CI_SERVER_URL=https://ci.example.com \
  -e REGISTRATION_TOKEN=replaceme \
  -e HOME=/root \
  -e GITLAB_SERVER_FQDN=gitlab.example.com \
  -v /absolute/path/to/your/home/.ssh/id_rsa:/root/.ssh/id_rsa:ro \
  molovo/docker-gitlab-ci-runner-nodejs
```

### Development

This docker image is based on molovo/docker-gitlab-ci-runner image. In order to build it, you need to execute the following
commands:

```
docker build -t molovo/docker-gitlab-ci-runner-php github.com/molovo/docker-gitlab-ci-runner/nodejs
```

## Gitlab CI setup

By starting Gitlab CI runners through Docker, assuming you passed a valid registration token, they will automatically
add themselves to your CI instance. You should see them in the "Runners" tab.

## Contributors

Many thanks to the awesome [@jubianchi](https://twitter.com/jubianchi) for its help with the base php image.
