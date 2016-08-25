---
layout:     post
title:      "Wildfly Elytron and Webservice Integration"
subtitle:   ""
date:       2016-08-24 12:00:00
author:     "Jim Ma"
header-img: "img/post-bg-06.jpg"
---
In this post, we'll go through the new security change in coming wildfly like Elytron, Elytron subsystem and what will this change effect webservice and resteasy subsystem. This document is initial to collect the ideas and comments for this change.  In the following sections, this post will cover:

 - Pickebox
 - Realms and Domains
 - Wildfly Elytron
 - Undertow and Elytron integration
 - Webservice and Elytron
 
## Pickebox

Picketbox (formerly JBoss Security) is the foundational security framework that provides the authentication, authorization, audit and mapping capabilities to Java applications. It provides these abilities : 
  1. Authentication
  2. Authorization/Access Control
  3. Auditing
  4. Mapping (Principal/Roles/Attribute)
In jbossws webservice integration, SecurityDomainContext is retrieved to  AuthenticationManager and AuthorizationManager to do security check:

```java
package org.jboss.as.webservices.security;

import java.security.AccessController;
import java.security.Principal;
import java.security.PrivilegedAction;
import java.util.Set;

import javax.security.auth.Subject;

import org.jboss.as.security.plugins.SecurityDomainContext;
import org.jboss.security.SecurityContext;
import org.jboss.security.SecurityContextAssociation;
import org.jboss.security.SecurityContextFactory;
public final class SecurityDomainContextAdaptor implements org.jboss.wsf.spi.security.SecurityDomainContext {

    private final SecurityDomainContext context;

    public SecurityDomainContextAdaptor(SecurityDomainContext context) {
        this.context = context;
    }

    @Override
    public boolean isValid(Principal principal, Object credential, Subject activeSubject) {
        return context.getAuthenticationManager().isValid(principal, credential, activeSubject);
    }

    @Override
    public boolean doesUserHaveRole(Principal principal, Set<Principal> roles) {
        return context.getAuthorizationManager().doesUserHaveRole(principal, roles);
    }

    @Override
    public String getSecurityDomain() {
        return context.getAuthenticationManager().getSecurityDomain();
    }

    @Override
    public Set<Principal> getUserRoles(Principal principal) {
        return context.getAuthorizationManager().getUserRoles(principal);
    }

    @Override
    public void pushSubjectContext(final Subject subject, final Principal principal, final Object credential) {
        AccessController.doPrivileged(new PrivilegedAction<Void>() {

            public Void run() {
                SecurityContext securityContext = SecurityContextAssociation.getSecurityContext();
                if (securityContext == null) {
                    securityContext = createSecurityContext(getSecurityDomain());
                    setSecurityContextOnAssociation(securityContext);
                }
                securityContext.getUtil().createSubjectInfo(principal, credential, subject);
                return null;
            }
        });
    }

    /**
     * Create a JBoss Security Context with the given security domain name
     *
     * @param domain the security domain name (such as "other" )
     * @return an instanceof {@code SecurityContext}
     */
    private static SecurityContext createSecurityContext(final String domain) {
        return AccessController.doPrivileged(new PrivilegedAction<SecurityContext>() {

            @Override
            public SecurityContext run() {
                try {
                    return SecurityContextFactory.createSecurityContext(domain);
                } catch (Exception e) {
                    throw new RuntimeException(e);
                }
            }
        });
    }

    /**
     * Set the {@code SecurityContext} on the {@code SecurityContextAssociation}
     *
     * @param sc the security context
     */
    private static void setSecurityContextOnAssociation(final SecurityContext sc) {
        AccessController.doPrivileged(new PrivilegedAction<Void>() {

            @Override
            public Void run() {
                SecurityContextAssociation.setSecurityContext(sc);
                return null;
            }
        });
    }
}
```
We can directly create AuthenticationManager with the securityDomainname and JBossCallbackHandler :

```java
authenticationManger = new JBossAuthenticationManager(securityDomain, new JBossCallbackHandler());    
```
Create JBossAuthorizationManager with  securityDomain :

```java
authorizationManager = new JBossAuthorizationManager(securityDomain);
```

PicketBox(jboss security) is there for couple of years and it works well. The configuration in wildfly standalone.xml is simple : 

