# About

This is a microservices chassis for building applications with JEE8/MicroProfile/Docker, based on Thorntail.

# Integrated frameworks:

- Thorntail 2.2.1
- Docker container built via Fabric8 Docker Maven Plugin
- Remote debugging in Docker container
- Lombok (add plugin to your IDE)
- Datasource with health check
- JAX-RS resource
- TLS (https)
- Flyway database migration
- JPA and transactions
- SLF4J logging and Thorntail logging configuration
- MicroProfile Health endpoint with JVM and system health (via MicroProfile Extensions)
- MicroProfile Metrics endpoint (with example Counter)
- MicroProfile Config configuration
- MicroProfile OpenAPI specification with SwaggerUI extension

# Test frameworks

- Arquillian integration testing
- Arquillian extension for adding test dependencies (AssertJ) to in-container test
- RestAssured integration tests for JAX-RS endpoints
- Selenium browser tests via Drone and Graphene
- AssertJ and AssertJ-DB fluent tests

# Endpoints

MicroProfile:
- Metrics: [http://localhost:8080/metrics](http://localhost:8080/metrics)
- OpenAPI: [http://localhost:8080/openapi](http://localhost:8080/openapi)
- Health: [http://localhost:8080/health](http://localhost:8080/health)

Microprofile extensions:
- Health UI: [http://localhost:8080/health-ui/](http://localhost:8080/health-ui/)
- Swagger UI: [http://localhost:8080/api/openapi-ui](http://localhost:8080/api/openapi-ui)

Resources:
- Ping [http://localhost:8080/api/ping](http://localhost:8080/api/ping)
- Ping counter: [http://localhost:8080/metrics/application/PingCounter](http://localhost:8080/metrics/application/PingCounter)
- Config: [http://localhost:8080/api/config/{key}](http://localhost:8080/api/config/{key})
- CRUD resource example: [http://localhost:8080/api/documents](http://localhost:8080/api/documents)
    
# Maven targets

Go to directory `myapp`.

Running, starting and stopping the application from Maven:
- `mvn thorntail:run`
- `mvn thorntail:start`
- `mvn thorntail:stop`

Build docker container:
- `mvn package -Pdocker`

Run Docker container:
- `mvn docker:run -Pdocker`
- `mvn docker:start -Pdocker`
- `mvn docker:stop -Pdocker`

# Java 8 in Docker

When running Java 8 in a container, the following JVM options should be specified:
- respect CPU and memory limits: `-XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap`
- use all available heap in Docker: `-XX:MaxRAMFraction=1`
- ensure sufficient entropy: `-Djava.security.egd=file:/dev/./urandom`

When building the container, an exec-style entrypoint should be specified, in order to launch a single process
that can receive Unix signals.

To access the container image:

    docker run --rm -it --entrypoint bash myapp

# HTTPS

Enable https by running:

    $ cd myapp; java -jar target/myapp-thorntail.jar -Shttps
    
See [project-https.yml](myapp/src/main/resources/project-https.yml) for an example https configuration
(adapt to your needs). Https is not configured by default, because storing passwords and certificates
in archives/containers is insecure and not portable across environments. 

Generate a self-signed certificate by running `gen_keystore.sh` in [myapp/security](myapp/security).

To run the Docker container with https enabled mount a host volume containing `keystore.jsk` at
 `/opt/security` and specify `-Shttps` as command-line argument.
 
# Unit-integration testing from the IDE

To run Arquillian integration tests from IntelliJ:
- Edit Run/Debug Configurations
- Add Arquillian Junit configuration
- Select Configure
- Add Manual container configuration
- Set name: "Thorntail 2.2.1"
- Add dependency, select Existing library: "Maven: io.thorntail:arquillian-adapter:2.2.1-Final"

# Arquillian

The `@DefaultDeployment` annotation does not bundle tests dependencies for in-container tests.
Therefore, a loadable extensions is added via the SPI mechanism in Java for adding test
libraries to the deployment.

Note that `@DefaultDeployment` only adds classes in the current package.

# Remote debugging

To enable remote debugging in a Docker container, start the application with the following environment variable:

    JAVA_TOOL_OPTIONS=-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005

The  `docker:run` Maven target has been configured for this (see `pom.xml`).

# References

Thorntail:
- [Thorntail 2.2.1 documentation](https://docs.thorntail.io/2.2.1.Final/)
- [Thorntail examples](https://github.com/thorntail/thorntail-examples/tree/2.2.1.Final)

MicroProfile:
- [MicroProfile Config](https://github.com/eclipse/microprofile-config)
- [MicroProfile Health](https://github.com/eclipse/microprofile-health)
- [MicroProfile JWT](https://github.com/MicroProfileJWT/eclipse-newsletter-sep-2017)
- [MicroProfile Metrics](https://github.com/eclipse/microprofile-metrics/blob/master/spec/src/main/asciidoc/metrics_spec.adoc)
- [MicroProfile Rest Client](https://github.com/eclipse/microprofile-rest-client)
- [MicroProfile OpenAPI](https://github.com/eclipse/microprofile-open-api/blob/master/spec/src/main/asciidoc/microprofile-openapi-spec.adoc)
- [MicroProfile Extensions](https://www.microprofile-ext.org)
- [Swagger UI on MicroProfile OpenAPI](https://www.phillip-kruger.com/post/microprofile_openapi_swaggerui/)
- [Thorntail examples](https://github.com/thorntail/thorntail-examples)

Testing:
- [Functional testing using Drone and Graphene](http://arquillian.org/guides/functional_testing_using_graphene/)
