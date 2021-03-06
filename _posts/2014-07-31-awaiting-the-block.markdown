---
layout: post
title:  "Awaiting the Block"
date:   2014-07-31
categories: csharp
---

The **await** keyword in C# is magical.  It seems to just work.  Throw an **await** in front of a resource intensive 
function  and slap an **async** on the method.  Do this on all the methods in the chain and it just works. Right?  

Although it does *just* work most of the time there are quite a few *gotcha's*.  There are many [best practices](http://msdn.microsoft.com/en-us/magazine/jj991977.aspx) that have been shared to alleviate some of the  scenario's you might run 
 into.
 
 I recently watched an awesome [Pluralsight Video](http://pluralsight.com/training/courses/TableOfContents?courseName=skeet-async&highlight=jon-skeet_skeet-async-m1-intro*3!jon-skeet_skeet-async-m2-await*0,2!jon-skeet_skeet-async-m5-testing*0!jon-skeet_skeet-async-m3-blocking#skeet-async-m1-intro)
  by Jon Skeet that covered Asynchronous code in C# 5.  I  recommend  the  video (and Pluralsight in general) as
   it  goes into 
  great depth on how async works under the 
 covers, which is out of scope for this article.  
  
  One of Async's best use cases is IO related tasks, such as calling web services. When adding **async** to a 
  webservice call you will likely see benefits.  If, however, you do intensive processing before or after 
  making the web service call you could end up in a blocked state.
 
## Blocked
 Take a look at the code below from a click even on a simple WinForms app.  What do you think will 
 happen to the responsiveness of the form?
 
{% highlight csharp %}
private void noAwaitClick(object sender, EventArgs e)
{
    label1.Text = "starting...";
    long prime = this.FindPrimeNumber(PRIME_TO_FIND);
    label1.Text = prime.ToString();
}
{% endhighlight %}

If you guessed that the form will freeze up until the prime number is calculated then you are correct.  Using the 
well known [methods](http://stackoverflow.com/questions/709187/accessing-ui-in-a-thread) to spin off worker threads 
and then calling invoke on the UI thread is one way to get responsiveness back.  This can be a bit tedious and 
difficult 
for beginners
 to 
understand. With **async** and **await** we have a new, simpler way to stay responsive.

## Async to the Rescue
Let's take a look at the code block below.  What do you think will happen to the responsiveness of the form this time?

{% highlight csharp %}
private async void async1Click(object sender, EventArgs e)
{
    label1.Text = "starting...";
    var prime=  await this.primenumberAsync();
    label1.Text = prime.ToString();
}
{% endhighlight %}

Did you guess that it would be responsive?  Good guess, but this was a trick question and the *gotcha* I would like 
to share with you.  To understand why this is not going to keep the UI responsive we have see the implementation of 
the ```primenumberAsync()``` and have basic understanding of what is happening when 
the code is compiled using the **await** and **async** keywords.

Let's see what is happening in the implementation of ```primenumberAsync()```:

{% highlight csharp %}
private async Task<long> primenumberAsync()
{
    long prime = this.FindPrimeNumber(PRIME_TO_FIND);
    return await Task.FromResult(prime);
}
{% endhighlight %}

We are simply calling ```FindPrimeNumber```  and creating a ```Task``` from the result.  Now if you are like me, 
you might be thinking "Doesn't **await** and **async** create new threads when ```primenumberAsync()``` is called?"  

## Under the Covers
To see why the ```async1Click()``` code above blocks we have to have a understand of how **await** and **async** 
gets compiled.
The key words **await** and **async** actually get compiled into a [state machine](http://en.wikipedia.org/wiki/Finite-state_machine).  The compiled state machine is how the asynchronous behavior is achieved.  I would 
recommend watching the [pluralsight](http://pluralsight.com/training/courses/TableOfContents?courseName=skeet-async&highlight=jon-skeet_skeet-async-m1-intro*3!jon-skeet_skeet-async-m2-await*0,2!jon-skeet_skeet-async-m5-testing*0!jon-skeet_skeet-async-m3-blocking#skeet-async-m1-intro) video or reading an overview of [c#'s asyncrounous statemachine](http://www.codeproject.com/Articles/535635/Async-Await-and-the-Generated-StateMachine)
article to get a better understanding of the generated state machine. 

The key take away here is that the **await** and **async** keywords do not create threads.  Instead the compiler 
generates a 
state machine which, in this case, runs on the same UI thread.  

Since the work is not being done on a separate thread and requires intensive processing power, the UI thread gets 
blocked.  You can see the code being executed on the main thread if you download the [sample code](https://github.com/jsturtevant/await-demo) from the github 
repository. Run the code, set a break point  inside of the ```FindPrimeNumber``` routine and use the [thread viewer]
(http://msdn.microsoft.com/en-us/library/w15yf86f.aspx).  

## Unblocked
To create a responsive UI we can leave our call in the button handler the same but re-implement ```primenumberAsync()```
to create a new thread:

{% highlight csharp %}
private async Task<long> primenumberAsync2()
{
    var prime = await Task.Run(() => this.FindPrimeNumber(PRIME_TO_FIND));
    return prime;
}
{% endhighlight %}

## Conclusion
**Await** and **async** are power additions to the C# language but with great power comes great [responsibility](http://en.wikiquote.org/wiki/Stan_Lee).  In 
this post we took a look at one scenario where **await** and **async** are not magic bullets and a little bit of care
 needs to be taken to make sure that any extra processing is done a separate thread.
 
 
 