```xml
        <subsystem xmlns="urn:jboss:domain:security:3.0">
            <security-domains>
                <security-domain name="other" cache-type="default">
                    <authentication>
                        <login-module code="Remoting" flag="optional">
                            <module-option name="password-stacking" value="useFirstPass"/>
                        </login-module>
                        <login-module code="RealmDirect" flag="required">
                            <module-option name="password-stacking" value="useFirstPass"/>
                        </login-module>
                    </authentication>
                </security-domain>
                <security-domain name="jboss-web-policy" cache-type="default">
                    <authorization>
                        <policy-module code="Delegating" flag="required"/>
                    </authorization>
                </security-domain>
                <security-domain name="jboss-ejb-policy" cache-type="default">
                    <authorization>
                        <policy-module code="Delegating" flag="required"/>
                    </authorization>
                </security-domain>
                <security-domain name="jaspitest" cache-type="default">
                    <authentication-jaspi>
                        <login-module-stack name="dummy">
                            <login-module code="Dummy" flag="optional"/>
                        </login-module-stack>
                        <auth-module code="Dummy"/>
                    </authentication-jaspi>
                </security-domain>
            </security-domains>
        </subsystem>
```

All these configuration is use to understand, and it uses *loginModule* to configure all the security check for a security domain. But these two domains are exceptional:*jboss-web-policy* and *jboss-ejb-policy*. 

From  [Forum message][1] and [AS issues][2], these profiles are required for correct operation of authorization. The profiles are used internally by picketbox.

## Realms  and Domains

Realms are used to security the transport layer in wildfly and security domains are for application. When we configure the security for web or ejb application, we always configure the security domain instead of security realms.

Josef Cacek explains this very well:

>The Security domains are used mainly for defining security of deployed applications. The standard authentication in security domains is based on JAAS javax.security.auth.spi.LoginModule implementations. Application can come up with custom login module(s).

>The Security realms are used mainly for configuration security of server management interfaces and remoting. The realm authentication is based on provided implementations of javax.security.auth.callback.CallbackHandler. AFAIK it's not possible to provide own CallbackHandler implementation.

>A security domain can delegate authentication to a security realm by using the "RealmDirect" login module.

>A security realm can delegate authentication to a security domain by using "jaas" authentication configuration

Darran Lofthouse gives more technical differential details :

>The difference between the two is that for a security domain you present a username and a credential to the domain for verification during the authentication process - this means that this information needs to be available at the time an authentication check is going to be performed. 
 
>The security realms on the other hand are implemented as CallbackHandlers handling various Callbacks, this means that it is no longer a case of present everything we know and get an authentication decision instead we can now integrate the authentication process at the transport level much more closely with the user stores without those stores needing to be aware of the details of the transport.
 
>One example is the difference between how BASIC and DIGEST authentication can be used in the two approaches.  For security domains you end up with some of the Digest process living in a JBoss Web Authenitcator and then some of the HTTP specific items for verification need to live in the login module, if you want to add a new login module and still use DIGEST authentication then your new login module needs to be aware of HTTP Digets authentication.  The realms on the other hand just respond to callbacks based on a supplied username and return either the users password or a hash of their password allowing the transport specific checks to be made without the access to the user repository now needing to know about HTTP Digest - in addition this same realm implementation is then compatible with Remoting with uses SASL for authentication - to achieve this in security domains would require login modules to now be aware of two different transports in use.

## Wildfly Elytron

Wildfy Elytron is a new picketbox replacement and it provides the security manager, a lot of SPI to allow easy integrate with undertow, ejb. To find more about features please go to [Elytron Features][5]

These two classes shows what Elytron can provider : 

```java
   public WildFlyElytronProvider() {
       super("WildFlyElytron", 1.0, "WildFly Elytron Provider");       
       putHttpAuthenticationMechanismImplementations();
        putKeyStoreImplementations();
        putPasswordImplementations();
        putSaslMechanismImplementations();
        putCredentialStoreProviderImplementations();
    }
```
```java
@MetaInfServices(SecurityManager.class)
public final class WildFlySecurityManager extends SecurityManager implements PermissionVerifier {
}
```
As stated in elytron's project summary, it provides the classes and apis that undertow and other system can directly uses. Here I list some important Elytron classes that involves in a http basic authetnication: 
### org.wildfly.security.http.HttpAuthenticator

