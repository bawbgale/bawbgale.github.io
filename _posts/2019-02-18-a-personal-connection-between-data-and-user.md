---
id: 274
title: Creating a personal connection between data and user with conditional narrative in Tableau
date: 2019-02-18T05:55:14+00:00
author: Bob Gale
layout: post
guid: https://www.bawbgale.com/?p=274
permalink: /a-personal-connection-between-data-and-user/
categories:
  - Uncategorized
tags:
  - data visualization
  - tableau
---
<figure class="wp-block-image">[<img src="https://www.bawbgale.com/wp-content/uploads/2019/02/00_final_viz-1-1024x778.png" alt="" class="wp-image-287" srcset="https://www.bawbgale.com/wp-content/uploads/2019/02/00_final_viz-1-1024x778.png 1024w, https://www.bawbgale.com/wp-content/uploads/2019/02/00_final_viz-1-300x228.png 300w, https://www.bawbgale.com/wp-content/uploads/2019/02/00_final_viz-1-768x583.png 768w" sizes="(max-width: 1024px) 100vw, 1024px" />](https://public.tableau.com/profile/bob.gale#!/vizhome/MakeoverMonday2018w48/Dashboard1)</figure> 

Last month’s [Seattle Tableau User Group](https://usergroups.tableau.com/Seattle) meeting featured a hands-on redesign session based on Makeover Monday. (Special thanks to [Gina Bremer](https://www.linkedin.com/in/ginabremer/) for organizing this and pushing a number of us to try it for the first time.) Below is my entry and some notes about my approach.  


For those who don’t know, [Makeover Monday](http://www.makeovermonday.co.uk) is a weekly data viz challenge hosted by Eva Murray and Andy Kriebel. They post a “before” viz and its underlying data every Monday, invite the worldwide data viz community to take a stab at improving it, and then host a webinar the following Wednesday to review submissions and provide feedback. So it is an intense quick-turn sprint, sure to push the limits of even seasoned professionals.  


For our SeaTUG session, Gina picked a fun topic from last year: a comparison of the [cost of a night out in 13 major cities](https://data.world/makeovermonday/2018w48), based on average prices of seven common activities. Here’s the “before” viz:<figure class="wp-block-image">

<img src="https://www.bawbgale.com/wp-content/uploads/2019/02/01_chartoftheday_14081_the_price_of_a_party_around_the_world_n.jpg" alt="" class="wp-image-275" srcset="https://www.bawbgale.com/wp-content/uploads/2019/02/01_chartoftheday_14081_the_price_of_a_party_around_the_world_n.jpg 960w, https://www.bawbgale.com/wp-content/uploads/2019/02/01_chartoftheday_14081_the_price_of_a_party_around_the_world_n-300x214.jpg 300w, https://www.bawbgale.com/wp-content/uploads/2019/02/01_chartoftheday_14081_the_price_of_a_party_around_the_world_n-768x547.jpg 768w" sizes="(max-width: 960px) 100vw, 960px" /> </figure> 

Overall, a fairly simple, attractive and understandable viz, but with plenty of room for improvement. My main criticism was that while it lets you easily compare the overall costs of each city, you can&#8217;t easily compare the costs of the individual activities, except for the &#8220;Club entry&#8221; bars aligned with the baseline. This is a common shortcoming of stacked bar charts. 

Here&#8217;s my redesign (animated to show the interactivity):<figure class="wp-block-image">

<img src="https://www.bawbgale.com/wp-content/uploads/2019/02/02_night_out.gif" alt="" class="wp-image-276" /> </figure> 

My main focus was increasing user engagement with the viz by personalizing it. Rather than just dispassionately comparing data across cities (or pretending that most people can actually jet off to any of them at will), I prompt the user to select a city they plan to visit, then give them some practical advice on what activities they can choose to get the most value for their money relative to other cities. The idea is to engage the user in the viz not just by making it visually interesting but, as [Steve Wexler says](https://www.datarevelations.com/its-your-data-not-the-viz-thats-boring.html), creating a personal connection between the data and the user. 

Presenting a selector and highlighting the selected city was easy, but dispensing advice with conditional narrative text was more challenging. Here’s how I did it:  


The source data consists of a simple table of cities, activities (called &#8220;item&#8221;) and costs. The items were further categorized as &#8220;Date night&#8221; or &#8220;Party night,&#8221; though I didn&#8217;t use that in my viz.<figure class="wp-block-image">

<img src="https://www.bawbgale.com/wp-content/uploads/2019/02/03_city_data.png" alt="" class="wp-image-280" srcset="https://www.bawbgale.com/wp-content/uploads/2019/02/03_city_data.png 500w, https://www.bawbgale.com/wp-content/uploads/2019/02/03_city_data-300x153.png 300w" sizes="(max-width: 500px) 100vw, 500px" /> </figure> 

The first step was to create a parameter from the City field to use as a selector menu:<figure class="wp-block-image">

<img src="https://www.bawbgale.com/wp-content/uploads/2019/02/04_parameter-1024x932.png" alt="" class="wp-image-281" srcset="https://www.bawbgale.com/wp-content/uploads/2019/02/04_parameter-1024x932.png 1024w, https://www.bawbgale.com/wp-content/uploads/2019/02/04_parameter-300x273.png 300w, https://www.bawbgale.com/wp-content/uploads/2019/02/04_parameter-768x699.png 768w, https://www.bawbgale.com/wp-content/uploads/2019/02/04_parameter.png 1184w" sizes="(max-width: 1024px) 100vw, 1024px" /> </figure> 

Then I used a simple Boolean calculation to check if each row of data pertains to the selected city: 

<pre class="brush: plain; title: ; wrap-lines: false; notranslate" title="">// [City is Selected?]
[City] = [City Parameter]
</pre>

Then I started experimenting with charts and calculations to compare the selected city&#8217;s costs with other cities. Initially I focused on comparing each city&#8217;s ranking (both overall and per activity). Tableau makes this very easy using table calculations, but in order to use these rankings to power my conditional narrative, I needed to work with rankings outside of the visible dimensions &#8212; i.e. using LOD expressions. That’s when I realized that ranking is not well supported in Tableau outside of table calculations. So, after [upvoting this Tableau feature idea](https://community.tableau.com/ideas/4553), I switched to a different way of comparing relative prices: percent of maximum: 

<pre class="brush: plain; title: ; wrap-lines: false; notranslate" title="">//[Selected City is Expensive?]
SUM( IF [City is Selected?] THEN [Cost] END )
/
MAX( { FIXED [City] : SUM([Cost]) } )
&gt; [Expensive Threshold]
</pre>

<pre class="brush: plain; title: ; wrap-lines: false; notranslate" title="">//[Selected City is Cheap?]
SUM( IF [City is Selected?] THEN [Cost] END )
/
MAX( { FIXED [City] : SUM([Cost]) } )
&lt; [Cheap Threshold]
</pre>

The numerator uses an IF THEN expression to select only the selected city&#8217;s costs before summing them. The denominator uses a FIXED LOD expression to sum the costs of each city before calculating the MAX. The resulting fractions are then compared with parameters containing the threshold that I consider &#8220;expensive&#8221; or &#8220;cheap.&#8221; Using parameters instead of hard-coding these values let me play with different thresholds and see how the viz reacted. 

Once I had a metric that worked well with LOD calculations, I could start writing conditional expressions to display different narratives &#8212; e.g. “Congrats, Prague is one of the least expensive cities for a night out.” I split this into three separate calculations so that I could apply different text formatting to each:

<pre class="brush: plain; title: ; wrap-lines: false; notranslate" title="">// [Narrative 1]
IF [City Parameter] != '0' THEN
    IF [Selected City is Expensive?] THEN 'Sorry,'
    ELSEIF [Selected City is Cheap?] THEN 'Congrats,'
    ELSE ''
    END
END
</pre>

<pre class="brush: plain; title: ; wrap-lines: false; notranslate" title="">// [Narrative 2]
IF [City Parameter] != '0' THEN
    ATTR([Selected City])
END
</pre>

<pre class="brush: plain; title: ; wrap-lines: false; notranslate" title="">// [Narrative 3]
IF [City Parameter] != '0' THEN
    IF [Selected City is Expensive?] THEN 'is one of the most expensive cities for a night out.'
    ELSEIF [Selected City is Cheap?] THEN 'is one of the least expensive cities for a night out.'
    ELSE 'is neither the most nor least expensive city for a night out.'
    END
END
</pre>

I displayed this narrative text above a basic bar chart that showed how the selected city ranked on overall cost, with a tool tip that shows an itemized bill: <figure class="wp-block-image">

<img src="https://www.bawbgale.com/wp-content/uploads/2019/02/08_tooltip-1024x719.png" alt="" class="wp-image-291" srcset="https://www.bawbgale.com/wp-content/uploads/2019/02/08_tooltip-1024x719.png 1024w, https://www.bawbgale.com/wp-content/uploads/2019/02/08_tooltip-300x211.png 300w, https://www.bawbgale.com/wp-content/uploads/2019/02/08_tooltip-768x539.png 768w, https://www.bawbgale.com/wp-content/uploads/2019/02/08_tooltip.png 1142w" sizes="(max-width: 1024px) 100vw, 1024px" /> </figure> 

Then I turned to creating narrative text based on individual activities. This proved to be trickier because I wanted to express nuances such as &#8220;Overall this is one of the most expensive cities, but some activities are relative bargains.&#8221; I decided to base this on categorizing each activity as relatively expensive or cheap compared with other cities, then counting them. My first attempts at this kept producing continuous measures rather than a dimension that I could list and count, until I found this knowledge base article: [Unable to Convert Measure to Dimension](https://kb.tableau.com/articles/issue/unable-to-convert-measure-to-dimension). The key was that although the calculation needs to perform an aggregation to get the MAX activity cost across cities, the end result of the calculation must not return an aggregation. So I created Boolean calculations to check if the activity&#8217;s cost is above or below the threshold for the selected city:

<pre class="brush: plain; title: ; wrap-lines: false; notranslate" title="">// [Activity is Relatively Expensive?]
[Cost] / {FIXED [Item] : MAX([Cost])} &gt; [Expensive Threshold]
</pre>

<pre class="brush: plain; title: ; wrap-lines: false; notranslate" title="">// [Activity is Relatively Cheap?]
[Cost] / {FIXED [Item] : MAX([Cost])} &lt; [Cheap Threshold]
</pre>

Then used an IF THEN expression to return [Item] only if it is above or below the threshold for the selected city:

<pre class="brush: plain; title: ; wrap-lines: false; notranslate" title="">// [Selected City Expensive Activities]
IF [City is Selected?] AND [Activity is Relatively Expensive?]
THEN [Item] 
END
</pre>

<pre class="brush: plain; title: ; wrap-lines: false; notranslate" title="">// [Selected City Cheap Activities]
IF [City is Selected?] AND [Activity is Relatively Cheap?]
THEN [Item]
END
</pre>

Then used a bunch of conditional expressions with COUNTD of these dimensions to output conditional narrative text:

<pre class="brush: plain; title: ; wrap-lines: false; notranslate" title="">// [Narrative (activities)]
IF COUNTD([Selected City Expensive Activities]) &gt; 0 AND COUNTD([Selected City Cheap Activities]) &gt; 0
THEN 'A mixed bag! Some activities are pricey while others are on the cheap side.'
ELSEIF COUNTD([Selected City Expensive Activities]) = 7 
THEN 'Brace yourself! All activities in ' + ATTR([Selected City]) + ' are among the most expensive.'
(etc...)
END
</pre>

In addition to this narrative text, I wanted a bar chart that shows not only how each city ranks within each activity, but also groups the activities as Expensive, Moderate and Cheap. So I created a calculation that returns this string for each city and activity:

<pre class="brush: plain; title: ; notranslate" title="">// [Activity Relative Cost Category]
IF [Activity is Relatively Expensive?] THEN 'Expensive'
ELSEIF [Activity is Relatively Cheap?] THEN 'Cheap'
ELSE 'Moderate'
END
</pre>

Then created a dimension to use on my Columns shelf only if a city is currently selected:

<pre class="brush: plain; title: ; wrap-lines: false; notranslate" title="">// [Selected City Activity Relative Cost Category]
IF [City Parameter] != '0' THEN
{ FIXED [Item] : MAX( IF [City is Selected?] THEN [Activity Relative Cost Category] END ) }
ELSE ''
END
</pre>

The chart&#8217;s columns are sorted by this dimension, so they rearrange themselves when the selected city is changed!<figure class="wp-block-image">

<img src="https://www.bawbgale.com/wp-content/uploads/2019/02/09_activity_chart-1024x556.png" alt="" class="wp-image-308" srcset="https://www.bawbgale.com/wp-content/uploads/2019/02/09_activity_chart-1024x556.png 1024w, https://www.bawbgale.com/wp-content/uploads/2019/02/09_activity_chart-300x163.png 300w, https://www.bawbgale.com/wp-content/uploads/2019/02/09_activity_chart-768x417.png 768w" sizes="(max-width: 1024px) 100vw, 1024px" /> </figure> 

Finally, I’m a big believer in using color very intentionally. I used the practice of first designing the viz with no color at all, then adding color for emphasis. So the initial viz is entirely gray, except for the bright orange “Pick a city” prompt. Once you select a city, that color is used to highlight the data for this city throughout the viz.  


Admittedly, with only 13 cities and 7 activities, it’s not hard to reach these same conclusions by just eyeballing the viz, without the fancy rules-based narrative. But this rules-based approach would scale up to work with a much larger dataset that you can&#8217;t easily eyeball.  


Now that I got my feet wet with Makeover Monday, I plan to participate with new sessions more regularly. You can ﻿view and download [my final viz on Tableau Public](https://public.tableau.com/profile/bob.gale#!/vizhome/MakeoverMonday2018w48/Dashboard1).