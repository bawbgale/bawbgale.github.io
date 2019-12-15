---
id: 222
title: Wrangling photos into Tableau (with a stopover in serverless functions)
date: 2019-01-14T16:48:12-08:00
author: Bob Gale
layout: post
guid: https://www.bawbgale.com/?p=222
permalink: /wrangling-photos-into-tableau-with-a-stopover-in-serverless-functions/
categories:
  - Uncategorized
tags:
  - aviation
  - aws
  - azure
  - data visualization
  - javascript
  - serverless
  - tableau
---
[In my last post](/visualizing-a-story-buried-in-faa-data/), I walked through how I built a Tableau Public viz about the amazing number of aircraft built in 1946 that are still flying. This post digs into how I incorporated some nifty aircraft photos into the viz.<figure class="wp-block-image">

<img src="https://www.bawbgale.com/wp-content/uploads/2019/01/13_caption-1024x307.png" alt="" class="wp-image-233" srcset="https://www.bawbgale.com/wp-content/uploads/2019/01/13_caption-1024x307.png 1024w, https://www.bawbgale.com/wp-content/uploads/2019/01/13_caption-300x90.png 300w, https://www.bawbgale.com/wp-content/uploads/2019/01/13_caption-768x231.png 768w" sizes="(max-width: 1024px) 100vw, 1024px" /> </figure> 

First the underlying question: Why? I mean, of course _I_ love to look at pictures of airplanes. That’s why the wallpaper on my devices looks like this:<figure class="wp-block-image">

<img src="https://www.bawbgale.com/wp-content/uploads/2019/01/01_wallpaper-1-1024x561.png" alt="" class="wp-image-224" srcset="https://www.bawbgale.com/wp-content/uploads/2019/01/01_wallpaper-1-1024x561.png 1024w, https://www.bawbgale.com/wp-content/uploads/2019/01/01_wallpaper-1-300x164.png 300w, https://www.bawbgale.com/wp-content/uploads/2019/01/01_wallpaper-1-768x421.png 768w, https://www.bawbgale.com/wp-content/uploads/2019/01/01_wallpaper-1.png 1184w" sizes="(max-width: 1024px) 100vw, 1024px" /> <figcaption>Aviation geek wallpaper. Bonus points if you can guess what I named each device.</figcaption></figure> 