```java
public class HttpAuthenticator {
    private final Supplier<List<HttpServerAuthenticationMechanism>> mechanismSupplier;
    private final HttpExchangeSpi httpExchangeSpi;
    private final boolean required;
    private final boolean ignoreOptionalFailures;
    private volatile boolean authenticated = false;

    private HttpAuthenticator(final Supplier<List<HttpServerAuthenticationMechanism>> mechanismSupplier, final HttpExchangeSpi httpExchangeSpi,
                              final boolean required, final boolean ignoreOptionalFailures) {
        this.mechanismSupplier = mechanismSupplier;
        this.httpExchangeSpi = httpExchangeSpi;
        this.required = required;
        this.ignoreOptionalFailures = ignoreOptionalFailures;
    }

    /**
     * Perform authentication for the request.
     *
     * @return {@code true} if the call should be allowed to continue within the web server, {@code false} if the call should be
     *         returning to the client.
     * @throws HttpAuthenticationException
     */
    public boolean authenticate() throws HttpAuthenticationException {
        return new AuthenticationExchange().authenticate();
    }
    ....
     private class AuthenticationExchange implements HttpServerRequest, HttpServerResponse {

        private volatile HttpServerAuthenticationMechanism currentMechanism;

        private volatile boolean authenticationAttempted = false;
        private volatile int statusCode = -1;
        private volatile boolean statusCodeAllowed = false;
        private volatile List<HttpServerMechanismsResponder> responders;
        private volatile HttpServerMechanismsResponder successResponder;

        private boolean authenticate() throws HttpAuthenticationException {
            List<HttpServerAuthenticationMechanism> authenticationMechanisms = mechanismSupplier.get();
            responders = new ArrayList<>(authenticationMechanisms.size());
            try {
                for (HttpServerAuthenticationMechanism nextMechanism : authenticationMechanisms) {
                    currentMechanism = nextMechanism;
                    nextMechanism.evaluateRequest(this);

                    if (isAuthenticated()) {
                        if (successResponder != null) {
                            statusCodeAllowed = true;
                            successResponder.sendResponse(this);
                            if (statusCode > 0) {
                                httpExchangeSpi.setStatusCode(statusCode);
                                return false;
                            }
                        }
                        return true;
                    }
                }
```

