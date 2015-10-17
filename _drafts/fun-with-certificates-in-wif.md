---
layout: post
title: "Fun with certificates in WIF"
date: "2015-10-15 15:42"
categories: csharp wif
---

Recently I've been developing a custom STS using Windows Identity Framework and this is joy oh joy. Certs flying here and there, public keys, private keys, policies, trust and lack of trust.
But here I only wanted to share with you two small gems that may help you go a similar path and maybe save some precious time in your life.

## MMC invisible UTF sign

While working with certificates in WIF and Windows, sooner or later you'll have to copy certificate thumbprint to put somewhere in the configuration. In order to do that you'll probably go to Microsoft Management Console and use Manage Computer Certificates Snap-In. Select, copy, paste, done? Almost...

![MMC certificate](\images\cert_mmc.png)

If you'll paste the value in web.config using Visual Studio, VS will happily accept that.

![Visual Studio thumbprint](\images\cert_vs.png)

But there will be some issues. See how the same value looks in notepad:

![Notepad thumbprint](\images\cert_notepad2.png)

I just saved you half a day of going after your own tail.

## Signing STS ticket without opening password protected certificate

STS ticket needs to be digitally signed, so Relaying Parties will know that you are you and they can trust in whatever you've sent them. To sign a ticket with a certificate, you need to access it first. Avoid accessing certificate file because this requires password and lacks granular access rights. So avoid doing this:

{% highlight C# %}
var signingCert = new X509Certificate2(signingCertFullPath, password, X509KeyStorageFlags.MachineKeySet);
{% endhighlight %}

Use the following code instead to go password-less and remember to grant your AppPool identity access to certificate in windows store.

{% highlight C# %}
public static X509Certificate2 FindByThumbprint(string thumbprint, StoreName storeName, StoreLocation storeLocation)
{
    var certificateStore = new X509Store(storeName, storeLocation);
    certificateStore.Open(OpenFlags.ReadOnly);

    foreach (var certificate in certificateStore.Certificates)
    {
        if (certificate == null || certificate.Thumbprint == null)
        {
            continue;
        }

        if (String.Equals(certificate.Thumbprint, thumbprint, StringComparison.CurrentCultureIgnoreCase))
        {
            certificateStore.Close();
            return certificate;
        }
    }

    throw new ArgumentException(string.Format("Cannot find certificate with thumbprint {0} in certificate store ", thumbprint));
}

string signingCertThumbprint = ConfigurationManager.AppSettings["SigningCertificateThumbprint"];
var signingCert = FindByThumbprint(signingCertThumbprint, StoreName.My, StoreLocation.LocalMachine);
{% endhighlight %}
