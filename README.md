Docker for Rails Development
Published Jun 01, 2018Last updated Nov 27, 2018
Docker for Rails Development
Developing Ruby on Rails applications in large teams could be frustrating because team members use different operative systems, languages, timezones and more.

Docker is a great tool for software development since it synchronises your team with the same setup for everyone who collaborates in your project. That's fantastic!

Why?
Your team will have an stable development environment
Development it's a mirror of Production
No bugs caused by environment
Because Windows XD
New team members will love it!
How?
Let's break this into multiple sections, we will cover everthing needed to build badass Rails applications in Dockerized environments.

Rails in a Docker Container
Running rails in a Docker Container is pretty straightforward, let's begin by creating a Rails application and then configure a custom Docker Image so we can run the Rails application from there.

Rails application
Let's start by creating a new Rails application.

I'm assuming you have Rails 5.2 installed if not please follow one of this Guides.

$ rails new myapp --skip-test --skip-bundle

Docker Image
Now that we have a Rails application, let's create a simple Dockerfile in our Rails application root.
This will help us running our Rails application in Docker.

Dockerfile

FROM ruby:2.5
RUN apt-get update -yqq
RUN apt-get install -yqq --no-install-recommends nodejs
COPY . /usr/src/app/
COPY Gemfile* /usr/src/app/
WORKDIR /usr/src/app
RUN bundle install
Line #1 Sets Ruby 2.5 as the Ruby version for our custom Docker Image
Line #2 Update the Package list
Line #3 Install Node.js (needed for the Asset Pipeline)
Line #4 Copy our Rails application files from our local directory into the container
Line #5 Gemfile caching
Line #6 Sets the working directory for the Docker Image
Line #7 Installs the Ruby gems

Please make sure you are in the Rails application root directory.

$ docker build .

...
Removing intermediate container 487d39dad5ff
 ---> c2183f884d23
Successfully built c2183f884d23
Take a look at the custom image identifier (c2183f884d23) because we will use it next. Your identifier will be different of course.

We should now be able to list all of our available Docker Images.

$ docker images

REPOSITORY          TAG                  IMAGE ID            CREATED              SIZE
<none>              <none>               c2183f884d23        About a minute ago   381MB
As you can see the custom Docker Image is available.

Now that we have created our own Docker Image, it's time to run the application.

Running Rails
To get the application up and running we will execute the following command which basically says "start a containter out of our image (c2183f884d23) and then run rails s -b 0.0.0.0 inside it". The -b tells our Rails server to bind all IP addresses not just localhost.

$ docker run -p 3000:3000 c2183f884d23 rails s -b 0.0.0.0

The Rails server boots up,

=> Booting Puma
=> Rails 5.2.0 application starting in development
=> Run `rails server -h` for more startup options
Puma starting in single mode...
* Version 3.11.4 (ruby 2.5.1-p57), codename: Love Song
* Min threads: 5, max threads: 5
* Environment: development
* Listening on tcp://0.0.0.0:3000
Visit localhost:3000 and you should see Rails welcome page. Yay! You’re on Rails!.

Improve the Docker Image
Using the custom image ID as a reference to start our Rails application is not cool at all because it is not easy to remember this number, let's tag this image so it is easy to remember:

$ docker tag c2183f884d23 app

If we list our Docker images:

REPOSITORY    TAG    IMAGE ID    CREATED    SIZE
app           latest      c2183f884d23        5 days ago          1.03GB

You can use tags to version your custom Docker Image. All about Docker tag command here.

With the new Docker Image name app we can make run the Rails application easily.

$ docker run -p 3000:3000 app rails s -b 0.0.0.0

Notice we are no longer using the Docker Image ID but instead the tag name.

Now let's add a default command to out image, using CMD:

Dockerfile

FROM ruby:2.5
RUN apt-get update -yqq
RUN apt-get install -yqq --no-install-recommends nodejs
COPY . /usr/src/app/
COPY Gemfile* /usr/src/app/
WORKDIR /usr/src/app
RUN bundle install
CMD ["rails", "s", "-b", "0.0.0.0"]
There are some files we can ignore in our Docker Image using .dockerignore and feel free to add more here as needed:

.dockerignore

.git
.gitignore
log/*
tmp/*
Then, let's rebuild the Docker Image with the new CMD instruction:

$ docker build -t app .

Start the Rails application without the rails s -b 0.0.0.0 using the new tag name app:

$ docker run -p 3000:3000 app

So far we have covered Dockerfile, .dockerignore and docker tag.
Next we will switch to Docker Compose!

Rocking with Docker Compose
With Docker Composeyou describe each of your application services.

Introducing docker-compose.yml

version: '3'
services:
  web:
    build: .
    ports:
      - "3000:3000"
    volumes:
      - .:/usr/src/app
    command: bash -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3000 -b '0.0.0.0'"

We will remove the CMD ["rails", "s", "-b", "0.0.0.0"] line from our Dockerfile since we are running the same command from the docker-compose.yml.

This step is very important since if we don't do it we could get this error:
A server is already running. Check /usr/src/app/tmp/pids/server.pid

Dockerfile

FROM ruby:2.5
RUN apt-get update -yqq
RUN apt-get install -yqq --no-install-recommends nodejs
COPY . /usr/src/app/
COPY Gemfile* /usr/src/app/
WORKDIR /usr/src/app
RUN bundle install
Now, let's build the Docker Image with Docker Compose:

$ docker-compose up --build

Our Rails application is up and running, thanks to Docker Compose, YAY!

Docker Compose useful commands
This Docker Compose commands will help you during development and you will use them everyday!

Builds, (re)creates, starts, and attaches to containers for a service:

$ docker-compose up

Lists containers:

$ docker-compose ps

Managing containers lifecycle:

$ docker-compose [start|stop|kill|restart|pause|unpause|rm] SERVICE

Displays log output from services.:

$ docker-compose logs [SERVICE...]

Run arbitrary commands in your services:

$ docker-compose exec SERVICE COMMAND

Runs a one-time command against a service.:

$ docker-compose run SERVICE [COMMAND]

Rebuilding a Docker Image:

$ docker-compose build [SERVICE...]

Summary
We learned how to use Docker for Rails development, we covered some key concepts like: Dockerfile and Docker Compose.

Now you should be ready to start developing Rails applications in a Dockerized environment!

Rails 5
Docker
Docker compose
