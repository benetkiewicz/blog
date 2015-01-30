---
layout: post
title:  "Network communication debugging in .net"
date:   2015-01-30 17:50:59
categories: rest wcf webservices
---
While working with web apps, you often need to go beyond Visual Studio debugger and see what's actually being pushed through the wire. This post is a cumulative cheat sheet on how to set up network capturing for various protocols present in .net ecosystem:

* REST, plain http
* Entity Framework, plain SQL
* WCF, regardless of protocol
* WebServices, SOAP

#### REST, http
Suppose you're using a poorly written client library for some REST service and it fails. You can use the following configuration snippet in your web/app.config to capture a lot of diagnostic info and maybe figure out a work-around:

{% highlight xml %}
<system.diagnostics>
<sources>
<source name="System.Net" tracemode="protocolonly" maxdatasize="1024">
<listeners>
<add name="System.Net"/>
</listeners>
</source>
<source name="System.Net.Http">
<listeners>
<add name="System.Net"/>
</listeners>
</source>
</sources>
<switches>
<add name="System.Net" value="Information"/>
<add name="System.Net.Http" value="Information"/>
</switches>
<sharedListeners>
<add name="System.Net" type="System.Diagnostics.TextWriterTraceListener" initializeData="c:\temp\logs\network.log"/>
</sharedListeners>
<trace autoflush="true"/>
</system.diagnostics>
{% endhighlight %}
<pre>
System.Net Information: 0 : [13844] Current OS installation type is 'Client'.
System.Net Information: 0 : [13844] RAS supported: True
System.Net Information: 0 : [13844] Associating HttpWebRequest#16506093 with ServicePoint#7448163
System.Net Information: 0 : [13844] Associating Connection#51986349 with HttpWebRequest#16506093
System.Net Information: 0 : [13844] Connection#51986349 - Created connection from 10.XXX:8145 to 10.XXX:80.
System.Net Information: 0 : [13844] Associating HttpWebRequest#16506093 with ConnectStream#24590318
System.Net Information: 0 : [13844] HttpWebRequest#16506093 - Request: GET /api/v1/bXXX HTTP/1.1
...
System.Net Error: 0 : [13844] Exception in HttpWebRequest#16506093::GetResponse - The remote server returned an error: (401) Unauthorized..
</pre>

#### Entity Framework
Most useful EF single liner ever:

{% highlight C# %}
using(var context = new DbContext())
{
    context.Database.Log = x => Diagnostics.Debug.WriteLine(x);
    return context.Table.ToList();
}
{% endhighlight %}

#### WCF
Add the following configuration to start logging entire SOAP/whatever communication between endpoints (works for both client and server):

{% highlight xml %}
<system.diagnostics>
<sources>
<source name="System.ServiceModel" switchValue="Information,ActivityTracing"
propagateActivity="true">
<listeners>
<add name="xml" />
</listeners>
</source>
<source name="System.ServiceModel.MessageLogging">
<listeners>
<add name="xml" />
</listeners>
</source>
</sources>
<sharedListeners>
<add initializeData="C:\\temp\\TracingAndLogging-service.svclog" type="System.Diagnostics.XmlWriterTraceListener"
name="xml" />
</sharedListeners>
<trace autoflush="true" />
</system.diagnostics>

<system.serviceModel>
<diagnostics wmiProviderEnabled="true">
<messageLogging
logEntireMessage="true"
logMalformedMessages="true"
logMessagesAtServiceLevel="true"
logMessagesAtTransportLevel="true"
maxMessagesToLog="3000"
maxSizeOfMessageToLog="20000"/>
</diagnostics>
</system.serviceModel>
{% endhighlight %}

The result file will be large and not exactly human-readable, but there's Microsoft Windows SDK tool, called `SvcTraceViewer.exe` which makes it much easier to read. If you have Visual Studio installed, you will probably find this tool in:
<pre>
C:\Program Files\Microsoft SDKs\Windows\v6.0A\Bin\
</pre>

#### ASMX Webservices

Tracing ASMX communication requires both coding and configuration. You need to add the following class into your project:

{% highlight C# %}
using System;
using System.IO;
using System.Web.Services.Protocols;
using System.Xml;

public class SoapTracingExtension : SoapExtension
{
    private string filename;

    private Stream newStream;

    private Stream oldStream;

    public override Stream ChainStream(Stream stream)
    {
        this.oldStream = stream;
        this.newStream = new MemoryStream();
        return this.newStream;
    }

    public override object GetInitializer(LogicalMethodInfo methodInfo, SoapExtensionAttribute attribute)
    {
        return null;
    }

    public override object GetInitializer(Type webServiceType)
    {
        return null;
    }

    public override void Initialize(object initializer)
    {
        this.filename = "c:\\log.txt";
    }

    public override void ProcessMessage(SoapMessage message)
    {
        switch (message.Stage)
        {
            case SoapMessageStage.BeforeSerialize:
            break;
            case SoapMessageStage.AfterSerialize:
            this.WriteOutput(message);
            break;
            case SoapMessageStage.BeforeDeserialize:
            this.WriteInput(message);
            break;
            case SoapMessageStage.AfterDeserialize:
            if (message.Action == "urn:completeOrderWithOrderId")
            {
                message.Exception = new SoapException("Timeout", new XmlQualifiedName("Timeout"));
            }

            break;
            default:
            throw new Exception("invalidstage");
        }
    }

    public void WriteInput(SoapMessage message)
    {
        this.Copy(this.oldStream, this.newStream);
        using (FileStream fs = new FileStream(this.filename, FileMode.Append, FileAccess.Write))
        using (StreamWriter w = new StreamWriter(fs))
        {
            string soapString = (message is SoapServerMessage) ? "SoapRequest" : "SoapResponse";
            w.WriteLine(soapString);
            w.Flush();
            this.newStream.Position = 0;
            this.Copy(this.newStream, fs);
            w.Close();
            this.newStream.Position = 0;
        }
    }

    public void WriteOutput(SoapMessage message)
    {
        this.newStream.Position = 0;
        using (FileStream fs = new FileStream(this.filename, FileMode.Append, FileAccess.Write))
        using (StreamWriter w = new StreamWriter(fs))
        {
            string soapString = (message is SoapServerMessage) ? "SoapResponse" : "SoapRequest";
            w.WriteLine(soapString);
            w.Flush();
            this.Copy(this.newStream, fs);
            w.Close();
            this.newStream.Position = 0;
            this.Copy(this.newStream, this.oldStream);
        }
    }

    private void Copy(Stream from, Stream to)
    {
        using (TextReader reader = new StreamReader(from))
        using (TextWriter writer = new StreamWriter(to))
        {
            writer.WriteLine(reader.ReadToEnd());
            writer.Flush();
        }
    }
}

{% endhighlight %}

With that code in place, add the following into app/web.config:

{% highlight xml %}
<system.web>
<webServices>
<soapExtensionTypes>
<add type="Proper.Namespace.SoapTracingExtension, Proper.Namespace" priority="1" group="0" />
</soapExtensionTypes>
</webServices>
</system.web>
{% endhighlight %}
