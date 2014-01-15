---
layout : default
title: SCSS gradient
blog: true
portfolio: 
permalink: /blog/SCSS_Gradient
excerpt : Fun with SCSS @while loop
comments : true
seo__desc : SCSS gradient twist made with a loop and numerous div's
seo__key : SCSS, self-education, code, Haml, CodePen 
---
Today I had the idea to experiment with using the SCSS Control Directive, @while, to create a gradient. The result turned out to be pretty cool as well as educational.   
<!-- /intro -->
In my daily persuit to self-educate I sought out to learn more about some of the advanced functionality possible with SCSS. I started reading through the documentation for [Sass](http://sass-lang.com/documentation/file.SASS_REFERENCE.html) and the idea of creating the gradient spawned. So with the documentation in onw window and [CodePen](http://codepen.io) in the other window and an idea in mind I was set. 

The initial idea was that I would have numerous (100?) div's with a height of only a couple pixels. For the creation of the duplicated div's I used some [Haml](http://haml.info/) which I had learned from another [pen](http://codepen.io/_jasonhodges/full/BsFgx) I had created late last year. Using Haml allowed me to quickly create as many div's with incrementing ID's under the same class. 

{% highlight haml %}
%div.gradient
    %div.line--container
        - (0...100).each do |div|
            %div{:id => "line" + div.to_s}
{% endhighlight %}

compiles to...

{% highlight html %}
<div class='gradient'>
  <div class='line--container'>
    <div id='line0'></div>
    <div id='line1'></div>
    <div id='line2'></div>
    <div id='line3'></div>
    ..on and on and on
{% endhighlight %}

Now that each div had been created and ID assigned I went straight to the SCSS. I will break each section out and explain:

###The Variables
{% highlight scss %}
/*Need 'lines' var*/
$lines: 100;
/*Starting color*/
$c: #006AFF;
/*Multiplyer-change this value to apply major adjustment to gradient*/
$m: 2;
/*Set the height of each line*/
$h: 2.5px;
/*Set the starting width of each line*/
$w: 9px;
{% endhighlight %}

These are the variables I set up to use in the rest of the SCSS. As you will see later in the post I used a `@while` loop to style each of the individual div's. `$lines` will tell the `@while` control directive how many lines to create style for. The other variables could have been placed inside the loop but I just wanted to stick them all together for easier manipulation. Here is the entire code snippet:

###The Loop
{% highlight scss %}
@while $lines > 0{
  	#line#{$lines}{height: $h;
                  width: $w * $lines;
                  margin: 1px;
                  @include vendorize(transform, rotate(81deg + $lines));
                  background-color: $c + $lines*$m;
                  }
   	$lines: $lines - 1;
}
{% endhighlight %}

So what's happening here? the loop is started because `$lines` is greater than 0. Stepping inside the loop the next line is basically saying "style div with ID of line(*number)". Since the last line inside the loop is subtracting 1 from `$line`, the loop is starting at #line100 and decrementing down to #line1 at which point the statement evaluates to false. The starting background-color `$c` is then changed on each div because of the `background-color: $c + $lines*$m;` line. Of course the color behind the variable `$c` is set but the other two variables in this line are what allow the color to actually be changed for each consecutive div. 

I couldn't leave it at just getting the gradient so I played around some more and added `rotate` to each div as well which create a pretty neat looking thing. I won't get into the other tweaks I made but you can check it out for yourself. 

###The Result
<p data-height="510" data-theme-id="0" data-slug-hash="tmdKL" data-user="_jasonhodges" data-default-tab="result" class='codepen'>See the Pen <a href='http://codepen.io/_jasonhodges/pen/tmdKL'>Pure SCSS Gradient</a> by Jason Hodges (<a href='http://codepen.io/_jasonhodges'>@_jasonhodges</a>) on <a href='http://codepen.io'>CodePen</a></p>
<script async src="//codepen.io/assets/embed/ei.js"></script>