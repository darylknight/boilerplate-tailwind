# Boilerplate Website

This repository is for the Boilerplate website at [insertdomainhere](https://insertdomainhere).

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. See deployment for notes on how to deploy the project on a live system.

-  The site is built with [Craft CMS 4](https://craftcms.com)
-  For general configuration settings, see `config/general.php`
-  See the [Craft docs](https://craftcms.com/docs/4.x/config/config-settings.html) for available config settings

### Prerequisites

Craft CMS is built on PHP, so it needs a local environment to run it. See Craft's basic requirements [here](https://craftcms.com/docs/4.x/requirements.html). This project should run in various dev environments, but it assumes [Laravel Valet](https://laravel-mix.com) by default. To run this site locally, you will need:

-  Composer 2.x
-  Apache or Nginx
-  PHP 8.0+
-  MySQL 5.7.8+ with InnoDB or MariaDB 10.5+
-  512MB+ of memory allocated to PHP
-  200MB+ of free disk space

This repository has been tested with both DDEV and Laravel Valet as a local environment. If you're using Apache instead of nginx, you'll need to download Craft's default `.htaccess` file and put it in the document root. If you're using DDEV, you'll need to prefix every command in this repository with `ddev`.

## DDEV + Vite Notes

I never got static assets working correctly with Vite, so I've added a post-start hook to DDEV (in config.yaml) that will run `npm run build` whenever the DDEV container starts. This will copy static assets like fonts from `src/public` into `web/dist` on build, so those files will be available when running npm run dev. This is also why @font-face rules are defined in `_includes/fonts.twig` instead of in CSS, because there was no way to get the font URLs resolving during `npm run dev` AND `npm run build`.
In addition, it will also run `composer install` to make sure local packages are in sync with the repo (assuming you pulled down first)

## Project Config

This Craft website uses [Project Config](https://craftcms.com/docs/4.x/project-config.html). This has a few implications when there are multiple developers working on the same project.

1. Whenever you start work on a project, check for changes in the remote branch. If there are, pull these down and run `composer install`. This will also run the scripts at the bottom of `composer.json`, which will run Project Config migrations on your local database.
2. If there are Project Config merge conflicts, it normally just means the `dateModifed` in `project.yaml` has changed, but please check you're not deleting files another developer has set up.
3. Make sure that when you deploy to the server, it runs `composer install --no-interaction`. This will also run the scripts listed at the bottom of `composer.json`.

Never overwrite the staging or production database with a local copy.
All database structure changes are made locally. Those changes are stored in Project Config, and applied to the staging and production databases on deployment. No database changes are permitted on staging or production sites.

## Installation

### DDEV Setup

If you're using DDEV, note the post-start hooks in config.yaml:

```
hooks:
   post-start:
      - exec: npm install // install npm packages so the build process can run
      - exec: npm run build // build the project and make sure assets have been copied from src/public to web/dist/
      - exec: composer install // install Craft + plugins, run migrations
```

These will run every time the container starts to make sure you have the same packages installed as any other developer working on the site.

### Creating a new site from this Boilerplate

-  Create a new repository using this one as a template
-  Clone the site
-  Create an empty database for the site
-  Duplicate the `.env.example` file as `.env`. Update the Database Configuration, change the `CRAFT_ENVIRONMENT` variable to `dev`, update the `PRIMARY_SITE_URL` and `BASE_PATH`
-  Enter a `CP_TRIGGER`. This defaults to `control` if left blank
-  Enter `on` for `SYSTEM_STATUS`
-  Run `npm update` to install the latest packages from `package.json`
-  Run `composer install` to install Craft and it's plugins from `composer.json`
-  Run `./craft setup`
-  Optionally, duplicate `scripts/.env.sh.example` as `scripts/.env.sh` and update it with the correct paths for your local environment if you want to use [Craft Scripts](https://github.com/nystudio107/craft-scripts) for pulling the database and assets through the command line.
-  Update the details in `package.json`
-  Update the site details in this README.md file

### Setting up an existing site based on this repository

-  Clone this repository
-  Duplicate the `.env.example` file as `.env`. Update the database connection details and change the `ENVIRONMENT` variable to `dev`
-  Enter a `CP_TRIGGER`. This defaults to `control` if left blank
-  Enter `on` for `SYSTEM_STATUS`
-  Run `npm update` to install the latest packages from `package.json`
-  Run `composer install` to install Craft and it's plugins from `composer.json`
-  Generate a new `APP_ID` for `.env` by running `./craft setup/app-id`
-  Copy the `SECURITY_KEY` from the server and update it in the `.env` file.
-  Optionally, duplicate `scripts/.env.sh.example` as `scripts/.env.sh` and update it with the correct paths for your local environment if you want to use [Craft Scripts](https://github.com/nystudio107/craft-scripts) for pulling the database and assets through the command line.
-  Import the database either by downloading a backup from the Utilities section inside Craft, or run `scripts/pull_db.sh` if you set up [Craft Scripts](https://github.com/nystudio107/craft-scripts)
-  Copy `config/license.key` from the server, as this isn't stored in the repository
-  You can download user-uploaded assets from the server either through SFTP, SSH, or with one of the rsync commands below

### Syncing assets

-  Staging to local: `rsync -rtP --delete ploi@SER.VER.IP.ADD.RESS:/home/ploi/staging.boilerplate.com/web/uploads/ web/uploads/`
-  Production to local:`rsync -rtP --delete ploi@SER.VER.IP.ADD.RESS:/home/ploi/boilerplate.com/web/uploads/ web/uploads/`

## Code Formatting

-  This project uses [Prettier](https://prettier.io) for automatic code formatting, with the [Prettier for Melody](https://github.com/trivago/prettier-plugin-twig-melody) plugin to make it work with Twig files. This is an opinionated way to format code which keeps spacing consistent between developers
-  The configuration for Prettier in this project is defined in `.prettierrc`
-  To ignore certain files or paths, add them to `.prettierignore`
-  It's easiest to set up Prettier to format files automatically on save (you can do this with Visual Studio Code). To do this, follow [Prettier with Twig in VS Code](https://codeknight.co.uk/blog/getting-prettier-working-with-twig-craft-cms).

### Prettier & Tailwind class sorting

The `prettier-plugin-tailwindcss` plugin is now compatible with `prettier-plugin-twig-melody`, so this project will now automatically sort Tailwind classes in the markup whenever the document is formatted. This was made possible by manually defining the pluing order in `.prettierrc`. See [Enabling Tailwind class sorting in Twig with Prettier](https://codeknight.co.uk/blog/enabling-prettier-class-sorting-in-twig-with-prettier) for details.

## Front End CSS (Tailwind)

-  This project uses [Tailwind CSS](https://tailwindcss.com)
-  Tailwind can be configured with the `tailwind.config.js` file in the project root
-  Tailwind uses PostCSS. This can be configured with the `postcss.config.js` file in the project root

## Front End Build Process (Vite)

Front end resources are compiled with [Laravel Vite](https://laravel.com/docs/10.x/vite) using nystudio107's [Vite plugin](https://nystudio107.com/docs/vite/).

-  NPM scripts are in the `package.json` file in the root
-  To start the dev server, run `npm run serve`. This will give you a list of IP addresses. Ignore them, and access the site as usual on your normal dev domain (depending on whether you're using DDEV or Valet). When you save Twig files, the page will refresh. When you save CSS or JS files, the page will reload those resources via Hot Module Replacement
-  To build front end assets for the production server, run `npm run build`
-  Vite can be configured in `vite.config.js` in the root of the repository

## Composer Scripts

Below is an explanation of what all the scripts are for in `composer.json`. These mostly relate to the deployment process.

-  `craft-update` Runs Craft migrations (see `post-update-cmd` and `post-install-cmd`), applies Project Config & clears caches
-  `deploy-staging` When deploying to the staging site via Ploi, this runs `composer install`
-  `deploy-production` When deploying to the production site via Ploi, we only need to run migrations because Ploi rsync's all the Composer packages from staging
-  `post-update-cmd` After `composer update`, run migrations
-  `post-install-cmd` After `composer install`, run migrations
-  `nuke` Removes all Composer packages & the lock file, then runs `composer update` again with a clear cache

Reference: [Composer Commands](https://getcomposer.org/doc/articles/scripts).

## Server & Hosting

-  The site is hosted on [UpCloud](https://upcloud.com/) under the client's own account
-  The server is provisioned with [Ploi](https://ploi.io) which handles deployment, security updates, databases & SSL

### Deployment

-  The staging site is deployed automatically when you push to the `main` branch
-  The production site is deployed manually by logging into Ploi > Servers > server-name > sitename.com > "Deploy to production"

## Built With

-  [Craft CMS 4](https://craftcms.com)
-  [Tailwind CSS](https://tailwindcss.com)
-  [Laravel Vite](https://laravel.com/docs/10.x/vite)

## Versioning

We use [Git](https://git-scm.com) for versioning.

## Authors

-  [CodeKnight](https://codeknight.co.uk)
