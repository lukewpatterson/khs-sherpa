khsSherpa [![Build Status](https://secure.travis-ci.org/in-the-keyhole/khs-sherpa.png?branch=master)](http://travis-ci.org/in-the-keyhole/khs-sherpa)
==========

Remote java object JSON data framework

About
-----
Turn Java application servers into a remote JSON data access mechanism for mobile and HTML 5/Java Script applications. 

This lightweight server side framework allows Java classes contained inside a JEE application server
to become JSON end points that can be consumed via HTTP by native mobile devices or HTML/Javascript clients. 

Many MVC frameworks exist, but khsSherpa is intended to allow access to server side java objects with HTTP/and JSON. It 
also, provides session support for client applications that exist outside of a browser.

Examples
--------
Example Web application - https://github.com/in-the-keyhole/khs-sherpa-example-webapp

Working HTML5 JQuery Mobile - http://sherpa.keyholekc.com

HTML5 JQuery Mobile application on GitHub https://github.com/in-the-keyhole/khs-sherpa-jquery

Features  
--------
 * Annotation Based Configuration
 * Authentication and Role based permissions
 * JSONP Cross Domain Support 
 * Session Support 
 * Plug-gable User Activity Logging
 * Type mapping
 * XSS prevention support
 * Works with any JEE application server

Getting Started
---------------
To build it clone then use Maven:

    $ git clone ...
	$ cd khs-sherpa
	$ mvn install

Using Maven: add this dependency in your 'pom.xml' (available in Maven central repo)

    <dependency>
   	 <groupId>com.keyholesoftware</groupId>
   	 <artifactId>khs-sherpa</artifactId>
   	<version>1.1.3</version>
    </dependency>
   
Not using Maven: include following jars in lib class path

    khs-sherpa-1.1.3.jar
	gson-2.2.1.jar
	commons-lang3-3.1.jar
	
Quick Start 
----------
Configure and create Java server side end point in a WAR project

	
	1) Register Sherpa Servlet in WEB.XML (see configuring WEB.XML below)
	
	2) Create the following java class in a package named com.khs.example.endpoint
	
	@Endpoint(authenticated = false)
	public class TestService {
	
	// hello world  method
	public Result helloWorld() {
		return new Result("Hello World");
	}
		
		class Result {	
			public Result(Object o) {
			result = o;
			}
			public Object result;		
		}
	}
	 

	3) Create sherpa.properties file in your project resource/classpath folder and add this entry
	   
	endpoint.package=com.khs.example.endpoints
	   
	4) Start app server and in a browser enter the following URL.

	http://<server>/<webapp>/sherpa?endpoint=TestService&action=helloWorld	



