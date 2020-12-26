
# ArduinoCI Ruby gem (`arduino_ci`)
[![Gem Version](https://badge.fury.io/rb/arduino_ci.svg)](https://rubygems.org/gems/arduino_ci)
[![Documentation](http://img.shields.io/badge/docs-rdoc.info-blue.svg)](http://www.rubydoc.info/gems/arduino_ci/1.1.0)
[![Gitter](https://badges.gitter.im/Arduino-CI/arduino_ci.svg)](https://gitter.im/Arduino-CI/arduino_ci?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)
[![GitHub Marketplace](https://img.shields.io/badge/Get_it-on_Marketplace-informational.svg)](https://github.com/marketplace/actions/arduino_ci)

You want to run tests on your Arduino library (bonus: without hardware present), but the IDE doesn't support that.  Arduino CI provides that ability.

You want to precisely replicate certain software states in your library, but you don't have sub-millisecond reflexes for physically faking the inputs, outputs, and serial port.   Arduino CI fakes 100% of the physical input and output of an Arduino board, including the clock.

You want your Arduino library to be automatically built and tested every time someone contributes code to your project on GitHub, but the Arduino IDE lacks the ability to run unit tests. [Arduino CI](https://github.com/Arduino-CI/arduino_ci) provides that ability.

`arduino_ci` is a cross-platform build/test system, consisting of a Ruby gem and a series of C++ mocks.  It enables tests to be run both locally and as part of a CI service like GitHub Actions, TravisCI, Appveyor, etc.  Any OS that can run the Arduino IDE can run `arduino_ci`.

> Note: for running tests in response to [GitHub events](https://docs.github.com/en/free-pro-team@latest/developers/webhooks-and-events/github-event-types), an [Arduino CI GitHub Action](https://github.com/marketplace/actions/arduino_ci) is available for your convenience.  This method of running `arduino_ci` is driven by Docker, which may also serve your local testing needs (as it does not require a ruby environment to be installed).


Platform | CI Status
---------|:---------
OSX      | [![OSX Build Status](https://github.com/Arduino-CI/arduino_ci/workflows/macos/badge.svg)](https://github.com/Arduino-CI/arduino_ci/actions?workflow=macos)
Linux    | [![Linux Build Status](https://github.com/Arduino-CI/arduino_ci/workflows/linux/badge.svg)](https://github.com/Arduino-CI/arduino_ci/actions?workflow=linux)
Windows  | [![Windows Build status](https://github.com/Arduino-CI/arduino_ci/workflows/windows/badge.svg)](https://github.com/Arduino-CI/arduino_ci/actions?workflow=windows)


## Quick Start

For a fairly minimal practical example that you can copy from, see [the `Arduino-CI/Blink` repository](https://github.com/Arduino-CI/Blink).

The complete set of C++ unit tests for the `arduino_ci` library itself are in the [SampleProjects/TestSomething](SampleProjects/TestSomething) project.  The [test files](SampleProjects/TestSomething/test/) are named after the type of feature being tested.

> Arduino expects all libraries to be in a specific `Arduino/libraries` directory on your system.  If your library is elsewhere, `arduino_ci` will _automatically_ create a symbolic link in the `libraries` directory that points to the directory of the project being tested.  This simplifieds working with project dependencies, but **it can have unintended consequences on Windows systems** because [in some cases deleting a folder that contains a symbolic link to another folder can cause the _entire linked folder_ to be removed instead of just the link itself](https://superuser.com/a/306618).
>
> If you use a Windows system **it is recommended that you only run `arduino_ci` from project directories that are already inside the `libraries` directory**

### You Need Ruby and Bundler

You'll need Ruby version 2.2 or higher, and to `gem install bundler` if it's not already there.


### You Need A Compiler (`g++`)

For unit testing, you will need a compiler; [g++](https://gcc.gnu.org/) is preferred.

* **Linux**: `gcc`/`g++` is likely pre-installed.
* **OSX**: `g++` is an alias for `clang`, which is provided by Xcode and the developer tools.  You are free to `brew install gcc` as well; this is also tested and working.
* **Windows**: you will need Cygwin, and the `mingw-gcc-g++` package.  A full set of (working) install instructions can be found in `appveyor.yml`, as this is how CI runs for this project.


### You _May_ Need `python`

ESP32 and ESP8266 boards have [a dependency on `python` that they don't install themselves](https://github.com/Arduino-CI/arduino_ci/issues/235#issuecomment-739629243).  If you intend to test on these platforms (which are included in the default list of platforms to test against), you will need to make `python` (and possibly `pyserial`) available in the test environment.

Alternately, you might configure `arduino_ci` to simply not test against these.  Consult the reference for those details.


### Changes to Your Repo

Add a file called `Gemfile` (no extension) to your Arduino project:

```ruby
source 'https://rubygems.org'
gem 'arduino_ci' '~> 1.1'
```

It would also make sense to add the following to your `.gitignore`, or copy [the `.gitignore` used by this project](.gitignore):

```
/.bundle/
/.yardoc
Gemfile.lock
/_yardoc/
/coverage/
/doc/
/pkg/
/spec/reports/
vendor
*.gem

# rspec failure tracking
.rspec_status

# C++ stuff
*.bin
*.bin.dSYM
```


### Installing the Dependencies

Fulfilling the `arduino_ci` library dependency is as easy as running either of these two commands:
```
$ bundle install   # adds packages to global library (may require admin rights)
$ bundle install --path vendor/bundle   # adds packages to local library
```


### Running tests

With that installed, just the following shell command each time you want the tests to execute:

```
$ bundle exec arduino_ci.rb
```

`arduino_ci.rb` is the main entry point for this library.  This command will iterate over all the library's `examples/` and attempt to compile them.  If you set up unit tests, it will run those as well.


### Reference

For more information on the usage of `arduino_ci.rb`, see [REFERENCE.md](REFERENCE.md).  It contains information such as:

* How to configure build options (platforms to test, Arduino library dependencies to install) with an `.arduino-ci.yml` file
* Where to put unit test files
* How to structure unit test files
* How to control the global (physical) state of the Arduino board
* How to modify the Arduino platforms, compilers, test plans, etc


## Setting up Pull Request Testing and/or External CI

The following prerequisites must be fulfilled:

* A GitHub (or other repository-hosting) project for your library
* A CI system like [Travis CI](https://travis-ci.org/) or [Appveyor](https://www.appveyor.com/) that is linked to your project


### Testing with remote CI

> **Note:** `arduino_ci.rb` expects to be run from the root directory of your Arduino project library.


#### GitHub Actions

GitHub Actions allows you to automate your workflows directly in GitHub.
No additional steps are needed.
Just create a YAML file with the information below in your repo under the `.github/workflows/` directory.

```yaml
on: [push, pull_request]
jobs:
  runTest:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: ruby/setup-ruby@v1
        with:
          ruby-version: 2.6
      - run: |
          bundle install
          bundle exec arduino_ci_remote.rb
```


#### Travis CI

You'll need to go to https://travis-ci.org/profile/ and enable testing for your Arduino project.  Once that happens, you should be all set.  The script will test all example projects of the library and all unit tests.

Next, you need this in `.travis.yml` in your repo

```yaml
sudo: false
language: ruby
script:
  - bundle install
  - bundle exec arduino_ci.rb
```


#### Appveyor CI

You'll need to go to https://ci.appveyor.com/projects and add your project.

Next, you'll need this in `appveyor.yml` in your repo.

```yaml
build: off
test_script:
  - bundle install
  - bundle exec arduino_ci.rb
```


## Known Problems

* The Arduino library is not fully mocked.
* I don't have preprocessor defines for all the Arduino board flavors
* https://github.com/Arduino-CI/arduino_ci/issues


## Author

This gem was written by Ian Katz (ianfixes@gmail.com) in 2018.  It's released under the Apache 2.0 license.


## See Also

* [Contributing](CONTRIBUTING.md)
* [Adafruit/ci-arduino](https://github.com/adafruit/ci-arduino) which inspired this project
* [mmurdoch/arduinounit](https://github.com/mmurdoch/arduinounit) from which the unit test macros were adopted
