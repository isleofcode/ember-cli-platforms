Ember-cli-platforms
===================

# Evolving Proposal

This addon is currently just a proposal (not implemented) and is becoming an RFC.
You can track the RFC here: https://github.com/ember-cli/rfcs/pull/35



----

Powerful workflow and build pipeline for cross-platform Ember CLI Applications.

`ember-cli-platforms` treats platforms as build targets, and gives you a powerful
workflow for producing builds customized to the platform.

Whether you're building for just one platform, or a dozen `ember-cli-platforms`,
provides streamlined tooling, enhanced resuability, and better separation of concerns.

## What are platforms?

Platforms are roughly equivalent to web views, or a shell which provides a web view
such as a browser.

Chrome and Firefox extensions as well as projects such as `electron`, `cordova`,
`crosswalk`, `cef`, and `NW.js` provide their own specialized web views which allow you
to ship your Ember application as an installable app.

## Installation

- `ember install ember-cli-platforms`

## Plugins

Platform addons should also implement a default deployment process for ember-cli-deploy.
Custom deployments (such as to phonegap-build or cloudfive or Telerik) should be their
own addons.

- [Cordova]() `ember install ember-cli-platform-cordova`
- [Crosswalk]() `ember install ember-cli-platform-crosswalk`
- [Chrome]() `ember install ember-cli-platform-chrome`
- [Firefox]() `ember install ember-cli-platform-firefox`
- [MacGap]() `ember install ember-cli-platform-macgap`
- [CEF]() `ember install ember-cli-platform-cef`

**Advanced Usage**

The following plugins are available for advanced user. These platforms
come with [security risks]() you should understand before you choose them.

- [Electron]() `ember install ember-cli-platform-electron`
- [NW.js]() `ember install ember-cli-platform-nwjs`


## Usage

