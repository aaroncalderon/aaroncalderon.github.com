# Deploy (electrode) app with iisnode

https://github.com/electrode-io/electrode/issues/752

I was able to deploy the sample app with the following configuration:

The configuration below is set for the node app to recide at the root of the site, and on port 80

## Steps

1.  Cereate a file on the root of your IIS site called `app.js`
    - You can choose the name as you wish, just make shure you update the `web.config` file to reflect your new file name. I used this approach because I was having issues with paths and include commands otherwise
        - [Nested Server](https://github.com/tjanczuk/iisnode/issues/338)
        - [Point iisnode at server.js file nested in folder in an iis website
](https://stackoverflow.com/a/23520004/3991654)
2. Configure your `web.config`. Sample is shown below.
3. ~~`/lib/server/express-server.js`~~ `/config/default.js` Make shure you tell your app to listhen to the right __PIPE__. The default configuration will cause issues and will always use an integer as defined on by the `defaultListenPort`.

## /app.js

```js
require(__dirname + '\\lib\\server\\index.js');
```

## /web.config

```xml
<configuration>
  <system.webServer>

    <!-- SAMPLE 

    Indicates that the app.js file is a node.js application 
    to be handled by the iisnode module -->

    <handlers>
      <add name="iisnode" path="app.js" verb="*" modules="iisnode" />
    </handlers>

    <!-- use URL rewriting to redirect the entire branch of the URL namespace
    to app.js node.js application; 
        
    Additional Configuration Ideas
    https://tomasz.janczuk.org/2012/05/yaml-configuration-support-in-iisnode.html
    -->
    
    <rewrite>
      <rules>
          <rule name="LogFile" patternSyntax="ECMAScript" stopProcessing="true">  
                <match url="iisnode"/>  
          </rule>  
          <rule name="NodeInspector" patternSyntax="ECMAScript" stopProcessing="true">                      
              <match url="^/app.js\/debug[\/]?" />  
          </rule>  
          <rule name="StaticContent">  
                <action type="Rewrite" url="{{REQUEST_URI}}"/>  
          </rule>  
          <rule name="DynamicContent">  
                <conditions>  
                    <add input="{{REQUEST_FILENAME}}" matchType="IsFile" negate="True"/>  
                </conditions>  
                <action type="Rewrite" url="app.js"/>  
          </rule>
      </rules>
    </rewrite>
        <!-- 
        The key elements are:
          maxNamedPipeConnectionRetry="100" 
          namedPipeConnectionRetryDelay="250"  
        -->
    <iisnode
        maxNamedPipeConnectionRetry="100"
        namedPipeConnectionRetryDelay="250"
    />
  </system.webServer>
</configuration>
```

## /config/default.js

Look for the line that reads 

```js
const defaultListenPort = 3000;
```

Change it to: 
```js
const defaultListenPort = process.env.PORT ||  3000;
```

~~I tryed addressing this issue on the `/config/defaults.js` file but had no success.~~
