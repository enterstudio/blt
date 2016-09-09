This document outlines the process for creating a new BLT release.

# Testing

## Prerequisites

In order to use these testing instructions:

* The `blt` alias must be installed.
* Your LAMP stack must have host entry for `http://blt8-release.localhost` pointing to `./blt8-release/docroot`.
* MySQL must use `mysql://drupal:drupal@localhost/drupal:3306`. If this is not the case, modify the instructions below for your credentials.
* In order to test Drupal VM, you must install VirtualBox and Vagrant. See [Drupal VM](https://github.com/geerlingguy/drupal-vm#quick-start-guide) for more information.

## Create a new project via acquia/blt-project.

This test verifies that a new project can be created using `acquia/blt-project` via composer. This also tests the `blt update` process.

    # This downloads the latest stable version.
    export COMPOSER_PROCESS_TIMEOUT=2000
    composer create-project acquia/blt-project:8.x-dev blt8-release --no-interaction
    cd blt8-release
    ./vendor/bin/drupal yaml:update:value project.yml project.local.hostname 'blt8-release.localhost'
    blt configure
    # update local.settings.php pw
    echo '$databases["default"]["default"]["username"] = "drupal";' >> docroot/sites/default/settings/local.settings.php
    echo '$databases["default"]["default"]["password"] = "drupal";' >> docroot/sites/default/settings/local.settings.php
    blt local:setup
    drush uli
    read -p "Press any key to continue"
    
    # This updates to the latest dev version.
    composer update acquia/blt:8.x-dev
    blt update
    composer update
    dr uli
    read -p "Press any key to continue"
    cd ../

## Creates a new project without acquia/blt-project, "from scratch"

    rm -rf blt8-release
    mkdir blt8-release
    cd blt8-release
    git init
    composer init --stability=dev --no-interaction
    composer config prefer-stable true
    composer require acquia/blt:8.x-dev --dev
    blt init
    ./vendor/bin/drupal yaml:update:value project.yml project.local.hostname 'blt8-release.localhost'
    blt configure
    composer update
    echo '$databases["default"]["default"]["username"] = "drupal";' >> docroot/sites/default/settings/local.settings.php
    echo '$databases["default"]["default"]["password"] = "drupal";' >> docroot/sites/default/settings/local.settings.php
    blt local:setup
    drush uli
    read -p "Press any key to continue"
    cd ../

## Creates a new project without acquia/blt-project, "from scratch", using Drupal VM
 
    rm -rf blt8-release
    mkdir blt8-release
    cd blt8-release
    git init
    composer init --stability=dev --no-interaction
    composer config prefer-stable true
    composer require acquia/blt:8.x-dev --dev
    blt init
    ./vendor/bin/drupal yaml:update:value project.yml project.local.hostname 'blt8-release.localhost'
    blt configure
    composer update
    blt vm:init
    vagrant up
    blt local:setup
    drush @blted8.local uli
    read -p "Press any key to continue"
    vagrant destroy
    cd ../


## Generate CHANGELOG.md

### Prerequisites

* Ruby 2.2.2+ must be installed. You may use [RVM](https://rvm.io/rvm/install) to use a directory specific version of Ruby. E.g., `rvm use 2.2.2`.
* [skywinder/github-changelog-generator](https://github.com/skywinder/github-changelog-generator) must be installed. E.g., `gem install github_changelog_generator`. 
* Procure a [github api token](https://github.com/skywinder/github-changelog-generator#github-token).
* Determine the version of your future release.

Then, generate your release notes via:

    github_changelog_generator --token [token] --future-release=[version]
    
This will update CHANGELOG.md. The information for the new release should be copied and pasted into the GitHub release draft.
