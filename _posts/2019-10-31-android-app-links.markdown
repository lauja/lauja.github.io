---
layout: post
title:  "Android App Links"
date:   2019-10-31
---
There are certain scenarios where your customers may find a product of yours on a search engine on the phone, but you want to direct them to your app experience when they click on the link. Take Amazon for example. If you search for a product you want to buy on Google and click on an Amazon link, it will direct you to the app to view the product. This has numerous benefits, the important ones being, an improved experience and not having to go through the cumbersome authentication process on your web site.

Enter Android App Links ([https://developer.android.com/training/app-links](https://developer.android.com/training/app-links)). This is the way to get your users to launch straight in to the app if they click on a particular link, and if you can prove that the domain is yours. This works for Marshmallow devices and above. Here are the steps to get this implemented.
<br/>
<br/>

## Prove ownership of domain
You will need to upload a file called assetlinks.json to a folder called ".well-known" in the root of the domain that you are trying to target:

`http://<domain>/.well-known/assetlinks.json`

You can use this generator to create an example and also test for ownership:
[https://developers.google.com/digital-asset-links/tools/generator](https://developers.google.com/digital-asset-links/tools/generator)
 
{% highlight json %}
[
  {
    "relation": [
      "delegate_permission/common.handle_all_urls"
    ],
    "target": {
      "namespace": "android_app",
      "package_name": "<package_name>",
      "sha256_cert_fingerprints": [
        "<sha256>"
      ]
    }
  }
]
{% endhighlight %}

Note that the sha256 fingerprint needs to be the one signed for that particular apk and package name, eg. you might have a testing flavour like com.myapp.testing.

You can get the sha256 fingerprint from the apk by running:

`keytool -list -printcert -jarfile <apk file>`

There are some security restrictions. For example, you can not have the url redirect you from the domain to another domain to get the json file. You will get an error message along the lines of (look in the browser console of the generator):

> redirects are disallowed for security reasons (NOT_FOLLOWED_MAX_FORWARDS) [2] while fetching Web statements

You will. Have to change your Android manifest for where you intend to intercept this link:

eg. Inside the intent-filter

{% highlight xml %}
<data
    android:scheme="https"
    android:host="<domain>"
    android:path="<path>" />
{% endhighlight %}
    
When you are installing the app, you need to have a data connection as the phone will try to verify that the link is indeed legitimate.

Check if your phone recognises the domain:

`adb shell dumpsys package d`

If the Status is "ask" it will show a "disambiguation dialog", which will ask the user to select the app that we want to open with.
If the Status is "always" the app will always open for links to this domain.

eg:

```
Package: au.com.realestate.app
  Domains: mobile.realestate.com.au www.realestate.com.au m.realestate.com.au realestate.com.au
  Status:  ask

Package: com.fairfax.domain
  Domains: www.domain.com.au sell.domain.com.au m.domain.com.au domain.com.au stage-sell.domain.com.au
  Status:  always : 200000017
```

In this case, the phone will ask you to select the app you want to view with if you go to **realestate.com.au** (eg. browser), and it will go straight to Domain app if we go to say **www.domain.com.au**

The last step is to show an appropriate webpage for when there is no app installed. For example, if we have added the applink: www.myapp.com/sales in to the app, but the user does not have the app installed, the user should still be able to continue what they wanted to do in the web experience.