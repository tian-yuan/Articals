# Overview

**cfg4j** ("configuration for Java") is a **configuration library for Java distributed apps** (and more).

#### Features:

- Open source
- **Easy to use**
- **Auto-reloads** configuration
- **Powerful** configuration mechanisms (interface **binding**, **multi-source** support with **fallback** strategy, **merging**, ...)
- **Distributed-environment friendly** ( **caching**, support for **multiple environments** [test, preprod, prod], ...)
- Reads configuration from: **Consul, Git repos (YAML and/or properties), Files, Classpath, ...**
- Modern design
  - Seamless integration with DI containers: **Spring**, **Guice** and others
  - Exposes performance metrics by integration with **Metrics library**
  - Extensible (see the list of **plugins** below)
  - Heavily tested (**99% code coverage**)
  - Well [documented](http://cfg4j.org/)
  - Java 7+ required

总结： cfg4j 从 Consul, Git, S3 后端等获取配置，目前只支持 JAVA，支持 fallback, merging 等，但是不支持配置回滚和灰度发布。