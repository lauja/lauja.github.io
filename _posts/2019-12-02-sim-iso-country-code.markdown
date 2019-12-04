---
layout: post
title:  "Getting a user's location via sim ISO"
date:   2019-12-02
---

There may come a time where you need to find out a user's physical location so that you can provide a different experience depending on where they are.
This can be done easily by doing: 

{% highlight kotlin %}
val telephonyManager = getSystemService(Context.TELEPHONY_SERVICE) as? TelephonyManager
val simCountry = telephonyManager?.simCountryIso
{% endhighlight %}

This is not completely foolproof as devices that do not have a sim will most likely return null. 
Also if the customer is on global roaming, the sim country will report from where the closest cell tower is rather than sim's originating country.

This can be tested easily on the emulator by doing the following:

```
adb shell

su
 
setprop gsm.sim.operator.iso-country <ISO_CODE eg. au>
```

NOTE: you will need to set up the correct emulator to be able to `su`.
Choose a Google APIs image rather than a Google Play image

you can get the current sim iso code by running:

```
getprop gsm.sim.operator.iso-country
```