Configuring WEB.XML
-------------------
Add the khsSherpa framework jar to your classpath/maven dependency list and add the 
SherpaServlet to the WEB-INF/web.xml as shown below. 

    <servlet>	
  		<servlet-name>sherpa</servlet-name>
		<display-name>sherpa</display-name>
		<servlet-class>com.khs.sherpa.servlet.SherpaServlet</servlet-class>	
	</servlet>
	<servlet-mapping>
		<servlet-name>sherpa</servlet-name>
		<url-pattern>/sherpa</url-pattern>
	</servlet-mapping>
	
	<servlet-mapping>
		<servlet-name>sherpa</servlet-name>
		<url-pattern>/sherpa/*</url-pattern>
	</servlet-mapping>
	

Endpoint Example
----------------
JSON endpoints are defined by annotation a Java class with the @Endpoint annotation. 
The java endpoint below has two methods that can be called remotely. 

    @Endpoint(authenticated = false)
	public class TestService {
	
	// hello world  method
	public Result helloWorld() {
		return new Result("Hello World");
	}
	
	// add two numbers method
	public Result add(@Param("x_value") double x, @Param("y_value") double y) {
		return new Result(x + y);
	}
		
		class Result {	
			public Result(Object o) {
			result = o;
			}
			public Object result;		
		}
	}

The @Param annotation is used to specify request parameters for an endpoint method. 

### Get/Post URL to access the TestService.helloWorld() java method is formatted in this manner 

	http://<server>/<webapp>/sherpa?endpoint=TestService&action=helloWorld
	     
### Get/Post URL to access the TestService.add(x,y) java method is formatted in this manner

	http://<server>/<webapp>/sherpa?endpoint=TestService&action=add&x_value=100&y_value=200
  
Configuring khsSherpa
---------------------
Define a sherpa.properties file in your webapp classpath. The only required entry is 
the endpoint.package entry, which tells sherpa where to find Java end points. 


    ##khsSherpa server properties

    #package where endpoints are located
    endpoint.package=com.khs.example.endpoints

Endpoint and Action Naming Conventions
-------------------------------------
By default, endpoint names are a classes simple name and action names are method names. 
Alternative names can be provided by specifying the value attribute of the @Endpoint annotation 
as shown below. 

	@Endpoint("HelloEndpoint")
	public class HelloWorld { 
	
Action names specified by applying the @Action annotation to a method implementation, as shown below. 

	@Action("hello")
	public String sayHello() {
	
The @Action annotation can also be specified to disable a method from execution with the disabled attribute. 

	@Action(disabled=true)
	public String myMethod() { 	
	
Restful based Action URLs
-------------------------
End point action methods can be mapped to a restful URL by applying the mapping attribute in the @Action 
annotation. An example restful mapping for the helloworld action is shown below

	@Action(mapping = "/helloworld", method = MethodRequest.GET)
	public String helloWorld() {		
		return "hello world";
	}

End point action an then be accessed with the following restful URL 

	http:<server>/<webapp>/sherpa/helloworld

Parameters can also be mapped to a restful URL by including a delimited parameter name in the mapping. 
The example below shows how a Vendor lookup id is mapped into the URL.
	
	@Action(mapping = "/findbyid/{id}", method = MethodRequest.GET)
	public Vendor fetchOne(@Param("id") Integer id) {
		return repository.findOne(id);
	}
	
Restful URL specifying a Vendor lookup id for the above action is shown below. 	
	
	http:<server>/<webapp>/sherpa/findbyid/100
	

Test Fixture
------------
A testing jsp, test-fixture.jsp has been created that will allow testing of khsSherpa end points, copy this 
file into your web contents web app directory, access the test-fixture.jsp with a browser and you will be able to invoke @Endpoint 
methods and view JSON results.  

Steps for testing json end points from a web app:

	create a java webapp project
	
	copy test-fixure.jsp into web content folder
	
	define a java class and annotate with @Endpoint, see above example
	
	define end point package name in sherpa.properties file
	
	start your webapp
	
	open text-fixture.jsp with this url 
	
		http://<server>/<webapp>/test-fixture.jsp
		
	Fill in end point simple class name, method name and parameter names and submit	


Authentication
--------------
End points can be configured to require authentication by setting the authentication attribute to true 
on the @Endpoint annotation. Authenticated end points can only be invoked if a valid user id and token
id is supplied. Valid token ids are obtained by invoking the framework authenticate action with valid 
credentials. 

An example authentication request URL is shown below. 

	http://<server>/<webapp>/sherpa?action=authenticate&userid=dpitt@keyholesoftware.com&password=password
         
If valid credentials, the following JSON token object will be returned. 

	{
	    "token": "1336103738643",
	    "timeout": 0,
	    "active": true,
	    "userid": "dpitt@keyholesoftware.com",
	    "lastActive": 1336103738643
	}	

The token id and userid values are supplied as parameters to @Endpoint method calls.
Authenticated URL's with token parameters will look like this...

	http://<server>/<webapp>/sherpa?endpoint=TestService&action=helloWord&userid=dpitt@keyholesoftware.com&token=1336103738643

Authentication Interface Implementation
---------------------------------------
The default authentication mechanism denies all credentials. Since various authentication mechanisms exist,
the framework supplies an interface, com.khs.sherpa.json.service.UserService. Concrete UserService implementations
are registered by defining the entry below in the sherpa.properties file.  

	## Authentication implementation
	user.service = com.example.LDAPUserService
	
Authentication implementations validate a user id and password and returns a list of valid roles for the user id. Roles 
are only required if role based authentication is applied.

Session Timeout
---------------

An authenticated user token will also define a timeout period in milliseconds. 0 indicates a session will never timeout
For timeout values greater than zero, the framework requires a new authentication token to be obtained through the 
authentication mechanism in order to continue to access an authenticated end point. Session timeouts can be set with 
the sherpa.properties file entry below. 

	## Session timeout (ms), default is 0 
	session.timeout=900000 

Role Based Security
-------------------
JEE javax.security roles can be applied to authenticated end point methods. These annotations are listed below. 
	
	javax.annotation.security.PermitAll
	javax.annotation.security.DenyAll
	javax.annotation.security.RolesAllowed
	
Declared roles for a user are applied to the authenticated users token. An example end point method that is 
restricted to only users with a "admin" role is shown below. 

	@RolesAllowed({"admin"})
	public Department create(@Param("number") int number,@Param("name")String name) {	
		Department dept = new Department();
		dept.number = number;
		dept.name = name;		
		return dept;		
	}	


Token Service
-------------
By default the framework supplies a default session token service implementation that maintains and in memory mapping of 
user tokens authenticated users. Token ids are generated by current milliseconds from the JVM. Sessions are active for specified
timeout periods and as long as the web application is started. 

If this default behavior is not sufficient, it can be replaced with an alternative implementation by implementing the framework 
supplied TokenService interface and registering it in the sherpa.properties file as shown below. 

JSONP Cross Domain Support
--------------------------
JSONP based URL's are supported by adding a callback parameter to a sherpa URL as shown below. 

	http://<server>/<webapp>/sherpa?endpoint=TestService&action=helloWord&callback=<function name> 

The framework will return a JSONP formatted result.

By default JSONP is disabled and the callback function will be ignored. To enable, set the sherpa.properties value shown below. 

	## Enable JSONP Support, default is false
	jsonp.support = true  

Data Type Mappings
------------------

khsSherpa maps request parameter types to java method argument types. The following mappings are applied 

	HTTP Request				Java
	
	Abc							String
	1.0							Float/Double/float/double
	1							Integer/Long/int/long
	0,1,y,n						Boolean/boolen
	mm/dd/yyyy					Date
	mm/dd/yyyy hh:mm:ss am		Calendar
	JSON String					Java Class Type
	
Date/Time format types can be changed framework wide by configuring the date.pattern or datetime.pattern in 
the sherpa.properties file. An example is shown below. 

	## Date format for date types, default is MM/dd/yyyy
	date.format=MM/dd/yyyy
	date.time.format=MM/dd/yyyy hh:mm:ss a

Date/Time  format types on an @Endpoint method level can be changed by specifying the format attribute on on @Param annotation
as shown below. 

	public Result time(@Param(value="cal", format="hh:mm:ss a") Calendar cal) {
		return new Result(cal);
	}

Encoding/XSS protection
-----------------------

End points with String parameter types can be automatically encoded to XML,HTML,or CSV formats. Encoding helps 
prevent XSS attacks from browser based clients. 

Encoding format for all String parameters can be enabled by setting the encode.format property in the 
sherpa.properties file as shown below. 

	encode.format = <possible values: HTML,XML,CSV>
	
Encoding can be applied at an end point action level by specifying the encoding format type in the
@Param annotation an example is shown below. 

	public Result encode(@Param(value="value",format=Encode.HTML) String value) {
		return new Result(value);	
	}

Activity Logging
----------------

By default end point execution will be logged via the java.util.logging.Logger. This can be turned off by setting 
the property below in sherpa.properties file. 

	acitivity.logging=false
	
An alternative logging implementation can be supplied and configured by implementing the com.khs.sherpa.ActivityService  
interface and registering in the sherpa.properties file as shown below. 

	activity.service.impl = <<qualified class name that implements com.khs.sherpa.service.ActivityService>>
 

Session Management Commands
---------------------------

Framework specific end point is available that will return active user sessions, and describe end point meta data. 
These actions must be invoked using the userid, passwords, and token for a user authenticated with the framework 
"SHERPA_ADMIN" role.

	Current Sessions
	
	http://<server>/<webapp>/sherpa?action=sessions&adminuserid=dpitt@keyholesoftware.com&adminpassword=password
	
  	Describe end point
  	
  	http://<server>/<webapp>/sherpa?action=describe&adminuserid=dpitt@keyholesoftware.com&adminpassword=password
  
  
  	Deactivate User id
  	
  	http://<server>/<webapp>/sherpa?action=deactivate&deactivate=jdoe@keyholesoftware.com&adminuserid=dpitt@keyholesoftware.com&adminpassword=password
  
  
  

  