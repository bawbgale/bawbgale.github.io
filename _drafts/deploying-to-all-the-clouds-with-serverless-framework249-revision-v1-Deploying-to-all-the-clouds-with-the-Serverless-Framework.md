---
id: 260
title: Deploying to all the clouds with the Serverless Framework
date: 2019-01-21T05:34:01+00:00
author: Bob Gale
layout: revision
guid: https://www.bawbgale.com/249-revision-v1/
permalink: /249-revision-v1/
---
In [my last post](/wrangling-photos-into-tableau-with-a-stopover-in-serverless-functions/), I described how I implemented some web scraping code to wrangle aircraft photos into Tableau. One possible approach was a serverless function (i.e. a snippet of cloud-hosted code that executes on-demand) to retrieve a photo of a particular aircraft in response to a click on the Tableau viz. In the end, I opted to pre-fetch the photo URLs using a batch process, but I gained some useful experience building and deploying serverless functions to different cloud platforms. This post digs how I did this using the Serverless Framework.

The [Serverless Framework](https://serverless.com/framework/) is an open-source project introduced in 2015 designed to automate some of the grunt work of deploying serverless functions. It originally targeted only [AWS Lambda](https://aws.amazon.com/lambda/) but has since grown to support other &#8220;Function as a Service&#8221; providers such as [Azure](https://azure.microsoft.com/en-us/services/functions/) and [Google Cloud](https://cloud.google.com/functions/), as well as [IBM/Apache OpenWhisk](https://console.bluemix.net/openwhisk/), [Oracle Fn](https://fnproject.io), [Kubeless](https://kubeless.io), [SpotInst](https://api.spotinst.com) and [Cloudflare](https://www.cloudflare.com/products/cloudflare-workers/). It seeks to provide a platform-agnostic interface into the parts of these platforms that are functionally equivalent (so to speak) but are defined using different formats. By making functions more portable, it helps address one of the biggest concerns organizations have when building in the cloud: How to avoid “platform lock-in” &#8212; i.e. being too dependent on one platform, and thus vulnerable to its pricing and availability constraints.

(Of course, you might ask whether this is just moving your dependency to another player, and a less known/established one at that. Fair point. The framework is maintained by Serverless Inc., a privately held, investor-backed company. However, it is open-sourced under the MIT license, has a healthy community of contributors, and doesn’t do any magic beyond translating your configuration into the target platform’s resource management format.)

The Serverless Framework also encourages the best practice of defining infrastructure as code. All the cloud platforms provide user interfaces for setting up services, but defining them in configuration files makes your setup far more reproducible, testable and shareable. There are other, more generic frameworks for achieving this, but which is right for you depends on whether your situation is best served by a Swiss Army knife or a single-purpose blade. 

So when my colleague Ian Cole and I set out to try the serverless approach to aircraft photo scraping, it made sense to try to Serverless Framework and to see if it truly makes it easy to move a function from one platform to another. We found that it was easy but not effortless. Below are the main steps I had to follow for AWS, Azure and Google Cloud, as well as some gotchas to beware of.

## Getting Started

The Serverless Framework uses Node.js, so even if your function is not written in Node.js, you will need to install it, then `npm install -g serverless`. The framework supports AWS out of the box, but if you’re deploying to Azure or Google, you’ll need to `npm install` the appropriate plugin. Then you can run `npm create --template {platform and language you’re using} --path {what you want to call it}` to generate scaffolding to get you started. 

## Account Setup

With all three services, you’ll need to set up an account and install local credentials to give the framework permission to deploy code on your behalf. This is inherently platform-specific but well documented for each platform in the Serverless [Getting](https://serverless.com/framework/docs/providers/aws/guide/credentials/) [Started](https://serverless.com/framework/docs/providers/azure/guide/credentials/) [Docs](https://serverless.com/framework/docs/providers/google/guide/credentials/).

If you don’t already have an account, all three offer generous free tiers and trial periods. AWS and Azure give you 1 million free requests per month. Google gives you 2 million. In addition, Azure gives you a $200 credit to spend on any services, but it expires after 30 days. Google gives you a $300 credit that lasts 12 months. AWS doesn’t have a trial credit, but many of its other services have free tiers that you can use to try them out.

One other pricing difference to note: While all three offer the requisite on-demand pricing model based on number of requests and compute time, Azure also offers fixed “App Service” pricing if you want to provision your function on an alway-on virtual machine.

## Serverless.yml

The heart of the Serverless Framework is the serverless.yml config file, which is generated when you run `serverless create`. For a basic function, the serverless.yml configs are remarkably compact and similar across platforms:

<pre class="brush: plain; title: ; wrap-lines: false; notranslate" title="">service: my-unique-name

provider:
  name: aws
  runtime: nodejs8.10
  
functions:
  hello:
    handler: handler.hello
</pre>

<pre class="brush: plain; title: ; wrap-lines: false; notranslate" title="">service: my-unique-name

provider:
  name: azure
  location: West US

plugins:
  - serverless-azure-functions

functions:
  hello:
    handler: handler.hello
    events:
      - http: true
        x-azure-settings:
          authLevel : anonymous
      - http: true
        x-azure-settings:
          direction: out
          name: res
</pre>

<pre class="brush: plain; title: ; wrap-lines: false; notranslate" title="">service: my-unique-name # NOTE: Don't put the word "google" in here

provider:
  name: google
  runtime: nodejs
  project: my-project
  credentials: ~/.gcloud/keyfile.json

plugins:
  - serverless-google-cloudfunctions

functions:
  hello:
    handler: hello
    events:
      - http: path
</pre>

These are just the bare-bones required settings. Many other options can be specified if you want to vary from defaults. 

## Function Portability

In addition to platform-specific differences in the serverless.yml, your function itself needs to be tailored somewhat to the specific platform. All three platforms support Node.js functions, but their method signatures and response formats are slightly different: 

<pre class="brush: jscript; title: ; wrap-lines: false; notranslate" title="">'use strict';

module.exports.hello = async (event, context) =&gt; {
  return {
    statusCode: 200,
    body: JSON.stringify({
      message: 'Go Serverless v1.0! Your function executed successfully!',
      input: event,
    }),
  };
</pre>

<pre class="brush: jscript; title: ; wrap-lines: false; notranslate" title="">'use strict';

module.exports.hello = function (context) {
  context.log('JavaScript HTTP trigger function processed a request.');

  context.res = {
    // status: 200, /* Defaults to 200 */
    body: 'Go Serverless v1.x! Your function executed successfully!',
  };

  context.done();
};
</pre>

<pre class="brush: jscript; title: ; wrap-lines: false; notranslate" title="">'use strict';

exports.http = (request, response) =&gt; {
  response.status(200).send('Hello World!');
};

exports.event = (event, callback) =&gt; {
  callback();
};
</pre>

Our function was pretty generic and didn’t rely on any other cloud services, like databases, file storage or message queues. So I expected it to be pretty portable across platforms. It was initially written for AWS Lambda, so porting it to Azure required a minor rewrite. Porting to Google Cloud required yet another. I had already refactored the core retriever logic out of the handler function, but now I had three different versions of the handler, including repeated error handling code. I refactored this one more time to keep the absolute minimum in the platform-specific handlers.

(Code snippet)

Dependencies on other common cloud services like file storage could be injected in a similar fashion. Obviously, relying on any very platform-specific features (like some of the amazing Machine Learning as a Service features [links]) will limit the portability of your code. 

## Testing

The Serverless CLI allows you to invoke your function locally (though it did not work for Google?) as long as there are no dependencies on other online resources &#8212; i.e. it does not provide local mocks for any other cloud services that your function might be calling. But isolating and mocking such dependencies should be part of your automated test coverage. Refactoring my core logic out of the handler module made it easier to write test cases using mocha and chai, and mock dependencies with sinon, moxios and mock-fs.

## Deploy

Once you’re ready to deploy, running \`serverless deploy\` takes care of generating all the platform-specific elements and pushing your code to the cloud. You can inspect all the deployment assets that it creates in the invisible .serverless directory, and looking at the resulting AWS CloudFormation or Google Resource Manager assets helps you appreciate the complexity that the framework spares you from. &nbsp;[Nothing generated for Azure?]

However, I encountered one major gotcha when trying to deploy the same function on multiple providers: There is currently no way to specify a different config file when you run \`serverless deploy\`. (I contributed to a pull request to add this CLI option, but as of this writing it is not merged.) As a workaround, you can use a simple bash script to swap config files when you deploy:

<pre class="brush: bash; title: ; wrap-lines: false; notranslate" title="">#!/bin/bash
if [ "$1" != "" ]; then
  echo "copying serverless-$1.yml to serverless.yml and running serverless deploy"
  cp serverless-$1.yml serverless.yml && sls deploy
else
    echo "Please append provider, like 'deploy.sh aws' or 'deploy.sh azure'"
fi
</pre>

## Performance Differences

Once I adapted the JS code to suit each service and deployed it, they all “functioned” as expected. But I did notice some performance differences. Azure had a much slower cold start performance. (Record some actual benchmarks after varying degrees of idle time) (Record differences in deployment time)

Note these are unscientific measurements. I have not tried to exhaustively benchmark the services under controlled conditions.

## A few other details

Two other minor things to note:  


Serverless does not pay attention to your .gitignore file. Every file in your directory will get deployed unless you add exclude rules to your serverless.yml  


Each service exposes different names in the function console and URL. (Some code snippet with highlighting?)

  * AWS
  * Azure
  * Google

## Conclusion

All in all, I found the Serverless Framework to be an easy on-ramp for trying out different cloud providers. The experience also spurred me to write more platform-independent code, which is always a good thing. All of these platforms are evolving fast, so it behooves you to trying them out, use design patterns that can translate across platforms, and avoid platform lock in.