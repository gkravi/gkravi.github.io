---
title: AEM Dispatcher Setup
layout: post
summary: Learn About AEM Dispatcher
author: Ravi Gupta
date: '2020-05-03 09:31:02'
category: AEM
thumbnail: code.jpg
---

# AEM Dispatcher installation:

**Step:1 Open the url https://www.adobeaemcloud.com/content/companies/public/adobe/dispatcher/dispatcher.html**

And download AEM dispatcher dispatcher-apache2.2-windows-x86-4.2.3.zip then unzip it in local folder.
It will contain:
	• disp_apache2.2.dll – This is my dispatcher module file, which we will plug in with web server.
	• dispatcher.any – It is our dispatcher configuration file.
	• author_dispatcher.any – Sample file for configuring our dispatcher.
	• httpd.conf.disp2.conf – sample file to configure apache webserver.


**Step:2 Now got to Apache installation directory C:\Program Files (x86)\Apache Software Foundation\Apache2.2\conf and open httpd.conf file.**

**Step:3 Open you httpd.conf.disp2 also copy below setting from this file to httpd.conf file**

	· Copy LoadModule dispatcher_module  modules/mod_dispatcher.so at (line 234) and paste anywhere in httpd.conf file, paste it at (line 130) where my load modules are ending.
	Note:This setting is used by apache webserver to load the dispatcher when next time Apache web server is started.
	· As my dispatcher module name is disp_apache2.2.dll. I will change modules/mod_dispatcher.so  in above line to modules/disp_apache2.2.dll.
	· Now copy the dispatcher level setting from httpd.conf.disp2.conf (Line 236 -284) as shown below and paste it to httpd.conf file after ending of </IfModule> tag. I am pasting it at (line 146) where my if module is ending. This setting consist of:
		• DispatcherConfig – location of the configuration file
		• DispatcherLog– location of the dispatcher log file
		• DispatcherPassError – to use your dispatcher for handling errors, if set to 1 then webserver will handle errors.
		• DispatcherKeepAliveTimeout – Time interval for which your request should be kept alive.(in seconds, zero disables the keep-alive)
		• SetHandler - to activate the dispatcher LoadModule.
		• ModMimeUsePathInfo to configure behavior of mod_mime. The mod_mime module is used to assign content metadata to the content selected for an HTTP response. When On , the ModMimeUsePathInfo parameter specifies that mod_mime is to determine the content type based on the complete URL; this means that virtual resources will have metainformation applied based on their extension.
		• DispatcherLogLevel : (0|1|2|3)
			Log level for the log file:
				• 0 - Errors
				• 1 - Warnings
				• 2 - Infos
				• 3 - Debug
				
		• DispatcherNoServerHeader - This parameter is deprecated and no longer has any effect. If set to Off then the http header won't get processed, else it will be sent on to directly AEM server.
		
		• DispatcherDeclineRoot - If turned On then it will decline requests to the root "/" (like, http://localhost:4503/)  are not handled by the dispatcher; use mod_alias for the correct mapping. This doesn't work in Apache it works in IIS.
		
		• DispatcherUseProcessedURL - Defines whether to use pre-processed URLs for all further processing by Dispatcher: 
		
			§ 0 - use the original URL passed to the web server.
			§ 1 - the dispatcher uses the URL already processed by the handlers that precede the dispatcher (i.e. mod_rewrite )
		• DispatcherPassError - Defines how to support error codes for ErrorDocument handling:
			§ 0 - Dispatcher spools all error responses to the client.
			§ 1 - Dispatcher does not spool an error response to the client (where the status code is greater or equal than 400)
			
		• DispatcherNoCanonURL - Setting this parameter to On will pass the raw URL to the backend instead of the canonicalised one and will override the settings of DispatcherUseProcessedURL. The default value is Off.
		
		• SetHandler - SetHandler statement to the context of your configuration ( <Directory> , <Location> ) for the Dispatcher to handle the incoming requests.
		```
					<Directory />  
					<IfModule disp_apache2.c>  
					SetHandler dispatcher-handler  
					</IfModule>  
					  
					Options FollowSymLinks  
					AllowOverride None  
					</Directory>  
```

Note : The filter rules in the Dispatcher configuration will always be evaluated against 
the sanitised URL not the raw URL.

Dispatcher specific configuration entries should look like this:
```		 
<IfModule disp_apache2.c>

DispatcherConfig conf/dispatcher.any
DispatcherLog logs/dispatcher.log DispatcherLogLevel 3
DispatcherNoServerHeader 0 DispatcherDeclineRoot 0
DispatcherUseProcessedURL 0
DispatcherPassError 0
DispatcherKeepAliveTimeout 60

</IfModule>
```

	
Note: Do not paste this if module under any existing if module.

**Step:4 Now copy the dispatcher handler from httpd.conf.disp2.conf (line 370) to httpd.conf file (line 238).**

It will appear like below:
```
	<Directory />
	    Options FollowSymLinks
	    AllowOverride None
	    Order deny,allow
	    Deny from all
	</Directory>
	<Directory />
	    <IfModule disp_apache2.c>
	        ModMimeUsePathInfo On
	        # enable dispatcher for ALL request. if this is too restrictive,
	        # move it to another location
	        SetHandler dispatcher-handler
	    </IfModule>
	
	    Options FollowSymLinks
	    AllowOverride None
	</Directory>
```
	
	

**Step:5 Now copy this disp_apache2.2.dll  file from downloaded dispatcher and paste it under C:\Program Files (x86)\Apache Software Foundation\Apache2.2\modules.**

**Step:6 Now copy dispatcher.any file from downloaded dispatcher and paste it under C:\Program Files (x86)\Apache Software Foundation\Apache2.2\conf directory.**

**Step:7 Open dispatcher.any file and make few modification to make our dispatcher up and running.**

	· Update /renders section: Go to line 31 and update renders section to point our publish instance. Update the hostname(to 127.0.0.1 for local instance or remote IP) if required currently it is pointing to localhost and port to 4503 (i.e. publish instance port).
	· Update /filter section: Go to line 77 and instead of denying all request lets allow all request for testing. For Publish instance you should deny all request and allow specific , For author instance you should allow all and deny specific.
```
/filter
			{
			# Deny everything first and then allow specific entries
			/0001 { /type "allow" /glob "*" }
```


	· Update /cache section: /docroot. It should exactly match my httpd.conf DocumentRoot path. Copy Document root path from httpd.conf file and paste it in dispatcher.any file at line 132.
	 C:/Program Files (x86)/Apache Software Foundation/Apache2.2/htdocs
	· Save all settings and Restart your Apache Web server in services.msc.


# Test Dispatcher installation:

	· Go to dispatcher.log and you can see Dispatcher initialized in log file. It means our dispatcher is up.

	• To check it is correctly plug in with our publish instance. Simply hit published page.
	http://localhost:81 it will redirect to the home page of the publish AEM site. Apache will serve the http://localhost:4503 via port 81 where Apache dispatcher is hosted.
	
	• Also check if the page browsed are cached in docroot folder defined in the dispatcher.any file. If not then go through below troubleshooting steps.

# Troubleshooting:

By default, requests that carry an authentication header are not cached. This is because authentication is not checked when a document is returned from the cache - so the document would be displayed for a user who does not have the necessary rights. However, in some setups it can be permissible to cache authenticated documents.

Workaround: There is a setting in dispatcher.any file /allowAuthorized "1" if set to "1" then authentication header of page will be ignored and pages will be cached without the auth header, which is not recommended setting for prod dispatcher configuration.