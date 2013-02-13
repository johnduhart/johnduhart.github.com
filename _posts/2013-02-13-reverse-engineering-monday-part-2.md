---
layout: post
title: "Reverse Engineering Monday (Part 2)"
category: reverse
---
Picking up from where we left off last, we now know a few key things.

First of all we now know that this is a .NET application, and decompiling it will be relatively easy. Secondly we know that it's using SMTP to send data back to it's owner through gmail. This is our target, we want to get those credentials in order to stop the owner from stealing any more data.

So let's get started!
### Wireshark
We're going to start off by looking doing some packet sniffing on the traffic coming out of the program. Hopefully we can learn something this way. So we set up Wireshark and then start the program aaaand then Wireshark closes.

Huh?

It would appear that the program is doing some protection against being reverse engineered and/or stopped. The same thing happens to task manager, it closes immediately. Luckily these sorts of tricks are easy to get around.

<a href="/images/2013-02-13/wiresharkFix.png"><img src="/images/2013-02-13/wiresharkFix.png" class="center" /></a>

With that out of the way we restart the application and, tada, we have an SMTP Stream!

<a href="/images/2013-02-13/stream.png"><img src="/images/2013-02-13/stream.png" class="center" /></a>

So now if we wanted we could take a left turn here, decrypt the TLS stream, extract the credentials and end our journey. Unfortunately I do not know the ins and outs of TLS, so I will be taking the scenic route.

If you're reading this and you know how to decrypt the stream to make it plaintex, by all means <a href="mailto:john@johnduhart.me">send me an e-mail</a>. I'd love to hear about.
###The Scenic Route
Like I said earlier, this is a .NET application. De-compiling this application will be a piece of cake, we just need to put the application into JetBrains dotPeek (or really any other MSIL decompiler).

<a href="/images/2013-02-13/easy.png"><img src="/images/2013-02-13/easy.png" class="center" /></a>

Like I said, this was going to be eas—

<a href="/images/2013-02-13/notsoeasy.png"><img src="/images/2013-02-13/notsoeasy.png" class="center" /></a>

Oh. What?

{% highlight csharp %}
using System;
using System.ComponentModel;
using System.Windows.Forms;

namespace amsu
{
	public class CustomControl6 : Control
	{
		private IContainer components;

		public CustomControl6()
		{
			this.InitializeComponent();
		}

		protected override void Dispose(bool disposing)
		{
			if (disposing && this.components != null)
			{
				this.components.Dispose();
			}
			base.Dispose(disposing);
		}

		private void InitializeComponent()
		{
			this.components = new Container();
		}

		protected override void OnPaint(PaintEventArgs pe)
		{
			base.OnPaint(pe);
		}
	}
}
{% endhighlight %}

Okay this is a bit odd. Each class inherits System.Windows.Form.Control, but seems to do nothing. The application doesn't even have a GUI! Let's take a step back here, and just search for Main().

{% highlight csharp %}
private static void Invoke()
{
	CustomControl27.Amci.Invoke(null, null);
}

internal static void Main()
{
	CustomControl24.Invoke();
}
{% endhighlight %}

Okay there it is, and it seems to be calling Invoke on a MethodInfo property in CustomControl27...

{% highlight csharp %}
public static MethodInfo Amci
{
	get
	{
		return CustomControl25.Amci;
	}
}
{% endhighlight %}

...which is a wrapper around CustomControl25.Amci...

{% highlight csharp %}
public static readonly MethodInfo Amci = CustomControl9.Oaoa.GetMethod("Dif");
{% endhighlight %}

...which loaded from CustomControl9.Oaoa...

{% highlight csharp %}
public static System.Type Oaoa = CustomControl2.Maip.GetType("ive.Spec");
{% endhighlight %}

...which is loaded from CustomControl2.Maip...

{% highlight csharp %}
public static Assembly Maip = Assembly.Load(CustomControl8.Prost());
{% endhighlight %}

...which is loading an assembly contained in a byte array from Prost()! Let's dig into Prost()

{% highlight csharp %}
public static byte[] Prost()
{
	CustomControl7.Key2();
	CustomControl5.Cumi();
	return CustomControl3._data;
}
{% endhighlight %}

There's two method calls in the start of Prost(), but we'll get back to that later. So, it's returning data,

{% highlight csharp %}
public static byte[] _data = Convert.FromBase64String(Resource1.String1);
{% endhighlight %}

Which is loading a Base64 encoded string stored in resoures! Fucking awesome.

<a href="/images/2013-02-13/resource.PNG"><img src="/images/2013-02-13/resource.PNG" class="center" /></a>

Essentially, it's a program inside a program.

<img src="http://userserve-ak.last.fm/serve/500/50441183/Xzibit.png" class="center" />

Sorry, that was pretty forced.

So the next steps for this would be to take the base64 string, convert it to binary, and save it to a DLL to open in dotPeek. Right?

Well, not exactly. Do you remember those two method calls in Prost()? Those two methods are actually manipulating the data array before it is return to be loaded as an assembly. Damn.

Instead of trying to extract all that codes which spans several CustomControl classes, I'm just going to convert the program to a C# project using JustDecompile and load it into Visual Studio.

<a href="/images/2013-02-13/convert.PNG"><img src="/images/2013-02-13/convert.PNG" class="center" /></a>

And now with the code open in VS, we're going to make the following change to write the changed contents to a dll to anaylze.

{% highlight csharp %}
public static byte[] Prost()
{
	CustomControl7.Key2();
	CustomControl5.Cumi();

	// haxxor code!!
	File.WriteAllBytes("inner.dll", CustomControl3._data);
	Environment.Exit(1337);

	return CustomControl3._data;
}
{% endhighlight %}

And the we run it...

<a href="/images/2013-02-13/decompiled.PNG"><img src="/images/2013-02-13/decompiled.PNG" class="center" /></a>

Success! We now have our inner DLL file!

### That's all for today
If you'd like to take a look at the decompiled code that we explored today, you can downloading it by [clicking on these words](/assets/reverseeng1/GPCode.zip). In the next edition, we'll enter another level of hell and take a look at this inner assembly and what it does.