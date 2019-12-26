# Serverless Nuxt Plugin

<p>
  <a href="https://npmcharts.com/compare/serverless-nuxt?minimal=true"><img alt="Downloads" src="https://img.shields.io/npm/dt/serverless-nuxt.svg?style=flat-square" /></a>
  <a href="https://www.npmjs.com/package/serverless-nuxt"><img alt="Version" src="https://img.shields.io/npm/v/serverless-nuxt.svg?style=flat-square" /></a>
  <a href="https://www.npmjs.com/package/serverless-nuxt"><img alt="License" src="https://img.shields.io/npm/l/serverless-nuxt.svg?style=flat-square" /></a>
  <img alt="Typescript" src="https://img.shields.io/badge/language-Typescript-007acc.svg?style=flat-square" />
  <br />
  <a href="https://david-dm.org/wan2land/serverless-nuxt"><img alt="dependencies Status" src="https://img.shields.io/david/wan2land/serverless-nuxt.svg?style=flat-square" /></a>
  <a href="https://david-dm.org/wan2land/serverless-nuxt?type=dev"><img alt="devDependencies Status" src="https://img.shields.io/david/dev/wan2land/serverless-nuxt.svg?style=flat-square" /></a>
</p>

Nuxt on AWS(Lambda + S3) with Serverless Framework.

## Installation

If you don't have a Nuxt project already, let's create it first. If you have already installed a Nuxt project, you can skip the following.

[Nuxt Official : Installation](https://nuxtjs.org/guide/installation/)

Install the plug-in in the project directory.

```bash
cd my-nuxt-project
npm i serverless-nuxt
npm i serverless-nuxt-plugin -D
```

**Important**

Lambda has a maximum file size of 256 MB. (cf. [AWS Lambda Limits](https://docs.aws.amazon.com/lambda/latest/dg/limits.html))
You will need to delete the package files you need during the build process. There is a good package for this.

```bash
npm i nuxt-start
npm i nuxt -D
```

Then,

We will touch a total of 3 files.

- `serverless.yml`
- `handler.js`
- `nuxt.config.js`

Once you've used the Serverless framework, it's easier. It is helpful to read [the Manual](https://serverless.com/framework/docs/providers/aws/guide/quick-start/) once.

### `serverless.yml`

Add the `serverless.yml` file. The `my-nuxt-project` part is set to your project name. Use the Cloud Formations to upload **assets**(`.nuxt/dist/client`) files to the **AWS S3** repository. If you have an S3 repository already created, remove Cloud Formation information(**resources** section).

Write your plugin settings in the `custom.nuxt` field.  The version(`custom.nuxt.version`) is used as a prefix when uploading assets files to S3. For more options, please see [here](#Options).

```yml
service:
  name: my-nuxt-project

plugins:
  - serverless-nuxt-plugin

resources:
  Resources:
    AssetsBucket:
      Type: AWS::S3::Bucket
      Properties:
        BucketName: ${self:custom.nuxt.bucketName}
        CorsConfiguration:
          CorsRules:
            - AllowedMethods:
              - GET
              - HEAD
              AllowedOrigins:
              - "*"

provider:
  name: aws
  stage: ${opt:stage, 'dev'}
  runtime: nodejs10.x
  environment:
    NODE_ENV: ${file(.env.${self:provider.stage}.yml):NODE_ENV}

custom:
  nuxt:
    version: v0.0.1-alpha
    bucketName: my-nuxt-project-${self:provider.stage}

functions:
  nuxt:
    handler: handler.render
    events:
      - http: ANY /
      - http: ANY /{proxy+}
```

You can modify the existing Nuxt configuration file (`nuxt.config.js`) to fit the following format:

```js
const pkg = require("./package.json")

// "export default" => "module.exports ="
module.exports = {
  mode: "universal",
  head: {
    title: pkg.name,
    meta: [
      { charset: "utf-8" },
      { name: "viewport", content: "width=device-width, initial-scale=1" },
      { hid: "description", name: "description", content: pkg.description },
    ],
    link: [
      { rel: "icon", type: "image/x-icon", href: "/favicon.ico" },
    ],
  },
  build: {
    // The "publicPath" value is automatically generated by the plug-in
  },
}
```

Finally, the handler(`handler.js`) is written as:

```js
const { createNuxtApp } = require("serverless-nuxt")
const config = require("./nuxt.config.js")

module.exports.render = createNuxtApp(config)
```

Proceed as follows: The serverless plug-in builds automatically at deployment time.

**Never** use the `npm run build` command.

```bash
sls deploy --stage dev
```

## Options

```yml
custom:
  nuxt:
    version: v1.0.0-alpha
    bucketName:
    assetsPath:
```

Name                 | Description | Default
---------------------| ----------- | ------- |
version (required)   | version     |
bucketName (required)| AWS S3 Bucket Name for static files
cdnPath              | CDN Path    | `null` 
assetsPath           |  | `".nuxt/dist/client"`


## References

- [Serverless plugin author's cheat sheet](https://gist.github.com/HyperBrain/50d38027a8f57778d5b0f135d80ea406)

## License

MIT
