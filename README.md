> **Work in Progress** 👀
>
> lerna-cola is being built in parallel to a large scale production grade project, thereby getting some serious dogfooding in order to work out the kinks and settle on a useful API. Whilst we have made a lot of progress this is still very much an alpha version of the project.

---

# Lerna Cola 🥤

Superpowers for your [Lerna](https://lernajs.io/) monorepos.

Clean, build, develop, and deploy your packages utilising a rich plugin ecosystem.

## TOC

- [Introduction](#introduction)
- [Requirements](#requirements)
- [(Not Really) Requirements](#not-really-requirements)
- [Installation](#installation)
- [Sample Application](#sample-application)
- [Configuration](#configuration)
  - [Example Configuration](#example-configuration)
  - [Configuration Schema](#configuration-schema)
- [CLI Commands](#cli-commands)
  - [build](#build)
  - [develop](#develop)
  - [deploy](#deploy)
- [Plugins](#plugins)
  - [Core Plugins](#core-plugins)
  - [Official Plugins](#official-plugins)
  - [3rd Party Plugins](#3rd-party-plugins)
- [Schemas](#schemas)
  - [`Package` Schema](#package-schema)
  - [`Config` Schema](#package-schema)
- [Plugin Development](#plugin-development)

## Introduction

[Lerna](https://lernajs.io/) makes it crazy easy to manage cross package dependencies and provides sane methods to version them. It takes away the fear of creating and maintaining packages allowing us to fully embrace the Node.js module ethos of creating packages with isolated responsibilities.

Lerna Cola wants to build on top of these core principles by providing the following additional features:

- Easily **enrich your packages** with a **compilation/transpilation/bundling** step (babel/flow/typescript/reasonml/webpack/parcel/etc/etc/etc).
- Take away the fear of building a wide set of **microservices/lambda packages** by providing a **rich development service** that handles **hot/auto reloading** of your packages. This allows for a **fluid development experience** reminiscent of old school single package/repository based development.
- **Deploy** your packages with a simple command to a **cloud provider** of your choice.

You access the features via one of the 4 CLI commands that Lerna Cola provides: [`clean`](#clean), [`build`](#build), [`develop`](#develop), and [`deploy`](#deploy).

The commands utilise a rich plugin eco-system, allowing for 3rd party contributions.

Lift your build, development and deployment configurations to the root of your monorepo, keep your packages clean, and utilise the full benefits of a monorepo structure.

## Requirements

- **Node >= 8**

  Version 8 was LTS at the time of writing this so a decision was made to run with it. This of course is only a requirement for the `lerna-cola` CLI, your packages can have their own individual engine requirements.

## (Not Really) Requirements

- **[Lerna](https://lernajs.io/)**

  To tell you the truth, we don't strictly require that you use Lerna. You could very well use straight up Yarn workspaces, but in our opinion you would be missing out on some cool features. We very very very much recommend that you use this library in conjunction with Lerna.

## Installation

Simply add the Lerna Cola CLI package as a dev dependency to the root of your monorepo:

```bash
yarn add @lerna-cola/cli -DW
```

_or, via NPM_:

```bash
npm i @lerna-cola/cli -D
```

## Sample Application

We definitely recommend that you read the documentation below to gain a full understanding of Lerna Cola, however, we want to highlight early on that a sample application is maintained here:

- https://github.com/ctrlplusb/lerna-cola-sample

It provides a great way for you to quickly clone and run a non-trivial project in order to assess the benefits that Lerna Cola could bring to your projects.

## Configuration

To use the Lerna Cola CLI you will need to create a configuration file named either `lerna-cola.json` or `lerna-cola.js` within the root of your project.

> Note: When creating a `.js` file ensure that you export the configuration object via `module.exports`.

### Example Configuration

Before we describe the configuration schema, it may be helpful to review a typical configuration:

```javascript
module.exports = {
  packages: {
    // The following two packages are microservices where we are using the babel
    // plugin to transpile them to support our current node version, and the
    // server develop plugin which will take care of executing and restarting
    // our microservices for any changes up their dependency trees.

    'my-microservice-1': {
      buildPlugin: '@lerna-cola/plugin-build-babel',
      developPlugin: 'plugin-develop-server',
    },

    'my-microservice-1': {
      buildPlugin: '@lerna-cola/plugin-build-babel',
      developPlugin: 'plugin-develop-server',
    },

    // The following package is a "Create React App" package which comes with
    // it's own build and develop (start) scripts. We will therefore use the
    // Lerna Cola script plugin to delegate to the CRA scripts.

    'my-ui': {
      buildPlugin: {
        name: 'plugin-script',
        options: {
          scriptName: 'build',
        },
      },
      developPlugin: {
        name: 'plugin-script',
        options: {
          scriptName: 'start',
        },
      },
    },
  },
}
```

Within this example we are providing a Lerna Cola configuration for 3 of our packages within our monorepo. We need not provide a configuration for every package within our monorepo; only the ones that we wish to execute Lerna Cola plugins against. Lerna Cola will still be aware of the other packages within our repo, for example providing hot reloading on our servers/apps when a change occurs on our shared utils package.

In our example we have two microservices that will be Babel transpiled by the [`build`](#build) command, and will be treated as server process (with execution and hot reloading) by the [`develop`](#develop) command. The third package configuration utilities a special [Create React App](https://TODO) plugin that contains configuration for both the [`build`](#build) and [`develop`](#develop) commands.

This is a very basic example, however, it is illustrative of how quickly you can provide a centralised and coordinated configuration for your monorepo packages. Plugins can of course be customised via options. We recommend you read the [plugin](#plugins) docs for detailed information on them.

### Configuration Schema

The configuration is an Object/JSON structure that supports the following schema:

- `commandHooks` (_Object_, **optional**)

  A set of hooks to allow you to execute a function pre/post each command. You can specify a set of hooks for each of the commands:

  - clean
  - build
  - develop
  - deploy

  Each of them support the following configuration _Object_:

  - `pre`: (_Function_, **optional**)

    Runs prior to the specifed command. Can return a `Promise`, which will be waited on to resolve before proceeding.

  - `post`: (_Function_, **optional**)

    Runs after to the specifed command completes/fails. Can return a `Promise`, which will be waited on to resolve before proceeding.

  Example:

  ```javascript
  // lerna-cola.js
  module.exports = {
    commandHooks: {
      build: {
        pre: () => Promise.resolve(),
        post: () => Promise.resolve(),
      },
    },
  }
  ```

- `packages` (_Object_, **_optional_**)

  An object where each key is the name of a package within your repository (matching the name of the package within the package's `package.json` file). The value against each key is considered a package configuration object, supporting the following schema:

  - `buildPlugin` (_string_ || _Object_, **optional**)

    The plugin to use when building the package. Please see the [Plugins](#plugins) documentation for more information.

  - `developPlugin` (_string_ || _Object_, **optional**)

    The plugin to use when developing the package. Please see the [Plugins](#plugins) documentation for more information.

  - `deployPlugin` (_string_ || _Object_, **optional**)

    The plugin to use when deploying the package. Please see the [Plugins](#plugins) documentation for more information.

  - `srcDir` (_string_, **optional**, **default**: 'src')

    The directory within your package containing the source.

  - `entryFile` (_string_, **optional**, **default**: 'index.js')

    The file within your `srcDir` that should be considered the "entry" file for your package.

  - `outputDir` (_string_, **optional**, **default**: 'build')

    The directory which should be targetted for build output by the build plugins. Also used by the clean plugin to know which directory should be removed.

  - `disableSrcWatching` (_boolean_, **optional**, **default**: false)

    Use this to disable watching of the source when running the develop command. This can be useful if you don't care about tracking changes for a specific package within your repository, or you have a seperate process that will already track source changes for the respective package (e.g. the `start` script within a Create React App project).

- `packageSources` (_Array&lt;string&gt;_, **_optional_**, **default**: _see below_)

  An array of globs, paths where your packages are contained. By default it uses the same configuration as Lerna, i.e.:

  ```json
    "packageSources": [
      "packages/*"
    ]
  ```

## CLI Commands

Below is an brief description of each command that the CLI provides. You can request for help via the CLI for all of the commands like so:

```bash
lerna-cola help
```

Or for an individual command like so:

```bash
lerna-cola build help
```

When executing commands all of the environment variables that are currently available (i.e. `process.env`) will be passed into the respective processes that are spawned for each package.

### clean

Clean the output from the the [`build`](#build) command.

By default the [`plugin-clean-build`](#plugin-clean-build) plugin is used, however, you can customise this within your [configuration](#configuration).

### build

Take advantage of one of the wide array of plugins to babel/flow/typescript/reasonml/etc/etc/etc transpile/build your packages with the ability to easily share and manage configuration at the root of your monorepo.

When executed it will run all of the configured plugins for all of your packages that have a `buildPlugin` configured within the `lerna-cola.json` [configuration](#configuration) file. The package build order is based upon on a topological sort against the dependency tree of your packages within the monorepo (i.e. they build in the correct order).

### develop

Run a full auto/hot reloading development environment in which any changes to one package cascade through your package dependency tree. No more need to manually rebuild/restart servers during development allow for a blazingly hot development experience.

For packages without a `developPlugin` configured, they will have their source or build output watched, with any changes causing notifications to be send down the dependency tree. Any packages running with a `developPlugin` can then respond to these and their own changes in order to auto/hot reload appropriately.

All of the logs/output from your packages will be managed and printed within the console window where you ran the `develop` command. The output contains a uniquely colored column to the left allowing you to easily identify the output for each respective package.

### deploy

Deploy your apps with a simple one line command to a cloud provider supported by the plugin system.

When executing this command your packages will be built in topological sort order (based on their dependency tree between each other) and then will subsequently be deployed via their configured plugins.

## Plugins

Lerna Cola is powered by a powerful plugin system that allows the build, develop, and deploy commands to support a multitude of targets. It is very easy to build your own plugin should you desire to so - please see the ["Plugin Development"](#plugin-development) section for more information.

Plugins are split by "core" plugins, which are bundled with the main `@lerna-cola/cli` package, and "package" plugins which could either be official Lerna Cola packages, 3rd party packages, or private packages of your own making.

You define plugins against each of your package configurations within your Lerna Cola [configuration](#configuration).

Plugins allow two forms of assignment.

**Form 1 - specify the plugin by name**

```javascript
// lerna-cola.js
module.exports = {
  packages: {
    'my-package': {
      buildPlugin: '@lerna-cola/plugin-build-babel',
    },
  },
}
```

**Form 2 - provide options to the plugin**

```javascript
// lerna-cola.js
module.exports = {
  packages: {
    'my-package': {
      buildPlugin: {
        name: '@lerna-cola/plugin-build-babel',
        options: {
          config: {
            presets: ['babel-preset-env'],
          },
        },
      },
    },
  },
}
```

You can pass down any arbitrary set of options, which will be made available to the respective plugin. Please refer to the documentation for each plugin in terms of what options it supports.

### Core Plugins

Below are the "core" plugins which will be immediately available when you add Lerna Cola to your repository.

#### `plugin-clean-build`

A [clean](#clean) command plugin.

This plugin will be automatically assigned to any package with a `buildPlugin` defined.

When executed it will remove the output directory targetted by your respective build plugin.

#### `plugin-develop-server`

A [develop](#develop) command plugin.

#### `plugin-script`

This is a special plugin that can be configured against any of your plugins for a package. i.e. `cleanPlugin`, `buildPlugin`, `developPlugin`, and `deployPlugin`.

It allows you to delegate to a script defined within the `package.json` of your targetted package.

This can be especially useful for packages that come with their own sets of scripts (e.g. A Create React App package).

**Options**

This plugin supports the following options:

- `scriptName` (_string_, **required**)

  The name of the script to run within your target package.

- `runForEveryChange` (_boolean_, **default**: false)

  Only used when assigned against the `developPlugin`. Setting this value to `true` will ensure that the your script will be executed any time a change is registered within your packages' source or against one of its dependencies. If your script spawns a long run child process (for example a Create React App server), then an existing running instance will be destroyed before the script is ran again.

**Example**

```javascript
// lerna-cola.js
module.exports = {
  packages: {
    'my-create-react-app': {
      buildPlugin: {
        name: 'script',
        options: {
          scriptName: 'build',
        },
      },
    },
  },
}
```

### Official Plugins

#### `@lerna-cola/plugin-build-babel`

A [build](#build) command plugin.

This plugin will transpile your package's `srcDir` (see the [configuration](#configuration)) using [Babel](https://babeljs.io), outputing the results into your packages `outputDir` (see the [configuration](#configuration)).

By default it will use a `.babelrc` found within your packages root directory, and if one is not found then it will look for a `.babelrc` within the root of your monorepo. You can alternatively provide a babel configuration via the plugin's options.

**Options**

This plugin supports the following options:

- `config` (_string_ || _Object_, **optional**)

  The [babel configuration](https://babeljs.io/docs/en/api.md) to use for the transpilation.

  This can be one of two things:

  - The name of a package where the main export is a babel configuration object.
  - An object containing the babel configuration.

  > Note: Lerna Cola ships a simple babel config package which you could use. It is called `@lerna-cola/babel-config`

We highly recommend that you enable sourcemaps output within your configuration to aid debugging.

**Example**

Specifying an inline config:

```javascript
// lerna-cola.js
module.exports = {
  packages: {
    'my-lib': {
      buildPlugin: {
        name: '@lerna-cola/plugin-build-babel',
        options: {
          config: {
            presets: ['babel-preset-env'],
          },
        },
      },
    },
  },
}
```

Specifying a package containing the configuration:

```javascript
// lerna-cola.js
module.exports = {
  packages: {
    'my-lib': {
      buildPlugin: {
        name: '@lerna-cola/plugin-build-babel',
        options: {
          config: 'my-babel-config-package',
        },
      },
    },
  },
}
```

#### `@lerna-cola/plugin-build-flow`

A [build](#build) command plugin.

This plugin will transpile your package's `srcDir` (see the [configuration](#configuration)), stripping your source of any flow annotations, and outputing the result (along with \*.flow) files into your package's `outputDir` (see the [configuration](#configuration)).

**Options**

This plugin supports the following options:

- `inputs` (_Array&lt;string&gt;_ , **optional**, **default**: _see below_)

  An array of glob strings which should be used to match the files that will be processed.

  Defaults to the following:

  ```json
  ["**/*.js", "!__tests__", "!test.js"]
  ```

**Example**

```javascript
// lerna-cola.js
module.exports = {
  packages: {
    'my-lib': {
      buildPlugin: '@lerna-cola/plugin-build-flow',
    },
  },
}
```

#### `@lerna-cola/plugin-deploy-now`

> TODO

### 3rd Party Plugins

Hopefully we will have some soon. 🤞

Please submit a PR to add yours here.

## Schemas

### `Package` Schema

The holy grail of information for your plugins.

### `Config` Schema

> TODO

## Plugin Development

Developing plugins are very easy. You can develop 3 different types of plugins; one for each command type.

To create a plugin create a new package, where "main" on its package.json file points to the plugin source.

Your plugin factory will receive two arguments:

- `pkg` (_Package_)

  The package for which the plugin is assigned. This contains lots of really useful information about the package, such as it's dependency tree, src and build output paths etc. Please see the full schema for the [Package Schema](#package-schema) to see what is available to you.

> TODO Plugin API etc
