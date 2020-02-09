---
layout: post
title:  "Using COM methods in PowerShell"
date:   2020-02-09 21:00:00 +0000
categories: powershell com
---
PowerShell can create instances of COM objects using `New-Object -ComObject`. Calling methods on these objects can be difficult though - it doesn't seem to properly handle methods which have VARIANTs in their signature.

I found [this StackOverflow answer](https://stackoverflow.com/a/30522402) which describes the fix. You effectively need to add `System.Runtime.InteropServices.VariantWrapper` as another level of indirection.

### Sample Code
{% highlight powershell %}
$comObject = New-Object -ComObject {...}
$handles = $null
$handleWrapper = New-Object System.Runtime.InteropServices.VariantWrapper $handles

$comObject.MethodCall([ref]$handleWrapper)

# Handles are now available in the $handleWrapper variable.
{% endhighlight %}
