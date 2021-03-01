---
layout:     page
title:      "Debug Quarkus Build Steps"
subtitle:   ""
date:       2021-03-01
author:     "Jim Ma"
header-img: "img/post-bg-06.jpg"
---
Quarkus moves the annotation scanning , build meta model, bytecode generation and does as much as possible things in build time 
instead of runtime to make application start up much faster with low memeory footprint. Without doubt, it does
a lot of things in build time to handle all things. The following are the build step list for a simple jaxrs 
service with cdi, resteasy-reactive and resteasy-reactive-jackson extensions enabled:

```java
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.ArcProcessor#exposeCustomScopeNames" in 4 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.netty.deployment.NettyProcessor#registerQualifiers" in 24 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.pkg.steps.PackageTypeVerificationBuildStep#builtins" in 8 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.resteasy.reactive.jackson.deployment.processor.ResteasyReactiveJacksonProcessor#feature" in 9 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.vertx.core.deployment.VertxCoreProcessor#filterNettyHostsFileParsingWarn" in 17 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.CommandLineArgumentsProcessor#commandLineArgs" in 69 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.resteasy.reactive.server.deployment.ResteasyReactiveProcessor#vertxIntegration" in 12 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.logging.LoggingResourceProcessor#setupLogFilters" in 16 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.StartupBuildSteps#addScope" in 26 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.vertx.http.deployment.VertxHttpProcessor#hostDefault" in 22 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.logging.LoggingResourceProcessor#setProperty" in 1 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.ExtensionLoader$$Lambda$107/914407920@36ad9dac" in 745 ms
[DEBUG] [io.quarkus.builder] Finished step "i0 o.quarkus.resteasy.reactive.server.deployment.ResteasyReactiveProcessor#securityExceptionMappers" in 770 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.netty.deployment.NettyProcessor#cleanup" in 776 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.vertx.http.deployment.VertxHttpProcessor#filterMultipleVertxInstancesWarning" in 3 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.HotDeploymentConfigBuildStep#configFile" in 5 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.UnremovableAnnotationsProcessor#unremovableBeans" in 805 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.resteasy.reactive.server.deployment.ResteasyReactiveProcessor#integrateSecurityOverrideSupport" in 59 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.resteasy.reactive.server.deployment.ResteasyReactiveProcessor#capability" in 14 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.resteasy.reactive.server.deployment.ResteasyReactiveCDIProcessor#contextInjection" in 795 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.vertx.http.deployment.VertxHttpProcessor#logging" in 20 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.logging.LoggingResourceProcessor#setUpDefaultLogCleanupFilters" in 11 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.steps.ReflectiveHierarchyStep#ignoreJavaClassWarnings" in 18 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.logging.LoggingResourceProcessor#setUpDarkeningDefault" in 15 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.ArcProcessor#loggerProducer" in 16 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.logging.LoggingResourceProcessor#setUpDefaultLevels" in 7 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.steps.AdditionalClassLoaderResourcesBuildStep#appendAdditionalClassloaderResources" in 4 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.pkg.steps.JarResultBuildStep#outputTarget" in 11 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.pkg.steps.FileSystemResourcesBuildStep#normalMode" in 3 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.ConfigBuildStep#registerConfigRootsAsBeans" in 7 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.pkg.steps.PackageTypeVerificationBuildStep#verify" in 8 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.vertx.http.deployment.HttpSecurityProcessor#builtins" in 2 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.resteasy.reactive.jackson.deployment.processor.ResteasyReactiveJacksonProcessor#jsonDefault" in 61 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.vertx.core.deployment.VertxCoreProcessor#build" in 34 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.steps.BannerProcessor#watchBannerChanges" in 35 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.jsonp.deployment.JsonpProcessor#build" in 35 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.vertx.http.deployment.HttpSecurityProcessor#initMtlsClientAuth" in 56 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.vertx.http.deployment.devmode.console.DevConsoleProcessor#hander" in 1020 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.ArcProcessor#launchMode" in 73 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.resteasy.reactive.server.deployment.ResteasyReactiveProcessor#buildSetup" in 33 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.TestsAsBeansProcessor#testAnnotations" in 36 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.ArcProcessor#marker" in 70 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.steps.ConfigBuildSteps#generateConfigSources" in 898 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.ArcProcessor#feature" in 30 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.vertx.core.deployment.VertxCoreProcessor#ioThreadDetector" in 999 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.resteasy.reactive.server.deployment.ResteasyReactiveVertxWebSocketIntegrationProcessor#scanner" in 12 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.logging.LoggingResourceProcessor#registerMetrics" in 131 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.ConfigBuildStep#bean" in 23 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.netty.deployment.NettyProcessor#setNettyMachineId" in 30 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.vertx.http.deployment.VertxHttpProcessor#frameworkRoot" in 1 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.steps.ApplicationInfoBuildStep#create" in 9 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.steps.BootstrapConfigSetupBuildStep#setupBootstrapConfig" in 57 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.steps.RuntimeConfigSetupBuildStep#setupRuntimeConfig" in 2 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.steps.DevModeBuildStep#watchChanges" in 3 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.resteasy.reactive.jackson.deployment.processor.ResteasyReactiveJacksonProcessor#additionalProviders" in 44 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.steps.MainClassBuildStep#applicationReflection" in 27 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.netty.deployment.NettyProcessor#limitArenaSize" in 8 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.resteasy.reactive.server.deployment.ResteasyReactiveScanningProcessor#asyncSupport" in 70 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.resteasy.reactive.server.deployment.ResteasyReactiveCDIProcessor#beanDefiningAnnotations" in 5 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.vertx.http.deployment.VertxHttpProcessor#additionalBeans" in 3 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.JniProcessor#setupJni" in 24 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.index.ApplicationArchiveBuildStep#addConfiguredIndexedDependencies" in 30 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.resteasy.reactive.server.deployment.ResteasyReactiveProcessor#registerCustomExceptionMappers" in 16 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.logging.LoggingResourceProcessor#setupLoggingStaticInit" in 75 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.dev.HotDeploymentConfigFileBuildStep#setupConfigFileHotDeployment" in 11 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.CollectionClassProcessor#setupCollectionClasses" in 6 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.TestsAsBeansProcessor#testClassBeans" in 11 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.vertx.http.deployment.HttpSecurityProcessor#initBasicAuth" in 29 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.vertx.core.deployment.VertxCoreProcessor#eventLoopCount" in 60 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.SslProcessor#setupNativeSsl" in 17 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.netty.deployment.NettyProcessor#eagerlyInitClass" in 25 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.vertx.http.deployment.HttpSecurityProcessor#initFormAuth" in 8 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.steps.ProfileBuildStep#defaultProfile" in 26 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.vertx.http.deployment.VertxHttpProcessor#cors" in 44 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.steps.BlockingOperationControlBuildStep#blockingOP" in 68 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.ArcProcessor#capability" in 24 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.StartupBuildSteps#unremovableBeans" in 52 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.vertx.core.deployment.VertxCoreProcessor#build" in 61 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.steps.ThreadPoolSetup#createExecutor" in 70 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.steps.BannerProcessor#recordBanner" in 87 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.ArcProcessor#quarkusMain" in 90 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.vertx.http.deployment.VertxHttpProcessor#bodyHandler" in 156 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.netty.deployment.NettyProcessor#registerEventLoopBeans" in 10 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.steps.ApplicationIndexBuildStep#build" in 158 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.logging.LoggingResourceProcessor#setupLoggingRuntimeInit" in 3 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.WrongAnnotationsProcessor#detect" in 2 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.ArcProcessor#setupExecutor" in 33 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.steps.ConfigGenerationBuildStep#checkForBuildTimeConfigChange" in 63 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.netty.deployment.NettyProcessor#build" in 1425 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.steps.NativeImageConfigBuildStep#build" in 6 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.index.ApplicationArchiveBuildStep#build" in 318 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.ObserverValidationProcessor#validateApplicationObserver" in 1 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.steps.CombinedIndexBuildStep#build" in 3 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.steps.MainClassBuildStep#mainClassBuildStep" in 2 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.steps.RegisterForReflectionBuildStep#build" in 4 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.BuildTimeEnabledProcessor#unlessBuildProfile" in 2 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.resteasy.reactive.common.deployment.ResteasyReactiveCommonProcessor#handleApplication" in 4 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.resteasy.reactive.server.deployment.ResteasyReactiveScanningProcessor#scanForParamConverters" in 0 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.resteasy.reactive.common.deployment.ResteasyReactiveCommonProcessor#scanForIOInterceptors" in 2 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.BuildTimeEnabledProcessor#ifBuildProfile" in 4 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.resteasy.reactive.server.deployment.ResteasyReactiveScanningProcessor#scanForParamConverters" in 10 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.ArcProcessor#quarkusApplication" in 9 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.jackson.deployment.JacksonProcessor#register" in 1 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.resteasy.reactive.server.deployment.ResteasyReactiveScanningProcessor#scanForFeatures" in 1 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.resteasy.reactive.server.deployment.ResteasyReactiveScanningProcessor#scanForInterceptors" in 5 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.jackson.deployment.JacksonProcessor#pushConfigurationBean" in 8 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.configproperties.ConfigPropertiesBuildStep#produceConfigPropertiesMetadata" in 6 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.vertx.core.deployment.VertxCoreProcessor#registerVerticleClasses" in 2 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.steps.CapabilityAggregationStep#build" in 1 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.configproperties.ConfigPropertiesBuildStep#setup" in 1 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.vertx.http.deployment.HttpSecurityProcessor#setupAuthenticationMechanisms" in 3 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.ConstructorPropertiesProcessor#build" in 7 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.resteasy.reactive.common.deployment.ResteasyReactiveCommonProcessor#scanResources" in 6 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.resteasy.reactive.server.deployment.ResteasyReactiveScanningProcessor#scanForDynamicFeatures" in 2 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.vertx.http.deployment.StaticResourcesProcessor#collectStaticResources" in 7 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.resteasy.reactive.server.deployment.ResteasyReactiveCDIProcessor#additionalBeans" in 8 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.resteasy.reactive.server.deployment.ResteasyReactiveProcessor#unremoveableBeans" in 3 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.jackson.deployment.JacksonProcessor#autoRegisterModules" in 7 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.vertx.http.deployment.StaticResourcesProcessor#staticInit" in 14 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.resteasy.reactive.server.deployment.ResteasyReactiveScanningProcessor#scanForContextResolvers" in 13 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.resteasy.reactive.jackson.deployment.processor.ResteasyReactiveJacksonProcessor#handleJsonAnnotations" in 1 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.resteasy.reactive.server.deployment.ResteasyReactiveProcessor#generateCustomProducer" in 18 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.jackson.deployment.JacksonProcessor#generateCustomizer" in 29 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.resteasy.reactive.server.deployment.ResteasyReactiveCDIProcessor#pathInterfaceImpls" in 14 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.resteasy.reactive.server.deployment.ResteasyReactiveProcessor#handleClassLevelExceptionMappers" in 31 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.resteasy.reactive.server.deployment.ResteasyReactiveScanningProcessor#handleCustomAnnotatedMethods" in 90 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.resteasy.reactive.server.deployment.ResteasyReactiveScanningProcessor#scanForExceptionMappers" in 6 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.resteasy.reactive.common.deployment.ResteasyReactiveCommonProcessor#buildResourceInterceptors" in 9 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.BeanArchiveProcessor#build" in 203 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.BuildTimeEnabledProcessor#unlessBuildProperty" in 2 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.BuildTimeEnabledProcessor#ifBuildProperty" in 1 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.BuildTimeEnabledProcessor#conditionTransformer" in 1 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.AutoProducerMethodsProcessor#annotationTransformer" in 4 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.AutoInjectFieldProcessor#autoInjectQualifiers" in 0 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.AutoInjectFieldProcessor#annotationTransformer" in 5 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.AutoAddScopeProcessor#annotationTransformer" in 4 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.ArcProcessor#initialize" in 37 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.steps.ConfigGenerationBuildStep#generateConfigClass" in 605 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.ArcProcessor#registerBeans" in 374 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.ConfigBuildStep#analyzeConfigPropertyInjectionPoints" in 1 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.ConfigBuildStep#generateConfigMappings" in 1 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.ConfigBuildStep#registerConfigMappings" in 6 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.staticmethods.InterceptedStaticMethodsProcessor#collectInterceptedStaticMethods" in 13 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.SyntheticBeansProcessor#initStatic" in 17 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.staticmethods.InterceptedStaticMethodsProcessor#processInterceptedStaticMethods" in 2 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.SyntheticBeansProcessor#initRegular" in 15 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.SyntheticBeansProcessor#initRuntime" in 9 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.ArcProcessor#registerSyntheticObservers" in 1 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.StartupBuildSteps#registerStartupObservers" in 4 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.ArcProcessor#validate" in 100 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.ArcProcessor#generateResources" in 172 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.staticmethods.InterceptedStaticMethodsProcessor#callInitializer" in 3 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.resteasy.reactive.common.deployment.ResteasyReactiveCommonProcessor#setupEndpoints" in 4 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.vertx.http.deployment.StaticResourcesProcessor#runtimeInit" in 3 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.ConfigBuildStep#validateConfigProperties" in 2 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.resteasy.reactive.server.deployment.ResteasyReactiveProcessor#setupEndpoints" in 86 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.vertx.http.deployment.VertxHttpProcessor#initializeRouter" in 3 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.resteasy.reactive.server.deployment.ResteasyReactiveProcessor#applyRuntimeConfig" in 3 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.vertx.http.deployment.VertxHttpProcessor#finalizeRouter" in 23 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.steps.ClassTransformingBuildStep#handleClassTransformation" in 2 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.arc.deployment.LifecycleEventsBuildStep#startupEvent" in 1 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.vertx.http.deployment.VertxHttpProcessor#openSocket" in 1 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.steps.ShutdownListenerBuildStep#setupShutdown" in 6 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.steps.ReflectiveHierarchyStep#build" in 23 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.steps.ReflectionDiagnosticProcessor#writeReflectionData" in 1 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.steps.MainClassBuildStep#build" in 115 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.pkg.steps.JarResultBuildStep#buildRunnerJar" in 227 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.pkg.steps.JarResultBuildStep#jarOutput" in 1 ms
[DEBUG] [io.quarkus.builder] Finished step "io.quarkus.deployment.pkg.steps.AppCDSBuildStep#build" in 0 ms
```
To know more details about what a specific buildsetp does, debugger can tell more things. All the BuildSteps are 
triggered in BuildContext, set a breakpoint in this class to figure out:

