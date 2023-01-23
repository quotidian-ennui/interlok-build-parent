# interlok-build-parent

[![OWASP SCA](https://github.com/quotidian-ennui/interlok-build-parent/actions/workflows/owasp.yaml/badge.svg)](https://github.com/quotidian-ennui/interlok-build-parent/actions/workflows/owasp.yaml) [![Gradle](https://github.com/quotidian-ennui/interlok-build-parent/actions/workflows/gradle.yaml/badge.svg)](https://github.com/quotidian-ennui/interlok-build-parent/actions/workflows/gradle.yaml) ![license](https://img.shields.io/github/license/quotidian-ennui/interlok-build-parent.svg)

The suggested name was `verbose-octo-disco`


> This is a divorced fork of https://github.com/adaptris/interlok-build-parent made for my own purposes. I don't need some of the features that the original parent provides.

- Single branch; my version of the bleeding edge is all you get.
- Generally targets the latest release of Interlok only.
- I'm using it to think about Interlok in a slightly different way.
- It doesn't mean you can't use it; but my itches will be different to yours.
- I may or may not be in sync with the original in terms of features.

## Usage

```
// build.gradle
ext {
  interlokVersion = '4.7-SNAPSHOT'
  interlokUiVersion = interlokVersion
  interlokParentGradle = "https://raw.githubusercontent.com/adaptris/interlok-build-parent/main/v3/build.gradle"
}

allprojects {
  apply from: "${interlokParentGradle}"
}

dependencies {
  interlokRuntime ("com.adaptris:interlok-json:$interlokVersion") { changing=true }
  interlokRuntime ("com.adaptris:interlok-filesystem:$interlokVersion") { changing=true }
  interlokRuntime ("com.adaptris:interlok-csv-json:$interlokVersion") { changing=true }
}
```

## Gradle Tasks

* `gradle clean check` will verify the configuration unmarshalls; execute service-tester if `src/test/interlok/service-test.xml` exists; and finally runs a version report.
* `gradle clean assemble` will build your distribution, you end up with an Interlok installed into `./build/distribution`
* `gradle clean build` will execute both the steps above
* `gradle dependencyCheckAnalyze` will execute the OWASP dependency check analysis; this isn't part of the normal build chain.

### Customising your build

There is support for build environments; you can pass in a gradle property to specify the build environment `gradle -PbuildEnv=myEnv clean check`. What this does is to copy the file `variables-local-{buildenv}.properties` to variables-local.properties so that you can potentially override various settings on a per build basis.

* __If the `buildEnv` is set to the _dev_ then the UI is copied into the distribution__.
* __If the `buildEnv` is set to the _dev_ then the service tester jars are copied into the distribution in the expectation that you will be using the UI service tester page__.


The same is possible with `log4j2.xml` by creating a file with the following pattern `log4j2.xml.{buildEnv}`.

If you don't want to assemble into `./build/distribution` then you can override that location by defining a `interlokDistDirectory=` in your gradle properties (or on the commandline). We generally discourage this, unless you are only running the assemble task.

If you have a local repository that you want to use (e.g. you have custom components not in our artifact repository), then you can specify an additional property `localInterlokRepo` to point to your artifact repository e.g. `gradle -PlocalInterlokRepo=http://my.artifact.repo/groups/interlok assemble`. Any _url_ supported by gradle may be used here.

### Additional Build Specific Configuration

If you need additional build specific configuration, these can be added by setting the follow properties `additionalTemplatedConfiguration` or `additionalTemplatedProperties` in your build.gradle (in the `ext{}` block) :

```
additionalTemplatedConfiguration = [
  'jetty.xml'
]

additionalTemplatedProperties = [
  'kinesis-local'
]
```

Template files are named slightly differently to property files since the expectation is that property files may need to be maintained by the UI so it needs to fit that default convention. `log42j.xml.{buildEnv}` and `variables-local-{buildEnv}.properties` are always automagically copied and overridden; additional files may be requested by explicitly setting them.

| Setting | BuildEnv | Behaviour |
|----|----|----|
| - | dev | If `src/main/interlok/config/log4j2.xml.dev` exists then this is used; otherwise log4j2.xml is the file used during `assemble` |
| - | dev | If `src/main/interlok/config/variables-local-dev.properties` exists then this is used; otherwise variables-local.properties is the file used during `assemble` |
| additionalTemplatedConfiguration = ['jetty.xml', 'ApplicationInsights.xml' ] | dev | If `src/main/interlok/config/jetty.xml.dev` exists then this is used; otherwise jetty.xml is the file used during `assemble`. The same behaviour is applied for `src/main/interlok/config/ApplicationInsights.xml.dev` |
| additionalTemplatedProperties = ['aws' ] | dev | If `src/main/interlok/config/aws-dev.properties` exists then this is used; otherwise aws.properties is the file used during `assemble` |


### Overriding system properties / environment variables during InterlokVerify

If you are expecting environment variables and system properties to be injected at runtime by your build pipeline, then you can spoof those things by setting two additional variables in your build.gradle, `interlokVerifyEnvironmentProperties` & `interlokVerifySystemProperties` respectively; this will makes `check` work w/o you defining additional properties.

```
interlokVerifyEnvironmentProperties = [
  DB_HOST: "localhost",
  DB_PORT: "3306",
  DB_NAME: "mydatabase",
  DB_USER: "root",
  DB_PASSWORD: "honestly-this-is-the-pasword-for-the-production-database",
  AWS_S3_BUCKET: "some-bucket",
  AWS_SQS_QUEUE: "queue",
]
interlokVerifySystemProperties = [
  "org.jruby.embed.localcontext.scope": "threadsafe",
  "org.jboss.logging.provider": "slf4j"
]
```