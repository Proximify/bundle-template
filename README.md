# bundle-template

This article is all about how to structure your reusable bundles to be configurable and extendable. Reusable bundles are those meant to be shared privately across many company projects or publicly so any Symfony project can install them.

# Bundle Name

A bundle is also a PHP namespace. The namespace must follow the PSR-4 interoperability standard for PHP namespaces and class names: it starts with a vendor segment, followed by zero or more category segments, and it ends with the namespace short name, which must end with Bundle.

A namespace becomes a bundle as soon as you add a bundle class to it. The bundle class name must follow these rules:

- Use only alphanumeric characters and underscores;
- Use a StudlyCaps name (i.e. camelCase with an uppercase first letter);
- Use a descriptive and short name (no more than two words);
- Prefix the name with the concatenation of the vendor (and optionally the category namespaces);
- Suffix the name with Bundle.


Here are some valid bundle namespaces and class names:

| Namespace                | Bundle Class Name |
|--------------------------|-------------------|
| `Acme\Bundle\BlogBundle` | AcmeBlogBundle    |
| `Acme\BlogBundle`        | AcmeBlogBundle    |


By convention, the getName() method of the bundle class should return the class name.

<pre>
<b>Note:</b> If you share your bundle publicly, you must use the bundle class name as the name of the repository (AcmeBlogBundle and not BlogBundle for instance).
</pre>

<pre>
Symfony core Bundles do not prefix the Bundle class with Symfony and always add a Bundle sub-namespace; for example: Symfony\Bundle\FrameworkBundle\FrameworkBundle.
</pre>

Each bundle has an alias, which is the lower-cased short version of the bundle name using underscores (acme_blog for AcmeBlogBundle). This alias is used to enforce uniqueness within a project and for defining bundle’s configuration options (see below for some usage examples).

# Directory Structure

The basic directory structure of an AcmeBlogBundle must read as follows:

<pre>
/ your-bundle
├─ AcmeBlogBundle.php
├─ Controller/
├─ README.md
├─ LICENSE
├─ Resources/
│   ├─ config/
│   ├─ doc/
│   │  └─ index.rst
│   ├─ translations/
│   ├─ views/
│   └─ public/
└─ Tests/
</pre>

<b>The following files are mandatory</b>, because they ensure a structure convention that automated tools can rely on:

- `AcmeBlogBundle.php`: This is the class that transforms a plain directory into a Symfony bundle (change this to your bundle’s name);
- `README.md`: This file contains the basic description of the bundle and it usually shows some basic examples and links to its full documentation (it can use any of the markup formats supported by GitHub, such as README.rst);
- `LICENSE`: The full contents of the license used by the code. Most third-party bundles are published under the MIT license, but you can choose any license;
- `Resources/doc/index.rst`: The root file for the Bundle documentation.

The depth of subdirectories should be kept to a minimum for the most used classes and files. Two levels is the maximum.

The bundle directory is read-only. If you need to write temporary files, store them under the cache/ or log/ directory of the host application. Tools can generate files in the bundle directory structure, but only if the generated files are going to be part of the repository.

The following classes and files have specific emplacements (some are mandatory and others are just conventions followed by most developers):

| Type                         | Directory              |
|------------------------------|------------------------|
| Commands                     | `Command/`             |
| Controllers                  | `Controller/`          |
| Service Container Extensions | `DependencyInjection/` |
| Doctrine ORM entities (when not using annotations) | `Entity/` |
| Doctrine ODM documents (when not using annotations) | `Document/` |
| Event Listeners | `EventListener/` |
| Configuration (routes, services, etc.) | `Resources/config/` |
| Web Assets (CSS, JS, images) | `Resources/public/` |
| Translation files | `Resources/translations/` |
| Validation (when not using annotations) | `Resources/config/validation/` |
| Serialization (when not using annotations) | `Resources/config/serialization/` |
| Templates | `Resources/views/`|
| Unit and Functional Tests	 | `Tests/`|

# Classes

The bundle directory structure is used as the namespace hierarchy. For instance, a ContentController controller which is stored in `Acme/BlogBundle/Controller/ContentController.php` would have the fully qualified class name of `Acme\BlogBundle\Controller\ContentController`.

