# Quarkus 1.4.1 Maven regression

This repo demonstrates a regression in what I believe to be io.quarkus.bootstrap.resolver.maven.

`master` branch: Project generated from code.quarkus.io, default everything, with the only change being to this README and the addition of `setup/settings.xml`

`q-1.4.1-maven-revision` branch: Only change from `master` is replacing <version> with ${release} and adding a Maven property defining that version

`q-1.3.2` branch: Only change from `master` is reverting back to Quarkus 1.3.2.Final

`q-1.3.2-maven-revision` branch: Only change from `q-1.3.2` is replacing <version> with ${release} and adding a Maven property defining that version

## Background

1. Maven `${revision}`

   Since 3.5.0-beta-1, maven has supported using a ${revision} placeholder for the version in pom files.

   https://maven.apache.org/maven-ci-friendly.html

   This is often used in CI systems to be able to inject version at build time via `mvn -Drevision=1.2.3 package`

2. Companies frequently require all dependencies to be pulled through an internal Maven repository that also proxies upstream. One easy way to accomplish this is by providing a `settings.xml` file that specifies that repository. For the purpose of this demonstration, we just override the name but leave the repository as Maven Central.

## Setup

Copy `setup/settings.xml` to `~/.m2/settings.xml`

This is necessary to demonstrate that `settings.xml` repository configuration is ignored when Quarkus downloads test dependencies. It configures a repository named "get-all-from-here" which simply points at maven central. In a real scenario, this would point at a company's private Maven repository.

## Expectation

All dependencies should be retrieved from "get-all-from-here" repository. If at any point we see dependencies pulled from "central", that proves our Maven configuration is not being honored.

## Test Steps

1. Purge local Maven repo to ensure all dependencies will be downloaded
2. Execute `mvn test` and search for lines that contain "central"

```
mvn clean
rm -Rf ~/.m2/repository
mvn test | grep central
```

Expectation is to have no output, meaning all dependencies were downloaded from our demonstration repository named "get-all-from-here"

## Results

* `master`
  ```
  git checkout master
  rm -Rf ~/.m2/repository
  mvn test | grep central
  ```
  
  Output:
  ```
  mvn test  49.28s user 5.27s system 48% cpu 1:51.89 total
  grep central  0.40s user 1.29s system 1% cpu 1:51.89 total
  ```
  
  Determination:
  * All dependencies pulled from "get-all-from-here" defined repository, as expected

