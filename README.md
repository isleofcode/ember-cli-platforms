Ember-cli-platforms
===================

## Installation

- `ember install ember-cli-platforms`
- `ember install ember-cli-platform-cordova`

## Api

- `ember s -p "cordova"`
- `ember s --platform="cordova"`
- `ember platform cordova`

## Pipeline Configuration

- `<project>/config/platforms/<platform>.js`

**Minimum Settings**
- `outputDestination`
- `addon-name`
- `commands: [...commands]` (for command proxying)

## Pipeline Hooks

Borrowed heavily from ember-cli-deploy

- configure
- setup
- willPipeAssets
- pipeAssets
- didPipeAssets
- teardown

## Remote Live Reload and Inspection

- [ember-cli-remote-inspector](https://github.com/joostdevries/ember-cli-remote-inspector)
- ember-cli-remote-livereload
  - auto add `<allow-navigation href="http://10.0.2.2:4200/*" />`
  - cordova.liveReload.enabled = true
  - cordova.liveReload.platform = 'android'
  - cordova.emberUrl
  - PR https://github.com/poetic/ember-cli-cordova/pull/56
  


## Pipeline and Deployment Addons

Addons implementing this pipeline should also implement
a deployment pipeline for ember-cli-deploy

- ember-cli-platform-cordova
