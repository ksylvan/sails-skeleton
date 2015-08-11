# sails-skeleton

These are scripts and instructions for creating a Sails.js (see http://sailsjs.org) based app, connected
to a database backend, all running in Docker containers.

## Requirements

Install Docker on your machine.

* https://www.docker.com/

On Windows, currently, this might involve installing Virtualbox and using Docker Machine to
bring up a docker host.

Once docker is installed, you'll be able to run docker-compose (included as a shell script) for the
orchestration of containers.

## Quick Start

    bin/workflow

Then follow the instructions. This lands you in a root shell in the current directory running a node image
(see https://registry.hub.docker.com/_/node/) and you have access to node and npm.

    # npm install -g sails

At this point, on Linux, in order to preserve ownership/permissions, you can run the following
command in the container:

    # /tmp/setup

It will create your user and group in the container so that files you create will be owned and
accessible by you (and not owned by root). The above is not needed on Windows.

Now, use the normal sails commands to generate your application.

    ksylvan@4bf72c677274:/usr/src/app$ sails new app
    info: Created a new Sails app `app`!

You can now exit the container. Type "exit" (or Control-D) till you are back at the prompt on your host.

Now, put the following in app/Dockerfile:

    FROM node:0.12.7-onbuild

    RUN npm install -g sails nodemon

    EXPOSE 1337

Check over the included docker-compose.yml and ensure that it is what you need. It is set up for a
sails web container connected to a redis backend (with a volume-only data container so that you
can destroy and recreate the web and db containers at will, without losing your persistent data).

At this point, you should be able to:

    ./bin/docker-compose build
    ./bin/docker-compose up

You can ignore the "WARN EACCESS" messages while the web container builds. AT this point, you will
be able to access your sails server at the usual

## Development Workflow

When you have your initial setup, you can go ahead and check your source code into your
enterprise source control repository.

Now, on Linux, you can use the following method of working with your sources:

    $ ./bin/docker-compose start db
    Starting exampleapp_db_1...

    $ ./bin/start
    About to run exampleapp_web container for development.
    Using args: -v /home/ksylvan/git/example_app/app:/usr/src/app -w /usr/src/app
    Current directory becomes /usr/src/app

    On Linux, continue by /tmp/setup in the container.
    This will preserve file ownership.

    Then continue using commands in the container.

Now, continue with running /tmp/setup as instructed, and then run "nodemon app.js":

    # /tmp/setup
    Running in the conatiner as root
    Adding group `ksylvan' (GID 1000) ...
    Done.
    [...]

    $ nodemon app.js

Now, on your host, you can use your editor of choice (e.g. https://atom.io/) and change the sources,
and nodemon will monitor and restart the sails framework automatically, allowing you to see the changes
instantly in your browser.

## Saving/Restoring the Data Volume container.

While the data volume is in its own container, which allows us to delete and recreate the database
container without losing the data, there are cases where we might need to delete all the containers
and start fresh.

In that case, we can use the "./bin/db" utility script.

    $ ./bin/db
    Usage: db help                   prints this help message.
           db save [filename]        save data to tmp/filename
           db restore [filename]     restore from tmp/filename
    If filename is not specified, defaults to data.tar.gz

Use it like this:

    $ ./bin/db save
    tar: removing leading '/' from member names
    data/
    data/dump.rdb
    Data container volumes backed up to tmp/data.tar.gz

Then, when you need to restore to a prior state of the DB, or you had to
re-install all the Docker related utilities and so had to trash everything,
do this:

    $ ./bin/db restore
    data/
    data/dump.rdb
    Data container volumes restored from tmp/data.tar.gz

This will restore the data directory from the backed up compressed archive.