All classes and files must follow the [Symfony coding standards](https://symfony.com/doc/current/contributing/code/standards.html).

Some classes should be seen as facades and should be as short as possible, like Commands, Helpers, Listeners and Controllers.

Classes that connect to the event dispatcher should be suffixed with `Listener`.

Exception classes should be stored in an `Exception` sub-namespace.

# Vendors

A bundle must not embed third-party PHP libraries. It should rely on the standard Symfony autoloading instead.

A bundle should also not embed third-party libraries written in JavaScript, CSS or any other language.

# Tests

A bundle should come with a test suite written with PHPUnit and stored under the Tests/ directory. Tests should follow the following principles:

- The test suite must be executable with a simple phpunit command run from a sample application;
- The functional tests should only be used to test the response output and some profiling information if you have some;
- The tests should cover at least 95% of the code base.


<pre>
<b>Note:</b> A test suite must not contain AllTests.php scripts, but must rely on the existence of a phpunit.xml.dist file.
</pre>

# Continuous Integration

Testing bundle code continuously, including all its commits and pull requests, is a good practice called Continuous Integration. There are several services providing this feature for free for open source projects. The most popular service for Symfony bundles is called [Travis CI] (https://travis-ci.org/).

Here is the recommended configuration file (`.travis.yml`) for Symfony bundles, which test the two latest LTS versions of Symfony and the latest beta release:

<pre>
language: php

cache:
    directories:
        - $HOME/.composer/cache/files
        - $HOME/symfony-bridge/.phpunit

env:
    global:
        - PHPUNIT_FLAGS="-v"
        - SYMFONY_PHPUNIT_DIR="$HOME/symfony-bridge/.phpunit"

matrix:
    fast_finish: true
    include:
          # Minimum supported dependencies with the latest and oldest PHP version
        - php: 7.2
          env: COMPOSER_FLAGS="--prefer-stable --prefer-lowest" SYMFONY_DEPRECATIONS_HELPER="max[self]=0"
        - php: 7.1
          env: COMPOSER_FLAGS="--prefer-stable --prefer-lowest" SYMFONY_DEPRECATIONS_HELPER="max[self]=0"

          # Test the latest stable release
        - php: 7.1
        - php: 7.2
          env: COVERAGE=true PHPUNIT_FLAGS="-v --coverage-text"

          # Test LTS versions. This makes sure we do not use Symfony packages with version greater
          # than 2 or 3 respectively. Read more at https://github.com/symfony/lts
        - php: 7.2
          env: DEPENDENCIES="symfony/lts:^2"
        - php: 7.2
          env: DEPENDENCIES="symfony/lts:^3"

          # Latest commit to master
        - php: 7.2
          env: STABILITY="dev"

    allow_failures:
          # Dev-master is allowed to fail.
        - env: STABILITY="dev"

before_install:
    - if [[ $COVERAGE != true ]]; then phpenv config-rm xdebug.ini || true; fi
    - if ! [ -z "$STABILITY" ]; then composer config minimum-stability ${STABILITY}; fi;
    - if ! [ -v "$DEPENDENCIES" ]; then composer require --no-update ${DEPENDENCIES}; fi;

install:
    - composer update ${COMPOSER_FLAGS} --prefer-dist --no-interaction
    - ./vendor/bin/simple-phpunit install

script:
    - composer validate --strict --no-check-lock
    # simple-phpunit is the PHPUnit wrapper provided by the PHPUnit Bridge component and
    # it helps with testing legacy code and deprecations (composer require symfony/phpunit-bridge)
    - ./vendor/bin/simple-phpunit $PHPUNIT_FLAGS
</pre>

Consider using the [Travis cron](https://docs.travis-ci.com/user/cron-jobs/) tool to make sure your project is built even if there are no new pull requests or commits.

# Installation

Bundles should set `"type": "symfony-bundle"` in their composer.json file. With this, [Symfony Flex](https://symfony.com/doc/current/setup.html#symfony-flex) will be able to automatically enable your bundle when it’s installed.

If your bundle requires any setup (e.g. configuration, new files, changes to .gitignore, etc), then you should create a [Symfony Flex recipe](https://github.com/symfony/recipes).

# Documentation

All classes and functions must come with full PHPDoc.

Extensive documentation should also be provided in the Resources/doc/ directory. The index file (for example `Resources/doc/index.rst or Resources/doc/index.md)` is the only mandatory file and must be the entry point for the documentation. The [reStructuredText (rST)](https://symfony.com/doc/current/contributing/documentation/format.html) is the format used to render the documentation on the Symfony website.

## Installation Instructions

<pre>
Installation
============

Make sure Composer is installed globally, as explained in the
[installation chapter](https://getcomposer.org/doc/00-intro.md)
of the Composer documentation.

Applications that use Symfony Flex
----------------------------------

Open a command console, enter your project directory and execute:

```console
$ composer require <package-name>
```

Applications that don't use Symfony Flex
----------------------------------------

### Step 1: Download the Bundle

Open a command console, enter your project directory and execute the
following command to download the latest stable version of this bundle:

```console
$ composer require <package-name>
```

### Step 2: Enable the Bundle

Then, enable the bundle by adding it to the list of registered bundles
in the `config/bundles.php` file of your project:

```php
// config/bundles.php

return [
    // ...
    <vendor>\<bundle-name>\<bundle-long-name>::class => ['all' => true],
];
```
</pre>

The example above assumes that you are installing the latest stable version of the bundle, where you don’t have to provide the package version number (e.g. composer require friendsofsymfony/user-bundle). If the installation instructions refer to some past bundle version or to some unstable version, include the version constraint (e.g. `composer require friendsofsymfony/user-bundle "~2.0@dev`").

Optionally, you can add more installation steps (Step 3, Step 4, etc.) to explain other required installation tasks, such as registering routes or dumping assets.


# Routing

If the bundle provides routes, they must be prefixed with the bundle alias. For example, if your bundle is called AcmeBlogBundle, all its routes must be prefixed with acme_blog_.

# Templates

If a bundle provides templates, they must use Twig. A bundle must not provide a main layout, except if it provides a full working application.

# Translation Files

If a bundle provides message translations, they must be defined in the XLIFF format; the domain should be named after the bundle name (`acme_blog`).

A bundle must not override existing messages from another bundle.

# Configuration

To provide more flexibility, a bundle can provide configurable settings by using the Symfony built-in mechanisms.

For simple configuration settings, rely on the default parameters entry of the Symfony configuration. Symfony parameters are simple key/value pairs; a value being any valid PHP value. Each parameter name should start with the bundle alias, though this is just a best-practice suggestion. The rest of the parameter name will use a period (`.`) to separate different parts (e.g. `acme_blog.author.email`).

The end user can provide values in any configuration file:

<b>YAML: </b> 

<pre>
# config/services.yaml
parameters:
    acme_blog.author.email: 'fabien@example.com'
</pre>

<b>PHP:</b> 

<pre>
// config/services.php
$container->setParameter('acme_blog.author.email', 'fabien@example.com');
</pre>

Retrieve the configuration parameters in your code from the container:

<pre>
$container->getParameter('acme_blog.author.email');
</pre>

While this mechanism requires the least effort, you should consider using the more advanced [semantic bundle configuration](https://symfony.com/doc/current/bundles/configuration.html) to make your configuration more robust.

# Versioning

Bundles must be versioned following the [Semantic Versioning Standard](https://semver.org/).

# Services

If the bundle defines services, they must be prefixed with the bundle alias instead of using fully qualified class names like you do in your project services. For example, AcmeBlogBundle services must be prefixed with acme_blog. The reason is that bundles shouldn’t rely on features such as service autowiring or autoconfiguration to not impose an overhead when compiling application services.

In addition, services not meant to be used by the application directly, should be [defined as private](https://symfony.com/doc/current/service_container/alias_private.html#container-private-services). For public services, [aliases should be created](https://symfony.com/doc/current/service_container/autowiring.html#service-autowiring-alias) from the interface/class to the service id. For example, in MonologBundle, an alias is created from `Psr\Log\LoggerInterface` to `logger` so that the `LoggerInterface` type-hint can be used for autowiring.

Services should not use autowiring or autoconfiguration. Instead, all services should be defined explicitly.

<b> Note: </b> You can learn much more about service loading in bundles reading this article: [How to Load Service Configuration inside a Bundle](https://symfony.com/doc/current/bundles/extension.html).

# Composer Metadata

The `composer.json` file should include at least the following metadata:

`name`
Consists of the vendor and the short bundle name. If you are releasing the bundle on your own instead of on behalf of a company, use your personal name (e.g. johnsmith/blog-bundle). Exclude the vendor name from the bundle short name and separate each word with an hyphen. For example: AcmeBlogBundle is transformed into blog-bundle and AcmeSocialConnectBundle is transformed into social-connect-bundle.

`description`
A brief explanation of the purpose of the bundle.

`type`
Use the symfony-bundle value.

`license`
a string (or array of strings) with a valid license identifier, such as MIT.

`autoload`
This information is used by Symfony to load the classes of the bundle. It’s recommended to use the PSR-4 autoload standard: use the namespace as key, and the location of the bundle’s main class (relative to `composer.json`) as value. For example, if the main class is located in the bundle root directory: `"autoload": { "psr-4": { "SomeVendor\\BlogBundle\\": "" } }`. If the main class is located in the src/ directory of the bundle: `"autoload": { "psr-4": { "SomeVendor\\BlogBundle\\": "src/" } }`.

In order to make it easier for developers to find your bundle, register it on Packagist, the official repository for Composer packages.

# Resources

If the bundle references any resources (config files, translation files, etc.), don’t use physical paths (e.g. `__DIR__/config/services.xml`) but logical paths (e.g. @FooBundle/Resources/config/services.xml).

The logical paths are required because of the bundle overriding mechanism that lets you override any resource/file of any bundle. See [Locating Resources](https://symfony.com/doc/current/components/http_kernel.html#http-kernel-resource-locator) for more details about transforming physical paths into logical paths.

Beware that templates use a simplified version of the logical path shown above. For example, an index.html.twig template located in the `Resources/views/Default/` directory of the FooBundle, is referenced as `@Foo/Default/index.html.twig`.

# Learn more

- [How to Load Service Configuration inside a Bundle](https://symfony.com/doc/current/bundles/extension.html)
- [How to Create Friendly Configuration for a Bundle](https://symfony.com/doc/current/bundles/configuration.html)