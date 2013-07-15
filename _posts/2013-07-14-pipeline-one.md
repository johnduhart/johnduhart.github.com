---
layout: post
title: "Breakdown: Pipeline (Part One)"
category: breakdown
---

<img src="http://i.imgur.com/q0Oj7j1.png" class="center" />

### Intro

In the past on [Facepunch](http://facepunch.com/forumdisplay.php?f=353) I have done satirical breakdowns of poorly coded websites, pointing out the various flaws in them while incorporating witty remarks. In this series of blog posts I'd like to bring that back, although with a more professional tone. I'll still manage to work some humor into these posts, however.

### Valve Pipeline

For those of you who aren't following Valve on YouTube, yesterday they uploaded a video introducing a new program that brings high school level students into Valve, allowing them to gain experience in the video game industry. There's probably a dozen articles covering the announcement so I'm not going to bother covering this in depth.

<img src="http://i.imgur.com/SfTD4zU.png" class="center" />

One thing interesting about the video is that apparently tank-tops are work appropriate clothing. Who knew.

In all seriousness though, given that these are teenagers as young as 15, I'm going to dull my bite quite a bit. They're not professionals and it would be unfair to assume as such. 

### The Server Itself

Before even getting to the website, I'd like to take a second to talk about the server that hosts [http://pipeline.valvesoftware.com/](http://pipeline.valvesoftware.com/). Whoever at Valve set this up realized that putting a server under control by a group of teenagers (likely pushing out insecure code) in one of their data centers would be a terrible idea, and instead decided to host it externally. Congratulations for that bit of foresight.

Specifically the IP belongs to Rackspace Cloud. It only seems to be listening on port 80. For the time being there's nothing else interesting about this.

### The Intro Page

<img src="http://i.imgur.com/sOeqiLb.png" class="center" />

The introduction/splash/work-in-progress page itself looks alright. I mean, I'm no designer but I think it looks *okay*.

<img src="http://i.imgur.com/O1CacRq.png" class="center" />

It does bug me that there is a lack of proper cursors over clickable elements. There's no indication that what I'm hovering my mouse cursor over can be clicked, and to be honest it makes my skin crawl a little.

The website also fails the "Usability without JavaScript test" as none of the secret links work without it.

#### What lurks beneath

Let's look at that HTML, shall we!

{% highlight html %}
<html>
<head>
	<title>Pipeline</title>
	<link rel="stylesheet" type="text/css" href="css/debut.css">
	<link rel="stylesheet" type="text/css" href="css/bootstrap.css">
	<script type="text/javascript" src="js/jquery.js"></script>
	<script type="text/javascript" src="js/informational.js"></script>
</head>
{% endhighlight %}

Okay, well it's missing a doc type, but it's a basic mistake so no big deal. Also, interesting decision on including bootstrap after your CSS. Since (external) CSS is applied in order of inclusion, I feel like you'd be fighting with bootstrap in entire time to get your own styling to come up.

We'll look at the contents of those files shortly.

{% highlight html %}
	<div class="video_black">
		<iframe style="margin-top: 60px" width="1200" height="600" src="//www.youtube.com/embed/wrH5ZbUJ_kw" frameborder="0" allowfullscreen></iframe>
	</div>
{% endhighlight %}

**Protip:** If you're going to embed a YouTube video and want it big, allow it to scale down on smaller screens. Your users will thank you.

{% highlight html %}
	<div class="blurbPhoto"></div>
{% endhighlight %}

Wait hold on, did I miss something? Where's the pipe and text here? Nope, looking at the css for .blurbPhoto you can see it's an image.

{% highlight css %}
.blurbPhoto{
	margin: 60 0 60 0; /*Makes float in center*/
	height: 400px;
	width: 100%;
	background:url('../images/section_bar.png') no-repeat;
	background-position: center;
}
{% endhighlight %}

[A 3000 pixel wide image](http://pipeline.valvesoftware.com/images/section_bar.png).

This is a hack. There's no other way to describe this. The proper way to do this would be to style the div with a fixed height and width with correct borders, and put the image content inside and center it. Ugh.

Moving on to the FAQ section

{% highlight html %}
<div id="FAQ">
	<br>
	<div class="question Q" id="Q1">Why is Valve doing this?</div><br>
	<div class="answer" id="A1">There are two main reasons that Valve is creating Pipeline.  
		The first is that we are frequently asked questions by teenagers about the videogame industry... </div><br>
{% endhighlight %}

Holy shit are they using a `<br>` tag? I have not seen one of those in forever. Adding margins in between questions/answers was the right way to go about this. Also, why are there two question classes?

I personally like using `<dd>`/`<dl>`/`<dt>` with FAQ pages because it gives the HTML semantic meaning.

{% highlight html %}
<table cellpadding="10" id="email_register_table">
	<tr>
		<!-- Name Field -->
		<td>Name:</td>
		<td><input type="text" name="Name" id="newName" placeholder="Name"></td>
	</tr>
	<tr>
		<!-- Email Field -->
		<td>Email:</td>
		<td><input type="email" id="newEmail" class="verifyEmail" name="Email" placeholder="Email"></td>
	</tr>
	<tr>
		<!-- Verify Email Field -->
		<td id="verifyEmailLabel">Verify-Email:</td>
		<td><input type="email" id="verifyNewEmail" class="verifyEmail" name="verifyEmail" placeholder="Verify-Email"></td>
		<td id="emailVarificationUserCheck"></td>
	</tr>
	<tr>
		<!-- Submit Button -->
		<td></td>
		<td><input type="button" id="email_signup" onClick="newletterSignUp();" disabled="disabled" class="btn" value="Sign Up"></td>
	</tr>
</table>
{% endhighlight %}

The lack of a `<form>` element means that this for has no hope of working without JavaScript (If you could ever get to it in the first place, however).

And finally,

{% highlight html %}
	<p><br /></p>
	<p><br /></p>
	<p><br /></p>
	<p><br /></p>
	<p><br /></p>
</div>
<html>
{% endhighlight %}

lrn2padding

#### Background Images

In `debut.css` I found the following lines

{% highlight css %}
background: url('http://webby.com/humor/i/near-Tuba.jpg') fixed no-repeat;
/* ... */
background: url('http://wallpapersus.com/wp-content/uploads/2012/10/Black-Background-Metal-Hole-Very-Small.jpg') fixed no-repeat; /* Subject to change */
{% endhighlight %}

This is not okay. The Pipeline website is currently getting a lot of attention due to its connection with Valve, and it's unfair to those website owners to have their bandwidth leeched.

This also brings up an issue of copyright. Being 15 isn't a good excuse for stealing somebody's work.

#### JavaScript

Time to examine the informational.js

{% highlight javascript %}
// These next two function control the show all, and close all divs.
// It loops through every element that contains the id Ai, i being a number. 
$(".openall").click(function(){
	for (var i = 0; i <= q_num; i++) {
		$("#A"+i).slideDown("slow");
	};
});
$(".closeall").click(function(){
	for (var i = 0; i <= q_num; i++) {
		$("#A"+i).slideUp("slow");
	};
});
{% endhighlight %}

This seems unnecessary, all the answers have a class of `.answer` and can be selected all once using that. (I realize there's a [performance benefit](http://jsperf.com/id-vs-class-vs-tag-selectors/2) for selecting by id however the difference in this case is negligible).

{% highlight javascript %}
// $('.Q').each() means do this to every element that has class="Q"
$('.Q').each( 
	function( i )
	{
		// adding one to q_num each time there is element with class Q found
		q_num++;
		// this next function is called whenever one of the elements with class Q is clicked
		$(this).click( 
			function() 
			{ 
				// gets the id of the element that was clicked
				id = $(this).attr( 'id' );
				// id looks like Q1, Q2, ... Q10, etc
				num = id.slice( 1 ); // returns everything after first character
				toggleAnswer( num );  // calls the toggleAnswer() function.
			} 
		);
	}
);

function toggleAnswer( answerNum )
{
	// see if the element is hidden
	if ( $('#A'+answerNum).is( ':hidden' ) )
	{
		// slide it down (expose it)
		$('#A'+answerNum).slideDown( 'slow' );
	}
	else
	{
		// slide it up (hide it)
		$('#A'+answerNum).slideUp( 'slow' );
	}
}
{% endhighlight %}

Both `id` and `num` are being created as [global variables](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Values,_variables,_and_literals#Global_variables).

Here's my fixed version of the FAQ code:

{% highlight javascript %}
$(document).ready(function() {
	$(".openall").click(function() {
		$(".answer").slideDown("slow");
	});
	$(".closeall").click(function() {
		$(".answer").slideUp("slow");
	});

	$(".question").each(function() {
		var questionId = $(this).attr("id").slide(1),
			$answer = $("#A" + questionId);

		$(this).click(function() {
			$answer.slideToggle("slow");
		});
	});
});
{% endhighlight %}

It's functionally the same and much more compact.

#### Newsletter Registration

As I mentioned above, the newsletter registration form only works with JavaScript, so we'll take a look at how that works.

{% highlight javascript %}
$(".verifyEmail").keyup(function(event){
	var email = $('input[id=newEmail]').val();
	var verifyEmail = $('input[id=verifyNewEmail]').val();
	if ((email != "") && (verifyEmail != "")) {
		if (email === verifyEmail) {
			// The emails do match
			$('#email_signup').removeAttr('disabled');
			$('#emailVarificationUserCheck').html('Match!');
			$('#emailVarificationUserCheck').css('color','green');
		} else{
			// The emails don't match
			$('#emailVarificationUserCheck').html('Sorry, those Emails don\'t match.');
			$('#emailVarificationUserCheck').css('color','red');
			$('#email_signup').attr('disabled','disabled');
		};
	}
	else
	{
		$('#emailVarificationUserCheck').html('');
		$('#email_signup').attr('disabled','disabled');
	};
});
{% endhighlight %}

This code is doing e-mail validation. I don't understand why it's required to make sure the person puts in their e-mail correctly, given that they will need to verify it.

`$('input[id=newEmail]')` This selector is also incorrect, the regular id selector should be used. (I ran a performance test to see if jQuery compensates for this. [It doesn't](http://jsperf.com/id-vs-class-vs-tag-selectors/144))

{% highlight javascript %}
function newletterSignUp () 
{
	newName = $('#newName').val(); // gets the newName value
	newEmail = $('#newEmail').val(); // gets the newEmail value
	$.ajax(
		{
			type: 'POST',
			url: 'action.php',
			data: 
				{ // passes the values into the php page as names retrievable by chkreq
					'action':'newletterSignUp',
					'newName':newName,
					'newEmail':newEmail
//					'recaptcha_challenge_field':$('#recaptcha_challenge_field').val(),
//					'recaptcha_response_field':$('#recaptcha_response_field').val()
				},
			dataType: 'json',
			success: function ( dataJSON )
				{
					console.log('trying');
					if ( dataJSON.success == 1) 
					{
						alert( "Thank you, you have been added to our Email system." );
						location.href = 'index.php';
					} 
					else
					{
						alert( "There was a problem adding you!\n"+dataJSON.errortxt );
					};
				}
		}
	);
}
{% endhighlight %}

Nothing too amazing here, it submits the form and gives back a result through an [`alert()`](https://developer.mozilla.org/en-US/docs/Web/API/window.alert).

So let's take a look at the JSON that gets returned.

<img src="http://i.imgur.com/23IYuR4.png" class="center" />

`authKey`?

<img src="http://i.imgur.com/1EyrBY7.png" class="center" />

Oh. Oh, no. That's not something to send back in a JSON response.

### To Be Continued

I'm going to break this post up in order to get it out quicker. There will be another installment soon.