But beyond self-indulgence, what value do the photos add to the viz? Data visualization is about creating compelling visuals from _data itself_. Are photos just eye-candy? Non-data ink? Ben Jones provided one answer in his presentation “[Using Data Visualization to Inform and Inspire](https://www.slideshare.net/dataremixed/using-data-visualization-to-inform-and-inspire)”: Using powerful imagery increases the _memorability_ of a data visualization.

Beyond memorability, showing photos of the top aircraft models built in 1946 helps convey what made these aircraft so popular. They established the basic form of private aircraft that has defined the category ever since: single-engine, propeller-driven monoplanes that are easy to fly and cheap to mass produce. Despite being more than 70 years old, they don’t look like museum relics. The photos help convey that part of the story. So I envisioned building a little slideshow of images that changes when you click on a chart of the top aircraft models. 

Being an aviation geek, I knew just where to find the photos. JetPhotos.com is a crowd-sourced repository of aircraft photos of all types (not just jets) taken by avid plane-spotting photographers. You can find photos of just about any aircraft if you know its “tail number.” 

But how to display the photos as a slideshow in Tableau? This is not really the sort of display that Tableau was designed for. But the Tableau community has always found clever ways to make Tableau jump through all kinds of hoops. The Information Lab has a [good overview of the different techniques for adding images to Tableau](https://www.theinformationlab.co.uk/2014/08/21/using-images-tableau/), but in a nutshell, they are:

  1. Shapes mark
  2. Background image
  3. Image object
  4. Web Page object

Each has its pros, cons and best uses, but for a data-driven display of multiple large images, Web Page object is the way to go. This allows a dashboard action (i.e. clicking on an aircraft models bar chart) to cause a parameterized URL to display in another section of the dashboard. This has the added benefit of allowing me to display the image files from the JetPhotos.com site, rather than downloading and including the files in my viz.

At first glance, this seemed pretty straightforward. JetPhotos.com doesn’t have a public API for looking up photos, but their URL scheme is relatively transparent. If you know the tail number of a particular aircraft, you can tack it onto a URL to find any photos they have of that plane. <figure class="wp-block-image">

<img src="https://www.bawbgale.com/wp-content/uploads/2019/01/02_jetphotos_url-1-1024x426.png" alt="" class="wp-image-226" srcset="https://www.bawbgale.com/wp-content/uploads/2019/01/02_jetphotos_url-1-1024x426.png 1024w, https://www.bawbgale.com/wp-content/uploads/2019/01/02_jetphotos_url-1-300x125.png 300w, https://www.bawbgale.com/wp-content/uploads/2019/01/02_jetphotos_url-1-768x319.png 768w, https://www.bawbgale.com/wp-content/uploads/2019/01/02_jetphotos_url-1.png 1121w" sizes="(max-width: 1024px) 100vw, 1024px" /> </figure> 

However, this serves up a full web page with a multiple photos and lots of surrounding navigation, rather than a nice clean individual image. Not very pretty in a Tableau dashboard.

Digging through the HTML of the JetPhotos.com pages, I was able to extract individual image URLs, but this presented another problem: Unlike page URLs, the image URLs are cryptic — i.e. not at all derivable based on tail number. <figure class="wp-block-image">

<img src="https://www.bawbgale.com/wp-content/uploads/2019/01/03_jetphotos_image_url-1024x474.png" alt="" class="wp-image-227" srcset="https://www.bawbgale.com/wp-content/uploads/2019/01/03_jetphotos_image_url-1024x474.png 1024w, https://www.bawbgale.com/wp-content/uploads/2019/01/03_jetphotos_image_url-300x139.png 300w, https://www.bawbgale.com/wp-content/uploads/2019/01/03_jetphotos_image_url-768x356.png 768w, https://www.bawbgale.com/wp-content/uploads/2019/01/03_jetphotos_image_url.png 1121w" sizes="(max-width: 1024px) 100vw, 1024px" /> </figure> 

Extracting image URLs based on tail numbers would require some web scraping code.

At this point, I decided to punt. To get my viz out the door, I used a temporary hack: I manually looked up 10 images for the 10 aircraft models that I wanted to feature and created a calculated field that returned the appropriate image URL based on aircraft model:<figure class="wp-block-image">

<img src="https://www.bawbgale.com/wp-content/uploads/2019/01/04_photo_url_hack.png" alt="" class="wp-image-228" srcset="https://www.bawbgale.com/wp-content/uploads/2019/01/04_photo_url_hack.png 575w, https://www.bawbgale.com/wp-content/uploads/2019/01/04_photo_url_hack-300x177.png 300w" sizes="(max-width: 575px) 100vw, 575px" /> </figure> 

Then I set up a URL Action with a reference to this field as the URL:<figure class="wp-block-image">

<img src="https://www.bawbgale.com/wp-content/uploads/2019/01/05_photo_url_action.png" alt="" class="wp-image-229" srcset="https://www.bawbgale.com/wp-content/uploads/2019/01/05_photo_url_action.png 503w, https://www.bawbgale.com/wp-content/uploads/2019/01/05_photo_url_action-280x300.png 280w" sizes="(max-width: 503px) 100vw, 503px" /> </figure> 

That let me publish my viz while I turned my attention to the challenge of automatically looking up photo URLs for any tail number in the FAA database. That’s when things got really interesting!

## Automating the Web scraping

Manually looking up photo URLs was fine for a few images, but I wanted a more scalable solution. The viz would be hosted on Tableau Public, so I wanted any click on the top aircraft chart to trigger an image lookup on the fly. To help with this, I consulted my friend and colleague [Ian Cole](http://colecoding.com). Together, we wrote up some Node.js code that takes a tail number as a parameter, requests a page from JetPhotos.com, and parses the first image URL out of the HTML. This relies on a pair of NPM packages (Axios and Cheerio) to retrieve the HTML page and extract the image URL from the DOM:

<pre class="brush: jscript; title: ; wrap-lines: false; notranslate" title="">'use strict';
const request = require('axios');
const cheerio = require('cheerio');

module.exports.getjetphoto = async (event, context, callback) =&gt; {
  const tailNum = event.tailNum;
  return request(`https://www.jetphotos.com/photo/keyword/${tailNum}`)
    .then(({data}) =&gt; {
      const $ = cheerio.load(data);
      const photoElements = $('img.result__photo');
      const photos = [];
      photoElements.each((i, el) =&gt; {
        photos.push($(el).attr('src'));
      });
      var html = `&lt;html&gt;&lt;img style="max-width: 500px;" src="${photos[0]}"/&gt;&lt;/html&gt;`;
      context.succeed(html);
    })
    .catch((err) =&gt; {
      context.succeed(err.response);
    });
};
</pre>

We tested the code locally and it worked like a charm. 

For hosting this code, we thought this would be a great scenario for a “serverless” function on AWS Lambda — i.e. instead hosting (and paying for) full round-the-clock web hosting, the snippet of code would run on-demand, and we would be charged per request (and only if traffic exceeds AWS’s generous free tier of one million requests per month, which was unlikely).

We also used this as an occasion to try out the [Serverless Framework](https://serverless.com), an open-source project to develop a platform-agnostic interface for all the major cloud providers. Once we set up the appropriate account credentials and permissions on AWS, the framework took care of a lot of the nuts-and-bolts of deploying both a Lambda function and an API Gateway endpoint. 

But when we deployed it to AWS, we hit a roadblock: A 403 error and a Captcha security check page:<figure class="wp-block-image">

<img src="https://www.bawbgale.com/wp-content/uploads/2019/01/07_aws.png" alt="" class="wp-image-230" srcset="https://www.bawbgale.com/wp-content/uploads/2019/01/07_aws.png 726w, https://www.bawbgale.com/wp-content/uploads/2019/01/07_aws-300x162.png 300w" sizes="(max-width: 726px) 100vw, 726px" /> <figcaption>AWS Lambda gets wanded</figcaption></figure> 

Looked like perhaps JetPhotos.com was blocking AWS IP addresses, probably to prevent bots from extracting all of their photos.

So here was our chance to see if the Serverless Framework lived up to its platform-agnostic goal. If it didn’t work on AWS, maybe it would work on Azure. We found that the Serverless Framework made this easier but not effortless. I’ll go into more details in a follow up post, but bottom line, we deployed to Azure, and it worked! No IP blocking here (for now).<figure class="wp-block-image">

<img src="https://www.bawbgale.com/wp-content/uploads/2019/01/08_azure.png" alt="" class="wp-image-231" srcset="https://www.bawbgale.com/wp-content/uploads/2019/01/08_azure.png 724w, https://www.bawbgale.com/wp-content/uploads/2019/01/08_azure-300x116.png 300w" sizes="(max-width: 724px) 100vw, 724px" /> <figcaption>Azure Function gets waved through (though in the slow lane)</figcaption></figure> 

However, there were still a few gotchas with this approach:

  * Cold start performance: The beauty of serverless is that you pay only per invocation of your function, not for continually running a server. The downside is that if your function has not been invoked in awhile, there can be a slight lag as the cloud provider re-deploys your function to a server. This is typically less than a second, but in the case of Azure, we found it could be upwards of 30 seconds. Not good!
  * Repeatedly reparsing the same pages: In the simplest implementation, every click on the top aircraft models chart would re-request and re-parse the same page, which seemed wasteful and redundant. Yes, we could implement caching of frequent requests, but that starts to get more complex.
  * Dealing with missing data: While JetPhoto.com’s photo collection is quite extensive, it by no means covers every tail number in existence. So some tail numbers would display a “no photo” message rather than a nice picture. 

This lead to a rethinking of the approach: Instead of parsing URLs on-the-fly, what about pre-fetching a batch of URLs that could be included with the dashboard’s data source? That brings us to the next chapter of the story. 

## Next stop: Batch mode

Using our web scraping code in batch mode required a bit of adaptation: Reading the tail numbers from a CSV file instead of a command line, and outputting to CSV rather than an HTML page snippet. 

The first step was a bit of refactoring. Both modes needed the same HTTP request and HTML parsing code, so it made sense to split that into a retriever module, separate from the differing input and output modes. Here’s a condensed version of how I structured that:

<pre class="brush: jscript; title: ; wrap-lines: false; notranslate" title="">// handler.js
const retriever = require('./retriever.js')

module.exports.getjetphoto = async (event, context, callback) =&gt; {
  return retriever.getjetphotos(event.tailNum, (result) =&gt; {
    context.succeed(wrapHtml(result[0]))
  })
}

function wrapHtml (photo) {
	// html stuff
}
</pre>

<pre class="brush: jscript; title: ; wrap-lines: false; notranslate" title="">// retriever.js
const request = require('axios')
const cheerio = require('cheerio')

module.exports.getjetphotos = async (tailNum, callback) =&gt; {
  return request(jetPhotosUrl(tailNum))
    .then(({ data }) =&gt; {
      callback(extractPhotos(data))
    })
    .catch((err) =&gt; {
      callback(err.response)
    })
}

function jetPhotosUrl (tailNum) {
	// jetphotos url stuff
}

function extractPhotos (data) {
	// cheerio stuff
}
</pre>

This set things up nicely to add a batcher module to handle reading and writing CSVs. This was greatly facilitated by the [PapaParse](https://www.papaparse.com) library. But because each call to the retriever is asynchronous (i.e. returns a Promise that is resolved when the async call finishes), I collected all the Promises in an array, then used Promises.all to wait for them all to be resolved before writing the output files: 

<pre class="brush: jscript; title: ; wrap-lines: false; notranslate" title="">// batcher.js
const fs = require('fs')
const Papa = require('papaparse')
const retriever = require('./retriever.js')

module.exports.getjetphotobatch = (inputFile = 'tail_numbers.csv', tailNumCol = 'Tail Number') =&gt; {
  // use fs and PapaParse read input CSV file
  fs.readFile(inputFile, 'utf8', (err, fileData) =&gt; {
    Papa.parse(fileData, {
      complete: (results) =&gt; {
        // do some validation 

        let aircraftListWithPhotos = results.data.map(row =&gt; {
          // build an array of promises that will get resolved when retriever responds
          return new Promise((resolve, reject) =&gt; {
            retriever.getjetphotos(row[tailNumCol], (result) =&gt; {...}
          }
        })

        // after all promises resolved, write output 
        Promise.all(aircraftListWithPhotos).then(aircraftListWithPhotos =&gt; {

          // create a log file with all the original rows plus status column
          let aircraftStatusList = ...

          // push all the photo urls rows to a separate array
          let photosUrlList = ...

          // output to CSVs named as the input file plus a suffix
          outputCsv('status', aircraftStatusList)
          outputCsv('photos', photosUrlList)
        })

        const outputCsv = (suffix, data) =&gt; {
          // Use PapaParse and fs to output the data to CSVs
        }
      }
    })
  })
}
</pre>

When this was all ready to go, I discovered another gotcha: Rate limiting! Not surprisingly, any popular website will prevent too many repeated requests from the same source. I wanted to respect that, so after a bit of trial and error, I was able to determine the rate limit threshold (no more than 25 requests every 2 minutes) and adapted the code request small batches, then pause for a cool-down period. The [Bottleneck](https://www.npmjs.com/package/bottleneck) library was a big help here. All I had to do is set up the Bottleneck timing options, then wrap the retriever call in a function that takes care of pausing and resuming accordingly:

<pre class="brush: jscript; title: ; wrap-lines: false; notranslate" title="">// batcher.js
...
const Bottleneck = require('bottleneck')
...

// set up Bottleneck options

let bottleneckRefreshInterval = 100 * 1000 // must be divisible by 250
let bottleneckOptions = {
  maxConcurrent: 1,
  minTime: 200,
  reservoir: 25, // initial value
  reservoirRefreshAmount: 25,
  reservoirRefreshInterval: bottleneckRefreshInterval
}
const limiter = new Bottleneck(bottleneckOptions)

module.exports.getjetphotobatch = (inputFile = 'tail_numbers.csv', tailNumCol = 'Tail Number') =&gt; {

	// all the same csv parsing and item iterating here	

	// wrap the retriever call with limiter.submit(). Bottleneck takes care of the rest!
	limiter.submit(
		retriever.getjetphotos, tailNum, (result) =&gt; {...}
	)
	
}
</pre>

So with the batch mode working, I was able to collect almost 400 photo URLs for the 10 aircraft models that I was featuring in my viz. Each aircraft model’s photos were in a separate CSV, so back in Tableau I joined them with the my aircraft data using a wildcard union:<figure class="wp-block-image">

<img src="https://www.bawbgale.com/wp-content/uploads/2019/01/12_union_csvs-1024x652.png" alt="" class="wp-image-232" srcset="https://www.bawbgale.com/wp-content/uploads/2019/01/12_union_csvs-1024x652.png 1024w, https://www.bawbgale.com/wp-content/uploads/2019/01/12_union_csvs-300x191.png 300w, https://www.bawbgale.com/wp-content/uploads/2019/01/12_union_csvs-768x489.png 768w" sizes="(max-width: 1024px) 100vw, 1024px" /> </figure> 

This also gave me the option of displaying multiple photos per aircraft model — i.e. potentially every tail number of that model in the JetPhotos collection. I played with a few ideas for choosing the “best” photo of each model, but settled on a simpler scheme of choosing the newest (which conveniently was the highest ID number).

As a final touch, I wanted to display not only the aircraft image, but also the tail number and photographer’s name in a caption. So I modified the web scraping code to read the photographer’s name as well as the URL, then used another dashboard action to display that as a caption below the photo.<figure class="wp-block-image">

<img src="https://www.bawbgale.com/wp-content/uploads/2019/01/13_caption-1024x307.png" alt="" class="wp-image-233" srcset="https://www.bawbgale.com/wp-content/uploads/2019/01/13_caption-1024x307.png 1024w, https://www.bawbgale.com/wp-content/uploads/2019/01/13_caption-300x90.png 300w, https://www.bawbgale.com/wp-content/uploads/2019/01/13_caption-768x231.png 768w" sizes="(max-width: 1024px) 100vw, 1024px" /> <figcaption>Final Top Aircraft Models chart with slideshow and caption on Tableau Public</figcaption></figure> 

In the end, I was able to populate my viz with a nice slideshow of aircraft photos, explore a number of possible solutions, and produce some versatile aircraft photo retrieval code that I can use for future aviation vizes. All the resulting code is on [GitHub](https://github.com/bawbgale/jetphotos-translator) if you want to explore it in more detail.