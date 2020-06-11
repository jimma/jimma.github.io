---
layout:     page
title:      "Use galleon to customize and install Wildfly"
subtitle:   ""
date:       2020-06-03
author:     "Jim Ma"
header-img: "img/post-bg-06.jpg"
---
Wildfly Galleon project is designed to install, uninstall patch product with a lot of customization. With a defined xml , user can install what they want. For a large size product like wildfly server, if user only wants web and messaging functionalities, they can use this tool to only install this two components in the distribution. 

Wildfly Galleon provides two tools to do software provisioning: CLI and maven plugin. Let's play with the CLI tool first. 

The latest Gallon release is 4.2.5.Final, we download from [here](https://github.com/wildfly/galleon/releases/tag/4.2.5.Final). unzip and cli tool will be ready for use. 
Let's try to install wildfly-core sever with this two steps:
- create an empty dir to put the wildfly-core server installation
- under this dir run **galleon-4.2.5.Final/bin/galleon.sh  install org.wildfly.core:wildfly-core-galleon-pack:12.0.0.Final**
  
```
Feature-packs resolved. 
Packages installed. 
JBoss modules installed. 
Configurations generated. 
Feature pack installed.
```
**gallon.sh** successfully fetch everything from maven repo and install wildfly-core server under this empty dir. The installation log shows the feature pack is resolved and installed. What's the magic behind this simple CLI execution ? Open this galleon pack file wildfly-core-galleon-pack-12.0.0.Final.zip, it shows:

![wildflycoregalleonpack.zip](/img/galleon/wildflycorepack.png)

The galleon feature pack entry file is the feature-pack.xml. This xml file saves which version wildfly-galleon-plugin handle this plugin and what should do to install this feature pack: 
```
<?xml version="1.0" ?>
<feature-pack xmlns="urn:jboss:galleon:feature-pack:2.0" location="wildfly-core@maven(org.jboss.universe:community-universe):current#12.0.1.Final-SNAPSHOT">
    <config model="standalone" name="standalone.xml"/>
    <config model="domain" name="domain.xml"/>
    <config model="host" name="host.xml"/>
    <config model="host" name="host-master.xml"/>
    <config model="host" name="host-slave.xml"/>
    <default-packages>
        <package name="bin"/>
        <package name="docs"/>
        <package name="modules.all"/>
    </default-packages>
    <plugins>
        <plugin id="wildfly-galleon-plugins" location="org.wildfly.galleon-plugins:wildfly-galleon-plugins:jar:4.2.6.Final"/>
    </plugins>
</feature-pack>
```

### Create default server binary

**package** in galleon is the basic task element to create, copy, delete or modify jobs. In some package there is ant tasks.xml job defines to do remove directory, xslt or whatever ant task can do. Above three default package will create bin, docs and modules directory. package can depend another package like one ant task depends another ant task. One package includes the resources it requires to copy or modify the installation file.

### Configuraiton provisioning 

The first element defines it use the standalone config in this zip file to compose standalone server configuration:

```
configs
├── domain
│   ├── domain.xml
│   │   └── config.xml
│   └── model.xml
├── host
│   ├── host-master.xml
│   │   └── config.xml
│   ├── host-slave.xml
│   │   └── config.xml
│   ├── host.xml
│   │   └── config.xml
│   └── model.xml
└── standalone
    ├── model.xml
    └── standalone.xml
        └── config.xml

```

The galleon plugin goes to find model.xml and config.xml under "standalone.xml" directory and follow the instructions from these two xml file to install more things as the xml defined. 

#### Create configuraiton directory, resources files and required jboss modules

The model.xml has content to do prepare all directory belongs to standalone configuraiton: 
```
    <packages>
        <package name="product.conf" optional="true"/>
        <package name="misc.standalone"/>
        <package name="org.jboss.as.standalone"/>
        <package name="org.jboss.as.domain-management"/>
        <!-- cleanup runtime dirs -->
        <package name="cleanup.standalone.config.history.dir" optional="true"/>
        <package name="cleanup.standalone.log.dir" optional="true"/>
        <package name="cleanup.standalone.data.dir" optional="true"/>
        <package name="cleanup.standalone.tmp.vfs" optional="true"/>

        <package name="org.wildfly.bootable-jar" optional="true"/>
        <package name="org.wildfly.embedded"/>
    </packages>
</config>
```

From the package name , it did the following theses :
- cleanup log,config history, log ,data tmp dir first
- create org.jboss.as.standalone module
- create org.jboss.as.domain-management module
- create org.wildfly.bootable-jar module
- create org.wildfly.embedded module 
- create standalone configuration directory by <misc.standalone> package and copy properties to standalone configuration directory : like application-roles.properties, application-user.properties.
- create standalone.sh under bin

#### Create final standalone configuration file

There is config.xml which defines various layer to compose the final standalone.xml server configuration file:
```
<config xmlns="urn:jboss:galleon:config:1.0" name="standalone.xml" model="standalone">
    <layers>
        <include name="legacy-management"/>
        <include name="elytron"/> 
        <include name="jmx-remoting"/> 
        <include name="logging"/>
        <include name="core-management"/>
        <include name="request-controller"/>
        <include name="security-manager"/>
        <include name="discovery"/>
        <include name="core-tools"/>
    </layers>
    <!-- package included in case configuration is provisioned without inheriting all modules -->
    <packages>
        <package name="org.jboss.as.patching.cli"/>
    </packages>
</config>
```
From galleon doc, the layer is:
>is meant to represent a certain configuration flavor that can be used on its own or in combination with other layers to produce the final configuration.

layer is dependable like : 

```
<?xml version="1.0" ?>
<layer-spec xmlns="urn:jboss:galleon:layer-spec:1.0" name="core-server">
    <dependencies>
        <layer name="core-security-realms" optional="true"/>
        <layer name="secure-management" optional="true"/>
        <layer name="jmx-remoting" optional="true"/> 
        <layer name="logging" optional="true"/>
        <layer name="core-management" optional="true"/>
        <layer name="request-controller" optional="true"/>
        <layer name="security-manager" optional="true"/>
    </dependencies>
    
</layer-spec>
```

and layer is composed with feature group or feature: 

```
<?xml version="1.0" ?>
<layer-spec xmlns="urn:jboss:galleon:layer-spec:1.0" name="base-server">
    <feature-group name="standalone-base"/>
    <feature-group name="public-interface"/>
</layer-spec>

```

feature-group is composed with feature:

```
<?xml version="1.0" encoding="UTF-8"?>
<feature-group-spec name="standalone-base" xmlns="urn:jboss:galleon:feature-group:1.0">
    <feature spec="server-root">
        <param name="server-root" value="/" />
    </feature>
    <feature spec="socket-binding-group">
        <param name="default-interface" value="public" />
        <param name="socket-binding-group" value="standard-sockets" />
        <param name="port-offset" value="${jboss.socket.binding.port-offset:0}"/>
    </feature>
</feature-group-spec>
```

feature is composed different elements:

```
<feature-spec xmlns="urn:jboss:galleon:feature-spec:1.0" name="server-root">
    <annotation name="jboss-op">
        <elem name="name" value="write-attribute"/>
        <elem name="op-params" value="organization,name"/>
        <elem name="addr-params" value="server-root"/>
    </annotation>
    <annotation name="feature-branch">
        <elem name="id" value="server-root"/>
    </annotation>
    <provides>
        <capability name="org.wildfly.management.path.jboss.server.log.dir"/>
        <capability name="org.wildfly.management.model-controller-client-factory"/>
        <capability name="org.wildfly.server.suspend-controller"/>
        <capability name="org.wildfly.management.executor"/>
        <capability name="org.wildfly.management.external-module"/>
        <capability name="org.wildfly.management.path.jboss.home.dir"/>
        <capability name="org.wildfly.management.path.jboss.server.temp.dir"/>
        <capability name="org.wildfly.management.path.jboss.server.data.dir"/>
        <capability name="org.wildfly.management.path.jboss.server.base.dir"/>
        <capability name="org.wildfly.management.process-state-notifier"/>
        <capability name="org.wildfly.management.path-manager"/>
        <capability name="org.wildfly.management.path.jboss.controller.temp.dir"/>
        <capability name="org.wildfly.management.console-availability"/>
        <capability name="org.wildfly.management.notification-handler-registry"/>
        <capability name="org.wildfly.management.path.jboss.server.config.dir"/>
    </provides>
    <params>
        <param name="name" nillable="true"/>
        <param name="organization" nillable="true"/>
        <param name="server-root" feature-id="true" default="/"/>
    </params>
    <packages>
        <package name="wildflyee.api" optional="true"/>
        <package name="ibm.jdk"/>
    </packages>
</feature-spec>
```

It looks the features are generated by galleon plugin instead of manually created. 
Till now, we have question about how does it generate standalone.xml ? 
Galleon follows these steps to generate all configuration file: 
- scan annotation element from above feature spec xml and extract operation can interact with wildfly ModelController : 
  ```
      <annotation name="jboss-op">
        <elem name="name" value="write-attribute"/>
        <elem name="op-params" value="organization,name"/>
        <elem name="addr-params" value="server-root"/>
    </annotation>
    
  ``` 
 - save all batch ModelNode operation or non batch ModelNode operation in a script file like: 
  
  ```
  standalone
--internal-empty-config,--admin-only,--internal-remove-config,--server-config,standalone-microprofile-ha.xml
batch
{"operation":"add","address":[{"interface":"public"}],"inet-address":"${jboss.bind.address:127.0.0.1}"}
{"operation":"add","address":[{"interface":"management"}],"inet-address":"${jboss.bind.address.management:127.0.0.1}"}
{"operation":"add","address":[{"interface":"private"}],"inet-address":"${jboss.bind.address.private:127.0.0.1}"}
{"operation":"add","address":[{"extension":"org.jboss.as.naming"}]}
{"operation":"add","address":[{"extension":"org.jboss.as.ee"}]}
{"operation":"add","address":[{"extension":"org.jboss.as.deployment-scanner"}]}
{"operation":"add","address":[{"extension":"org.wildfly.extension.io"}]}
{"operation":"add","address":[{"extension":"org.wildfly.extension.undertow"}]}
{"operation":"add","address":[{"extension":"org.wildfly.extension.elytron"}]}
{"operation":"add","address":[{"extension":"org.jboss.as.jmx"}]}
{"operation":"add","address":[{"extension":"org.jboss.as.logging"}]}
{"operation":"add","address":[{"extension":"org.wildfly.extension.core-management"}]}
{"operation":"add","address":[{"extension":"org.wildfly.extension.request-controller"}]}
{"operation":"add","address":[{"extension":"org.wildfly.extension.security.manager"}]}
{"operation":"add","address":[{"extension":"org.jboss.as.transactions"}]}
{"operation":"add","address":[{"extension":"org.jboss.as.connector"}]}
{"operation":"add","address":[{"extension":"org.jboss.as.jaxrs"}]}
{"operation":"add","address":[{"extension":"org.wildfly.extension.bean-validation"}]}
{"operation":"add","address":[{"extension":"org.jboss.as.weld"}]}
{"operation":"add","address":[{"extension":"org.wildfly.extension.ee-security"}]}
{"operation":"add","address":[{"extension":"org.wildfly.extension.messaging-activemq"}]}
{"operation":"add","address":[{"extension":"org.wildfly.extension.microprofile.config-smallrye"}]}
{"operation":"add","address":[{"extension":"org.wildfly.extension.microprofile.metrics-smallrye"}]}
{"operation":"add","address":[{"extension":"org.wildfly.extension.microprofile.opentracing-smallrye"}]}
{"operation":"add","address":[{"extension":"org.wildfly.extension.microprofile.health-smallrye"}]}
{"operation":"add","address":[{"extension":"org.wildfly.extension.microprofile.fault-tolerance-smallrye"}]}
{"operation":"add","address":[{"extension":"org.wildfly.extension.microprofile.jwt-smallrye"}]}
{"operation":"add","address":[{"extension":"org.wildfly.extension.microprofile.openapi-smallrye"}]}
{"operation":"add","address":[{"extension":"org.jboss.as.jpa"}]}
{"operation":"add","address":[{"extension":"org.jboss.as.clustering.jgroups"}]}
{"operation":"add","address":[{"extension":"org.jboss.as.clustering.infinispan"}]}
{"operation":"add","address":[{"extension":"org.wildfly.extension.clustering.web"}]}
run-batch
{"operation":"add","address":[{"socket-binding-group":"standard-sockets"}],"default-interface":"public","port-offset":"${jboss.socket.binding.port-offset:0}"}
{"operation":"add","address":[{"subsystem":"naming"}]}

```

  The full script file can be found [here](https://gist.github.com/jimma/8fbf69f029a69fdc534ee0fc5631b83d)
  
-  start embedded server and execute this script to generate all configurations. This is the full command to run these operations and generate configuration file like standalone.xml:
  
  ```
  [jdk1.8.0_162/jre/bin/java, -server, -cp, maven_repo/org/jboss/galleon/galleon-maven-plugin/4.2.5.Final/galleon-maven-plugin-4.2.5.Final.jar:maven_repo/org/jboss/galleon/galleon-core/4.2.5.Final/galleon-core-4.2.5.Final.jar:maven_repo/org/jboss/staxmapper/1.1.0.Final/staxmapper-1.1.0.Final.jar:maven_repo/org/jboss/galleon/galleon-maven-universe/4.2.5.Final/galleon-maven-universe-4.2.5.Final.jar:maven_repo/org/apache/maven/plugin-tools/maven-plugin-annotations/3.5/maven-plugin-annotations-3.5.jar:maven_repo/org/eclipse/aether/aether-util/1.1.0/aether-util-1.1.0.jar:maven_repo/org/apache/maven/shared/maven-artifact-transfer/0.9.1/maven-artifact-transfer-0.9.1.jar:maven_repo/org/codehaus/plexus/plexus-utils/3.0.24/plexus-utils-3.0.24.jar:/tmp/873ff52b-af6f-411b-81fe-441f4367d680/plugins/wildfly-galleon-plugins.jar:/tmp/873ff52b-af6f-411b-81fe-441f4367d680/resources/wildfly/wildfly-config-gen.jar:maven_repo/org/jboss/modules/jboss-modules/1.10.0.Final/jboss-modules-1.10.0.Final.jar:maven_repo/org/wildfly/core/wildfly-cli/11.0.0.Final/wildfly-cli-11.0.0.Final-client.jar, org.wildfly.galleon.plugin.server.ForkedEmbeddedUtil, /tmp/wfgp6707939363760510761sysprops, org.wildfly.galleon.plugin.config.generator.WfConfigGenerator, /home/jimma/code/wildfly/stage/wildfly/dist/target/wildfly-20.0.0.Beta1-SNAPSHOT, /tmp/873ff52b-af6f-411b-81fe-441f4367d680/tmp/forkedembedded.txt]
  ```
  
  The next question is what does galleon read to generate the annotation element in feature's spec.xml ? 
  as create configuration file , it starts a fork process with these args:

 ```
 java, -server, -cp, maven_repo/org/wildfly/galleon-plugins/wildfly-galleon-maven-plugin/4.2.6.Final/wildfly-galleon-maven-plugin-4.2.6.Final.jar:maven_repo/org/wildfly/core/wildfly-embedded/12.0.0.Beta2/wildfly-embedded-12.0.0.Beta2.jar:maven_repo/org/jboss/logging/jboss-logging/3.4.1.Final/jboss-logging-3.4.1.Final.jar:maven_repo/org/jboss/modules/jboss-modules/1.10.0.Final/jboss-modules-1.10.0.Final.jar:maven_repo/org/wildfly/core/wildfly-controller-client/12.0.0.Beta2/wildfly-controller-client-12.0.0.Beta2.jar:maven_repo/org/jboss/jboss-dmr/1.5.0.Final/jboss-dmr-1.5.0.Final.jar:maven_repo/org/jboss/threads/jboss-threads/2.3.3.Final/jboss-threads-2.3.3.Final.jar:maven_repo/org/wildfly/common/wildfly-common/1.5.2.Final/wildfly-common-1.5.2.Final.jar:maven_repo/org/wildfly/galleon-plugins/wildfly-galleon-plugins/4.2.6.Final/wildfly-galleon-plugins-4.2.6.Final.jar:maven_repo/org/jboss/galleon/galleon-core/4.2.5.Final/galleon-core-4.2.5.Final.jar:maven_repo/org/jboss/staxmapper/1.1.0.Final/staxmapper-1.1.0.Final.jar:maven_repo/org/jboss/galleon/galleon-maven-plugin/4.2.5.Final/galleon-maven-plugin-4.2.5.Final.jar:maven_repo/org/eclipse/aether/aether-util/1.1.0/aether-util-1.1.0.jar:maven_repo/org/jboss/galleon/galleon-maven-universe/4.2.5.Final/galleon-maven-universe-4.2.5.Final.jar:maven_repo/org/apache/maven/plugin-tools/maven-plugin-annotations/3.5.2/maven-plugin-annotations-3.5.2.jar:maven_repo/org/wildfly/galleon-plugins/wildfly-config-gen/4.2.6.Final/wildfly-config-gen-4.2.6.Final.jar:maven_repo/org/wildfly/galleon-plugins/wildfly-feature-spec-gen/4.2.6.Final/wildfly-feature-spec-gen-4.2.6.Final.jar:maven_repo/org/apache/maven/shared/maven-filtering/3.1.1/maven-filtering-3.1.1.jar:maven_repo/javax/enterprise/cdi-api/1.0/cdi-api-1.0.jar:maven_repo/org/eclipse/sisu/org.eclipse.sisu.inject/0.3.0/org.eclipse.sisu.inject-0.3.0.jar:maven_repo/org/sonatype/sisu/sisu-guice/3.2.5/sisu-guice-3.2.5-no_aop.jar:maven_repo/aopalliance/aopalliance/1.0/aopalliance-1.0.jar:maven_repo/com/google/guava/guava/16.0.1/guava-16.0.1.jar:maven_repo/org/sonatype/plexus/plexus-sec-dispatcher/1.3/plexus-sec-dispatcher-1.3.jar:maven_repo/org/sonatype/plexus/plexus-cipher/1.4/plexus-cipher-1.4.jar:maven_repo/org/apache/maven/shared/maven-shared-utils/3.0.0/maven-shared-utils-3.0.0.jar:maven_repo/com/google/code/findbugs/jsr305/2.0.1/jsr305-2.0.1.jar:maven_repo/org/codehaus/plexus/plexus-utils/3.1.0/plexus-utils-3.1.0.jar:maven_repo/org/codehaus/plexus/plexus-interpolation/1.22/plexus-interpolation-1.22.jar:maven_repo/org/sonatype/plexus/plexus-build-api/0.0.7/plexus-build-api-0.0.7.jar:maven_repo/org/apache/commons/commons-lang3/3.5/commons-lang3-3.5.jar:maven_repo/org/codehaus/plexus/plexus-archiver/3.4/plexus-archiver-3.4.jar:maven_repo/org/codehaus/plexus/plexus-io/2.7.1/plexus-io-2.7.1.jar:maven_repo/commons-io/commons-io/2.5/commons-io-2.5.jar:maven_repo/org/apache/commons/commons-compress/1.11/commons-compress-1.11.jar:maven_repo/org/iq80/snappy/snappy/0.4/snappy-0.4.jar:maven_repo/org/tukaani/xz/1.5/xz-1.5.jar:maven_repo/org/apache/maven/shared/maven-artifact-transfer/0.9.1/maven-artifact-transfer-0.9.1.jar:maven_repo/org/codehaus/plexus/plexus-component-annotations/1.7.1/plexus-component-annotations-1.7.1.jar:maven_repo/org/apache/maven/shared/maven-common-artifact-filters/3.0.1/maven-common-artifact-filters-3.0.1.jar:maven_repo/org/sonatype/sisu/sisu-inject-bean/1.4.2/sisu-inject-bean-1.4.2.jar:maven_repo/org/sonatype/sisu/sisu-guice/2.1.7/sisu-guice-2.1.7-noaop.jar:maven_repo/commons-codec/commons-codec/1.6/commons-codec-1.6.jar:maven_repo/org/wildfly/maven/plugins/licenses-plugin/2.0.0.Final/licenses-plugin-2.0.0.Final.jar:maven_repo/org/apache/maven/maven-builder-support/3.3.1/maven-builder-support-3.3.1.jar:maven_repo/org/apache/maven/plugin-testing/maven-plugin-testing-harness/3.3.0/maven-plugin-testing-harness-3.3.0.jar, org.wildfly.galleon.plugin.server.ForkedEmbeddedUtil, /tmp/wfgp1019281718254681233sysprops, org.wildfly.galleon.plugin.featurespec.generator.FeatureSpecGenerator, wildfly/stage/wildfly/ee-galleon-pack/target/wildfly, /tmp/wfgp4280664821083887940standalone-specs, /tmp/wfgp2444550083207653964domain-specs
 
 ```

The functional class **FeatureSpecGenerator** takes care of start an embedded server with the extensions configured :

```
<?xml version='1.0' encoding='UTF-8'?>
<server xmlns="urn:jboss:domain:6.0">
<extensions>
<extension module="org.jboss.as.mail"/>
<extension module="org.jboss.as.jpa"/>
<extension module="org.wildfly.extension.undertow"/>
<extension module="org.wildfly.extension.microprofile.health-smallrye"/>
<extension module="org.jboss.as.naming"/>
<extension module="org.jboss.as.clustering.infinispan"/>
<extension module="org.jboss.as.sar"/>
<extension module="org.wildfly.extension.microprofile.opentracing-smallrye"/>
<extension module="org.jboss.as.modcluster"/>
<extension module="org.jboss.as.pojo"/>
<extension module="org.jboss.as.transactions"/>
<extension module="org.wildfly.extension.clustering.singleton"/>
<extension module="org.wildfly.iiop-openjdk"/>
<extension module="org.wildfly.extension.security.manager"/>
<extension module="org.wildfly.extension.elytron"/>
<extension module="org.wildfly.extension.discovery"/>
<extension module="org.jboss.as.ejb3"/>
<extension module="org.wildfly.extension.batch.jberet"/>
<extension module="org.jboss.as.jdr"/>
<extension module="org.jboss.as.remoting"/>
<extension module="org.jboss.as.clustering.jgroups"/>
<extension module="org.jboss.as.xts"/>
<extension module="org.wildfly.extension.picketlink"/>
<extension module="org.jboss.as.ee"/>
<extension module="org.jboss.as.jsf"/>
<extension module="org.jboss.as.connector"/>
<extension module="org.jboss.as.logging"/>
<extension module="org.jboss.as.jaxrs"/>
<extension module="org.wildfly.extension.microprofile.config-smallrye"/>
<extension module="org.wildfly.extension.core-management"/>
<extension module="org.wildfly.extension.rts"/>
<extension module="org.jboss.as.weld"/>
<extension module="org.wildfly.extension.microprofile.metrics-smallrye"/>
<extension module="org.wildfly.extension.bean-validation"/>
<extension module="org.wildfly.extension.ee-security"/>
<extension module="org.jboss.as.jsr77"/>
<extension module="org.jboss.as.webservices"/>
<extension module="org.jboss.as.jmx"/>
<extension module="org.wildfly.extension.clustering.web"/>
<extension module="org.jboss.as.deployment-scanner"/>
<extension module="org.wildfly.extension.messaging-activemq"/>
<extension module="org.wildfly.extension.request-controller"/>
<extension module="org.wildfly.extension.io"/>
<extension module="org.jboss.as.security"/>
</extensions>
</server>
```

And execute :read-feature-description operation recursively to dump all info to a file with all feature description info like :

```
"subsystem.jaxrs" =>
   {"name" => "subsystem.jaxrs",
    "annotation" => {"name" => "add",
                   "addr-params" => "subsystem","op-params" => "jaxrs-2-0-request-matching,         
                       resteasy-add-charset, resteasy-buffer-exception-entity,resteasy-disable-html-sanitizer,resteasy-disable-providers,resteasy-document-expand-entity-references,resteasy-document-secure-disableDTDs,resteasy-document-secure-processing-feature,resteasy-gzip-max-input,resteasy-jndi-resources,resteasy-language-mappings,resteasy-media-type-mappings,resteasy-media-type-param-mapping,resteasy-prefer-jackson-over-jsonb,resteasy-providers,resteasy-rfc7232preconditions,resteasy-role-based-security,resteasy-secure-random-max-use,resteasy-use-builtin-providers,resteasy-use-container-form-params,resteasy-wider-request-matching"}
                 ,"params" => [{"name" => "subsystem","default" => "jaxrs","feature-id" => true},{"name" => "jaxrs-2-0-request-matching","nillable" => true},{"name" => "resteasy-add-charset","nillable" => true},{"name" => "resteasy-buffer-exception-entity","nillable" => true},{"name" => "resteasy-disable-html-sanitizer","nillable" => true},{"name" => "resteasy-disable-providers","nillable" => true,"type" => "List<String>"},{"name" => "resteasy-document-expand-entity-references","nillable" => true},{"name" => "resteasy-document-secure-disableDTDs","nillable" => true},{"name" => "resteasy-document-secure-processing-feature","nillable" => true},{"name" => "resteasy-gzip-max-input","nillable" => true},{"name" => "resteasy-jndi-resources","nillable" => true,"type" => "List<String>"},{"name" => "resteasy-language-mappings","nillable" => true},{"name" => "resteasy-media-type-mappings","nillable" => true},{"name" => "resteasy-media-type-param-mapping","nillable" => true},{"name" => "resteasy-prefer-jackson-over-jsonb","nillable" => true},{"name" => "resteasy-providers","nillable" => true,"type" => "List<String>"},{"name" => "resteasy-rfc7232preconditions","nillable" => true},{"name" => "resteasy-role-based-security","nillable" => true},{"name" => "resteasy-secure-random-max-use","nillable" => true},{"name" => "resteasy-use-builtin-providers","nillable" => true},{"name" => "resteasy-use-container-form-params","nillable" => true},{"name" => "resteasy-wider-request-matching","nillable" => true},{"name" => "extension","default" => "org.jboss.as.jaxrs"}],"refs" => [{"feature" => "extension","include" => true}],
                 
                 
                 "packages" => [{"package" => "org.jboss.resteasy.resteasy-validator-provider","optional" => true,"passive" => true},{"package" => "org.jboss.resteasy.resteasy-validator-provider-11","optional" => true,"passive" => true},{"package" => "org.jboss.resteasy.resteasy-json-binding-provider","optional" => true},{"package" => "com.fasterxml.jackson.datatype.jackson-datatype-jdk8","optional" => true},{"package" => "org.jboss.resteasy.resteasy-json-p-provider","optional" => true},{"package" => "org.jboss.resteasy.resteasy-multipart-provider","optional" => true},{"package" => "org.eclipse.microprofile.restclient","optional" => true},{"package" => "javax.ws.rs.api"},{"package" => "org.jboss.resteasy.resteasy-cdi","optional" => true,"passive" => true},{"package" => "org.jboss.resteasy.resteasy-jaxrs","optional" => true},{"package" => "org.jboss.resteasy.resteasy-client-microprofile","optional" => true},{"package" => "org.jboss.resteasy.resteasy-spring","optional" => true},{"package" => "javax.xml.bind.api"},{"package" => "javax.json.api"},{"package" => "org.codehaus.jackson.jackson-core-asl","optional" => true},{"package" => "org.jboss.resteasy.resteasy-jsapi","optional" => true},{"package" => "org.jboss.resteasy.resteasy-crypto","optional" => true},{"package" => "org.jboss.resteasy.resteasy-jettison-provider","optional" => true},{"package" => "org.jboss.resteasy.resteasy-jackson2-provider","optional" => true},{"package" => "org.jboss.resteasy.resteasy-yaml-provider","optional" => true},{"package" => "com.fasterxml.jackson.datatype.jackson-datatype-jsr310","optional" => true},{"package" => "org.jboss.resteasy.resteasy-atom-provider","optional" => true},{"package" => "org.jboss.resteasy.resteasy-jackson-provider","optional" => true},{"package" => "org.jboss.resteasy.resteasy-jaxb-provider","optional" => true},{"package" => "org.jboss.as.jaxrs"}],"children" => {}}
```

The full content can be found from [here](https://gist.github.com/jimma/f5e982a19f0c0d42febd83a00874f22e/raw/55b0883a1bb458427493667ed46b34b213ea2a77/gistfile1.txt)

The subsystem path address, parameter etc can be get from the ModelNode. The package is read from registration as each subystem add the RuntimePackage while create the subystem definition like what jaxrs subystem does:

```
    @Override
    public void registerAdditionalRuntimePackages(ManagementResourceRegistration resourceRegistration) {
        resourceRegistration.registerAdditionalRuntimePackages(RuntimePackageDependency.passive(RESTEASY_CDI.getName()),
                    RuntimePackageDependency.passive(RESTEASY_VALIDATOR_11.getName()),
                    RuntimePackageDependency.passive(RESTEASY_VALIDATOR.getName()),
                    RuntimePackageDependency.required(JAXRS_API.getName()),
                    RuntimePackageDependency.required(JAXB_API.getName()),
                    RuntimePackageDependency.required(JSON_API.getName()),
                    RuntimePackageDependency.optional(RESTEASY_ATOM.getName()),
                    RuntimePackageDependency.optional(RESTEASY_JAXRS.getName()),
                    RuntimePackageDependency.optional(RESTEASY_CLIENT_MICROPROFILE.getName()),
                    RuntimePackageDependency.optional(RESTEASY_JAXB.getName()),
                    RuntimePackageDependency.optional(RESTEASY_JACKSON2.getName()),
                    RuntimePackageDependency.optional(RESTEASY_JSON_P_PROVIDER.getName()),
                    RuntimePackageDependency.optional(RESTEASY_JSON_B_PROVIDER.getName()),
                    RuntimePackageDependency.optional(RESTEASY_JSAPI.getName()),
                    RuntimePackageDependency.optional(RESTEASY_MULTIPART.getName()),
                    RuntimePackageDependency.optional(RESTEASY_YAML.getName()),
                    RuntimePackageDependency.optional(JACKSON_CORE_ASL.getName()),
                    RuntimePackageDependency.optional(RESTEASY_CRYPTO.getName()),
                    RuntimePackageDependency.optional(JACKSON_DATATYPE_JDK8.getName()),
                    RuntimePackageDependency.optional(JACKSON_DATATYPE_JSR310.getName()),
                    RuntimePackageDependency.optional(MP_REST_CLIENT.getName()),
                    // The following ones are optional dependencies located in org.jboss.as.jaxrs module.xml
                    // To be provisioned, they need to be explicitly added as optional packages.
                    RuntimePackageDependency.optional("org.jboss.resteasy.resteasy-jettison-provider"),
                    RuntimePackageDependency.optional("org.jboss.resteasy.resteasy-jackson-provider"),
                    RuntimePackageDependency.optional("org.jboss.resteasy.resteasy-spring"));
    }

```

 galleon-maven-plugin can be configured with layers to generate differnt wildfly distrubitons:

```
                    <plugin>
                        <groupId>org.jboss.galleon</groupId>
                        <artifactId>galleon-maven-plugin</artifactId>
                        <executions>
                            <execution>
                                <id>cloud-server-provisioning-test</id>
                                <goals>
                                    <goal>provision</goal>
                                </goals>
                                <phase>compile</phase>
                                <configuration>
                                    <install-dir>${layers.install.dir}/cloud-server</install-dir>
                                    <record-state>false</record-state>
                                    <log-time>${galleon.log.time}</log-time>
                                    <offline>true</offline>
                                    <plugin-options>
                                        <jboss-maven-dist/>
                                        <jboss-fork-embedded>${galleon.fork.embedded}</jboss-fork-embedded>
                                        <optional-packages>passive+</optional-packages>
                                    </plugin-options>
                                    <feature-packs>
                                        <feature-pack>
                                            <groupId>${testsuite.ee.galleon.pack.groupId}</groupId>
                                            <artifactId>${testsuite.ee.galleon.pack.artifactId}</artifactId>
                                            <version>${testsuite.ee.galleon.pack.version}</version>
                                            <inherit-configs>false</inherit-configs>
                                            <inherit-packages>false</inherit-packages>
                                        </feature-pack>
                                    </feature-packs>
                                    <configurations>
                                        <config>
                                            <model>standalone</model>
                                            <name>standalone.xml</name>
                                            <layers>
                                                <layer>cloud-server</layer>
                                            </layers>
                                        </config>
                                    </configurations>
                                </configuration>
                            </execution>
```

Or provides with a custom provisioning xml file for galleon.sh:

```
<installation xmlns="urn:jboss:galleon:provisioning:3.0">
    <feature-pack location="wildfly@maven(org.jboss.universe:community-universe):current">
        <default-configs inherit="false"/>
        <packages inherit="false">
            <!-- If docs/licenses is desired, uncomment this line -->
            <include name="docs.licenses"/>
        </packages>
    </feature-pack>
    <feature-pack location="org.wildfly.extras.wildfly-feature-pack-template:template-feature-pack:1.0.0.Alpha-SNAPSHOT">
        <default-configs inherit="false"/>
        <packages inherit="false">
            <!-- If docs/licenses is desired, uncomment this line -->
            <include name="docs.licenses.merge"/>
        </packages>
    </feature-pack>
    <config model="standalone" name="standalone.xml">
        <layers>
            <!-- The base server -->
            <include name="cloud-profile"/>

            <!-- Our layer(s) -->
            <include name="template-layer"/>
        </layers>
    </config>
    <options>
        <option name="optional-packages" value="passive+"/>
    </options>
</installation>
``` 

Or provides layers with CLI :

```
galleon.sh install wildfly:current --layers=cdi,jaxrs,jpa --dir=/path/to/wildfly
```

```
galleon.sh  install wildfly:current --layers=undertow --dir=/path/to/wildfly
```

```
galleon.sh  install wildfly:current --layers=web-server --dir=/path/to/wildfly
```

## Summary

When move to microservices/cloud architecture from monolith, we usually want the binary is as small as possible. This requires the wildfly or EAP can be trimmed and only includes the application needs. Galleon can help with this. If you want to slim or trim the wildfly distribution or even provide your custom layer, please give gallen a try. 

### Reference

- [EAP Galleon doc](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.3/html/getting_started_with_jboss_eap_for_openshift_online/capability-trimming-eap-foropenshift_default)

- [Galleon WFLY layer test](https://github.com/wildfly/wildfly/blob/master/testsuite/layers/)
 
- https://docs.wildfly.org/galleon









 