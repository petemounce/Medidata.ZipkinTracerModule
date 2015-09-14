# Medidata.ZipkinTracerModule
A .NET implementation of the Zipkin Tracer client.

## Overview
This nuget package implements the zipkin tracer client for .net applications.  This sln produces 2 nuget packages.

1) Medidata.ZipkinTracer.Core : core library for generating zipkin spans from ids sent through from CrossApplicationTracer and sending it to the zipkin collector using thrift protocol

2) Medidata.ZipkinTracer.HttpModule : httpModule injects zipkin trace into every request which has a zipkin traceId and isSampled of true in the header. Includes config transforms that automatically modifies the config for zipkin to work.

For more information and implementations in other languages, please check [Openzipkin](https://github.com/openzipkin/).

### Enable/Disable zipkin tracing

Zipkin relies on CrossApplicationTracer library (https://github.com/mdsol/Medidata.CrossApplicationTracer)'s TraceProvider to retrieve ids which are passed through the http request headers.

Zipkin will record traces if TraceProvider's IsSampled is true.  

This will happen if :

a) the caller of the app has set the IsSampled http header value to true.

OR

b) the url request is not in the mAuthWhitelist appsetting config, and using the zipkinSampleRate, CrossApplicationTracer will determine whether or not to trace this request. zipkinSampleRate is the approximate percentage of traces being recorded in zipkin.

### Config transformations
Below are the config transformations that is needed.  

These transformations will be made automatically if the Medidata.ZipkinTracer.HttpModule nuget package is installed.  These changes will need to be made manually for Medidata.ZipkinTracer.Core nuget package.

1) appsettings.template.config, app.config

Add the below additional configurations. Please verify these values and modify them according to your service/environment.

```
<appSettings>
  <add key="zipkinScribeServerName" value="zipkinvm.cloudapp.net" />
  <add key="zipkinScribeServerPort" value="9410" />
  <add key="ServiceName" value="Name of your Service i.e.Gambit" />
  <add key="spanProcessorBatchSize" value="10" />
  <add key="zipkinSampleRate" value="0.5" />
</appSettings>
```

	zipkinScribeServerName - the zipkin scribe/collector server name to connect to send the Spans

	zipkinScribeServerPort - the zipkin scribe/collector server port to connect to send the Spans

	ServiceName- name of your Service that zipkin will use to label the trace

	spanProcessorBatchSize - how many Spans should be sent to the zipkin scribe/collector in one go.
	
	zipkinSampleRate - 1 decimal point float value between 0 and 1.  this value will determine randomly if the current request will be traced or not.

	
2) parameters.xml

This is used in opscode's xml when deploying service (i.e. Gambit) to customize the values to be used in appsettings.

The values are the same as appsettings.template.config

```
<parameters>
  <parameter name="Zipkin Scribe Server Name" description="Zipkin scribe server name" defaultValue="zipkinvm.cloudapp.net">
    <parameterEntry kind="XmlFile" scope="\\appsettings.config$" match="//appSettings/add[@key='zipkinScribeServerName']/@value" />
  </parameter>
  <parameter name="Zipkin Scribe Server Port" description="Zipkin scribe server port" defaultValue="9410">
    <parameterEntry kind="XmlFile" scope="\\appsettings.config$" match="//appSettings/add[@key='zipkinScribeServerPort']/@value" />
  </parameter>
  <parameter name="Service Name" description="Service name to be traced in Zipkin" defaultValue="Gambit">
    <parameterEntry kind="XmlFile" scope="\\appsettings.config$" match="//appSettings/add[@key='ServiceName']/@value" />
  </parameter>
  <parameter name="Span Processor Batch Size" description="Number of spans to send to zipkin collector in one go" defaultValue="10">
    <parameterEntry kind="XmlFile" scope="\\appsettings.config$" match="//appSettings/add[@key='spanProcessorBatchSize']/@value" />
  </parameter>
  <parameter name="Zipkin Sample Rate" description="float between 0 and 1 to determine whether to send a zipkin trace" defaultValue="0.5">
    <parameterEntry kind="XmlFile" scope="\\appsettings.config$" match="//appSettings/add[@key='zipkinSampleRate']/@value" />
  </parameter>
</parameters>
```

#### Config transformations for HttpModule package

1) web.config 

The following will be added to add the httpModule to your project.  Please don't modify this.

 ```
  <system.webServer>
    <modules>
      <add name="ZipkinRequestContextModule" type="Medidata.ZipkinTracerModule.HttpModule.ZipkinRequestContextModule" />
    </modules>
  </system.webServer>
  ```

## Contributors
ZipkinTracer is (c) Medidata Solutions Worldwide and owned by its major contributors:
* Tomoko Kwan
* [Kenya Matsumoto](https://github.com/kenyamat)