HttpAuthenticator basically get a HttpServerAuthenticationMechanism list to check request by calling evaluateRequest() method. For example, simple username password verification is handled by UsernamePasswordAuthenticationMechanism:
![UsernamePasswordAuthenticationMechanism](https://raw.githubusercontent.com/jimma/images/master/usernamepwd.png)

### org.wildfly.security.auth.server.AbstractMechanismAuthenticationFactory
This class is responsible for create SecurityMechanism. It requires a securityDomain to create mechanism:
![enter image description here](https://github.com/jimma/images/blob/master/securitymechanism.png?raw=true)
In createMetchnism() method, there is ServerAuthenticationContext is instantiated and callBackhandler is created to handle all callbacks in mechanism.

AbstractMechanismAuthenticationFactory is the class to put SecurityDomain, realms and mechanisms together. In Elytron, there are two authenticationFactory: 
 
  * HttpAuthenticationFactory
  * SaslAuthenticationFactory

Factory class gets everything prepared and get the configured securityMechanisms to authenticate and authorize each requests. 
As now, we can understand the each configuration item in wildfly-elytron.xml:

```xml
        <subsystem xmlns="urn:wildfly:elytron:1.0">
            <security-domains>
                <security-domain name="ApplicationDomain" default-realm="ApplicationRealm" permission-mapper="login-permission-mapper" role-mapper="combined-role-mapper">
                    <realm name="ApplicationRealm" role-decoder="groups-to-roles"/>
                </security-domain>
                <security-domain name="ManagementDomain" default-realm="ManagementRealm" permission-mapper="login-permission-mapper" role-mapper="combined-role-mapper">
                    <realm name="ManagementRealm" role-decoder="groups-to-roles"/>
                </security-domain>
            </security-domains>
            <security-realms>
                <properties-realm name="ApplicationRealm">
                    <users-properties path="application-users.properties" relative-to="jboss.server.config.dir"/>
                    <groups-properties path="application-roles.properties" relative-to="jboss.server.config.dir"/>
                </properties-realm>
                <properties-realm name="ManagementRealm">
                    <users-properties path="mgmt-users.properties" relative-to="jboss.server.config.dir"/>
                    <groups-properties path="mgmt-groups.properties" relative-to="jboss.server.config.dir"/>
                </properties-realm>
            </security-realms>
            <mappers>
                <simple-permission-mapper name="login-permission-mapper">
                    <permission-mapping roles="All">
                        <permission class-name="org.wildfly.security.auth.permission.LoginPermission"/>
                    </permission-mapping>
                </simple-permission-mapper>
                <simple-role-decoder name="groups-to-roles" attribute="groups"/>
                <constant-role-mapper name="constant-roles" roles="All"/>
                <logical-role-mapper name="combined-role-mapper" logical-operation="OR" left="constant-roles"/>
            </mappers>
            <http>
                <http-authentication-factory name="management-http-authentication" http-server-mechanism-factory="global" security-domain="ManagementDomain">
                    <mechanism-configuration>
                        <mechanism mechanism-name="BASIC">
                            <mechanism-realm realm-name="Management Realm"/>
                        </mechanism>
                    </mechanism-configuration>
                </http-authentication-factory>
                <http-authentication-factory name="application-http-authentication" http-server-mechanism-factory="global" security-domain="ApplicationDomain">
                    <mechanism-configuration>
                        <mechanism mechanism-name="BASIC">
                            <mechanism-realm realm-name="Application Realm"/>
                        </mechanism>
                        <mechanism mechanism-name="FORM"/>
                    </mechanism-configuration>
                </http-authentication-factory>
                <provider-http-server-mechanism-factory name="global"/>
            </http>
            <sasl>
                <sasl-authentication-factory name="management-sasl-authentication" sasl-server-factory="global" security-domain="ManagementDomain"/>
                <sasl-authentication-factory name="application-sasl-authentication" sasl-server-factory="global" security-domain="ApplicationDomain"/>
                <provider-sasl-server-factory name="global"/>
            </sasl>
        </subsystem>
```

## Undertow and Elytron integration

To isolate the different security implementation , undertow defines the securityContext interface for undertow to use. This also allows Security for undertow is plugable. The underling security implementation can be picketbox or Elytron.
 
In undertow subsystem, when it meets the Elytron security configuration element **application-security-domain** in wildfly startup configuration : 

```xml
        <subsystem xmlns="urn:jboss:domain:undertow:4.0">
            <buffer-cache name="default"/>
            <server name="default-server">
                <http-listener name="default" redirect-socket="https" socket-binding="http" enable-http2="true"/>
                <https-listener name="https" socket-binding="https" security-realm="ApplicationRealm" enable-http2="true"/>
                <host name="default-host" alias="localhost">
                    <location name="/" handler="welcome-content"/>
                    <filter-ref name="server-header"/>
                    <filter-ref name="x-powered-by-header"/>
                </host>
            </server>
            <servlet-container name="default">
                <jsp-config/>
                <websockets/>
            </servlet-container>
            <handlers>
                <file name="welcome-content" path="${jboss.home.dir}/welcome-content"/>
            </handlers>
            <filters>
                <response-header name="server-header" header-value="WildFly/10" header-name="Server"/>
                <response-header name="x-powered-by-header" header-value="Undertow/1" header-name="X-Powered-By"/>
            </filters>
            <application-security-domains>
                <application-security-domain security-domain="other" http-authentication-factory="application-http-authentication"/>
            </application-security-domains>
        </subsystem>
```

This will create the *ApplicationSecurityDomainService* and add *ElytronContextAssociationHandler* to secure http request. 
*ElytronContextAssoicationHandler* is place to bind securityMechanisms for undertow and create the SecurityContext: 

```java
 public class ElytronContextAssociationHandler extends AbstractSecurityContextAssociationHandler {
    private final Supplier<List<HttpServerAuthenticationMechanism>> mechanismSupplier;
    private final Function<HttpServerExchange, ElytronHttpExchange> httpExchangeSupplier;

    private ElytronContextAssociationHandler(Builder builder) {
        super(checkNotNullParam("next", builder.next));
        this.mechanismSupplier = checkNotNullParam("mechanismSupplier", builder.mechanismSupplier);
        this.httpExchangeSupplier = checkNotNullParam("httpExchangeSupplier", builder.httpExchangeSupplier);
    }

    /**
     * Create a new Elytron backed {@link SecurityContext}.
     */
    @Override
    public SecurityContext createSecurityContext(HttpServerExchange exchange) {
        return SecurityContextImpl.builder()
                .setExchange(exchange)
                .setMechanismSupplier(mechanismSupplier)
                .setHttpExchangeSupplier(this.httpExchangeSupplier.apply(exchange))
                .build();
    }
```

Here you can see, with different subsystem configuraition , and different ContextAssoicationHandler to put in undertow server. This is the other advanced thing in wildfly/undertow design. 
  
## Webservice and Elytron

 - In webservice, there is SecurityDomainContext interface for the same purpose with undertow's SecurityContext. This requires revise and implement the securityContext for Elytron. If it requires plugable in wildfly, we may need to look at how to get undertow's securityContext .
 - in jaspi, the only thing we can get is the security domain name , then we need to secure the soap request:

```java
authenticationManger = new JBossAuthenticationManager(securityDomain, new JBossCallbackHandler());
```

   In Elytron,it would be better if there is some utility class we can use to easily integrate with Elytron. 
 
[1]: https://developer.jboss.org/message/752117    "Forum message"
[2]: https://issues.jboss.org/browse/AS7-5319 "AS Issues" 
[3]: https://docs.jboss.org/author/display/WFLY/Security+Realms "Security Realms"
[4]: https://docs.jboss.org/author/display/WFLY/Security+subsystem+configuration "Security subystem"   
[5]:https://developer.jboss.org/wiki/WildFlyElytron-ProjectSummary  "Elytron Features"                               
