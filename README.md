# buildpack-sentry-sourcemaps

A [Heroku buildpack][] built to upload sourcemaps to [Sentry][], as described in [the Sentry docs][docs]. Designed to work with Node and JS builds during the Heroku build phase.

## Usage

Define the following configuration variables within Herkou app. See [Heroku Documentaiton](https://devcenter.heroku.com/articles/config-vars) for more informaiton.

- `SENTRY_AUTH_TOKEN`: the Sentry API authentication token
- `SENTRY_ORG`: the Sentry organization the project lives under
- `SENTRY_PROJECT`: the Sentry project the source maps belong too

## Getting Sentry Auth Token

You can get it on the [API page][]. The token needs the `project:write` scope to be able to upload. The token value would be saved as the `SENTRY_AUTH_TOKEN` configuration variables.

## Determining your Sentry organziation and project

When viewing your project within Sentry, the organization and project will be found within the URL.

> `https://sentry.io/<SENTRY_ORG>/<SENTRY_PROJECT>`

## Make sure you BUILD

When using webpack, babel, or uglifyJS, you need to build the sourcemaps before they will upload. Typically this can be done within your `package.json` file. Example using babel to create sourcemaps with `-s` option. The `heroku-postbuild` step runs after dependencies are downloaded, allowing you to build during the deploy. Read more about [Heroku specific build steps](https://devcenter.heroku.com/articles/nodejs-support#heroku-specific-build-steps).

### Babel

```
// package.json

"scripts": {
  "heroku-postbuild": "mkdir -p dist && babel src -s -D -d dist",
},
```

### Next.js (version 5+)

```js
// next.config.js

module.exports = {
  webpack(config, { dev }) {
    if (!dev) {
      config.devtool = "source-map";
    }

    return config;
  }
};
```

```js
// package.json

"scripts": {
  "heroku-postbuild": "next build",
},
```

Then add this buildpack to your app:

    heroku buildpacks:add https://github.com/WebGrind/buildpack-sentry-sourcemaps

And push a new relase.

## Defining the Sentry release

The buildpack will use the current git commit number via the environment variable `HEROKU_SLUG_COMMIT` to mark the Sentry release. More information can be found about this [here](https://devcenter.heroku.com/articles/dyno-metadata).

## What Changed

This is a modified version of the great work done by _Schnouki_ found [here](https://github.com/Schnouki/buildpack-sentry-sourcemaps). Things were changed to be a little more robust, and support more standard configuration variables with Sentry.

- Envrioment Variables were changed to match the [Sentry CLI Configuration](https://docs.sentry.io/learn/cli/configuration/) variables. This allowed the buildpack to be a drop-in replacement for WebPack configs, Sentry CLI, and other methods.
- The prefix Enviroment Variable was deprecated and now defaults to using `~` which means the full domain does not need specified. View the sentry [documentation][docs] for more details on how this work.
- The original buildpack was great, but would not dig through nested folders. Instead of looking for Source Maps in a single folder, one level deep, it looks through the entire projects directory for source maps. This creates a more universial solution to work across multiple projects. This also means that certain folders are ignored by default, include `node_modules` and `.heroku`.

I did not submit a PR for these changes because of the modification of Enviroment Variables. This means this is considered a breaking change, and can not be used as a drop-in replacement for the work _Schanouki_ did.

## License

MIT.

[heroku buildpack]: https://devcenter.heroku.com/articles/buildpacks
[sentry]: https://sentry.io/
[docs]: https://docs.sentry.io/clients/javascript/sourcemaps/
[api page]: https://sentry.io/api/
