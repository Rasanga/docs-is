# Re-branding the Default Login Page for Your Application

WSO2 Identity Server usually displays a default login page for all service provider applications that send authentication requests to it. WSO2 IS supports re-branding this login page according to your requirements for SAML2 SSO, OAuth, or OpenID Connect. 

The login pages and other pages, such as error and notification screens,
of SAML SSO, OAuth, OpenID Connect, and Passive STS are located in the
**authenticationendpoint** webapp file, which is found in the
`<IS_HOME>/repository/deployment/server/webapps` directory.

You can easily re-brand these pages within this web application by
changing the respective JSPs, JavaScript and CSS. If you want to point
to a different web application, you can do so by redirecting or
forwarding from **authenticationendpoint** to your webapp. In the case
of SAML SSO, the ' **issuer** ' id of the service provider is also sent
to this webapp. Therefore, different login pages can be given to
different service providers by looking at the ' **issuer** ' request
parameter.

**This page guides you through re-branding the login page for a sample SAML2 SSO web application. **

## Configuring the two service providers

First, register two service providers in WSO2 Identity Server. 

1.  [Deploy traveloctity](../../learn/deploying-the-sample-app/#deploying-the-travelocity-webapp) sample application.

2.  [Download a copy of `travlocity.com.war`](../../learn/deploying-the-sample-app/#deploy-the-sample-web-application)
    and place it in the `<TOMCAT_HOME>/webapps/` folder. 

3.  Rename the file to `avis.com.war` and restart the Tomcat server.

    !!! note 
        -   Open the ` <TOMCAT_HOME>/webapps/travelocity.com/travelocity.properties ` file and check the following
        configurations.
            ``` 
            #The URL of the SAML 2.0 Assertion Consumer
            SAML2.AssertionConsumerURL=http://wso2is.local:8080/travelocity.com/home.jsp
                
        
            #openid.return_to parameter
            OpenId.ReturnToURL=http://wso2is.local:8080/travelocity.com/home.jsp
            ```

        - Open the `<TOMCAT_HOME>/webapps/avis.com/avis.properties` file  and check the following configurations.
            ```
            #The URL of the SAML 2.0 Assertion Consumer
            SAML2.AssertionConsumerURL=http://wso2.is:8080/avis.com/home.jsp
        
            #openid.return_to parameter
            OpenId.ReturnToURL=http://wso2.is:8080/avis.com/home.jsp
            ```

4. Register a new service provider with the name `avis.com` similarly by following the steps given in the [configuring service provider for travelocity sample section](../../deploying-the-sample-app/#configuring-the-service-provider) while replacing `travelocity.com` references to `avis.com`.
   
4.  Start the application server and access the following URLs to make sure both apps are running.

    **travelocity.com**

    URL:
    [http://wso2is.local:8080/travelocity.com/index.jsp](http://localhost:8080/travelocity.com/index.jsp)

    ![Travelocity screen](../assets/img/using-wso2-identity-server/travelocity-screen.png) 

    **avis.com**

    URL:
    [http://wso2is.local:8080/avis.com/index.jsp](http://localhost:8080/avis.com/index.jsp)

    ![Avis screen](../assets/img/using-wso2-identity-server/avis-screen.png) 

## Registering the two service providers in WSO2 Identity Server

1.  Sign in to the WSO2 Identity Server [Management
    Console](../../setup/getting-started-with-the-management-console).
2.  On the **Main** menu, click **Identity \> Service Providers \>**
    **Add**.
3.  Enter `          travelocity.com         ` as the **Service Provider
    Name** in the form that appears and click **Register**. The Service
    Providers page appears.  
    ![Add new service provider](../assets/img/using-wso2-identity-server/add-new-service-provider.png) 
4.  Under the **Inbound Authentication Configuration** section, click
    **SAML2 Web SSO Configuration \> Configure**. The Register New
    Service Provider page appears.  
    ![SAML2 Web SSO Configuration option](../assets/img/using-wso2-identity-server/saml2-web-sso-configuration-option.png) 
5.  Configure the following details for travelocity.com and repeat steps
    1 to 6 and configure details for avis.com.

    ![New application screen](../assets/img/using-wso2-identity-server/new-application-screen-saml.png) 

    **travelocity.com**

    -   ***Issuer*** : travelocity.com **
    -   ***Assertion Consumer URL*** :
        http://wso2is.local:8080/travelocity.com/home.jsp
    -   Select ***Enable Response Signing***
    -   Select ***Enable Single Logout***

    **avis.com**

    -   ***Issuer*** : avis.com **
    -   ***Assertion Consumer URL*** :
        http://wso2is.local:8080/avis.com/home.jsp
    -   Select ***Enable Response Signing***
    -   Select ***Enable Single Logout***

6.  When attempting to "login with SAML from WSO2 Identity Server" in
    Travelocity.com and Avis.com, you can see the following default
    page located at
    `          <IS_HOME>/repository/deployment/server/webapps/authenticationendpoint/login.jsp         `
    .  
    ![Identity Server sign in screen](../assets/img/using-wso2-identity-server/identity-server-sign-in-screen.png) 

## Configuring the login page

### Understanding the authenticationendpoint web application

The login page that is displayed during SAML2 SSO, OAuth, OpenID Connect
and Passive-STS flows is located inside the webapp named
**authenticationendpoint**. The reason for storing this in a web app is:

-   to easily customize the page according to user requirements
-   if needed, place that whole web application in an external
    application server

WSO2 Identity Server is aware of the location of this web application as it is
specified with the following configurations in the `<IS_HOME>/repository/conf/deployment.toml` file. 

```
[authentication.endpoints] 
login_url="/authenticationendpoint/login.do"
retry_url="/authenticationendpoint/retry.do" 
```

By default, the configuration points to a location inside WSO2 Identity Server itself
thus, the relative path is given. If it is necessary to point to an
external application, the full path should be given instead.

!!! note
    
    If this web app is moved outside the Identity Server, ensure
    that no one can access the login credentials that are passed between
    this application and the Identity Server. This means that the external
    location should ideally be either inside a secured intranet or the
    transport should be HTTPS. Other similar precautions may be necessary to
    secure the communication.
    

The following is the structure of this web app.

![Web app folder strcture](../assets/img/using-wso2-identity-server/web-app-folder-strcture.png)

The **authenticationendpoint** web application uses a carbon component
called
`         org.wso2.carbon.identity.application.authentication.endpoint.util        `
. This bundle includes a filter called the
`         org.wso2.carbon.identity.application.authentication.endpoint.util.filter        `
. `         AuthenticationEndpointFilter        `, `        ` which
acts as the Front Controller.

When a request is made to the **authenticationendpoint** web
application, based on the authentication protocol type identified by the
request parameter ‘type’, the controller first forwards the request to
the protocol based login url patterns defined. For example, if the
request to the **authenticationendpoint** web application is initiated
as a result of a SAML SSO authentication request, the controller will
forward the request to the url pattern
`         /samlsso_login.do        ` . If you look inside the
**web.xml,** you will see that this url pattern is mapped to the
**login.jsp** file. The request is finally forwarded to this
**login.jsp** page.

Everything on the **authenticationendpoint** web application can be modified according to your requirements. You can either add JSP pages or modify them and configure the **web.xml** respectively.

The only restriction involved is that the content already sent back by
the pages inside the default web app must be submitted to the Identity
Server. Additionally, you must point to the correct location via the
`         <IS_HOME>/repository/conf/identity/application-authentication.xml        `
file.

### Re-branding the login page

When a request comes to the default login page, you will see several parameters being passed in the address bar. For this example, the
focus is on the following two parameters:

-   **sessionDataKey** : This is an identifier used by the Identity
    Server to maintain state information related to this particular
    request by the service provider.
    !!! note
        The 'sessionDataKey' query parameter is used to coordinate the request state across components participating in the request flow. It does not correlate with the user session. Furthermore, the request state maintained against the 'sessionDataKey' parameter value is cleared by each participating component at the end of request flow. This means that even if an external party grabs the 'sessionDataKey' they will not be able to get into the authentication sequence, as the user session is not associated with that key.

 -   **relyingParty** : This is the value we gave for the "Issuer" field
    when we registered the SAML2 SSO service provider (e.g.,travelocity.com). This value is used to
    display different login pages to different service providers.

When modifying the pages, ensure that the following is applied.

1.  Form submissions should happen to the "commonauth" servlet as a
    POST.

    ``` xml
    <form id="form" name="form" action="../../commonauth" method="POST"> 
    ```

2.  Make sure to send back the "sessionDataKey" with the form
    submission, by using a hidden input field.

    ``` xml
    <%@ page import="org.owasp.encoder.Encode" %>

    <input type="hidden" name="sessionDataKey" value="<%=Encode.forHtmlAttribute(request.getParameter("sessionDataKey"))%>"/>
    ```

#### **Using a JSP to redirect to SP relevant pages**

1.  Rename the existing 'login.jsp' to 'default\_login.jsp'
2.  Create a new file with the name 'login.jsp' including the following
    code.

    ``` java
    <%  
    String relyingParty = request.getParameter("relyingParty");

    if (relyingParty.equals("travelocity.com")) {
        RequestDispatcher dispatcher = request.getRequestDispatcher("travelocity_login.jsp");
        dispatcher.forward(request, response);
    } else {
        RequestDispatcher dispatcher = request.getRequestDispatcher("default_login.jsp");
        dispatcher.forward(request, response);
    } 
        %>
    ```

    What this code basically does is it forwards the request to a
    different login page by checking the value of
    `           relyingParty          ` parameter.

3.  Get the 'travelocity\_login.jsp' from
    [here](https://svn.wso2.org/repos/wso2/people/malithim/samples/is-510/authenticationendpoint-customize-login-per-sp/)
    and place it at the same level as 'login.jsp'. Also, download the
    contents of the 'css' and 'images' folders from that same link and
    put them inside the respective folders in the
    authenticationendpoint.

4.  Log in to the travelocity.com web app
    again. You are presented with the re-branded page.  
    ![Travelocity login screen](../assets/img/using-wso2-identity-server/travelocity-login-screen.png)   
    If you access avis.com, it still displays the
    default login page.