![debug-build-step](/img/buildstep/debug.png)

To look at the execution time of these generated byte code in ApplicationImpl, use this flag to start the runner jar file :
```java
java -Dquarkus.debug.print-startup-times=true -jar quarkus-run.jar
```
And all startup task will be printed out : 
```java
Build step LoggingResourceProcessor.setupLoggingStaticInit completed in: 9ms
Build step DevConsoleProcessor.hander completed in: 3ms
Build step VertxCoreProcessor.ioThreadDetector completed in: 4ms
Build step BlockingOperationControlBuildStep.blockingOP completed in: 0ms
Build step NativeImageConfigBuildStep.build completed in: 1ms
Build step JacksonProcessor.pushConfigurationBean completed in: 1ms
Build step StaticResourcesProcessor.staticInit completed in: 1ms
Build step SyntheticBeansProcessor.initStatic completed in: 1ms
Build step ArcProcessor.generateResources completed in: 182ms
Build step ResteasyReactiveProcessor.setupEndpoints completed in: 188ms
Build step BootstrapConfigSetupBuildStep.setupBootstrapConfig completed in: 0ms
Build step NettyProcessor.eagerlyInitClass completed in: 3ms
Build step RuntimeConfigSetupBuildStep.setupRuntimeConfig completed in: 64ms
Build step VertxHttpProcessor.cors completed in: 0ms
Build step ThreadPoolSetup.createExecutor completed in: 21ms
Build step HttpSecurityProcessor.initBasicAuth completed in: 1ms
Build step VertxCoreProcessor.build completed in: 1ms
Build step VertxCoreProcessor.eventLoopCount completed in: 1ms
Build step BannerProcessor.recordBanner completed in: 0ms
Build step ArcProcessor.setupExecutor completed in: 0ms
__  ____  __  _____   ___  __ ____  ______ 
 --/ __ \/ / / / _ | / _ \/ //_/ / / / __/ 
 -/ /_/ / /_/ / __ |/ , _/ ,< / /_/ /\ \   
--\___\_\____/_/ |_/_/|_/_/|_|\____/___/   
2021-03-01 18:14:21,387 WARN  [io.qua.config] (main) Unrecognized configuration key "quarkus.debug.print-startup-times" was provided; it will be ignored; verify that the dependency extension for this configuration is set or that you did not make a typo
Build step LoggingResourceProcessor.setupLoggingRuntimeInit completed in: 45ms
Build step VertxHttpProcessor.bodyHandler completed in: 5ms
Build step ConfigGenerationBuildStep.checkForBuildTimeConfigChange completed in: 11ms
Build step ConfigBuildStep.registerConfigMappings completed in: 1ms
Build step SyntheticBeansProcessor.initRuntime completed in: 1ms
Build step ConfigBuildStep.validateConfigProperties completed in: 0ms
Build step StaticResourcesProcessor.runtimeInit completed in: 6ms
Build step ResteasyReactiveProcessor.applyRuntimeConfig completed in: 1ms
Build step VertxHttpProcessor.initializeRouter completed in: 98ms
Build step VertxHttpProcessor.finalizeRouter completed in: 5ms
Build step LifecycleEventsBuildStep.startupEvent completed in: 0ms
Build step VertxHttpProcessor.openSocket completed in: 124ms
Build step ShutdownListenerBuildStep.setupShutdown completed in: 1ms
2021-03-01 18:14:22,174 INFO  [io.quarkus] (main) hello 1.0.0-SNAPSHOT on JVM (powered by Quarkus 999-SNAPSHOT) started in 1.120s. Listening on: http://0.0.0.0:8080
2021-03-01 18:14:22,175 INFO  [io.quarkus] (main) Profile prod activated. 
2021-03-01 18:14:22,175 INFO  [io.quarkus] (main) Installed features: [cdi, resteasy-reactive, resteasy-reactive-jackson]

```

Here the ResteasyReactiveProcessor.setupEndpoints is completely new generated method. It is set with all things the recorder 
requries and get it started. Here is the [decompiled code](https://gist.github.com/jimma/736646641c702db87a93e5d2e260f062), 
please check out. 



