# Gradle Swagger Generator Plugin [![CircleCI](https://circleci.com/gh/int128/gradle-swagger-generator-plugin.svg?style=shield)](https://circleci.com/gh/int128/gradle-swagger-generator-plugin) [![Gradle Status](https://gradleupdate.appspot.com/int128/gradle-swagger-generator-plugin/status.svg)](https://gradleupdate.appspot.com/int128/gradle-swagger-generator-plugin/status)

This is a Gradle plugin for the following tasks:

- Validate an OpenAPI YAML.
- Generate source from an OpenAPI YAML using [Swagger Codegen](https://github.com/swagger-api/swagger-codegen) V2 and V3.
- Generate Swagger UI with an OpenAPI YAML.
- Generate ReDoc with an OpenAPI YAML.

See also the following examples:

- [Swagger UI example](https://int128.github.io/gradle-swagger-generator-plugin/ui-v3/) (generated by the [ui-v3/basic-example](/acceptance-test/ui-v3/basic-example/build.gradle) project)
- [ReDoc example](https://int128.github.io/gradle-swagger-generator-plugin/redoc/) (generated by the [redoc](/acceptance-test/redoc/build.gradle) project)
- [HTML document example](https://int128.github.io/gradle-swagger-generator-plugin/html/) (generated by the [codegen-v2/html](/acceptance-test/codegen-v2/html/build.gradle) project)


## Code Generation

Create a project with the following build script.

```groovy
plugins {
  id 'org.hidetake.swagger.generator' version '2.16.0'
}

repositories {
  jcenter()
}

dependencies {
  swaggerCodegen 'io.swagger:swagger-codegen-cli:2.3.1'             // Swagger Codegen V2
  swaggerCodegen 'io.swagger.codegen.v3:swagger-codegen-cli:3.0.0'  // or Swagger Codegen V3
}

swaggerSources {
  petstore {
    inputFile = file('petstore.yaml')
    code {
      language = 'spring'
    }
  }
}
```

The task generates source code into `build/swagger-code-petstore`.

```
% ./gradlew generateSwaggerCode
:resolveSwaggerTemplate NO-SOURCE
:generateSwaggerCodePetstore
:generateSwaggerCode NO-SOURCE
```


## Document Generation

### Swagger UI

Create a project with the following build script.

```groovy
plugins {
  id 'org.hidetake.swagger.generator' version '2.16.0'
}

repositories {
  jcenter()
}

dependencies {
  swaggerUI 'org.webjars:swagger-ui:3.10.0'
}

swaggerSources {
  petstore {
    inputFile = file('petstore.yaml')
  }
}
```

The task generates an API document as `build/swagger-ui-petstore`.

```
% ./gradlew generateSwaggerUI
:generateSwaggerUIPetstore
:generateSwaggerUI NO-SOURCE
```


### ReDoc

Create a project with the following build script.

```groovy
plugins {
  id 'org.hidetake.swagger.generator' version '2.16.0'
}

swaggerSources {
  petstore {
    inputFile = file('petstore.yaml')
  }
}
```

The task generates an API document as `build/redoc-petstore`.

```
% ./gradlew generateSwaggerReDoc
:generateReDocPetstore
:generateReDoc NO-SOURCE
```


### HTML

Create a project with the following build script.

```groovy
plugins {
  id 'org.hidetake.swagger.generator' version '2.16.0'
}

repositories {
  jcenter()
}

dependencies {
  swaggerCodegen 'io.swagger:swagger-codegen-cli:2.3.1'             // Swagger Codegen V2
  swaggerCodegen 'io.swagger.codegen.v3:swagger-codegen-cli:3.0.0'  // or Swagger Codegen V3
}

swaggerSources {
  petstore {
    inputFile = file('petstore.yaml')
    code {
      language = 'html'  // html or html2
    }
  }
}
```

The task generates a static HTML into `build/swagger-code-petstore`.

```
% ./gradlew generateSwaggerCode
:resolveSwaggerTemplate NO-SOURCE
:generateSwaggerCodePetstore
:generateSwaggerCode NO-SOURCE
```


## Recipes

See the examples in [acceptance-test](acceptance-test).

### Use configuration file

We can use a [JSON configuration file](https://github.com/swagger-api/swagger-codegen#customizing-the-generator) as follows:

```groovy
swaggerSources {
  petstore {
    inputFile = file('petstore.yaml')
    code {
      language = 'spring'
      configFile = file('config.json')
    }
  }
}
```

`config.json` depends on the language and framework. For example,

```json
{
  "library": "spring-mvc",
  "modelPackage": "example.model",
  "apiPackage": "example.api",
  "invokerPackage": "example"
}
```

Run the task with `Help` postfix to show available JSON configuration.

```
% ./gradlew generateSwaggerCodePetstoreHelp
:generateSwaggerCodePetstoreHelp
=== Available raw options
NAME
        swagger-codegen-cli generate - Generate code with chosen lang

SYNOPSIS
        swagger-codegen-cli generate
                [(-a <authorization> | --auth <authorization>)]
...

=== Available JSON configuration for language spring:

CONFIG OPTIONS
	sortParamsByRequiredFlag
...
```


### Build generated code

It is recommended to generate code into an ephemeral directory (e.g. `build`) and exclude it from a Git repository.
We can compile generated code as follows:

```groovy
swaggerSources {
  petstore {
    inputFile = file('petstore.yaml')
    code {
      language = 'spring'
      configFile = file('config.json')
    }
  }
}

// Configure compile task dependency and source
compileJava.dependsOn swaggerSources.petstore.code
sourceSets.main.java.srcDir "${swaggerSources.petstore.code.outputDir}/src/main/java"
sourceSets.main.resources.srcDir "${swaggerSources.petstore.code.outputDir}/src/main/resources"
```

For more, see [codegen-v2/java-spring](acceptance-test/codegen-v2/java-spring) and [codegen-v3/java-spring](acceptance-test/codegen-v3/java-spring).


### Validate YAML before code genetation

It is recommended to validate an OpenAPI YAML before code generation in order to avoid invalid code generated.
We can validate a YAML as follows:

```groovy
swaggerSources {
  petstore {
    inputFile = file('petstore.yaml')
    code {
      language = 'spring'
      configFile = file('config.json')
      // Validate YAML before code generation
      dependsOn validation
    }
  }
}
```


### Selective generation

We can control output of code generation.
At default everything is generated but only models and APIs are generated in the following:

```groovy
swaggerSources {
  petstore {
    inputFile = file('petstore.yaml')
    code {
      language = 'spring'
      configFile = file('config.json')
      // Generate only models and controllers
      components = ['models', 'apis']
    }
  }
}
```

`components` property accepts a list of strings or map.

```groovy
// generates only models
components = ['models']
components = [models: true]

// generate only User and Pet models
components = [models: ['User', 'Pet']]
components = [models: 'User,Pet']

// generate only APIs (without tests)
components = [apis: true, apiTests: false]
components = [apis: true, apiTests: null]
```

For details, see [selective generation section](https://github.com/swagger-api/swagger-codegen#selective-generation).


### Use custom template

We can use a custom template for the code generation as follows:

```groovy
// build.gradle
swaggerSources {
  inputFile = file('petstore.yaml')
  petstore {
    language = 'spring'
    // Path to the template directory
    templateDir = file('templates/spring-mvc')
  }
}
```

For more, see [codegen-v2/custom-template](acceptance-test/codegen-v2/custom-template).


### Use custom generator class

We can use a custom generator class for the code generation as follows:

```groovy
// build.gradle
swaggerSources {
  petstore {
    inputFile = file('petstore.yaml')
    code {
      // FQCN of the custom generator class
      language = 'CustomGenerator'
    }
  }
}
```

```groovy
// buildSrc/build.gradle
repositories {
  jcenter()
}

dependencies {
  // Add dependency here (do not specify in the parent project)
  compile 'io.swagger:swagger-codegen-cli:2.3.1'
}
```

```groovy
// buildSrc/src/main/groovy/CustomGenerator.groovy
import io.swagger.codegen.languages.SpringCodegen

class CustomGenerator extends SpringCodegen {
}
```

For more, see [codegen-v2/custom-class](acceptance-test/codegen-v2/custom-class) and [codegen-v3/custom-class](acceptance-test/codegen-v3/custom-class).


### Externalize template or generator class

In some large use case, we can release a template or generator to an external repository and use them from projects.

```groovy
// build.gradle
repositories {
  // Use external repository for the template and the generator class
  maven {
    url 'https://example.com/nexus-or-artifactory'
  }
  jcenter()
}

dependencies {
  swaggerCodegen 'io.swagger:swagger-codegen-cli:2.3.1'
  // Add dependency for the template
  swaggerTemplate 'com.example:swagger-templates:1.0.0'
  // Add dependency for the generator class
  swaggerCodegen 'com.example:swagger-generators:1.0.0'
}

swaggerSources {
  petstore {
    inputFile = file('petstore.yaml')
    code {
      language = 'spring'
      // The plugin automatically extracts template JAR into below destination
      templateDir = file("${resolveSwaggerTemplate.destinationDir}/spring-mvc")
    }
  }
}
```

For more, see
[codegen-v2/externalize-template](acceptance-test/codegen-v2/externalize-template) and
[codegen-v2/externalize-class](acceptance-test/codegen-v2/externalize-class).


### Use multiple sources

We can handle multiple sources in a project as follows:

```groovy
// build.gradle
swaggerSources {
    petstoreV1 {
        inputFile = file('v1-petstore.yaml')
        code {
            language = 'spring'
            configFile = file('v1-config.json')
        }
    }
    petstoreV2 {
        inputFile = file('v2-petstore.yaml')
        code {
            language = 'spring'
            configFile = file('v2-config.json')
        }
    }
}

compileJava.dependsOn swaggerSources.petstoreV1.code, swaggerSources.petstoreV2.code
sourceSets.main.java.srcDirs "${swaggerSources.petstoreV1.code.outputDir}/src/main/java", "${swaggerSources.petstoreV2.code.outputDir}/src/main/java"
sourceSets.main.resources.srcDirs "${swaggerSources.petstoreV1.code.outputDir}/src/main/resources", "${swaggerSources.petstoreV2.code.outputDir}/src/main/resources"
```

For more, see [codegen-v2/multiple-sources](acceptance-test/codegen-v2/multiple-sources) and [codegen-v3/multiple-sources](acceptance-test/codegen-v3/multiple-sources).


### Switch version of Swagger Codegen

We can use multiple versions of Swagger Codegen as follows:

```groovy
// build.gradle
configurations {
    swaggerCodegenV2
    swaggerCodegenV3
}

dependencies {
    swaggerCodegenV2 'io.swagger:swagger-codegen-cli:2.3.1'
    swaggerCodegenV3 'io.swagger.codegen.v3:swagger-codegen-cli:3.0.0'
}

swaggerSources {
    petstoreV2 {
        inputFile = file('v2-petstore.yaml')
        code {
            language = 'spring'
            configuration = configurations.swaggerCodegenV2
        }
    }
    petstoreV3 {
        inputFile = file('v3-petstore.yaml')
        code {
            language = 'spring'
            configuration = configurations.swaggerCodegenV3
        }
    }
}
```

For more, see [codegen-v3/multiple-codegen-versions](acceptance-test/codegen-v3/multiple-codegen-versions).


### Configure Swagger UI

We can [configure Swagger UI](https://github.com/swagger-api/swagger-ui/blob/master/docs/usage/configuration.md)
by overwriting [the default `index.html`](src/main/resources/swagger-ui.html) as follows:

```groovy
swaggerSources {
  petstore {
    inputFile = file('petstore.yaml')
    ui {
      doLast {
        copy {
          from 'index.html'
          into outputDir
        }
      }
    }
  }
}
```

You can create an `index.html` from [the Swagger UI official one](https://github.com/swagger-api/swagger-ui/blob/master/dist/index.html).
It must satisfy the followings:

- Put `<script src="./swagger-spec.js">` in order to load a Swagger spec.
  The plugin exports the Swagger spec as `swagger-spec.js` file while generation.
- Set `spec: window.swaggerSpec` in `SwaggerUIBundle()` parameters.
- Set `validatorUrl: null` in `SwaggerUIBundle()` parameters in order to turn off the validator badge.

For more, see [ui-v3/basic-example](acceptance-test/ui-v3/basic-example).
If you need Swagger UI 2.x, see [ui-v2/basic-example](acceptance-test/ui-v2/basic-example).


## Settings

The plugin adds `validateSwagger`, `generateSwaggerCode`, `generateSwaggerUI` and `GenerateReDoc` tasks.
A task will be skipped if no input file is given.


### Task type `ValidateSwagger`

The task accepts below properties.

Key           | Type              | Value                                   | Default value
--------------|-------------------|-----------------------------------------|--------------
`inputFile`   | File              | Swagger spec file.                      | Mandatory
`reportFile`  | File              | File to write validation report.        | `$buildDir/tmp/validateSwagger/report.yaml`

It depends on the following JSON schema:

- [OpenAPI Specification version 2.0](https://github.com/OAI/OpenAPI-Specification/blob/master/schemas/v2.0/schema.json)


### Task type `GenerateSwaggerCode`

The task accepts below properties.

Key           | Type              | Value                                   | Default value
--------------|-------------------|-----------------------------------------|--------------
`language`    | String            | Language to generate.                   | Mandatory
`inputFile`   | File              | Swagger spec file.                      | Mandatory
`outputDir`   | File              | Directory to write generated files.     | `$buildDir/swagger-code`
`wipeOutputDir` | Boolean         | Wipe the `outputDir` before generation. | `true`
`library`     | String            | Library type.                           | None
`configFile`  | File              | [JSON configuration file](https://github.com/swagger-api/swagger-codegen#customizing-the-generator). | None
`templateDir` | File              | Directory containing the template.      | None
`components`  | List or Map       | [Components to generate](https://github.com/swagger-api/swagger-codegen#selective-generation) that is a list of `models`, `apis` and `supportingFiles`. | All components
`additionalProperties` | Map of String, String | [Additional properties](https://github.com/swagger-api/swagger-codegen#to-generate-a-sample-client-library). | None
`rawOptions`  | List of Strings   | Raw command line options for Swagger Codegen | None
`configuration` | String or Configuration | Configuration for Swagger Codegen | `configurations.swaggerCodegen`


### Task type `GenerateSwaggerUI`

The task accepts below properties.

Key           | Type              | Value                                   | Default value
--------------|-------------------|-----------------------------------------|--------------
`inputFile`   | File              | Swagger spec file.                      | Mandatory
`outputDir`   | File              | Directory to write Swagger UI files.    | `$buildDir/swagger-ui`
`wipeOutputDir` | Boolean         | Wipe the `outputDir` before generation. | `true`

Note that `options` and `header` are no longer supported since 2.10.0.
See the [Migration Guide](https://github.com/int128/gradle-swagger-generator-plugin/issues/81) for details.


### Task type `GenerateReDoc`

The task accepts below properties.

Key           | Type              | Value                                   | Default value
--------------|-------------------|-----------------------------------------|--------------
`inputFile`   | File              | Swagger spec file.                      | Mandatory
`outputDir`   | File              | Directory to write ReDoc files.         | `$buildDir/swagger-redoc`
`wipeOutputDir` | Boolean         | Wipe the `outputDir` before generation. | `true`
`scriptSrc`   | String            | URL to ReDoc JavaScript.                | `//rebilly.github.io/ReDoc/releases/latest/redoc.min.js`
`title`       | String            | HTML title.                             | `ReDoc - $filename`
`options`     | Map of Strings    | [ReDoc tag attributes](https://github.com/Rebilly/ReDoc#redoc-tag-attributes). | Empty map


## Contributions

This is an open source software licensed under the Apache License Version 2.0.
Feel free to open issues or pull requests.

CircleCI builds the plugin continuously.
Following variables should be set.

Environment Variable        | Purpose
----------------------------|--------
`$GRADLE_PUBLISH_KEY`       | Publish the plugin to Gradle Plugins
`$GRADLE_PUBLISH_SECRET`    | Publish the plugin to Gradle Plugins