These are the docs for using `ember-cli-platforms` with an installed platform plugin.
For the docs on creating a platform plugin, [skip ahead](./README.md#creating-plugins).

**Create a platform.**
```cli
ember platform <type> [-t "<name>"]
```

Generates a new platform of `<type>` with an optional `<name>` parameter used when more than
one instance of a platform is desired.

For all commands, `platform` can be abbreviated as `p`, and
commands that mirror `ember-cli` commands preserve the `ember-cli`
shorthands for those methods as well.

**Serve a specific platform.**
```cli
ember p:s <type>[-<name>]
```

This is shorthand for:
```cli
ember platform:serve --platform="<type>" [--target="<name>"] --environment="development"
```

**Build a specific platform.**
```cli
ember p:b <type>[-<name>]
```

**Test a specific platform.**
```cli
ember p:t <type>[-<name>] <deployTarget>
```

**Deploy a specific platform.**
```cli
ember p:d <type>[-<name>] <deployTarget>
```

This is short hand for:
```cli
ember platform:deploy --platform="<type>" [--target="<name>"] --deployTarget="<deployTarget>" --environment="development"
```


### Platform Specific Code

`ember-cli-platforms` leverages in-repo-addon semantics and Broccoli plugins to enable
you to sprinkle platform specific code into your app as needed with "platform addons".

A platform addon is generated automatically for each platform you create with `ember platform`.

```
<project>
  /lib
    /<type>[-<name>]
```

These "addons" function differently from normal addons. Instead of code in the app overriding or importing
code from the addon, the addon take precedence.  This let's you selectively replace or add entire modules
as needed.

You can import the `app` version of a module into the addon.

```js
import Foo from '<project-name>/routes/foo';

// our platform version of routes/foo
export default Foo.extend({
  hello: 'World!'
});
```

### Platform Specific Config

A platform config file is generated automatically for each platform you create with `ember platform`.
```
<project>
  /config
    /platforms
      /<type>[-<name>].js
```

This configuration will be merged with your primary configuration.


## Creating Plugins

`ember-cli-platforms` has two forms of plugins:

1. `process hooks` which add behaviors to ember-cli-platforms.
2. `platform plugins` which implement support for a specific platform.
3. `deploy plugins` which implement support for deployment

Platform plugins (generally) are also process hooks.

## All plugins

ember-cli-platforms plugins are nothing more than standard ember-cli addons
with 3 small ember-cli-platforms specific traits:

1. they contain a package.json keyword to identify them as plugins
2. they are named ember-cli-platform-*
3. they return an object that implements one or more of the ember-cli-platforms pipeline hooks


### Creating a Process Hook

#### Create an addon

```cli
ember addon ember-cli-platform-<name>
```

#### Mark the addon as a plugin

Identify your addon as a plugin by updating the package.json like so:

```js
// package.json

"keywords": [
  "ember-addon",

  // if present, indicates this addon contains process hooks for ember-cli-platforms
  "ember-cli-platform-plugin",
  
  // if present, indicates this addon contains a platform implementation for ember-cli-platforms
  "ember-cli-platform",
  
  // if present, indicates this addon implements deployment hooks
  "ember-cli-platform-deploy"
]
```


#### Implement one or more pipeline hooks

In order for a plugin to be useful it must implement one or more of the ember-cli-platforms
pipeline hooks.

To do this you must implement a function called `createPlatformPlugin` in your `index.js` file.
This function must return an object that contains:

1. A name property which is what your plugin will be referred to.
2. One or more implemented pipeline hooks

**Example:**

```js
module.exports = {
  name: 'ember-cli-platform-live-reload',

  createPlatformPlugin: function(options) {
    return {
      name: "live-reload",

      didBuild: function(context) {
        //do something amazing here once the project has been built
      }
    };
  }

};
```

#### Process Hooks

- configure
- setup
- willBuild
- build
- didBuild
- teardown



### Creating a Deployment Plugin

This is built overtop of [ember-cli-deploy](http://ember-cli.com/ember-cli-deploy/docs/v0.5.x/pipeline-hooks/),
but integrates with your platform flow.

To create a deploy plugin, follow the same steps as for `process hooks`, making sure to
add the keyword `ember-cli-platform-deploy` to `package.json`. The following is a full list
of hooks which can be returned by `createPlatformPlugin` for use with deployment.

#### All Hooks (including deployment)

- configure
- setup
- willDeploy
- willBuild, build, didBuild,
- willPrepare, prepare, didPrepare,
- willUpload, upload, didUpload,
- willActivate, activate, didActivate, (only if --activate flag is passed)
- didDeploy,
- teardown



### Creating a Platform Plugin

Platform plugins have several concerns they need to manage.  They should

- implement the build hooks of the pipeline
- provide a starter kit for the platform
- provide a CLI proxy for SDK commands
- provide a blueprint for project installation
- run any necessary SDK installation commands
- handle SDK specific file inclusion

**Command Runners**

- ember-cli-platform supplied a commandRunner you can use to run CLI commands directly from
  the pipeline if necessary.

**Command Proxies**

- If your ember-cli-platform-plugin specifies a `commandName`, it will be used to generate
 a command proxy for that platform.  For instance:
 
```cli
{
  commandName: 'cordova'
}
```

If a user generates a cordova platform with the following command

```cli
ember platform cordova -t "ios"
```

Then an `npm` command script will be generated and installed that proxies use of
`cordova-ios` into the cordova project directory ios, allowing you to easily use
the cordova-cli.

**Minimum Settings**
- `outputDestination`
- `addon-name`



## FAQ

**Q: Can I have multiple projects for a given platform?**

Yes.  With Cordova this might look like the following.

Default setup (one project)
```cli
ember platform cordova
```

Advanced setup (multiple projects)
```cli
ember platform cordova -t "android"
```

**Q: Can I use Live Reload?**

Yes: platform plugins implement hooks to allow you to live reload when working with simulators
or attached devices.  When you `ember s`, it will just work.

> N.B. here's how ember-cli-cordova was approaching this
> - ember-cli-remote-livereload
> - auto add `<allow-navigation href="http://10.0.2.2:4200/*" />`
> - cordova.liveReload.enabled = true
> - cordova.liveReload.platform = 'android'
> - cordova.emberUrl
> - PR https://github.com/poetic/ember-cli-cordova/pull/56

**Q: Can I use the Ember Inspector?**

Yes. There's a project specifically for that: [ember-cli-remote-inspector](https://github.com/joostdevries/ember-cli-remote-inspector)

**Q: Are there platform focused addons I should use?**

[Suggested Addons]()


**Challenge List (unresolved implementation questions)**

The following concerns are difficult challenges that will need to be fleshed out to properly provide
an abstract process for each platform to be able to utilize.

- ember serve doesn't resolve promise when serving, nw.js had to hook it
- testing: run tests within the deployment target
  - view based tests
  - special test runner
  - special qunit adapter for testem
  - special test command
- testing: run tests for each platform on travis/circle/etc.
- a way to configure `environment.js` if needed (beyond the platform config file, probably just a merge)

## Wish List

- automated iTunesStore and GooglePlay release processes
- device hot-reload deployment
- develop multiple projects with live-reload at once
