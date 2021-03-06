# Pebble Spring Boot Starter
Spring Boot starter for autoconfiguring Pebble as an MVC ViewResolver for Spring Boot 2.x.

## Basic (Local) Usage

This is a stop-gap project meant to be used until Pebble officially supports Spring Boot 2.x. Until then, you can deploy this to a local repo and use in your projects.

Add a "repo" directory to your project

```bash
mkdir repo
```

Add the local Maven repo directory to your pom.xml

```XML
  <repositories>
    <repository>
      <id>project.local</id>
      <name>project</name>
      <url>file:${project.basedir}/repo</url>
    </repository>
  </repositories>
```

Download and unzip the build jars from [releases](https://github.com/jpolete/pebble-spring-boot-2-starter/releases).

Deploy the starter to your local Maven repository directory. (Be sure to change the paths below)

```bash
mvn deploy:deploy-file -Durl=file:///path/to/your/local/maven/repo \
-Dfile=/path/to/unzipped/download/pebble-spring-boot-2-starter-1.0.0-SNAPSHOT.jar \
-DgroupId=com.mitchellbosecke \
-DartifactId=pebble-spring-boot-2-starter \
-Dpackaging=jar \
-Dversion=1.0.0-SNAPSHOT
```

Add the starter dependency to your pom.xml:
```XML
<dependency>
	<groupId>com.mitchellbosecke</groupId>
	<artifactId>pebble-spring-boot-2-starter</artifactId>
	<version>1.0.0-SNAPSHOT</version>
</dependency>
```

This is enough for autoconfiguration to kick in. This includes:

* a Loader that will pick template files ending in ``.pebble`` from ``/templates/`` dir on the classpath
* a PebbleEngine with default settings, configured with the previous loader
* a ViewResolver that will output ``text/html`` in ``UTF-8``

PLEASE NOTE: the starter depends on ``spring-boot-starter-web`` but is marked as optional, you'll need to add the dependency yourself or configure Spring MVC appropriately.

## Compatibility matrix
Pebble vs tested Boot versions (may work on older Boot releases).

| Pebble Boot 2 Starter | Spring Boot |
| --- | --- |
| 1.0.0 | 2.0.0 |

## Boot externalized configuration
A number of properties can be defined in Spring Boot externalized configuration, eg. ``application.properties``, starting with the prefix ``pebble``. See the corresponding [PebbleProperties.java](https://github.com/PebbleTemplates/pebble-spring-boot-starter/blob/master/src/main/java/com/mitchellbosecke/pebble/boot/autoconfigure/PebbleProperties.java) for your starter version. Notable properties are:

* ``pebble.prefix``: defines the prefix that will be prepended to the mvc view name. Defaults to ``/templates/``
* ``pebble.suffix``: defines the suffix that will be appended to the mvc view name. Defaults to ``.pebble``
* ``pebble.cache``: enables or disables PebbleEngine caches. Defaults to ``true``
* ``pebble.contentType``: defines the content type that will be used to configure the ViewResolver. Defaults to ``text/html``
* ``pebble.encoding``: defines the text encoding that will be used to configure the ViewResolver. Defaults to ``UTF-8``
* ``pebble.exposeRequestAttributes``: defines whether all request attributes should be added to the model prior to merging with the template for the ViewResolver. Defaults to ``false``
* ``pebble.exposeSessionAttributes``: defines whether all session attributes should be added to the model prior to merging with the template for the ViewResolver. Defaults to ``false``
* ``pebble.defaultLocale``: defines the default locale that will be used to configure the PebbleEngine. Defaults to ``null``
* ``pebble.strictVariables``: enable or disable the strict variable checking in the PebbleEngine. Defaults to ``false``

## Customizing Pebble
### Pebble extensions
Extensions defined as beans will be picked up and added to the PebbleEngine automatically:
```Java
@Bean
public Extension myPebbleExtension1() {
   return new MyPebbleExtension1();
}

@Bean
public Extension myPebbleExtension2() {
   return new MyPebbleExtension2();
}
```
CAVEAT: Spring will not gather all the beans if they're scattered across multiple @Configuration classes. If you use this mechanism, bundle all Extension @Beans in a single @Configuration class.

### Customizing the Loader
The autoconfigurer looks for a bean named ``pebbleLoader`` in the context. You can define a custom loader with that name and it will be used to configure the default PebbleEngine:
```Java
@Bean
public Loader<?> pebbleLoader() {
   return new MyCustomLoader();
}
```
PLEASE NOTE: this loader's prefix and suffix will be both overwritten when the ViewResolver is configured. You should use the externalized configuration for changing these properties.

### Customizing the PebbleEngine
Likewise, you can build a custom engine and make it the default by using the bean name ``pebbleEngine``:
```Java
@Bean
public PebbleEngine pebbleEngine() {
   return new PebbleEngine.Builder().build();
}
```

### Customizing the ViewResolver
And the same goes for the ViewResolver, using the bean name ``pebbleViewResolver``: 
```Java
@Bean
public PebbleViewResolver pebbleViewResolver() {
   return new PebbleViewResolver();
}
```
PLEASE NOTE: you need to change the Loader's prefix and suffix to match the custom ViewResolver's values.

### Using Pebble for other tasks
The main role of this starter is to configure Pebble for generating MVC View results (the typical HTML). You may define more PebbleEngine/Loader beans for other usage patterns (like generating email bodies). Bear in mind that you should not reuse the default Loader for other Engine instances.
