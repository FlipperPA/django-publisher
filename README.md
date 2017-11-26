# Django Publisher

This package provides Django management commands to publish your site to various instances (develop, stage, production) over SSH via Paramiko.

## Pre-Requisites

With great power comes great responsibility! Target servers (`DJANGO_PUBLISHER_INSTANCES['instance']['servers']`) must each have `git` and `virtualenv` installed, and support Linux-style OS commands. Target servers must have a user (`DJANGO_PUBLISHER_INSTANCES['instance']['server_user']`) with keys set up from the control machine where you run the Django command from. This typically means installing the control machine account's public key into the target server's user account's `AUTHORIZED_KEYS`.

## Installation and Required Django Settings

Install via pip into your development environment:

```bash
pip install django-publisher
```

Then add `django_publisher` to your `INSTALLED_APPS`. Next, we need to configure your instances in Django's settings; these can live in your development settings, as they won't be required by production.

```python
DJANGO_PUBLISHER_INSTANCES = {
    'develop': {
        'name': 'django-publisher',
        'repository': 'git@github.com:FlipperPA/django-publisher.git',
        'branch': 'develop',
        'settings': 'config.settings.develop',
        'requirements': 'requirements/develop.txt',
        'code_path': '/var/django/sites',
        'virtualenv_path': '/var/django/virtualenvs',
        'virtualenv_python_path': '/usr/bin/python3.6',
        'servers': ['devserver.example.com'],
        'server_user': 'deploy_user',
    },
    'production': {
        'name': 'django-publisher',
        'repository': 'git@github.com:FlipperPA/django-publisher.git',
        'branch': 'master',
        'settings': 'config.settings.master',
        'requirements': 'requirements/master.txt',
        'code_path': '/var/django/sites',
        'virtualenv_path': '/var/django/virtualenvs',
        'virtualenv_python_path': '/usr/bin/python3.6',
        'servers': ['prodserver1.example.com', 'prodserver2.example.com'],
        'server_user': 'deploy_user',
    },
}
```

* `name`: A name for your project.
* `repository`: The repository for your Django project, which will be cloned on each target server.
* `branch`: The branch to check out for the instance.
* `settings`: A full path to the Django settings for the instnace.
* `requirements`: A relative path to a `requirements` file to be `pip install`'d for the instance.
* `code_path`: The root path for your code repository to be checked out to on the target servers.
* `virtualenv_path`: The root path for your `virtualenv` to be created on the target servers.
* `virtualenv_python_path`: The full path to the version of Python for the `virtualenv` to use on the target servers.
* `servers`: A list of servers to deploy the Django project to.
* `server_user`: The user on the target servers which has been set up with keys from the control machine.

## Running the Command

```bash
python manage.py publish --instance=develop
```

* `instance`: Required. The name of the instance to publish in `DJANGO_PUBLISHER_INSTANCES`. In the example above, either `develop` or `production`.
* `quiet`: Less verbose output. Does not display the output of the commands being run to the terminal.
* `stamp`: By default, Django Publisher will append a datetime stamp to each code clone and virtualenv. This overrides the datetime default.

## What It Does

The `publish` command will SSH to each server in `servers` as the `server_user`, and perform the following functions:

* clone the repository from git with a stamp
* create a `virtualenv` with a stamp
* run the `collectstatic` command
* run the `migrate` command
* create symlinks to point to the completed publish

## Known Limitations and Issues

* Windows servers are not supported, however, you can use Windows as your control machine.
* Your repository's host must be in your target server's known hosts list, as git checkouts over SSH require an initial fingerprint.
* This is not meant to be a replacement for a fully featured continous integration product, like Jenkins.

## Contributors

* Timothy Allen (https://github.com/FlipperPA)