* `q-1.4.1-maven-revision`
  ```
  git checkout q-1.4.1-maven-revision
  rm -Rf ~/.m2/repository
  mvn test | grep central
  ```
  
  Output:
  ```
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy/1.4.1.Final/quarkus-resteasy-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy/1.4.1.Final/quarkus-resteasy-1.4.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy-parent/1.4.1.Final/quarkus-resteasy-parent-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy-parent/1.4.1.Final/quarkus-resteasy-parent-1.4.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-build-parent/1.4.1.Final/quarkus-build-parent-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-build-parent/1.4.1.Final/quarkus-build-parent-1.4.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-parent/1.4.1.Final/quarkus-parent-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-parent/1.4.1.Final/quarkus-parent-1.4.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/jboss-parent/36/jboss-parent-36.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/jboss-parent/36/jboss-parent-36.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-bom-deployment/1.4.1.Final/quarkus-bom-deployment-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-bom-deployment/1.4.1.Final/quarkus-bom-deployment-1.4.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-bom/1.4.1.Final/quarkus-bom-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-bom/1.4.1.Final/quarkus-bom-1.4.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/netty/netty-bom/4.1.45.Final/netty-bom-4.1.45.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/netty/netty-bom/4.1.45.Final/netty-bom-4.1.45.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/sonatype/oss/oss-parent/7/oss-parent-7.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/sonatype/oss/oss-parent/7/oss-parent-7.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/com/fasterxml/jackson/jackson-bom/2.10.3/jackson-bom-2.10.3.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/com/fasterxml/jackson/jackson-bom/2.10.3/jackson-bom-2.10.3.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/com/fasterxml/jackson/jackson-parent/2.10/jackson-parent-2.10.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/com/fasterxml/jackson/jackson-parent/2.10/jackson-parent-2.10.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/com/fasterxml/oss-parent/38/oss-parent-38.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/com/fasterxml/oss-parent/38/oss-parent-38.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/resteasy/resteasy-bom/4.5.3.Final/resteasy-bom-4.5.3.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/resteasy/resteasy-bom/4.5.3.Final/resteasy-bom-4.5.3.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/jboss-parent/35/jboss-parent-35.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/jboss-parent/35/jboss-parent-35.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/junit/junit-bom/5.6.2/junit-bom-5.6.2.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/junit/junit-bom/5.6.2/junit-bom-5.6.2.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/fabric8/kubernetes-client-bom/4.9.0/kubernetes-client-bom-4.9.0.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/fabric8/kubernetes-client-bom/4.9.0/kubernetes-client-bom-4.9.0.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-stack-depchain/3.8.5/vertx-stack-depchain-3.8.5.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-stack-depchain/3.8.5/vertx-stack-depchain-3.8.5.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-stack/3.8.5/vertx-stack-3.8.5.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-stack/3.8.5/vertx-stack-3.8.5.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-parent/14/vertx-parent-14.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-parent/14/vertx-parent-14.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-dependencies/3.8.5/vertx-dependencies-3.8.5.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-dependencies/3.8.5/vertx-dependencies-3.8.5.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/optaplanner/optaplanner-bom/7.36.1.Final/optaplanner-bom-7.36.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/optaplanner/optaplanner-bom/7.36.1.Final/optaplanner-bom-7.36.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/kie/kie-user-bom-parent/7.36.1.Final/kie-user-bom-parent-7.36.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/kie/kie-user-bom-parent/7.36.1.Final/kie-user-bom-parent-7.36.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/jboss-parent/31/jboss-parent-31.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/jboss-parent/31/jboss-parent-31.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-vertx-http/1.4.1.Final/quarkus-vertx-http-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-vertx-http/1.4.1.Final/quarkus-vertx-http-1.4.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-vertx-http-parent/1.4.1.Final/quarkus-vertx-http-parent-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-vertx-http-parent/1.4.1.Final/quarkus-vertx-http-parent-1.4.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-core/1.4.1.Final/quarkus-core-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-core/1.4.1.Final/quarkus-core-1.4.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-core-parent/1.4.1.Final/quarkus-core-parent-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-core-parent/1.4.1.Final/quarkus-core-parent-1.4.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/jakarta/annotation/jakarta.annotation-api/1.3.5/jakarta.annotation-api-1.3.5.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/jakarta/annotation/jakarta.annotation-api/1.3.5/jakarta.annotation-api-1.3.5.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/jakarta/annotation/ca-parent/1.3.5/ca-parent-1.3.5.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/jakarta/annotation/ca-parent/1.3.5/ca-parent-1.3.5.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/eclipse/ee4j/project/1.0.5/project-1.0.5.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/eclipse/ee4j/project/1.0.5/project-1.0.5.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/jakarta/enterprise/jakarta.enterprise.cdi-api/2.0.2/jakarta.enterprise.cdi-api-2.0.2.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/jakarta/enterprise/jakarta.enterprise.cdi-api/2.0.2/jakarta.enterprise.cdi-api-2.0.2.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/jakarta/el/jakarta.el-api/3.0.3/jakarta.el-api-3.0.3.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/jakarta/el/jakarta.el-api/3.0.3/jakarta.el-api-3.0.3.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/jakarta/interceptor/jakarta.interceptor-api/1.2.5/jakarta.interceptor-api-1.2.5.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/jakarta/interceptor/jakarta.interceptor-api/1.2.5/jakarta.interceptor-api-1.2.5.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/jakarta/inject/jakarta.inject-api/1.0/jakarta.inject-api-1.0.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/jakarta/inject/jakarta.inject-api/1.0/jakarta.inject-api-1.0.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-ide-launcher/1.4.1.Final/quarkus-ide-launcher-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-ide-launcher/1.4.1.Final/quarkus-ide-launcher-1.4.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-development-mode-spi/1.4.1.Final/quarkus-development-mode-spi-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-development-mode-spi/1.4.1.Final/quarkus-development-mode-spi-1.4.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/smallrye/config/smallrye-config/1.7.0/smallrye-config-1.7.0.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/smallrye/config/smallrye-config/1.7.0/smallrye-config-1.7.0.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/smallrye/config/smallrye-config-parent/1.7.0/smallrye-config-parent-1.7.0.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/smallrye/config/smallrye-config-parent/1.7.0/smallrye-config-parent-1.7.0.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/smallrye/smallrye-parent/16/smallrye-parent-16.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/smallrye/smallrye-parent/16/smallrye-parent-16.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/arquillian/arquillian-bom/1.6.0.Final/arquillian-bom-1.6.0.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/arquillian/arquillian-bom/1.6.0.Final/arquillian-bom-1.6.0.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/shrinkwrap/shrinkwrap-bom/1.2.6/shrinkwrap-bom-1.2.6.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/shrinkwrap/shrinkwrap-bom/1.2.6/shrinkwrap-bom-1.2.6.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/shrinkwrap/resolver/shrinkwrap-resolver-bom/3.1.4/shrinkwrap-resolver-bom-3.1.4.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/shrinkwrap/resolver/shrinkwrap-resolver-bom/3.1.4/shrinkwrap-resolver-bom-3.1.4.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven/3.6.3/maven-3.6.3.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven/3.6.3/maven-3.6.3.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-parent/33/maven-parent-33.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-parent/33/maven-parent-33.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/apache/21/apache-21.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/apache/21/apache-21.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/shrinkwrap/descriptors/shrinkwrap-descriptors-bom/2.0.0/shrinkwrap-descriptors-bom-2.0.0.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/shrinkwrap/descriptors/shrinkwrap-descriptors-bom/2.0.0/shrinkwrap-descriptors-bom-2.0.0.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/smallrye/config/smallrye-config-common/1.7.0/smallrye-config-common-1.7.0.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/smallrye/config/smallrye-config-common/1.7.0/smallrye-config-common-1.7.0.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/eclipse/microprofile/config/microprofile-config-api/1.4/microprofile-config-api-1.4.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/eclipse/microprofile/config/microprofile-config-api/1.4/microprofile-config-api-1.4.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/eclipse/microprofile/config/microprofile-config-parent/1.4/microprofile-config-parent-1.4.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/eclipse/microprofile/config/microprofile-config-parent/1.4/microprofile-config-parent-1.4.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/arquillian/arquillian-bom/1.1.13.Final/arquillian-bom-1.1.13.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/arquillian/arquillian-bom/1.1.13.Final/arquillian-bom-1.1.13.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/shrinkwrap/resolver/shrinkwrap-resolver-bom/2.2.4/shrinkwrap-resolver-bom-2.2.4.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/shrinkwrap/resolver/shrinkwrap-resolver-bom/2.2.4/shrinkwrap-resolver-bom-2.2.4.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/shrinkwrap/descriptors/shrinkwrap-descriptors-bom/2.0.0-alpha-10/shrinkwrap-descriptors-bom-2.0.0-alpha-10.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/shrinkwrap/descriptors/shrinkwrap-descriptors-bom/2.0.0-alpha-10/shrinkwrap-descriptors-bom-2.0.0-alpha-10.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/logging/jboss-logging/3.3.2.Final/jboss-logging-3.3.2.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/logging/jboss-logging/3.3.2.Final/jboss-logging-3.3.2.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/jboss-parent/15/jboss-parent-15.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/jboss-parent/15/jboss-parent-15.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/logmanager/jboss-logmanager-embedded/1.0.4/jboss-logmanager-embedded-1.0.4.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/logmanager/jboss-logmanager-embedded/1.0.4/jboss-logmanager-embedded-1.0.4.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/jboss-parent-mr-jar/32/jboss-parent-mr-jar-32.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/jboss-parent-mr-jar/32/jboss-parent-mr-jar-32.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/jboss-parent/32/jboss-parent-32.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/jboss-parent/32/jboss-parent-32.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/wildfly/common/wildfly-common/1.5.4.Final-format-001/wildfly-common-1.5.4.Final-format-001.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/wildfly/common/wildfly-common/1.5.4.Final-format-001/wildfly-common-1.5.4.Final-format-001.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/jboss-parent/34/jboss-parent-34.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/jboss-parent/34/jboss-parent-34.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/logging/jboss-logging-annotations/2.1.0.Final/jboss-logging-annotations-2.1.0.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/logging/jboss-logging-annotations/2.1.0.Final/jboss-logging-annotations-2.1.0.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/logging/jboss-logging-tools-parent/2.1.0.Final/jboss-logging-tools-parent-2.1.0.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/logging/jboss-logging-tools-parent/2.1.0.Final/jboss-logging-tools-parent-2.1.0.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/jboss-parent/20/jboss-parent-20.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/jboss-parent/20/jboss-parent-20.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/threads/jboss-threads/3.1.1.Final/jboss-threads-3.1.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/threads/jboss-threads/3.1.1.Final/jboss-threads-3.1.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/slf4j/slf4j-api/1.7.30/slf4j-api-1.7.30.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/slf4j/slf4j-api/1.7.30/slf4j-api-1.7.30.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/slf4j/slf4j-parent/1.7.30/slf4j-parent-1.7.30.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/slf4j/slf4j-parent/1.7.30/slf4j-parent-1.7.30.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/slf4j/slf4j-jboss-logging/1.2.0.Final/slf4j-jboss-logging-1.2.0.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/slf4j/slf4j-jboss-logging/1.2.0.Final/slf4j-jboss-logging-1.2.0.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/graalvm/sdk/graal-sdk/19.3.1/graal-sdk-19.3.1.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/graalvm/sdk/graal-sdk/19.3.1/graal-sdk-19.3.1.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/security/quarkus-security/1.1.0.Final/quarkus-security-1.1.0.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/security/quarkus-security/1.1.0.Final/quarkus-security-1.1.0.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/smallrye/reactive/mutiny/0.4.4/mutiny-0.4.4.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/smallrye/reactive/mutiny/0.4.4/mutiny-0.4.4.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/smallrye/reactive/mutiny-project/0.4.4/mutiny-project-0.4.4.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/smallrye/reactive/mutiny-project/0.4.4/mutiny-project-0.4.4.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/smallrye/smallrye-parent/15/smallrye-parent-15.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/smallrye/smallrye-parent/15/smallrye-parent-15.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/arquillian/arquillian-bom/1.5.0.Final/arquillian-bom-1.5.0.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/arquillian/arquillian-bom/1.5.0.Final/arquillian-bom-1.5.0.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/shrinkwrap/resolver/shrinkwrap-resolver-bom/2.2.6/shrinkwrap-resolver-bom-2.2.6.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/shrinkwrap/resolver/shrinkwrap-resolver-bom/2.2.6/shrinkwrap-resolver-bom-2.2.6.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/reactivestreams/reactive-streams/1.0.3/reactive-streams-1.0.3.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/reactivestreams/reactive-streams/1.0.3/reactive-streams-1.0.3.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-vertx-core/1.4.1.Final/quarkus-vertx-core-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-vertx-core/1.4.1.Final/quarkus-vertx-core-1.4.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-vertx-core-parent/1.4.1.Final/quarkus-vertx-core-parent-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-vertx-core-parent/1.4.1.Final/quarkus-vertx-core-parent-1.4.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-arc/1.4.1.Final/quarkus-arc-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-arc/1.4.1.Final/quarkus-arc-1.4.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-arc-parent/1.4.1.Final/quarkus-arc-parent-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-arc-parent/1.4.1.Final/quarkus-arc-parent-1.4.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/arc/arc/1.4.1.Final/arc-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/arc/arc/1.4.1.Final/arc-1.4.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/arc/arc-parent/1.4.1.Final/arc-parent-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/arc/arc-parent/1.4.1.Final/arc-parent-1.4.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/jakarta/transaction/jakarta.transaction-api/1.3.3/jakarta.transaction-api-1.3.3.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/jakarta/transaction/jakarta.transaction-api/1.3.3/jakarta.transaction-api-1.3.3.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/eclipse/microprofile/context-propagation/microprofile-context-propagation-api/1.0.1/microprofile-context-propagation-api-1.0.1.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/eclipse/microprofile/context-propagation/microprofile-context-propagation-api/1.0.1/microprofile-context-propagation-api-1.0.1.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/eclipse/microprofile/context-propagation/microprofile-context-propagation-parent/1.0.1/microprofile-context-propagation-parent-1.0.1.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/eclipse/microprofile/context-propagation/microprofile-context-propagation-parent/1.0.1/microprofile-context-propagation-parent-1.0.1.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-netty/1.4.1.Final/quarkus-netty-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-netty/1.4.1.Final/quarkus-netty-1.4.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-netty-parent/1.4.1.Final/quarkus-netty-parent-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-netty-parent/1.4.1.Final/quarkus-netty-parent-1.4.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/netty/netty-codec/4.1.45.Final/netty-codec-4.1.45.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/netty/netty-codec/4.1.45.Final/netty-codec-4.1.45.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/netty/netty-parent/4.1.45.Final/netty-parent-4.1.45.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/netty/netty-parent/4.1.45.Final/netty-parent-4.1.45.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/sonatype/oss/oss-parent/9/oss-parent-9.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/sonatype/oss/oss-parent/9/oss-parent-9.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/netty/netty-common/4.1.45.Final/netty-common-4.1.45.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/netty/netty-common/4.1.45.Final/netty-common-4.1.45.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/netty/netty-buffer/4.1.45.Final/netty-buffer-4.1.45.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/netty/netty-buffer/4.1.45.Final/netty-buffer-4.1.45.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/netty/netty-transport/4.1.45.Final/netty-transport-4.1.45.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/netty/netty-transport/4.1.45.Final/netty-transport-4.1.45.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/netty/netty-resolver/4.1.45.Final/netty-resolver-4.1.45.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/netty/netty-resolver/4.1.45.Final/netty-resolver-4.1.45.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/netty/netty-handler/4.1.45.Final/netty-handler-4.1.45.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/netty/netty-handler/4.1.45.Final/netty-handler-4.1.45.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-core/3.8.5/vertx-core-3.8.5.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-core/3.8.5/vertx-core-3.8.5.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/netty/netty-handler-proxy/4.1.45.Final/netty-handler-proxy-4.1.45.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/netty/netty-handler-proxy/4.1.45.Final/netty-handler-proxy-4.1.45.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/netty/netty-codec-socks/4.1.45.Final/netty-codec-socks-4.1.45.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/netty/netty-codec-socks/4.1.45.Final/netty-codec-socks-4.1.45.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/netty/netty-codec-http/4.1.45.Final/netty-codec-http-4.1.45.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/netty/netty-codec-http/4.1.45.Final/netty-codec-http-4.1.45.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/netty/netty-codec-http2/4.1.45.Final/netty-codec-http2-4.1.45.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/netty/netty-codec-http2/4.1.45.Final/netty-codec-http2-4.1.45.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/netty/netty-resolver-dns/4.1.45.Final/netty-resolver-dns-4.1.45.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/netty/netty-resolver-dns/4.1.45.Final/netty-resolver-dns-4.1.45.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/netty/netty-codec-dns/4.1.45.Final/netty-codec-dns-4.1.45.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/netty/netty-codec-dns/4.1.45.Final/netty-codec-dns-4.1.45.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/com/fasterxml/jackson/core/jackson-core/2.10.3/jackson-core-2.10.3.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/com/fasterxml/jackson/core/jackson-core/2.10.3/jackson-core-2.10.3.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/com/fasterxml/jackson/jackson-base/2.10.3/jackson-base-2.10.3.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/com/fasterxml/jackson/jackson-base/2.10.3/jackson-base-2.10.3.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-web/3.8.5/vertx-web-3.8.5.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-web/3.8.5/vertx-web-3.8.5.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-web-parent/3.8.5/vertx-web-parent-3.8.5.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-web-parent/3.8.5/vertx-web-parent-3.8.5.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-ext-parent/34/vertx-ext-parent-34.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-ext-parent/34/vertx-ext-parent-34.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-ext/34/vertx-ext-34.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-ext/34/vertx-ext-34.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-web-common/3.8.5/vertx-web-common-3.8.5.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-web-common/3.8.5/vertx-web-common-3.8.5.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-auth-common/3.8.5/vertx-auth-common-3.8.5.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-auth-common/3.8.5/vertx-auth-common-3.8.5.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-auth/3.8.5/vertx-auth-3.8.5.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-auth/3.8.5/vertx-auth-3.8.5.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-bridge-common/3.8.5/vertx-bridge-common-3.8.5.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-bridge-common/3.8.5/vertx-bridge-common-3.8.5.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy-server-common/1.4.1.Final/quarkus-resteasy-server-common-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy-server-common/1.4.1.Final/quarkus-resteasy-server-common-1.4.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy-server-common-parent/1.4.1.Final/quarkus-resteasy-server-common-parent-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy-server-common-parent/1.4.1.Final/quarkus-resteasy-server-common-parent-1.4.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy-common/1.4.1.Final/quarkus-resteasy-common-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy-common/1.4.1.Final/quarkus-resteasy-common-1.4.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy-common-parent/1.4.1.Final/quarkus-resteasy-common-parent-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy-common-parent/1.4.1.Final/quarkus-resteasy-common-parent-1.4.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/resteasy/resteasy-core/4.5.3.Final/resteasy-core-4.5.3.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/resteasy/resteasy-core/4.5.3.Final/resteasy-core-4.5.3.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/resteasy/resteasy-jaxrs-all/4.5.3.Final/resteasy-jaxrs-all-4.5.3.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/resteasy/resteasy-jaxrs-all/4.5.3.Final/resteasy-jaxrs-all-4.5.3.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/resteasy/resteasy-dependencies/4.5.3.Final/resteasy-dependencies-4.5.3.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/resteasy/resteasy-dependencies/4.5.3.Final/resteasy-dependencies-4.5.3.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/arquillian/arquillian-bom/1.4.1.Final/arquillian-bom-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/arquillian/arquillian-bom/1.4.1.Final/arquillian-bom-1.4.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/projectreactor/reactor-bom/Dysprosium-SR2/reactor-bom-Dysprosium-SR2.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/projectreactor/reactor-bom/Dysprosium-SR2/reactor-bom-Dysprosium-SR2.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/spec/javax/ws/rs/jboss-jaxrs-api_2.1_spec/2.0.1.Final/jboss-jaxrs-api_2.1_spec-2.0.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/spec/javax/ws/rs/jboss-jaxrs-api_2.1_spec/2.0.1.Final/jboss-jaxrs-api_2.1_spec-2.0.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/spec/javax/xml/bind/jboss-jaxb-api_2.3_spec/2.0.0.Final/jboss-jaxb-api_2.3_spec-2.0.0.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/spec/javax/xml/bind/jboss-jaxb-api_2.3_spec/2.0.0.Final/jboss-jaxb-api_2.3_spec-2.0.0.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/spec/javax/xml/bind/jboss-jaxb-api_2.3_spec-parent/2.0.0.Final/jboss-jaxb-api_2.3_spec-parent-2.0.0.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/spec/javax/xml/bind/jboss-jaxb-api_2.3_spec-parent/2.0.0.Final/jboss-jaxb-api_2.3_spec-parent-2.0.0.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/resteasy/resteasy-core-spi/4.5.3.Final/resteasy-core-spi-4.5.3.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/resteasy/resteasy-core-spi/4.5.3.Final/resteasy-core-spi-4.5.3.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/jakarta/validation/jakarta.validation-api/2.0.2/jakarta.validation-api-2.0.2.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/jakarta/validation/jakarta.validation-api/2.0.2/jakarta.validation-api-2.0.2.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/com/sun/activation/jakarta.activation/1.2.1/jakarta.activation-1.2.1.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/com/sun/activation/jakarta.activation/1.2.1/jakarta.activation-1.2.1.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/com/sun/activation/all/1.2.1/all-1.2.1.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/com/sun/activation/all/1.2.1/all-1.2.1.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/eclipse/ee4j/project/1.0.2/project-1.0.2.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/eclipse/ee4j/project/1.0.2/project-1.0.2.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-junit5/1.4.1.Final/quarkus-junit5-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-junit5/1.4.1.Final/quarkus-junit5-1.4.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-test-framework/1.4.1.Final/quarkus-test-framework-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-test-framework/1.4.1.Final/quarkus-test-framework-1.4.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-bootstrap-core/1.4.1.Final/quarkus-bootstrap-core-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-bootstrap-core/1.4.1.Final/quarkus-bootstrap-core-1.4.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-bootstrap-parent/1.4.1.Final/quarkus-bootstrap-parent-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-bootstrap-parent/1.4.1.Final/quarkus-bootstrap-parent-1.4.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/ow2/asm/asm/7.3.1/asm-7.3.1.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/ow2/asm/asm/7.3.1/asm-7.3.1.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/ow2/ow2/1.5/ow2-1.5.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/ow2/ow2/1.5/ow2-1.5.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-embedder/3.6.3/maven-embedder-3.6.3.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-embedder/3.6.3/maven-embedder-3.6.3.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-settings/3.6.3/maven-settings-3.6.3.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-settings/3.6.3/maven-settings-3.6.3.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/codehaus/plexus/plexus-utils/3.2.1/plexus-utils-3.2.1.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/codehaus/plexus/plexus-utils/3.2.1/plexus-utils-3.2.1.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/codehaus/plexus/plexus/5.1/plexus-5.1.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/codehaus/plexus/plexus/5.1/plexus-5.1.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-settings-builder/3.6.3/maven-settings-builder-3.6.3.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-settings-builder/3.6.3/maven-settings-builder-3.6.3.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-builder-support/3.6.3/maven-builder-support-3.6.3.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-builder-support/3.6.3/maven-builder-support-3.6.3.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/codehaus/plexus/plexus-interpolation/1.25/plexus-interpolation-1.25.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/codehaus/plexus/plexus-interpolation/1.25/plexus-interpolation-1.25.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-core/3.6.3/maven-core-3.6.3.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-core/3.6.3/maven-core-3.6.3.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-model/3.6.3/maven-model-3.6.3.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-model/3.6.3/maven-model-3.6.3.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-repository-metadata/3.6.3/maven-repository-metadata-3.6.3.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-repository-metadata/3.6.3/maven-repository-metadata-3.6.3.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-artifact/3.6.2/maven-artifact-3.6.2.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-artifact/3.6.2/maven-artifact-3.6.2.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven/3.6.2/maven-3.6.2.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven/3.6.2/maven-3.6.2.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/commons/commons-lang3/3.9/commons-lang3-3.9.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/commons/commons-lang3/3.9/commons-lang3-3.9.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/commons/commons-parent/48/commons-parent-48.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/commons/commons-parent/48/commons-parent-48.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-plugin-api/3.6.3/maven-plugin-api-3.6.3.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-plugin-api/3.6.3/maven-plugin-api-3.6.3.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/eclipse/sisu/org.eclipse.sisu.plexus/0.3.4/org.eclipse.sisu.plexus-0.3.4.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/eclipse/sisu/org.eclipse.sisu.plexus/0.3.4/org.eclipse.sisu.plexus-0.3.4.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/eclipse/sisu/sisu-plexus/0.3.4/sisu-plexus-0.3.4.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/eclipse/sisu/sisu-plexus/0.3.4/sisu-plexus-0.3.4.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/codehaus/plexus/plexus-component-annotations/2.1.0/plexus-component-annotations-2.1.0.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/codehaus/plexus/plexus-component-annotations/2.1.0/plexus-component-annotations-2.1.0.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/codehaus/plexus/plexus-containers/2.1.0/plexus-containers-2.1.0.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/codehaus/plexus/plexus-containers/2.1.0/plexus-containers-2.1.0.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-model-builder/3.6.3/maven-model-builder-3.6.3.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-model-builder/3.6.3/maven-model-builder-3.6.3.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-resolver-provider/3.6.3/maven-resolver-provider-3.6.3.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-resolver-provider/3.6.3/maven-resolver-provider-3.6.3.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/resolver/maven-resolver-api/1.4.1/maven-resolver-api-1.4.1.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/resolver/maven-resolver-api/1.4.1/maven-resolver-api-1.4.1.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/resolver/maven-resolver/1.4.1/maven-resolver-1.4.1.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/resolver/maven-resolver/1.4.1/maven-resolver-1.4.1.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/resolver/maven-resolver-spi/1.4.1/maven-resolver-spi-1.4.1.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/resolver/maven-resolver-spi/1.4.1/maven-resolver-spi-1.4.1.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/resolver/maven-resolver-util/1.4.1/maven-resolver-util-1.4.1.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/resolver/maven-resolver-util/1.4.1/maven-resolver-util-1.4.1.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/resolver/maven-resolver-impl/1.4.1/maven-resolver-impl-1.4.1.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/resolver/maven-resolver-impl/1.4.1/maven-resolver-impl-1.4.1.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/shared/maven-shared-utils/3.2.1/maven-shared-utils-3.2.1.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/shared/maven-shared-utils/3.2.1/maven-shared-utils-3.2.1.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/shared/maven-shared-components/30/maven-shared-components-30.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/shared/maven-shared-components/30/maven-shared-components-30.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-parent/30/maven-parent-30.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-parent/30/maven-parent-30.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/apache/18/apache-18.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/apache/18/apache-18.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/commons-io/commons-io/2.6/commons-io-2.6.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/commons-io/commons-io/2.6/commons-io-2.6.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/commons/commons-parent/42/commons-parent-42.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/commons/commons-parent/42/commons-parent-42.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/com/google/inject/guice/4.2.1/guice-4.2.1.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/com/google/inject/guice/4.2.1/guice-4.2.1.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/com/google/inject/guice-parent/4.2.1/guice-parent-4.2.1.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/com/google/inject/guice-parent/4.2.1/guice-parent-4.2.1.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/com/google/google/5/google-5.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/com/google/google/5/google-5.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/aopalliance/aopalliance/1.0/aopalliance-1.0.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/aopalliance/aopalliance/1.0/aopalliance-1.0.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/com/google/guava/guava/27.0.1-jre/guava-27.0.1-jre.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/com/google/guava/guava/27.0.1-jre/guava-27.0.1-jre.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/com/google/guava/guava-parent/27.0.1-jre/guava-parent-27.0.1-jre.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/com/google/guava/guava-parent/27.0.1-jre/guava-parent-27.0.1-jre.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/com/google/guava/failureaccess/1.0.1/failureaccess-1.0.1.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/com/google/guava/failureaccess/1.0.1/failureaccess-1.0.1.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/com/google/guava/guava-parent/26.0-android/guava-parent-26.0-android.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/com/google/guava/guava-parent/26.0-android/guava-parent-26.0-android.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/com/google/guava/listenablefuture/9999.0-empty-to-avoid-conflict-with-guava/listenablefuture-9999.0-empty-to-avoid-conflict-with-guava.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/com/google/guava/listenablefuture/9999.0-empty-to-avoid-conflict-with-guava/listenablefuture-9999.0-empty-to-avoid-conflict-with-guava.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/com/google/code/findbugs/jsr305/3.0.2/jsr305-3.0.2.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/com/google/code/findbugs/jsr305/3.0.2/jsr305-3.0.2.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/com/google/errorprone/error_prone_annotations/2.2.0/error_prone_annotations-2.2.0.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/com/google/errorprone/error_prone_annotations/2.2.0/error_prone_annotations-2.2.0.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/com/google/errorprone/error_prone_parent/2.2.0/error_prone_parent-2.2.0.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/com/google/errorprone/error_prone_parent/2.2.0/error_prone_parent-2.2.0.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/com/google/j2objc/j2objc-annotations/1.1/j2objc-annotations-1.1.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/com/google/j2objc/j2objc-annotations/1.1/j2objc-annotations-1.1.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/codehaus/mojo/animal-sniffer-annotations/1.17/animal-sniffer-annotations-1.17.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/codehaus/mojo/animal-sniffer-annotations/1.17/animal-sniffer-annotations-1.17.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/codehaus/mojo/animal-sniffer-parent/1.17/animal-sniffer-parent-1.17.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/codehaus/mojo/animal-sniffer-parent/1.17/animal-sniffer-parent-1.17.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/codehaus/mojo/mojo-parent/40/mojo-parent-40.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/codehaus/mojo/mojo-parent/40/mojo-parent-40.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/commons-cli/commons-cli/1.4/commons-cli-1.4.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/commons-cli/commons-cli/1.4/commons-cli-1.4.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/sonatype/plexus/plexus-sec-dispatcher/1.4/plexus-sec-dispatcher-1.4.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/sonatype/plexus/plexus-sec-dispatcher/1.4/plexus-sec-dispatcher-1.4.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/sonatype/spice/spice-parent/12/spice-parent-12.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/sonatype/spice/spice-parent/12/spice-parent-12.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/sonatype/forge/forge-parent/4/forge-parent-4.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/sonatype/forge/forge-parent/4/forge-parent-4.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/sonatype/plexus/plexus-cipher/1.4/plexus-cipher-1.4.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/sonatype/plexus/plexus-cipher/1.4/plexus-cipher-1.4.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/resolver/maven-resolver-connector-basic/1.4.1/maven-resolver-connector-basic-1.4.1.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/resolver/maven-resolver-connector-basic/1.4.1/maven-resolver-connector-basic-1.4.1.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/resolver/maven-resolver-transport-wagon/1.4.1/maven-resolver-transport-wagon-1.4.1.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/resolver/maven-resolver-transport-wagon/1.4.1/maven-resolver-transport-wagon-1.4.1.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/wagon/wagon-http/3.3.4/wagon-http-3.3.4.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/wagon/wagon-http/3.3.4/wagon-http-3.3.4.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/wagon/wagon-providers/3.3.4/wagon-providers-3.3.4.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/wagon/wagon-providers/3.3.4/wagon-providers-3.3.4.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/wagon/wagon/3.3.4/wagon-3.3.4.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/wagon/wagon/3.3.4/wagon-3.3.4.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/wagon/wagon-http-shared/3.3.4/wagon-http-shared-3.3.4.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/wagon/wagon-http-shared/3.3.4/wagon-http-shared-3.3.4.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jsoup/jsoup/1.12.1/jsoup-1.12.1.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jsoup/jsoup/1.12.1/jsoup-1.12.1.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/httpcomponents/httpclient/4.5.12/httpclient-4.5.12.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/httpcomponents/httpclient/4.5.12/httpclient-4.5.12.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/httpcomponents/httpcomponents-client/4.5.12/httpcomponents-client-4.5.12.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/httpcomponents/httpcomponents-client/4.5.12/httpcomponents-client-4.5.12.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/httpcomponents/httpcomponents-parent/11/httpcomponents-parent-11.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/httpcomponents/httpcomponents-parent/11/httpcomponents-parent-11.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/httpcomponents/httpcore/4.4.13/httpcore-4.4.13.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/httpcomponents/httpcore/4.4.13/httpcore-4.4.13.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/httpcomponents/httpcomponents-core/4.4.13/httpcomponents-core-4.4.13.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/httpcomponents/httpcomponents-core/4.4.13/httpcomponents-core-4.4.13.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/commons-codec/commons-codec/1.13/commons-codec-1.13.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/commons-codec/commons-codec/1.13/commons-codec-1.13.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/wagon/wagon-provider-api/3.3.4/wagon-provider-api-3.3.4.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/wagon/wagon-provider-api/3.3.4/wagon-provider-api-3.3.4.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/wagon/wagon-file/3.3.4/wagon-file-3.3.4.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/wagon/wagon-file/3.3.4/wagon-file-3.3.4.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/logging/commons-logging-jboss-logging/1.0.0.Final/commons-logging-jboss-logging-1.0.0.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/logging/commons-logging-jboss-logging/1.0.0.Final/commons-logging-jboss-logging-1.0.0.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/jboss-parent/18/jboss-parent-18.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/jboss-parent/18/jboss-parent-18.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-test-common/1.4.1.Final/quarkus-test-common-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-test-common/1.4.1.Final/quarkus-test-common-1.4.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-core-deployment/1.4.1.Final/quarkus-core-deployment-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-core-deployment/1.4.1.Final/quarkus-core-deployment-1.4.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/gizmo/gizmo/1.0.2.Final/gizmo-1.0.2.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/gizmo/gizmo/1.0.2.Final/gizmo-1.0.2.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/ow2/asm/asm-util/7.3.1/asm-util-7.3.1.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/ow2/asm/asm-util/7.3.1/asm-util-7.3.1.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/ow2/asm/asm-tree/7.3.1/asm-tree-7.3.1.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/ow2/asm/asm-tree/7.3.1/asm-tree-7.3.1.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/ow2/asm/asm-analysis/7.3.1/asm-analysis-7.3.1.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/ow2/asm/asm-analysis/7.3.1/asm-analysis-7.3.1.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/jandex/2.1.3.Final/jandex-2.1.3.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/jandex/2.1.3.Final/jandex-2.1.3.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/jboss-parent/12/jboss-parent-12.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/jboss-parent/12/jboss-parent-12.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-builder/1.4.1.Final/quarkus-builder-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-builder/1.4.1.Final/quarkus-builder-1.4.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-jsonp-deployment/1.4.1.Final/quarkus-jsonp-deployment-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-jsonp-deployment/1.4.1.Final/quarkus-jsonp-deployment-1.4.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-jsonp-parent/1.4.1.Final/quarkus-jsonp-parent-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-jsonp-parent/1.4.1.Final/quarkus-jsonp-parent-1.4.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-jsonp/1.4.1.Final/quarkus-jsonp-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-jsonp/1.4.1.Final/quarkus-jsonp-1.4.1.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/glassfish/jakarta.json/1.1.6/jakarta.json-1.1.6.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/glassfish/jakarta.json/1.1.6/jakarta.json-1.1.6.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/glassfish/json/1.1.6/json-1.1.6.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/glassfish/json/1.1.6/json-1.1.6.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/junit/jupiter/junit-jupiter/5.6.2/junit-jupiter-5.6.2.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/junit/jupiter/junit-jupiter/5.6.2/junit-jupiter-5.6.2.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/junit/jupiter/junit-jupiter-api/5.6.2/junit-jupiter-api-5.6.2.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/junit/jupiter/junit-jupiter-api/5.6.2/junit-jupiter-api-5.6.2.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apiguardian/apiguardian-api/1.1.0/apiguardian-api-1.1.0.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apiguardian/apiguardian-api/1.1.0/apiguardian-api-1.1.0.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/opentest4j/opentest4j/1.2.0/opentest4j-1.2.0.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/opentest4j/opentest4j/1.2.0/opentest4j-1.2.0.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/junit/platform/junit-platform-commons/1.6.2/junit-platform-commons-1.6.2.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/junit/platform/junit-platform-commons/1.6.2/junit-platform-commons-1.6.2.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/junit/jupiter/junit-jupiter-params/5.6.2/junit-jupiter-params-5.6.2.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/junit/jupiter/junit-jupiter-params/5.6.2/junit-jupiter-params-5.6.2.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/junit/jupiter/junit-jupiter-engine/5.6.2/junit-jupiter-engine-5.6.2.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/junit/jupiter/junit-jupiter-engine/5.6.2/junit-jupiter-engine-5.6.2.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/junit/platform/junit-platform-engine/1.6.2/junit-platform-engine-1.6.2.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/junit/platform/junit-platform-engine/1.6.2/junit-platform-engine-1.6.2.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/rest-assured/rest-assured/4.3.0/rest-assured-4.3.0.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/rest-assured/rest-assured/4.3.0/rest-assured-4.3.0.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/rest-assured/rest-assured-parent/4.3.0/rest-assured-parent-4.3.0.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/rest-assured/rest-assured-parent/4.3.0/rest-assured-parent-4.3.0.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/sonatype/oss/oss-parent/5/oss-parent-5.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/sonatype/oss/oss-parent/5/oss-parent-5.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/codehaus/groovy/groovy/3.0.2/groovy-3.0.2.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/codehaus/groovy/groovy/3.0.2/groovy-3.0.2.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/codehaus/groovy/groovy-xml/3.0.2/groovy-xml-3.0.2.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/codehaus/groovy/groovy-xml/3.0.2/groovy-xml-3.0.2.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/httpcomponents/httpmime/4.5.3/httpmime-4.5.3.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/httpcomponents/httpmime/4.5.3/httpmime-4.5.3.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/httpcomponents/httpcomponents-client/4.5.3/httpcomponents-client-4.5.3.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/httpcomponents/httpcomponents-client/4.5.3/httpcomponents-client-4.5.3.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/httpcomponents/project/7/project-7.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/httpcomponents/project/7/project-7.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/apache/13/apache-13.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/apache/13/apache-13.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/hamcrest/hamcrest/2.1/hamcrest-2.1.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/hamcrest/hamcrest/2.1/hamcrest-2.1.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/ccil/cowan/tagsoup/tagsoup/1.2.1/tagsoup-1.2.1.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/ccil/cowan/tagsoup/tagsoup/1.2.1/tagsoup-1.2.1.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/rest-assured/json-path/4.3.0/json-path-4.3.0.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/rest-assured/json-path/4.3.0/json-path-4.3.0.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/codehaus/groovy/groovy-json/3.0.2/groovy-json-3.0.2.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/codehaus/groovy/groovy-json/3.0.2/groovy-json-3.0.2.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/rest-assured/rest-assured-common/4.3.0/rest-assured-common-4.3.0.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/rest-assured/rest-assured-common/4.3.0/rest-assured-common-4.3.0.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/rest-assured/xml-path/4.3.0/xml-path-4.3.0.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/rest-assured/xml-path/4.3.0/xml-path-4.3.0.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/jakarta/xml/bind/jakarta.xml.bind-api/2.3.2/jakarta.xml.bind-api-2.3.2.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/jakarta/xml/bind/jakarta.xml.bind-api/2.3.2/jakarta.xml.bind-api-2.3.2.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/jakarta/xml/bind/jakarta.xml.bind-api-parent/2.3.2/jakarta.xml.bind-api-parent-2.3.2.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/jakarta/xml/bind/jakarta.xml.bind-api-parent/2.3.2/jakarta.xml.bind-api-parent-2.3.2.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/jakarta/activation/jakarta.activation-api/1.2.1/jakarta.activation-api-1.2.1.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/jakarta/activation/jakarta.activation-api/1.2.1/jakarta.activation-api-1.2.1.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/sling/org.apache.sling.javax.activation/0.1.0/org.apache.sling.javax.activation-0.1.0.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/sling/org.apache.sling.javax.activation/0.1.0/org.apache.sling.javax.activation-0.1.0.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/sling/sling/15/sling-15.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/sling/sling/15/sling-15.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/apache/12/apache-12.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/apache/12/apache-12.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy/1.4.1.Final/quarkus-resteasy-1.4.1.Final.jar
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-vertx-http/1.4.1.Final/quarkus-vertx-http-1.4.1.Final.jar
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-development-mode-spi/1.4.1.Final/quarkus-development-mode-spi-1.4.1.Final.jar
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/security/quarkus-security/1.1.0.Final/quarkus-security-1.1.0.Final.jar
  Downloading from central: https://repo.maven.apache.org/maven2/io/smallrye/reactive/mutiny/0.4.4/mutiny-0.4.4.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy/1.4.1.Final/quarkus-resteasy-1.4.1.Final.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/reactivestreams/reactive-streams/1.0.3/reactive-streams-1.0.3.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/reactivestreams/reactive-streams/1.0.3/reactive-streams-1.0.3.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/jakarta/enterprise/jakarta.enterprise.cdi-api/2.0.2/jakarta.enterprise.cdi-api-2.0.2.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/security/quarkus-security/1.1.0.Final/quarkus-security-1.1.0.Final.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/jakarta/el/jakarta.el-api/3.0.3/jakarta.el-api-3.0.3.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/jakarta/enterprise/jakarta.enterprise.cdi-api/2.0.2/jakarta.enterprise.cdi-api-2.0.2.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-vertx-http/1.4.1.Final/quarkus-vertx-http-1.4.1.Final.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/jakarta/interceptor/jakarta.interceptor-api/1.2.5/jakarta.interceptor-api-1.2.5.jar
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-vertx-core/1.4.1.Final/quarkus-vertx-core-1.4.1.Final.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/io/smallrye/reactive/mutiny/0.4.4/mutiny-0.4.4.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-development-mode-spi/1.4.1.Final/quarkus-development-mode-spi-1.4.1.Final.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-netty/1.4.1.Final/quarkus-netty-1.4.1.Final.jar
  Downloading from central: https://repo.maven.apache.org/maven2/io/netty/netty-codec/4.1.45.Final/netty-codec-4.1.45.Final.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-vertx-core/1.4.1.Final/quarkus-vertx-core-1.4.1.Final.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/netty/netty-handler/4.1.45.Final/netty-handler-4.1.45.Final.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/jakarta/el/jakarta.el-api/3.0.3/jakarta.el-api-3.0.3.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/jakarta/interceptor/jakarta.interceptor-api/1.2.5/jakarta.interceptor-api-1.2.5.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/io/netty/netty-codec/4.1.45.Final/netty-codec-4.1.45.Final.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-core/3.8.5/vertx-core-3.8.5.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-netty/1.4.1.Final/quarkus-netty-1.4.1.Final.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/netty/netty-buffer/4.1.45.Final/netty-buffer-4.1.45.Final.jar
  Downloading from central: https://repo.maven.apache.org/maven2/io/netty/netty-transport/4.1.45.Final/netty-transport-4.1.45.Final.jar
  Downloading from central: https://repo.maven.apache.org/maven2/io/netty/netty-common/4.1.45.Final/netty-common-4.1.45.Final.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/io/netty/netty-buffer/4.1.45.Final/netty-buffer-4.1.45.Final.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/netty/netty-handler-proxy/4.1.45.Final/netty-handler-proxy-4.1.45.Final.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/io/netty/netty-transport/4.1.45.Final/netty-transport-4.1.45.Final.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/io/netty/netty-common/4.1.45.Final/netty-common-4.1.45.Final.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/netty/netty-codec-socks/4.1.45.Final/netty-codec-socks-4.1.45.Final.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/io/netty/netty-handler/4.1.45.Final/netty-handler-4.1.45.Final.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/netty/netty-codec-http/4.1.45.Final/netty-codec-http-4.1.45.Final.jar
  Downloading from central: https://repo.maven.apache.org/maven2/io/netty/netty-codec-http2/4.1.45.Final/netty-codec-http2-4.1.45.Final.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-core/3.8.5/vertx-core-3.8.5.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/netty/netty-resolver/4.1.45.Final/netty-resolver-4.1.45.Final.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/io/netty/netty-codec-http/4.1.45.Final/netty-codec-http-4.1.45.Final.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/io/netty/netty-handler-proxy/4.1.45.Final/netty-handler-proxy-4.1.45.Final.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/io/netty/netty-resolver/4.1.45.Final/netty-resolver-4.1.45.Final.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/io/netty/netty-codec-socks/4.1.45.Final/netty-codec-socks-4.1.45.Final.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/com/fasterxml/jackson/core/jackson-core/2.10.3/jackson-core-2.10.3.jar
  Downloading from central: https://repo.maven.apache.org/maven2/io/netty/netty-codec-dns/4.1.45.Final/netty-codec-dns-4.1.45.Final.jar
  Downloading from central: https://repo.maven.apache.org/maven2/io/netty/netty-resolver-dns/4.1.45.Final/netty-resolver-dns-4.1.45.Final.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/io/netty/netty-codec-http2/4.1.45.Final/netty-codec-http2-4.1.45.Final.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-web/3.8.5/vertx-web-3.8.5.jar
  Downloading from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-web-common/3.8.5/vertx-web-common-3.8.5.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/io/netty/netty-codec-dns/4.1.45.Final/netty-codec-dns-4.1.45.Final.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-auth-common/3.8.5/vertx-auth-common-3.8.5.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/io/netty/netty-resolver-dns/4.1.45.Final/netty-resolver-dns-4.1.45.Final.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/com/fasterxml/jackson/core/jackson-core/2.10.3/jackson-core-2.10.3.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-bridge-common/3.8.5/vertx-bridge-common-3.8.5.jar
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy-server-common/1.4.1.Final/quarkus-resteasy-server-common-1.4.1.Final.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-web/3.8.5/vertx-web-3.8.5.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-arc/1.4.1.Final/quarkus-arc-1.4.1.Final.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-web-common/3.8.5/vertx-web-common-3.8.5.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/arc/arc/1.4.1.Final/arc-1.4.1.Final.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy-server-common/1.4.1.Final/quarkus-resteasy-server-common-1.4.1.Final.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/arc/arc/1.4.1.Final/arc-1.4.1.Final.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-arc/1.4.1.Final/quarkus-arc-1.4.1.Final.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-bridge-common/3.8.5/vertx-bridge-common-3.8.5.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/io/vertx/vertx-auth-common/3.8.5/vertx-auth-common-3.8.5.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/resteasy/resteasy-core/4.5.3.Final/resteasy-core-4.5.3.Final.jar
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy-common/1.4.1.Final/quarkus-resteasy-common-1.4.1.Final.jar
  Downloading from central: https://repo.maven.apache.org/maven2/org/eclipse/microprofile/context-propagation/microprofile-context-propagation-api/1.0.1/microprofile-context-propagation-api-1.0.1.jar
  Downloading from central: https://repo.maven.apache.org/maven2/jakarta/transaction/jakarta.transaction-api/1.3.3/jakarta.transaction-api-1.3.3.jar
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/spec/javax/ws/rs/jboss-jaxrs-api_2.1_spec/2.0.1.Final/jboss-jaxrs-api_2.1_spec-2.0.1.Final.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/jakarta/transaction/jakarta.transaction-api/1.3.3/jakarta.transaction-api-1.3.3.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy-common/1.4.1.Final/quarkus-resteasy-common-1.4.1.Final.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/spec/javax/xml/bind/jboss-jaxb-api_2.3_spec/2.0.0.Final/jboss-jaxb-api_2.3_spec-2.0.0.Final.jar
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/resteasy/resteasy-core-spi/4.5.3.Final/resteasy-core-spi-4.5.3.Final.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/eclipse/microprofile/context-propagation/microprofile-context-propagation-api/1.0.1/microprofile-context-propagation-api-1.0.1.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/resteasy/resteasy-core/4.5.3.Final/resteasy-core-4.5.3.Final.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/eclipse/microprofile/config/microprofile-config-api/1.4/microprofile-config-api-1.4.jar
  Downloading from central: https://repo.maven.apache.org/maven2/com/sun/activation/jakarta.activation/1.2.1/jakarta.activation-1.2.1.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/spec/javax/ws/rs/jboss-jaxrs-api_2.1_spec/2.0.1.Final/jboss-jaxrs-api_2.1_spec-2.0.1.Final.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/jakarta/validation/jakarta.validation-api/2.0.2/jakarta.validation-api-2.0.2.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/resteasy/resteasy-core-spi/4.5.3.Final/resteasy-core-spi-4.5.3.Final.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/jakarta/validation/jakarta.validation-api/2.0.2/jakarta.validation-api-2.0.2.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/org/eclipse/microprofile/config/microprofile-config-api/1.4/microprofile-config-api-1.4.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/ow2/asm/asm/7.3.1/asm-7.3.1.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/com/sun/activation/jakarta.activation/1.2.1/jakarta.activation-1.2.1.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-bootstrap-core/1.4.1.Final/quarkus-bootstrap-core-1.4.1.Final.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/spec/javax/xml/bind/jboss-jaxb-api_2.3_spec/2.0.0.Final/jboss-jaxb-api_2.3_spec-2.0.0.Final.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-junit5/1.4.1.Final/quarkus-junit5-1.4.1.Final.jar
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-settings/3.6.3/maven-settings-3.6.3.jar
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-embedder/3.6.3/maven-embedder-3.6.3.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-settings/3.6.3/maven-settings-3.6.3.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-core/3.6.3/maven-core-3.6.3.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-embedder/3.6.3/maven-embedder-3.6.3.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-artifact/3.6.2/maven-artifact-3.6.2.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-junit5/1.4.1.Final/quarkus-junit5-1.4.1.Final.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/codehaus/plexus/plexus-component-annotations/2.1.0/plexus-component-annotations-2.1.0.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/ow2/asm/asm/7.3.1/asm-7.3.1.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-bootstrap-core/1.4.1.Final/quarkus-bootstrap-core-1.4.1.Final.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-plugin-api/3.6.3/maven-plugin-api-3.6.3.jar
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-model/3.6.3/maven-model-3.6.3.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-model/3.6.3/maven-model-3.6.3.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-plugin-api/3.6.3/maven-plugin-api-3.6.3.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-artifact/3.6.2/maven-artifact-3.6.2.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-core/3.6.3/maven-core-3.6.3.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/org/codehaus/plexus/plexus-component-annotations/2.1.0/plexus-component-annotations-2.1.0.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/resolver/maven-resolver-util/1.4.1/maven-resolver-util-1.4.1.jar
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/resolver/maven-resolver-api/1.4.1/maven-resolver-api-1.4.1.jar
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-builder-support/3.6.3/maven-builder-support-3.6.3.jar
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-model-builder/3.6.3/maven-model-builder-3.6.3.jar
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/shared/maven-shared-utils/3.2.1/maven-shared-utils-3.2.1.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-builder-support/3.6.3/maven-builder-support-3.6.3.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/commons-io/commons-io/2.6/commons-io-2.6.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/resolver/maven-resolver-util/1.4.1/maven-resolver-util-1.4.1.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/resolver/maven-resolver-api/1.4.1/maven-resolver-api-1.4.1.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/com/google/inject/guice/4.2.1/guice-4.2.1-no_aop.jar
  Downloading from central: https://repo.maven.apache.org/maven2/com/google/guava/guava/27.0.1-jre/guava-27.0.1-jre.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-model-builder/3.6.3/maven-model-builder-3.6.3.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/shared/maven-shared-utils/3.2.1/maven-shared-utils-3.2.1.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/com/google/guava/failureaccess/1.0.1/failureaccess-1.0.1.jar
  Downloading from central: https://repo.maven.apache.org/maven2/com/google/guava/listenablefuture/9999.0-empty-to-avoid-conflict-with-guava/listenablefuture-9999.0-empty-to-avoid-conflict-with-guava.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/com/google/inject/guice/4.2.1/guice-4.2.1-no_aop.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/com/google/guava/listenablefuture/9999.0-empty-to-avoid-conflict-with-guava/listenablefuture-9999.0-empty-to-avoid-conflict-with-guava.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/com/google/guava/failureaccess/1.0.1/failureaccess-1.0.1.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/com/google/guava/guava/27.0.1-jre/guava-27.0.1-jre.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/commons-cli/commons-cli/1.4/commons-cli-1.4.jar
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-settings-builder/3.6.3/maven-settings-builder-3.6.3.jar
  Downloading from central: https://repo.maven.apache.org/maven2/org/eclipse/sisu/org.eclipse.sisu.plexus/0.3.4/org.eclipse.sisu.plexus-0.3.4.jar
  Downloading from central: https://repo.maven.apache.org/maven2/org/codehaus/plexus/plexus-utils/3.2.1/plexus-utils-3.2.1.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/commons-io/commons-io/2.6/commons-io-2.6.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/codehaus/plexus/plexus-interpolation/1.25/plexus-interpolation-1.25.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/commons-cli/commons-cli/1.4/commons-cli-1.4.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/sonatype/plexus/plexus-sec-dispatcher/1.4/plexus-sec-dispatcher-1.4.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/codehaus/plexus/plexus-interpolation/1.25/plexus-interpolation-1.25.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/sonatype/plexus/plexus-cipher/1.4/plexus-cipher-1.4.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/codehaus/plexus/plexus-utils/3.2.1/plexus-utils-3.2.1.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-resolver-provider/3.6.3/maven-resolver-provider-3.6.3.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-settings-builder/3.6.3/maven-settings-builder-3.6.3.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/org/eclipse/sisu/org.eclipse.sisu.plexus/0.3.4/org.eclipse.sisu.plexus-0.3.4.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-repository-metadata/3.6.3/maven-repository-metadata-3.6.3.jar
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/resolver/maven-resolver-spi/1.4.1/maven-resolver-spi-1.4.1.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-resolver-provider/3.6.3/maven-resolver-provider-3.6.3.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/resolver/maven-resolver-spi/1.4.1/maven-resolver-spi-1.4.1.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/org/sonatype/plexus/plexus-cipher/1.4/plexus-cipher-1.4.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-repository-metadata/3.6.3/maven-repository-metadata-3.6.3.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/org/sonatype/plexus/plexus-sec-dispatcher/1.4/plexus-sec-dispatcher-1.4.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/wagon/wagon-http/3.3.4/wagon-http-3.3.4.jar
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/resolver/maven-resolver-transport-wagon/1.4.1/maven-resolver-transport-wagon-1.4.1.jar
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/resolver/maven-resolver-connector-basic/1.4.1/maven-resolver-connector-basic-1.4.1.jar
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/resolver/maven-resolver-impl/1.4.1/maven-resolver-impl-1.4.1.jar
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/wagon/wagon-http-shared/3.3.4/wagon-http-shared-3.3.4.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/wagon/wagon-http-shared/3.3.4/wagon-http-shared-3.3.4.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/resolver/maven-resolver-transport-wagon/1.4.1/maven-resolver-transport-wagon-1.4.1.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/resolver/maven-resolver-impl/1.4.1/maven-resolver-impl-1.4.1.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/resolver/maven-resolver-connector-basic/1.4.1/maven-resolver-connector-basic-1.4.1.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/wagon/wagon-http/3.3.4/wagon-http-3.3.4.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/logging/jboss-logging/3.3.2.Final/jboss-logging-3.3.2.Final.jar
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/wagon/wagon-file/3.3.4/wagon-file-3.3.4.jar
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/wagon/wagon-provider-api/3.3.4/wagon-provider-api-3.3.4.jar
  Downloading from central: https://repo.maven.apache.org/maven2/org/jsoup/jsoup/1.12.1/jsoup-1.12.1.jar
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/logging/commons-logging-jboss-logging/1.0.0.Final/commons-logging-jboss-logging-1.0.0.Final.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/logging/jboss-logging/3.3.2.Final/jboss-logging-3.3.2.Final.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-test-common/1.4.1.Final/quarkus-test-common-1.4.1.Final.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jsoup/jsoup/1.12.1/jsoup-1.12.1.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/wagon/wagon-file/3.3.4/wagon-file-3.3.4.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/wagon/wagon-provider-api/3.3.4/wagon-provider-api-3.3.4.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/gizmo/gizmo/1.0.2.Final/gizmo-1.0.2.Final.jar
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-core-deployment/1.4.1.Final/quarkus-core-deployment-1.4.1.Final.jar
  Downloading from central: https://repo.maven.apache.org/maven2/org/ow2/asm/asm-util/7.3.1/asm-util-7.3.1.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/logging/commons-logging-jboss-logging/1.0.0.Final/commons-logging-jboss-logging-1.0.0.Final.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/ow2/asm/asm-tree/7.3.1/asm-tree-7.3.1.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/gizmo/gizmo/1.0.2.Final/gizmo-1.0.2.Final.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/ow2/asm/asm-analysis/7.3.1/asm-analysis-7.3.1.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-test-common/1.4.1.Final/quarkus-test-common-1.4.1.Final.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-builder/1.4.1.Final/quarkus-builder-1.4.1.Final.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-core-deployment/1.4.1.Final/quarkus-core-deployment-1.4.1.Final.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/org/ow2/asm/asm-util/7.3.1/asm-util-7.3.1.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-jsonp-deployment/1.4.1.Final/quarkus-jsonp-deployment-1.4.1.Final.jar
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-jsonp/1.4.1.Final/quarkus-jsonp-1.4.1.Final.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/ow2/asm/asm-tree/7.3.1/asm-tree-7.3.1.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/glassfish/jakarta.json/1.1.6/jakarta.json-1.1.6.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/ow2/asm/asm-analysis/7.3.1/asm-analysis-7.3.1.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/jandex/2.1.3.Final/jandex-2.1.3.Final.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-builder/1.4.1.Final/quarkus-builder-1.4.1.Final.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/org/glassfish/jakarta.json/1.1.6/jakarta.json-1.1.6.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/junit/jupiter/junit-jupiter-api/5.6.2/junit-jupiter-api-5.6.2.jar
  Downloading from central: https://repo.maven.apache.org/maven2/org/junit/jupiter/junit-jupiter/5.6.2/junit-jupiter-5.6.2.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-jsonp-deployment/1.4.1.Final/quarkus-jsonp-deployment-1.4.1.Final.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-jsonp/1.4.1.Final/quarkus-jsonp-1.4.1.Final.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apiguardian/apiguardian-api/1.1.0/apiguardian-api-1.1.0.jar
  Downloading from central: https://repo.maven.apache.org/maven2/org/opentest4j/opentest4j/1.2.0/opentest4j-1.2.0.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/jandex/2.1.3.Final/jandex-2.1.3.Final.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/junit/platform/junit-platform-commons/1.6.2/junit-platform-commons-1.6.2.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/junit/jupiter/junit-jupiter/5.6.2/junit-jupiter-5.6.2.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/junit/jupiter/junit-jupiter-params/5.6.2/junit-jupiter-params-5.6.2.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apiguardian/apiguardian-api/1.1.0/apiguardian-api-1.1.0.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/junit/jupiter/junit-jupiter-engine/5.6.2/junit-jupiter-engine-5.6.2.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/junit/jupiter/junit-jupiter-api/5.6.2/junit-jupiter-api-5.6.2.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/junit/platform/junit-platform-engine/1.6.2/junit-platform-engine-1.6.2.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/opentest4j/opentest4j/1.2.0/opentest4j-1.2.0.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-core/1.4.1.Final/quarkus-core-1.4.1.Final.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/junit/platform/junit-platform-commons/1.6.2/junit-platform-commons-1.6.2.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/jakarta/annotation/jakarta.annotation-api/1.3.5/jakarta.annotation-api-1.3.5.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/junit/jupiter/junit-jupiter-engine/5.6.2/junit-jupiter-engine-5.6.2.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/jakarta/inject/jakarta.inject-api/1.0/jakarta.inject-api-1.0.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/junit/jupiter/junit-jupiter-params/5.6.2/junit-jupiter-params-5.6.2.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-ide-launcher/1.4.1.Final/quarkus-ide-launcher-1.4.1.Final.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-core/1.4.1.Final/quarkus-core-1.4.1.Final.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/org/junit/platform/junit-platform-engine/1.6.2/junit-platform-engine-1.6.2.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/smallrye/config/smallrye-config/1.7.0/smallrye-config-1.7.0.jar
  Downloading from central: https://repo.maven.apache.org/maven2/io/smallrye/config/smallrye-config-common/1.7.0/smallrye-config-common-1.7.0.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/jakarta/annotation/jakarta.annotation-api/1.3.5/jakarta.annotation-api-1.3.5.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/logmanager/jboss-logmanager-embedded/1.0.4/jboss-logmanager-embedded-1.0.4.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/jakarta/inject/jakarta.inject-api/1.0/jakarta.inject-api-1.0.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/logging/jboss-logging-annotations/2.1.0.Final/jboss-logging-annotations-2.1.0.Final.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/logmanager/jboss-logmanager-embedded/1.0.4/jboss-logmanager-embedded-1.0.4.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/threads/jboss-threads/3.1.1.Final/jboss-threads-3.1.1.Final.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/io/smallrye/config/smallrye-config/1.7.0/smallrye-config-1.7.0.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/io/smallrye/config/smallrye-config-common/1.7.0/smallrye-config-common-1.7.0.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/slf4j/slf4j-api/1.7.30/slf4j-api-1.7.30.jar
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/slf4j/slf4j-jboss-logging/1.2.0.Final/slf4j-jboss-logging-1.2.0.Final.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-ide-launcher/1.4.1.Final/quarkus-ide-launcher-1.4.1.Final.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/graalvm/sdk/graal-sdk/19.3.1/graal-sdk-19.3.1.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/logging/jboss-logging-annotations/2.1.0.Final/jboss-logging-annotations-2.1.0.Final.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/threads/jboss-threads/3.1.1.Final/jboss-threads-3.1.1.Final.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/wildfly/common/wildfly-common/1.5.4.Final-format-001/wildfly-common-1.5.4.Final-format-001.jar
  Downloading from central: https://repo.maven.apache.org/maven2/io/rest-assured/rest-assured/4.3.0/rest-assured-4.3.0.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/slf4j/slf4j-api/1.7.30/slf4j-api-1.7.30.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/codehaus/groovy/groovy/3.0.2/groovy-3.0.2.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/slf4j/slf4j-jboss-logging/1.2.0.Final/slf4j-jboss-logging-1.2.0.Final.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/codehaus/groovy/groovy-xml/3.0.2/groovy-xml-3.0.2.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/io/rest-assured/rest-assured/4.3.0/rest-assured-4.3.0.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/org/graalvm/sdk/graal-sdk/19.3.1/graal-sdk-19.3.1.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/httpcomponents/httpclient/4.5.12/httpclient-4.5.12.jar
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/httpcomponents/httpcore/4.4.13/httpcore-4.4.13.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/wildfly/common/wildfly-common/1.5.4.Final-format-001/wildfly-common-1.5.4.Final-format-001.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/commons-codec/commons-codec/1.13/commons-codec-1.13.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/codehaus/groovy/groovy/3.0.2/groovy-3.0.2.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/httpcomponents/httpmime/4.5.3/httpmime-4.5.3.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/codehaus/groovy/groovy-xml/3.0.2/groovy-xml-3.0.2.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/hamcrest/hamcrest/2.1/hamcrest-2.1.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/commons-codec/commons-codec/1.13/commons-codec-1.13.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/httpcomponents/httpcore/4.4.13/httpcore-4.4.13.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/ccil/cowan/tagsoup/tagsoup/1.2.1/tagsoup-1.2.1.jar
  Downloading from central: https://repo.maven.apache.org/maven2/io/rest-assured/json-path/4.3.0/json-path-4.3.0.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/httpcomponents/httpclient/4.5.12/httpclient-4.5.12.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/codehaus/groovy/groovy-json/3.0.2/groovy-json-3.0.2.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/httpcomponents/httpmime/4.5.3/httpmime-4.5.3.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/rest-assured/rest-assured-common/4.3.0/rest-assured-common-4.3.0.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/hamcrest/hamcrest/2.1/hamcrest-2.1.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/rest-assured/xml-path/4.3.0/xml-path-4.3.0.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/io/rest-assured/json-path/4.3.0/json-path-4.3.0.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/org/ccil/cowan/tagsoup/tagsoup/1.2.1/tagsoup-1.2.1.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/commons/commons-lang3/3.9/commons-lang3-3.9.jar
  Downloading from central: https://repo.maven.apache.org/maven2/jakarta/xml/bind/jakarta.xml.bind-api/2.3.2/jakarta.xml.bind-api-2.3.2.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/codehaus/groovy/groovy-json/3.0.2/groovy-json-3.0.2.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/jakarta/activation/jakarta.activation-api/1.2.1/jakarta.activation-api-1.2.1.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/io/rest-assured/rest-assured-common/4.3.0/rest-assured-common-4.3.0.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/sling/org.apache.sling.javax.activation/0.1.0/org.apache.sling.javax.activation-0.1.0.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/jakarta/xml/bind/jakarta.xml.bind-api/2.3.2/jakarta.xml.bind-api-2.3.2.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/commons/commons-lang3/3.9/commons-lang3-3.9.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/jakarta/activation/jakarta.activation-api/1.2.1/jakarta.activation-api-1.2.1.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/io/rest-assured/xml-path/4.3.0/xml-path-4.3.0.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/sling/org.apache.sling.javax.activation/0.1.0/org.apache.sling.javax.activation-0.1.0.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy-deployment/1.4.1.Final/quarkus-resteasy-deployment-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy-deployment/1.4.1.Final/quarkus-resteasy-deployment-1.4.1.Final.pom (3.8 kB at 84 kB/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy-server-common-deployment/1.4.1.Final/quarkus-resteasy-server-common-deployment-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy-server-common-deployment/1.4.1.Final/quarkus-resteasy-server-common-deployment-1.4.1.Final.pom (2.2 kB at 41 kB/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jsoup/jsoup/1.11.3/jsoup-1.11.3.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jsoup/jsoup/1.11.3/jsoup-1.11.3.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/wagon/wagon-provider-api/3.3.3/wagon-provider-api-3.3.3.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/wagon/wagon-provider-api/3.3.3/wagon-provider-api-3.3.3.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/wagon/wagon/3.3.3/wagon-3.3.3.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/wagon/wagon/3.3.3/wagon-3.3.3.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy-common-deployment/1.4.1.Final/quarkus-resteasy-common-deployment-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy-common-deployment/1.4.1.Final/quarkus-resteasy-common-deployment-1.4.1.Final.pom (2.0 kB at 36 kB/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy-common-spi/1.4.1.Final/quarkus-resteasy-common-spi-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy-common-spi/1.4.1.Final/quarkus-resteasy-common-spi-1.4.1.Final.pom (817 B at 17 kB/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-arc-deployment/1.4.1.Final/quarkus-arc-deployment-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-arc-deployment/1.4.1.Final/quarkus-arc-deployment-1.4.1.Final.pom (1.6 kB at 38 kB/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/arc/arc-processor/1.4.1.Final/arc-processor-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/arc/arc-processor/1.4.1.Final/arc-processor-1.4.1.Final.pom (1.7 kB at 40 kB/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy-server-common-spi/1.4.1.Final/quarkus-resteasy-server-common-spi-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy-server-common-spi/1.4.1.Final/quarkus-resteasy-server-common-spi-1.4.1.Final.pom (889 B at 20 kB/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-undertow-spi/1.4.1.Final/quarkus-undertow-spi-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-undertow-spi/1.4.1.Final/quarkus-undertow-spi-1.4.1.Final.pom (2.0 kB at 32 kB/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-http-parent/1.4.1.Final/quarkus-http-parent-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-http-parent/1.4.1.Final/quarkus-http-parent-1.4.1.Final.pom (780 B at 17 kB/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/http/quarkus-http-servlet/3.0.6.Final/quarkus-http-servlet-3.0.6.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/http/quarkus-http-servlet/3.0.6.Final/quarkus-http-servlet-3.0.6.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/http/quarkus-http-parent/3.0.6.Final/quarkus-http-parent-3.0.6.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/http/quarkus-http-parent/3.0.6.Final/quarkus-http-parent-3.0.6.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/jboss-parent/28/jboss-parent-28.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/jboss-parent/28/jboss-parent-28.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/http/quarkus-http-core/3.0.6.Final/quarkus-http-core-3.0.6.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/http/quarkus-http-core/3.0.6.Final/quarkus-http-core-3.0.6.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/http/quarkus-http-http-core/3.0.6.Final/quarkus-http-http-core-3.0.6.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/http/quarkus-http-http-core/3.0.6.Final/quarkus-http-http-core-3.0.6.Final.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/jakarta/servlet/jakarta.servlet-api/4.0.3/jakarta.servlet-api-4.0.3.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/jakarta/servlet/jakarta.servlet-api/4.0.3/jakarta.servlet-api-4.0.3.pom (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/metadata/jboss-metadata-web/11.0.0.Final/jboss-metadata-web-11.0.0.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/metadata/jboss-metadata-web/11.0.0.Final/jboss-metadata-web-11.0.0.Final.pom (4.7 kB at 108 kB/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/metadata/jboss-as-parent-metadata/11.0.0.Final/jboss-as-parent-metadata-11.0.0.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/metadata/jboss-as-parent-metadata/11.0.0.Final/jboss-as-parent-metadata-11.0.0.Final.pom (17 kB at 173 kB/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/jboss-parent/16/jboss-parent-16.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/jboss-parent/16/jboss-parent-16.pom (32 kB at 358 kB/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/metadata/jboss-metadata-common/11.0.0.Final/jboss-metadata-common-11.0.0.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/metadata/jboss-metadata-common/11.0.0.Final/jboss-metadata-common-11.0.0.Final.pom (4.0 kB at 119 kB/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-vertx-http-deployment/1.4.1.Final/quarkus-vertx-http-deployment-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-vertx-http-deployment/1.4.1.Final/quarkus-vertx-http-deployment-1.4.1.Final.pom (3.0 kB at 35 kB/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-vertx-core-deployment/1.4.1.Final/quarkus-vertx-core-deployment-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-vertx-core-deployment/1.4.1.Final/quarkus-vertx-core-deployment-1.4.1.Final.pom (2.2 kB at 50 kB/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-netty-deployment/1.4.1.Final/quarkus-netty-deployment-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-netty-deployment/1.4.1.Final/quarkus-netty-deployment-1.4.1.Final.pom (1.6 kB at 29 kB/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-kubernetes-spi/1.4.1.Final/quarkus-kubernetes-spi-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-kubernetes-spi/1.4.1.Final/quarkus-kubernetes-spi-1.4.1.Final.pom (862 B at 12 kB/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-kubernetes-parent/1.4.1.Final/quarkus-kubernetes-parent-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-kubernetes-parent/1.4.1.Final/quarkus-kubernetes-parent-1.4.1.Final.pom (786 B at 16 kB/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-security-spi/1.4.1.Final/quarkus-security-spi-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-security-spi/1.4.1.Final/quarkus-security-spi-1.4.1.Final.pom (1.4 kB at 13 kB/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-security-parent/1.4.1.Final/quarkus-security-parent-1.4.1.Final.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-security-parent/1.4.1.Final/quarkus-security-parent-1.4.1.Final.pom (819 B at 6.3 kB/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/graalvm/nativeimage/svm/19.3.1/svm-19.3.1.pom
  Downloaded from central: https://repo.maven.apache.org/maven2/org/graalvm/nativeimage/svm/19.3.1/svm-19.3.1.pom (2.7 kB at 31 kB/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy-common-spi/1.4.1.Final/quarkus-resteasy-common-spi-1.4.1.Final.jar
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy-common-deployment/1.4.1.Final/quarkus-resteasy-common-deployment-1.4.1.Final.jar
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy-server-common-spi/1.4.1.Final/quarkus-resteasy-server-common-spi-1.4.1.Final.jar
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy-server-common-deployment/1.4.1.Final/quarkus-resteasy-server-common-deployment-1.4.1.Final.jar
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/http/quarkus-http-http-core/3.0.6.Final/quarkus-http-http-core-3.0.6.Final.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/http/quarkus-http-http-core/3.0.6.Final/quarkus-http-http-core-3.0.6.Final.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/http/quarkus-http-core/3.0.6.Final/quarkus-http-core-3.0.6.Final.jar
  Progress (4): quarkus-resteasy-common-deployment-1.4.1.Final.jar (8.2/26 kB) | quarkus-resteasy-server-common-deployment-1.4.1.Final.jar (11/29 kB) | quarkus-resteasy-server-common-spi-1.4.1.Final.jar (2.7/5.2 kB) | quarkus-resteasy-common-spProgress (4): quarkus-resteasy-common-deployment-1.4.1.Final.jar (8.2/26 kB) | quarkus-resteasy-server-common-deployment-1.4.1.Final.jar (11/29 kB) | quarkus-resteasy-server-common-spi-1.4.1.Final.jar (5.2 kB) | quarkus-resteasy-common-spi-1.Progress (4): quarkus-resteasy-common-deployment-1.4.1.Final.jar (8.2/26 kB) | quarkus-resteasy-server-common-deployment-1.4.1.Final.jar (11/29 kB) | quarkus-resteasy-server-common-spi-1.4.1.Final.jar (5.2 kB) | quarkus-resteasy-common-spi-1.Progress (4): quarkus-resteasy-common-deployment-1.4.1.Final.jar (11/26 kB) | quarkus-resteasy-server-common-deployment-1.4.1.Final.jar (11/29 kB) | quarkus-resteasy-server-common-spi-1.4.1.Final.jar (5.2 kB) | quarkus-resteasy-common-spi-1.4Progress (4): quarkus-resteasy-common-deployment-1.4.1.Final.jar (14/26 kB) | quarkus-resteasy-server-common-deployment-1.4.1.Final.jar (11/29 kB) | quarkus-resteasy-server-common-spi-1.4.1.Final.jar (5.2 kB) | quarkus-resteasy-common-spi-1.4Progress (4): quarkus-resteasy-common-deployment-1.4.1.Final.jar (16/26 kB) | quarkus-resteasy-server-common-deployment-1.4.1.Final.jar (11/29 kB) | quarkus-resteasy-server-common-spi-1.4.1.Final.jar (5.2 kB) | quarkus-resteasy-common-spi-1.4Progress (4): quarkus-resteasy-common-deployment-1.4.1.Final.jar (16/26 kB) | quarkus-resteasy-server-common-deployment-1.4.1.Final.jar (14/29 kB) | quarkus-resteasy-server-common-spi-1.4.1.Final.jar (5.2 kB) | quarkus-resteasy-common-spi-1.4Progress (4): quarkus-resteasy-common-deployment-1.4.1.Final.jar (16/26 kB) | quarkus-resteasy-server-common-deployment-1.4.1.Final.jar (16/29 kB) | quarkus-resteasy-server-common-spi-1.4.1.Final.jar (5.2 kB) | quarkus-resteasy-common-spi-1.4Progress (4): quarkus-resteasy-common-deployment-1.4.1.Final.jar (19/26 kB) | quarkus-resteasy-server-common-deployment-1.4.1.Final.jar (16/29 kB) | quarkus-resteasy-server-common-spi-1.4.1.Final.jar (5.2 kB) | quarkus-resteasy-common-spi-1.4Progress (4): quarkus-resteasy-common-deployment-1.4.1.Final.jar (19/26 kB) | quarkus-resteasy-server-common-deployment-1.4.1.Final.jar (19/29 kB) | quarkus-resteasy-server-common-spi-1.4.1.Final.jar (5.2 kB) | quarkus-resteasy-common-spi-1.4Progress (4): quarkus-resteasy-common-deployment-1.4.1.Final.jar (22/26 kB) | quarkus-resteasy-server-common-deployment-1.4.1.Final.jar (19/29 kB) | quarkus-resteasy-server-common-spi-1.4.1.Final.jar (5.2 kB) | quarkus-resteasy-common-spi-1.4Progress (4): quarkus-resteasy-common-deployment-1.4.1.Final.jar (22/26 kB) | quarkus-resteasy-server-common-deployment-1.4.1.Final.jar (22/29 kB) | quarkus-resteasy-server-common-spi-1.4.1.Final.jar (5.2 kB) | quarkus-resteasy-common-spi-1.4Progress (4): quarkus-resteasy-common-deployment-1.4.1.Final.jar (25/26 kB) | quarkus-resteasy-server-common-deployment-1.4.1.Final.jar (22/29 kB) | quarkus-resteasy-server-common-spi-1.4.1.Final.jar (5.2 kB) | quarkus-resteasy-common-spi-1.4Progress (4): quarkus-resteasy-common-deployment-1.4.1.Final.jar (26 kB) | quarkus-resteasy-server-common-deployment-1.4.1.Final.jar (22/29 kB) | quarkus-resteasy-server-common-spi-1.4.1.Final.jar (5.2 kB) | quarkus-resteasy-common-spi-1.4.1.Progress (4): quarkus-resteasy-common-deployment-1.4.1.Final.jar (26 kB) | quarkus-resteasy-server-common-deployment-1.4.1.Final.jar (25/29 kB) | quarkus-resteasy-server-common-spi-1.4.1.Final.jar (5.2 kB) | quarkus-resteasy-common-spi-1.4.1.Progress (4): quarkus-resteasy-common-deployment-1.4.1.Final.jar (26 kB) | quarkus-resteasy-server-common-deployment-1.4.1.Final.jar (27/29 kB) | quarkus-resteasy-server-common-spi-1.4.1.Final.jar (5.2 kB) | quarkus-resteasy-common-spi-1.4.1.Progress (4): quarkus-resteasy-common-deployment-1.4.1.Final.jar (26 kB) | quarkus-resteasy-server-common-deployment-1.4.1.Final.jar (29 kB) | quarkus-resteasy-server-common-spi-1.4.1.Final.jar (5.2 kB) | quarkus-resteasy-common-spi-1.4.1.Fin                                                                                                                                                                                                                                                  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy-server-common-spi/1.4.1.Final/quarkus-resteasy-server-common-spi-1.4.1.Final.jar (5.2 kB at 92 kB/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy-common-spi/1.4.1.Final/quarkus-resteasy-common-spi-1.4.1.Final.jar (3.4 kB at 60 kB/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/http/quarkus-http-servlet/3.0.6.Final/quarkus-http-servlet-3.0.6.Final.jar
  Downloading from central: https://repo.maven.apache.org/maven2/jakarta/servlet/jakarta.servlet-api/4.0.3/jakarta.servlet-api-4.0.3.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/http/quarkus-http-core/3.0.6.Final/quarkus-http-core-3.0.6.Final.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/metadata/jboss-metadata-common/11.0.0.Final/jboss-metadata-common-11.0.0.Final.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy-common-deployment/1.4.1.Final/quarkus-resteasy-common-deployment-1.4.1.Final.jar (26 kB at 265 kB/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/jboss/metadata/jboss-metadata-web/11.0.0.Final/jboss-metadata-web-11.0.0.Final.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/jakarta/servlet/jakarta.servlet-api/4.0.3/jakarta.servlet-api-4.0.3.jar (0 B at 0 B/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/http/quarkus-http-servlet/3.0.6.Final/quarkus-http-servlet-3.0.6.Final.jar (0 B at 0 B/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-undertow-spi/1.4.1.Final/quarkus-undertow-spi-1.4.1.Final.jar
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-netty-deployment/1.4.1.Final/quarkus-netty-deployment-1.4.1.Final.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy-server-common-deployment/1.4.1.Final/quarkus-resteasy-server-common-deployment-1.4.1.Final.jar (29 kB at 277 kB/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-vertx-core-deployment/1.4.1.Final/quarkus-vertx-core-deployment-1.4.1.Final.jar
  Progress (5): jboss-metadata-common-11.0.0.Final.jar (66/475 kB) | jboss-metadata-web-11.0.0.Final.jar (52/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (2.7/10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (2.7/8.1 kB) | quarkus-uProgress (5): jboss-metadata-common-11.0.0.Final.jar (69/475 kB) | jboss-metadata-web-11.0.0.Final.jar (52/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (2.7/10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (2.7/8.1 kB) | quarkus-uProgress (5): jboss-metadata-common-11.0.0.Final.jar (69/475 kB) | jboss-metadata-web-11.0.0.Final.jar (52/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (5.5/10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (2.7/8.1 kB) | quarkus-uProgress (5): jboss-metadata-common-11.0.0.Final.jar (69/475 kB) | jboss-metadata-web-11.0.0.Final.jar (52/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (5.5/10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (5.5/8.1 kB) | quarkus-uProgress (5): jboss-metadata-common-11.0.0.Final.jar (69/475 kB) | jboss-metadata-web-11.0.0.Final.jar (52/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (5.5/10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (5.5/8.1 kB) | quarkus-uProgress (5): jboss-metadata-common-11.0.0.Final.jar (71/475 kB) | jboss-metadata-web-11.0.0.Final.jar (52/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (5.5/10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (5.5/8.1 kB) | quarkus-uProgress (5): jboss-metadata-common-11.0.0.Final.jar (71/475 kB) | jboss-metadata-web-11.0.0.Final.jar (52/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (8.2/10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (5.5/8.1 kB) | quarkus-uProgress (5): jboss-metadata-common-11.0.0.Final.jar (71/475 kB) | jboss-metadata-web-11.0.0.Final.jar (52/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (8.2/10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-underProgress (5): jboss-metadata-common-11.0.0.Final.jar (71/475 kB) | jboss-metadata-web-11.0.0.Final.jar (52/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (8.2/10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-underProgress (5): jboss-metadata-common-11.0.0.Final.jar (74/475 kB) | jboss-metadata-web-11.0.0.Final.jar (52/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (8.2/10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-underProgress (5): jboss-metadata-common-11.0.0.Final.jar (74/475 kB) | jboss-metadata-web-11.0.0.Final.jar (52/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertow-Progress (5): jboss-metadata-common-11.0.0.Final.jar (74/475 kB) | jboss-metadata-web-11.0.0.Final.jar (55/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertow-Progress (5): jboss-metadata-common-11.0.0.Final.jar (74/475 kB) | jboss-metadata-web-11.0.0.Final.jar (55/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertow-Progress (5): jboss-metadata-common-11.0.0.Final.jar (77/475 kB) | jboss-metadata-web-11.0.0.Final.jar (55/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertow-Progress (5): jboss-metadata-common-11.0.0.Final.jar (77/475 kB) | jboss-metadata-web-11.0.0.Final.jar (58/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertow-Progress (5): jboss-metadata-common-11.0.0.Final.jar (77/475 kB) | jboss-metadata-web-11.0.0.Final.jar (58/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertow-Progress (5): jboss-metadata-common-11.0.0.Final.jar (77/475 kB) | jboss-metadata-web-11.0.0.Final.jar (60/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertow-Progress (5): jboss-metadata-common-11.0.0.Final.jar (80/475 kB) | jboss-metadata-web-11.0.0.Final.jar (60/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertow-Progress (5): jboss-metadata-common-11.0.0.Final.jar (80/475 kB) | jboss-metadata-web-11.0.0.Final.jar (60/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertow-Progress (5): jboss-metadata-common-11.0.0.Final.jar (82/475 kB) | jboss-metadata-web-11.0.0.Final.jar (60/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertow-Progress (5): jboss-metadata-common-11.0.0.Final.jar (82/475 kB) | jboss-metadata-web-11.0.0.Final.jar (63/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertow-Progress (5): jboss-metadata-common-11.0.0.Final.jar (82/475 kB) | jboss-metadata-web-11.0.0.Final.jar (63/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertow-Progress (5): jboss-metadata-common-11.0.0.Final.jar (85/475 kB) | jboss-metadata-web-11.0.0.Final.jar (63/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertow-Progress (5): jboss-metadata-common-11.0.0.Final.jar (85/475 kB) | jboss-metadata-web-11.0.0.Final.jar (66/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertow-Progress (5): jboss-metadata-common-11.0.0.Final.jar (85/475 kB) | jboss-metadata-web-11.0.0.Final.jar (66/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertow-Progress (5): jboss-metadata-common-11.0.0.Final.jar (88/475 kB) | jboss-metadata-web-11.0.0.Final.jar (66/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertow-Progress (5): jboss-metadata-common-11.0.0.Final.jar (88/475 kB) | jboss-metadata-web-11.0.0.Final.jar (69/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertow-Progress (5): jboss-metadata-common-11.0.0.Final.jar (88/475 kB) | jboss-metadata-web-11.0.0.Final.jar (69/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertow-Progress (5): jboss-metadata-common-11.0.0.Final.jar (90/475 kB) | jboss-metadata-web-11.0.0.Final.jar (69/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertow-Progress (5): jboss-metadata-common-11.0.0.Final.jar (90/475 kB) | jboss-metadata-web-11.0.0.Final.jar (71/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertow-Progress (5): jboss-metadata-common-11.0.0.Final.jar (93/475 kB) | jboss-metadata-web-11.0.0.Final.jar (71/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertow-Progress (5): jboss-metadata-common-11.0.0.Final.jar (93/475 kB) | jboss-metadata-web-11.0.0.Final.jar (74/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertow-Progress (5): jboss-metadata-common-11.0.0.Final.jar (96/475 kB) | jboss-metadata-web-11.0.0.Final.jar (74/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertow-Progress (5): jboss-metadata-common-11.0.0.Final.jar (96/475 kB) | jboss-metadata-web-11.0.0.Final.jar (77/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertow-Progress (5): jboss-metadata-common-11.0.0.Final.jar (99/475 kB) | jboss-metadata-web-11.0.0.Final.jar (77/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertow-Progress (5): jboss-metadata-common-11.0.0.Final.jar (99/475 kB) | jboss-metadata-web-11.0.0.Final.jar (80/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertow-Progress (5): jboss-metadata-common-11.0.0.Final.jar (101/475 kB) | jboss-metadata-web-11.0.0.Final.jar (80/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertowProgress (5): jboss-metadata-common-11.0.0.Final.jar (101/475 kB) | jboss-metadata-web-11.0.0.Final.jar (82/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertowProgress (5): jboss-metadata-common-11.0.0.Final.jar (104/475 kB) | jboss-metadata-web-11.0.0.Final.jar (82/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertowProgress (5): jboss-metadata-common-11.0.0.Final.jar (104/475 kB) | jboss-metadata-web-11.0.0.Final.jar (85/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertowProgress (5): jboss-metadata-common-11.0.0.Final.jar (107/475 kB) | jboss-metadata-web-11.0.0.Final.jar (85/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertowProgress (5): jboss-metadata-common-11.0.0.Final.jar (107/475 kB) | jboss-metadata-web-11.0.0.Final.jar (88/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertowProgress (5): jboss-metadata-common-11.0.0.Final.jar (110/475 kB) | jboss-metadata-web-11.0.0.Final.jar (88/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertowProgress (5): jboss-metadata-common-11.0.0.Final.jar (110/475 kB) | jboss-metadata-web-11.0.0.Final.jar (90/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertowProgress (5): jboss-metadata-common-11.0.0.Final.jar (112/475 kB) | jboss-metadata-web-11.0.0.Final.jar (90/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertowProgress (5): jboss-metadata-common-11.0.0.Final.jar (112/475 kB) | jboss-metadata-web-11.0.0.Final.jar (93/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertowProgress (5): jboss-metadata-common-11.0.0.Final.jar (115/475 kB) | jboss-metadata-web-11.0.0.Final.jar (93/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertowProgress (5): jboss-metadata-common-11.0.0.Final.jar (118/475 kB) | jboss-metadata-web-11.0.0.Final.jar (93/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertowProgress (5): jboss-metadata-common-11.0.0.Final.jar (121/475 kB) | jboss-metadata-web-11.0.0.Final.jar (93/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertowProgress (5): jboss-metadata-common-11.0.0.Final.jar (123/475 kB) | jboss-metadata-web-11.0.0.Final.jar (93/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertowProgress (5): jboss-metadata-common-11.0.0.Final.jar (126/475 kB) | jboss-metadata-web-11.0.0.Final.jar (93/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertowProgress (5): jboss-metadata-common-11.0.0.Final.jar (129/475 kB) | jboss-metadata-web-11.0.0.Final.jar (93/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertowProgress (5): jboss-metadata-common-11.0.0.Final.jar (132/475 kB) | jboss-metadata-web-11.0.0.Final.jar (93/468 kB) | quarkus-netty-deployment-1.4.1.Final.jar (10 kB) | quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB) | quarkus-undertow                                                                                                                                                                                                                                                  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-netty-deployment/1.4.1.Final/quarkus-netty-deployment-1.4.1.Final.jar (10 kB at 36 kB/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-kubernetes-spi/1.4.1.Final/quarkus-kubernetes-spi-1.4.1.Final.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-vertx-core-deployment/1.4.1.Final/quarkus-vertx-core-deployment-1.4.1.Final.jar (8.1 kB at 28 kB/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-vertx-http-deployment/1.4.1.Final/quarkus-vertx-http-deployment-1.4.1.Final.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-undertow-spi/1.4.1.Final/quarkus-undertow-spi-1.4.1.Final.jar (24 kB at 74 kB/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/arc/arc-processor/1.4.1.Final/arc-processor-1.4.1.Final.jar
  Progress (5): jboss-metadata-common-11.0.0.Final.jar (225/475 kB) | jboss-metadata-web-11.0.0.Final.jar (175/468 kB) | quarkus-vertx-http-deployment-1.4.1.Final.jar (22 kB) | quarkus-kubernetes-spi-1.4.1.Final.jar (8.1 kB) | arc-processor-1.4Progress (5): jboss-metadata-common-11.0.0.Final.jar (228/475 kB) | jboss-metadata-web-11.0.0.Final.jar (175/468 kB) | quarkus-vertx-http-deployment-1.4.1.Final.jar (22 kB) | quarkus-kubernetes-spi-1.4.1.Final.jar (8.1 kB) | arc-processor-1.4Progress (5): jboss-metadata-common-11.0.0.Final.jar (228/475 kB) | jboss-metadata-web-11.0.0.Final.jar (178/468 kB) | quarkus-vertx-http-deployment-1.4.1.Final.jar (22 kB) | quarkus-kubernetes-spi-1.4.1.Final.jar (8.1 kB) | arc-processor-1.4Progress (5): jboss-metadata-common-11.0.0.Final.jar (228/475 kB) | jboss-metadata-web-11.0.0.Final.jar (178/468 kB) | quarkus-vertx-http-deployment-1.4.1.Final.jar (22 kB) | quarkus-kubernetes-spi-1.4.1.Final.jar (8.1 kB) | arc-processor-1.4Progress (5): jboss-metadata-common-11.0.0.Final.jar (230/475 kB) | jboss-metadata-web-11.0.0.Final.jar (178/468 kB) | quarkus-vertx-http-deployment-1.4.1.Final.jar (22 kB) | quarkus-kubernetes-spi-1.4.1.Final.jar (8.1 kB) | arc-processor-1.4Progress (5): jboss-metadata-common-11.0.0.Final.jar (230/475 kB) | jboss-metadata-web-11.0.0.Final.jar (181/468 kB) | quarkus-vertx-http-deployment-1.4.1.Final.jar (22 kB) | quarkus-kubernetes-spi-1.4.1.Final.jar (8.1 kB) | arc-processor-1.4Progress (5): jboss-metadata-common-11.0.0.Final.jar (230/475 kB) | jboss-metadata-web-11.0.0.Final.jar (181/468 kB) | quarkus-vertx-http-deployment-1.4.1.Final.jar (22 kB) | quarkus-kubernetes-spi-1.4.1.Final.jar (8.1 kB) | arc-processor-1.4                                                                                                                                                                                                                                                  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-kubernetes-spi/1.4.1.Final/quarkus-kubernetes-spi-1.4.1.Final.jar (8.1 kB at 18 kB/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-arc-deployment/1.4.1.Final/quarkus-arc-deployment-1.4.1.Final.jar
  Progress (5): jboss-metadata-common-11.0.0.Final.jar (266/475 kB) | jboss-metadata-web-11.0.0.Final.jar (211/468 kB) | quarkus-vertx-http-deployment-1.4.1.Final.jar (22 kB) | arc-processor-1.4.1.Final.jar (44/310 kB) | quarkus-arc-deployment-Progress (5): jboss-metadata-common-11.0.0.Final.jar (266/475 kB) | jboss-metadata-web-11.0.0.Final.jar (211/468 kB) | quarkus-vertx-http-deployment-1.4.1.Final.jar (22 kB) | arc-processor-1.4.1.Final.jar (44/310 kB) | quarkus-arc-deployment-Progress (5): jboss-metadata-common-11.0.0.Final.jar (269/475 kB) | jboss-metadata-web-11.0.0.Final.jar (211/468 kB) | quarkus-vertx-http-deployment-1.4.1.Final.jar (22 kB) | arc-processor-1.4.1.Final.jar (44/310 kB) | quarkus-arc-deployment-Progress (5): jboss-metadata-common-11.0.0.Final.jar (269/475 kB) | jboss-metadata-web-11.0.0.Final.jar (214/468 kB) | quarkus-vertx-http-deployment-1.4.1.Final.jar (22 kB) | arc-processor-1.4.1.Final.jar (44/310 kB) | quarkus-arc-deployment-                                                                                                                                                                                                                                                  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-vertx-http-deployment/1.4.1.Final/quarkus-vertx-http-deployment-1.4.1.Final.jar (22 kB at 43 kB/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-security-spi/1.4.1.Final/quarkus-security-spi-1.4.1.Final.jar
  Progress (5): jboss-metadata-common-11.0.0.Final.jar (335/475 kB) | jboss-metadata-web-11.0.0.Final.jar (235/468 kB) | arc-processor-1.4.1.Final.jar (60/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (27/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (337/475 kB) | jboss-metadata-web-11.0.0.Final.jar (235/468 kB) | arc-processor-1.4.1.Final.jar (60/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (27/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (337/475 kB) | jboss-metadata-web-11.0.0.Final.jar (239/468 kB) | arc-processor-1.4.1.Final.jar (60/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (27/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (337/475 kB) | jboss-metadata-web-11.0.0.Final.jar (239/468 kB) | arc-processor-1.4.1.Final.jar (60/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (27/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (337/475 kB) | jboss-metadata-web-11.0.0.Final.jar (239/468 kB) | arc-processor-1.4.1.Final.jar (63/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (27/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (337/475 kB) | jboss-metadata-web-11.0.0.Final.jar (243/468 kB) | arc-processor-1.4.1.Final.jar (63/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (27/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (337/475 kB) | jboss-metadata-web-11.0.0.Final.jar (247/468 kB) | arc-processor-1.4.1.Final.jar (63/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (27/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (337/475 kB) | jboss-metadata-web-11.0.0.Final.jar (251/468 kB) | arc-processor-1.4.1.Final.jar (63/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (27/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (337/475 kB) | jboss-metadata-web-11.0.0.Final.jar (251/468 kB) | arc-processor-1.4.1.Final.jar (63/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (30/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (340/475 kB) | jboss-metadata-web-11.0.0.Final.jar (251/468 kB) | arc-processor-1.4.1.Final.jar (63/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (30/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (340/475 kB) | jboss-metadata-web-11.0.0.Final.jar (251/468 kB) | arc-processor-1.4.1.Final.jar (66/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (30/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (340/475 kB) | jboss-metadata-web-11.0.0.Final.jar (251/468 kB) | arc-processor-1.4.1.Final.jar (66/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (33/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (343/475 kB) | jboss-metadata-web-11.0.0.Final.jar (251/468 kB) | arc-processor-1.4.1.Final.jar (66/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (33/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (343/475 kB) | jboss-metadata-web-11.0.0.Final.jar (251/468 kB) | arc-processor-1.4.1.Final.jar (69/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (33/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (343/475 kB) | jboss-metadata-web-11.0.0.Final.jar (251/468 kB) | arc-processor-1.4.1.Final.jar (69/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (36/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (345/475 kB) | jboss-metadata-web-11.0.0.Final.jar (251/468 kB) | arc-processor-1.4.1.Final.jar (69/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (36/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (345/475 kB) | jboss-metadata-web-11.0.0.Final.jar (251/468 kB) | arc-processor-1.4.1.Final.jar (71/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (36/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (348/475 kB) | jboss-metadata-web-11.0.0.Final.jar (251/468 kB) | arc-processor-1.4.1.Final.jar (71/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (36/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (348/475 kB) | jboss-metadata-web-11.0.0.Final.jar (251/468 kB) | arc-processor-1.4.1.Final.jar (74/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (36/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (348/475 kB) | jboss-metadata-web-11.0.0.Final.jar (251/468 kB) | arc-processor-1.4.1.Final.jar (74/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (38/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (351/475 kB) | jboss-metadata-web-11.0.0.Final.jar (251/468 kB) | arc-processor-1.4.1.Final.jar (74/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (38/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (351/475 kB) | jboss-metadata-web-11.0.0.Final.jar (251/468 kB) | arc-processor-1.4.1.Final.jar (74/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (41/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (351/475 kB) | jboss-metadata-web-11.0.0.Final.jar (251/468 kB) | arc-processor-1.4.1.Final.jar (77/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (41/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (354/475 kB) | jboss-metadata-web-11.0.0.Final.jar (251/468 kB) | arc-processor-1.4.1.Final.jar (77/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (41/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (354/475 kB) | jboss-metadata-web-11.0.0.Final.jar (256/468 kB) | arc-processor-1.4.1.Final.jar (77/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (41/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (354/475 kB) | jboss-metadata-web-11.0.0.Final.jar (260/468 kB) | arc-processor-1.4.1.Final.jar (77/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (41/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (354/475 kB) | jboss-metadata-web-11.0.0.Final.jar (264/468 kB) | arc-processor-1.4.1.Final.jar (77/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (41/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (354/475 kB) | jboss-metadata-web-11.0.0.Final.jar (268/468 kB) | arc-processor-1.4.1.Final.jar (77/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (41/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (354/475 kB) | jboss-metadata-web-11.0.0.Final.jar (268/468 kB) | arc-processor-1.4.1.Final.jar (80/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (41/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (356/475 kB) | jboss-metadata-web-11.0.0.Final.jar (268/468 kB) | arc-processor-1.4.1.Final.jar (80/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (41/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (356/475 kB) | jboss-metadata-web-11.0.0.Final.jar (268/468 kB) | arc-processor-1.4.1.Final.jar (82/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (41/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (359/475 kB) | jboss-metadata-web-11.0.0.Final.jar (268/468 kB) | arc-processor-1.4.1.Final.jar (82/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (41/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (359/475 kB) | jboss-metadata-web-11.0.0.Final.jar (268/468 kB) | arc-processor-1.4.1.Final.jar (85/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (41/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (362/475 kB) | jboss-metadata-web-11.0.0.Final.jar (268/468 kB) | arc-processor-1.4.1.Final.jar (85/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (41/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (362/475 kB) | jboss-metadata-web-11.0.0.Final.jar (268/468 kB) | arc-processor-1.4.1.Final.jar (88/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (41/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (365/475 kB) | jboss-metadata-web-11.0.0.Final.jar (268/468 kB) | arc-processor-1.4.1.Final.jar (88/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (41/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (365/475 kB) | jboss-metadata-web-11.0.0.Final.jar (268/468 kB) | arc-processor-1.4.1.Final.jar (90/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (41/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (367/475 kB) | jboss-metadata-web-11.0.0.Final.jar (268/468 kB) | arc-processor-1.4.1.Final.jar (90/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (41/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (367/475 kB) | jboss-metadata-web-11.0.0.Final.jar (272/468 kB) | arc-processor-1.4.1.Final.jar (90/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (41/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (367/475 kB) | jboss-metadata-web-11.0.0.Final.jar (276/468 kB) | arc-processor-1.4.1.Final.jar (90/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (41/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (367/475 kB) | jboss-metadata-web-11.0.0.Final.jar (276/468 kB) | arc-processor-1.4.1.Final.jar (93/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (41/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (367/475 kB) | jboss-metadata-web-11.0.0.Final.jar (280/468 kB) | arc-processor-1.4.1.Final.jar (93/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (41/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (367/475 kB) | jboss-metadata-web-11.0.0.Final.jar (284/468 kB) | arc-processor-1.4.1.Final.jar (93/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (41/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (370/475 kB) | jboss-metadata-web-11.0.0.Final.jar (284/468 kB) | arc-processor-1.4.1.Final.jar (93/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (41/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (370/475 kB) | jboss-metadata-web-11.0.0.Final.jar (284/468 kB) | arc-processor-1.4.1.Final.jar (96/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (41/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (373/475 kB) | jboss-metadata-web-11.0.0.Final.jar (284/468 kB) | arc-processor-1.4.1.Final.jar (96/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (41/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (373/475 kB) | jboss-metadata-web-11.0.0.Final.jar (284/468 kB) | arc-processor-1.4.1.Final.jar (96/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (44/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (373/475 kB) | jboss-metadata-web-11.0.0.Final.jar (284/468 kB) | arc-processor-1.4.1.Final.jar (99/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (44/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (376/475 kB) | jboss-metadata-web-11.0.0.Final.jar (284/468 kB) | arc-processor-1.4.1.Final.jar (99/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (44/142 kB) | quarkus-security-spi-1.4.1Progress (5): jboss-metadata-common-11.0.0.Final.jar (376/475 kB) | jboss-metadata-web-11.0.0.Final.jar (284/468 kB) | arc-processor-1.4.1.Final.jar (101/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (44/142 kB) | quarkus-security-spi-1.4.Progress (5): jboss-metadata-common-11.0.0.Final.jar (378/475 kB) | jboss-metadata-web-11.0.0.Final.jar (284/468 kB) | arc-processor-1.4.1.Final.jar (101/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (44/142 kB) | quarkus-security-spi-1.4.Progress (5): jboss-metadata-common-11.0.0.Final.jar (381/475 kB) | jboss-metadata-web-11.0.0.Final.jar (284/468 kB) | arc-processor-1.4.1.Final.jar (101/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (44/142 kB) | quarkus-security-spi-1.4.Progress (5): jboss-metadata-common-11.0.0.Final.jar (384/475 kB) | jboss-metadata-web-11.0.0.Final.jar (284/468 kB) | arc-processor-1.4.1.Final.jar (101/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (44/142 kB) | quarkus-security-spi-1.4.Progress (5): jboss-metadata-common-11.0.0.Final.jar (384/475 kB) | jboss-metadata-web-11.0.0.Final.jar (288/468 kB) | arc-processor-1.4.1.Final.jar (101/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (44/142 kB) | quarkus-security-spi-1.4.Progress (5): jboss-metadata-common-11.0.0.Final.jar (387/475 kB) | jboss-metadata-web-11.0.0.Final.jar (288/468 kB) | arc-processor-1.4.1.Final.jar (101/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (44/142 kB) | quarkus-security-spi-1.4.Progress (5): jboss-metadata-common-11.0.0.Final.jar (387/475 kB) | jboss-metadata-web-11.0.0.Final.jar (292/468 kB) | arc-processor-1.4.1.Final.jar (101/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (44/142 kB) | quarkus-security-spi-1.4.Progress (5): jboss-metadata-common-11.0.0.Final.jar (387/475 kB) | jboss-metadata-web-11.0.0.Final.jar (297/468 kB) | arc-processor-1.4.1.Final.jar (101/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (44/142 kB) | quarkus-security-spi-1.4.Progress (5): jboss-metadata-common-11.0.0.Final.jar (387/475 kB) | jboss-metadata-web-11.0.0.Final.jar (301/468 kB) | arc-processor-1.4.1.Final.jar (101/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (44/142 kB) | quarkus-security-spi-1.4.Progress (5): jboss-metadata-common-11.0.0.Final.jar (389/475 kB) | jboss-metadata-web-11.0.0.Final.jar (301/468 kB) | arc-processor-1.4.1.Final.jar (101/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (44/142 kB) | quarkus-security-spi-1.4.Progress (5): jboss-metadata-common-11.0.0.Final.jar (392/475 kB) | jboss-metadata-web-11.0.0.Final.jar (301/468 kB) | arc-processor-1.4.1.Final.jar (101/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (44/142 kB) | quarkus-security-spi-1.4.Progress (5): jboss-metadata-common-11.0.0.Final.jar (395/475 kB) | jboss-metadata-web-11.0.0.Final.jar (301/468 kB) | arc-processor-1.4.1.Final.jar (101/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (44/142 kB) | quarkus-security-spi-1.4.Progress (5): jboss-metadata-common-11.0.0.Final.jar (398/475 kB) | jboss-metadata-web-11.0.0.Final.jar (301/468 kB) | arc-processor-1.4.1.Final.jar (101/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (44/142 kB) | quarkus-security-spi-1.4.Progress (5): jboss-metadata-common-11.0.0.Final.jar (400/475 kB) | jboss-metadata-web-11.0.0.Final.jar (301/468 kB) | arc-processor-1.4.1.Final.jar (101/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (44/142 kB) | quarkus-security-spi-1.4.Progress (5): jboss-metadata-common-11.0.0.Final.jar (403/475 kB) | jboss-metadata-web-11.0.0.Final.jar (301/468 kB) | arc-processor-1.4.1.Final.jar (101/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (44/142 kB) | quarkus-security-spi-1.4.Progress (5): jboss-metadata-common-11.0.0.Final.jar (406/475 kB) | jboss-metadata-web-11.0.0.Final.jar (301/468 kB) | arc-processor-1.4.1.Final.jar (101/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (44/142 kB) | quarkus-security-spi-1.4.Progress (5): jboss-metadata-common-11.0.0.Final.jar (409/475 kB) | jboss-metadata-web-11.0.0.Final.jar (301/468 kB) | arc-processor-1.4.1.Final.jar (101/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (44/142 kB) | quarkus-security-spi-1.4.Progress (5): jboss-metadata-common-11.0.0.Final.jar (411/475 kB) | jboss-metadata-web-11.0.0.Final.jar (301/468 kB) | arc-processor-1.4.1.Final.jar (101/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (44/142 kB) | quarkus-security-spi-1.4.Progress (5): jboss-metadata-common-11.0.0.Final.jar (414/475 kB) | jboss-metadata-web-11.0.0.Final.jar (301/468 kB) | arc-processor-1.4.1.Final.jar (101/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (44/142 kB) | quarkus-security-spi-1.4.Progress (5): jboss-metadata-common-11.0.0.Final.jar (414/475 kB) | jboss-metadata-web-11.0.0.Final.jar (301/468 kB) | arc-processor-1.4.1.Final.jar (104/310 kB) | quarkus-arc-deployment-1.4.1.Final.jar (44/142 kB) | quarkus-security-spi-1.4.                                                                                                                                                                                                                                                  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-security-spi/1.4.1.Final/quarkus-security-spi-1.4.1.Final.jar (3.3 kB at 4.8 kB/s)
  Downloading from central: https://repo.maven.apache.org/maven2/org/graalvm/nativeimage/svm/19.3.1/svm-19.3.1.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/metadata/jboss-metadata-common/11.0.0.Final/jboss-metadata-common-11.0.0.Final.jar (475 kB at 530 kB/s)
  Downloading from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy-deployment/1.4.1.Final/quarkus-resteasy-deployment-1.4.1.Final.jar
  Downloaded from central: https://repo.maven.apache.org/maven2/org/jboss/metadata/jboss-metadata-web/11.0.0.Final/jboss-metadata-web-11.0.0.Final.jar (468 kB at 427 kB/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-resteasy-deployment/1.4.1.Final/quarkus-resteasy-deployment-1.4.1.Final.jar (22 kB at 20 kB/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/quarkus-arc-deployment/1.4.1.Final/quarkus-arc-deployment-1.4.1.Final.jar (142 kB at 121 kB/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/io/quarkus/arc/arc-processor/1.4.1.Final/arc-processor-1.4.1.Final.jar (310 kB at 242 kB/s)
  Downloaded from central: https://repo.maven.apache.org/maven2/org/graalvm/nativeimage/svm/19.3.1/svm-19.3.1.jar (10 MB at 773 kB/s)
  mvn test  52.19s user 5.37s system 58% cpu 1:39.18 total
  grep central  0.44s user 1.30s system 1% cpu 1:39.18 total
  ```
  
  Determination:
  * Quarkus test dependencies pulled from "central", ignoring repository configuration defined in ~/.m2/settings.xml. If you run the above `mvn test` command without the `| grep central` you can see that when Quarkus runs its tests during the `maven-surefire-plugin` lifecycle, it downloads from the default "central" repository. 
  
* `q-1.3.2
  ```
  git checkout q-1.3.2
  rm -Rf ~/.m2/repository
  mvn test | grep central
  ```

  Output:
  ```
  mvn test  43.33s user 5.12s system 61% cpu 1:19.07 total
  grep central  0.35s user 1.09s system 1% cpu 1:19.07 total
  ```
  
  Determination:
  * All dependencies pulled from "get-all-from-here" defined repository, as expected
  
* `q-1.3.2-maven-revision`
  ```
  git checkout q-1.3.2-maven-revision
  rm -Rf ~/.m2/repository
  mvn test | grep central
  ```

  Output:
  ```
  mvn test  46.44s user 5.01s system 68% cpu 1:15.10 total
  grep central  0.34s user 1.06s system 1% cpu 1:15.10 total
  ```
  
  Determination:
  * All dependencies pulled from "get-all-from-here" defined repository, as expected
  