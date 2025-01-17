# Development
This chapter provides a detailed guide to install Build locally for development.
This setup is best if you want to contribute to the codebase.

## Ruby, NodeJS, and dependencies
As root user, upgrade the system, install [NodeJS](https://nodejs.org/) and 
[Ruby](https://www.ruby-lang.org/en/documentation/installation/).

```
apt update
apt upgrade

# See: https://github.com/nodesource/distributions/blob/master/README.md#debinstall
cd ~
curl -sL https://deb.nodesource.com/setup_14.x -o nodesource_setup.sh
sh nodesource_setup.sh
apt install nodejs
# Verify version, here 14.x:
nodejs -v

# Review Ruby version
ruby -v
# Build 0.3.5 to 0.4.0 requires a Ruby upgrade
gem update --system
```

If you run into trouble while resolving dependencies, try updating Bundler: `gem update --system && gem install bundler`.

Note: If you run into this error while executing `bundle install`
```
An error occurred while installing pg (<some version>), and Bundler cannot continue.
Make sure that `gem install pg -v '<some version>'` succeeds before bundling.
```

Then installing `pg` with `gem install pg -v '0.19.0'` would solve the issue. 
Sometimes it might complain that the `libpq` library is not found. This might happen for several reasons:

* Lib paths are mis-configured.
* `libpq` library is not installed at all. 
* On some platforms setting the `ARCHFLAGS` env to `-arch x86_64` and then installing pg would work. 
  So run this command: `sudo ARCHFLAGS="-arch x86_64" gem install pg -v '<some version>'`

## Code
Clone the Build code to a local folder of your choice. 

```
cd ~/projects
git clone git@github.com:getodk/build.git
cd build
```

The subsequent commands are run inside the cloned Build repository.

## Database
Follow the official [Postgres docs](https://www.postgresql.org/) to setup a cluster.
You can find the currently used Postgres version in `docker-compose.yml`.
Configure access for local users, then create a role and database `odkbuild`.

```
create role odkbuild with login password 'odkbuild';
create database odkbuild with owner='odkbuild' encoding='utf8';
```

Create `config.yml` as a copy of the template `config.yml.sample`.
If you have used different credentials to the code example above, update them in `config.yml`.
As `config.yml` contains a number of secret keys and tokens, it is excluded from both source control (via `.gitignore`)
and the Docker build (via `.dockerignore`).

Finally, run migrations against the database.

```
rake db:migrate
```

## Install
Install Build's Ruby gems and compile the Javascript assets.

```
bundle config set --local deployment 'true'
bundle install
bundle exec rake deploy:build
```

## build2xlsform
The export to XLSForm depends on [build2xlsform](https://github.com/getodk/build2xlsform). 
Follow its README to install `build2xlsform` locally as a separate project and run it on its default port 8686.

## Run
To finally run Build, run either `bundle exec rackup config.ru` to start the server, or `bundle exec shotgun config.ru` if you want the application to automatically detect your changes to source code and load them up when you refresh the app in your web browser.

```
# Static
bundle exec rackup config.ru

# Live reload
bundle exec shotgun config.ru
```

In production, assets are bundled by a pre-processor. It is worth running this to verify that the assets build correctly. 
In some cases, Javascript code can run fine through `bundle exec shotgun config.ru`, but fails to bundle correctly and therefore crashes the production deployment.

```
rake deploy:build
```

If JS files are not compiled cleanly by the JS compressor (Yui), the asset compilation tool may not throw any error. We just get a silent failure and blank line in the compiled assets. These errors can come e.g. from XForms terms which are reserved keywords, or at least trip over the compiler. One such example is the key "short". Be also aware that the JS code does not support ES6 syntax such as parameter defaults.

To view compilation errors, run the Yui jar file directly:

* Determine the path of the Java JAR `yuicompressor.jar` with `bundle info yui-compressor`. 
  Note that the Ruby Gem `yui-compressor` (in the example v0.9.3) contains the Java JAR `yuicompressor-x.x.x.jar` (in the example v2.4.2).
* Use the path to `yui-compressor.jar` to compile each asset you changed with option `-v` to see any resulting errors.

```
bundle info yui-compressor
java -jar /srv/odkbuild/releases/0.3.6/vendor/bundle/ruby/2.7.0/gems/yui-compressor-0.9.3/lib/yuicompressor-2.4.2.jar -v --type js public/javascripts/data.js
```

## Contribute
Once a contribution is ready, consult the [contributing guide](contribute.md) on preparing a Pull Request.

## Release
Once Build is ready for a new release, a team member with "maintainer" privileges will create a "release issue" and follow instructions therein.
