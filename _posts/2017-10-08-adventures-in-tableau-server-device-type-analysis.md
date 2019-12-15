---
id: 115
title: Adventures in Tableau Server device type analysis
date: 2017-10-08T15:18:22-08:00
author: Bob Gale
layout: post
guid: http://www.bawbgale.com/?p=115
permalink: /adventures-in-tableau-server-device-type-analysis/
categories:
  - Uncategorized
tags:
  - data analytics
  - python
  - tableau
---
I work on a team that builds Tableau dashboards for hundreds of users spread across the US. We’ve been looking at ways to improve our overall user experience, and have started investigating how many users are accessing our dashboards via desktop vs. mobile devices. Our goal was to make a data-driven decision about how much effort to put into using Tableau’s device-specific layout capabilities. That simple question lead to a small odyssey through the data that Tableau Server collects about itself.

## Finding the right data

The first place we looked is the [Tableau Server workgroups database](https://onlinehelp.tableau.com/current/server/en-us/data_dictionary.html), which contains a wealth of data about dashboard usage. In particular, the `view_stats` table has a `device_type` column that seemed to be exactly what we were looking for.

<pre class="brush: sql; title: ; notranslate" title="">SELECT device_type, SUM(nviews)
FROM views_stats
GROUP BY device_type
</pre>

Or so we thought:

<div id="attachment_116" style="width: 710px" class="wp-caption alignnone">
  <a href="http://www.bawbgale.com/wp-content/uploads/2017/10/tableau_views_by_device_type.png"><img class="wp-image-116 size-large" src="http://www.bawbgale.com/wp-content/uploads/2017/10/tableau_views_by_device_type-1024x584.png" alt="tableau_views_by_device_type" width="700" height="399" srcset="https://www.bawbgale.com/wp-content/uploads/2017/10/tableau_views_by_device_type-1024x584.png 1024w, https://www.bawbgale.com/wp-content/uploads/2017/10/tableau_views_by_device_type-300x171.png 300w, https://www.bawbgale.com/wp-content/uploads/2017/10/tableau_views_by_device_type.png 1130w" sizes="(max-width: 700px) 100vw, 700px" /></a>
  
  <p class="wp-caption-text">
    There’s no way we have that many tablet users
  </p>
</div>

This data showed surprisingly high tablet usage, which did not pass a sniff test. There’s no way we have that many tablet users! I confirmed that these numbers were fishy by browsing from my Windows 10 machine, then querying this table for my `user_id`. I saw that my views were indeed getting classified as “tablet” but another colleague’s Windows 10 views were getting classified as “desktop.”

Doing a little more digging, I determined that this table is recording not actual device type, but Tableau Server’s guess of your device type based on screen size, as described [here](https://onlinehelp.tableau.com/current/pro/desktop/en-us/dashboards_dsd_create.html#Test_the_dashboard_):

<table>
  <colgroup> <col /> <col /></colgroup> <tr>
    <th>
      If the smallest iframe dimension is…
    </th>
    
    <th>
      This device layout appears…
    </th>
  </tr>
  
  <tr>
    <td>
      500 pixels or less
    </td>
    
    <td>
      Phone
    </td>
  </tr>
  
  <tr>
    <td>
      Between 501 and 800 pixels
    </td>
    
    <td>
      Tablet
    </td>
  </tr>
  
  <tr>
    <td>
      Greater than 800 pixels
    </td>
    
    <td>
      Desktop
    </td>
  </tr>
</table>

Another test with my browser window maximized confirmed that this is indeed what’s going on.

Luckily, there’s another place we can look: `http_user_agent` in the `http_requests` table:

<pre class="brush: sql; title: ; notranslate" title="">SELECT http_user_agent
FROM http_requests
WHERE action = 'show'
</pre>

This column contains strings like this:

<pre class="brush: plain; title: ; wrap-lines: false; notranslate" title="">Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; Trident/4.0)</pre>

Anyone who’s worked with Web server logs will recognize this as the standard way each Web browser identifies not only itself (e.g. Chrome, Safari, IE) but also what platform it is running on (e.g. Windows, Mac, iOS, Android).

However, this table presents a couple of challenges: 1) `tabadmin backup` removes all but the last 7 days of data, and 2) the `http_user_agent` string needs to be parsed to extract useful classifications from it.

## Capturing the data

There are three ways we could deal with the truncation problem:

1. Skip truncation during backup: As of Tableau Server version 10.3, server admins can skip the truncation of this table during backup:

<pre class="brush: bash; title: ; wrap-lines: false; notranslate" title="">tabadmin backup --skip-http-truncate E:\backup\test.tsbak</pre>

Of course you will need to be careful that the table does not grow too large.

2. Scheduled export: We could periodically export the data and archive it elsewhere (in either another database or flat files).

3. Incrementally refreshed extract: Within Tableau, we could pull the data into a data source extract that is incrementally refreshed so that new data is captured without removing old data. I decided to try this approach first to see if we can do this totally within Tableau.

## Parsing the data

HTTP user agent strings follow the same general format but vary widely from browser to browser and change as new browser versions are released. So extracting useful categorical information requires some tricky text matching or parsing. You could possibly do this in a Tableau calculated field. Something like:

<pre class="brush: plain; title: ; notranslate" title="">IF CONTAINS(LOWER([http_user_agent]), 'windows') OR CONTAINS(LOWER([http_user_agent]), 'mac') THEN 'desktop'
ELSEIF CONTAINS(LOWER([http_user_agent]), 'ipad') OR CONTAINS(LOWER([http_user_agent]), 'tablet') THEN 'tablet'
ELSEIF CONTAINS(LOWER([http_user_agent]), 'iphone') OR CONTAINS(LOWER([http_user_agent]), 'mobile') THEN 'phone'
ELSE 'other'
END
</pre>

But that’s not only ugly and barely scratches the surface of different user agents, but also would be difficult to maintain as new browser versions are released. Also, why reinvent the wheel? This parsing is already built into numerous Web analytics products like Google Analytics, Web development frameworks, and open source libraries. So I looked around for a Tableau-friendly option.

What I found is an open-source library called [woothee](https://woothee.github.io/), which has implementations for many different programming languages including that favorite of data geeks, Python. Its maintainers regularly update it as new browser versions are released, and it even recognizes known search engine bots, which it labels as “crawler.”

Woothee accepts an HTTP user agent string as input and returns a JSON string containing several useful attributes:

<pre class="brush: python; title: ; notranslate" title="">import woothee
woothee.parse("Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; Trident/4.0)")
# {'name': 'Internet Explorer', 'category': 'pc', 'os': 'Windows 7', 'version': '8.0', 'vendor': 'Microsoft', 'os_version': 'NT 6.1'}
</pre>

So how can we use this in Tableau? Perhaps this is a job for [TabPy](https://github.com/tableau/TabPy), Tableau’s tool for running Python code within Tableau calculations.

## Trying TabPy

TabPy runs as a separate service on your desktop or server and allows Tableau calculations to execute Python code. [Installing TabPy](https://github.com/tableau/TabPy/blob/master/server.md) on your desktop is pretty easy using [Anaconda](https://www.anaconda.com/), an open source, cross-platform framework for installing Python and managing multiple environments with different combinations of Python versions and add-on packages.

Once you get TabPy installed and configured as a Tableau [External Services Connection](https://onlinehelp.tableau.com/current/pro/desktop/en-us/r_connection_manage.html#Configure), you can run Python code in a Tableau calculated field using one of the `SCRIPT_` functions (`SCRIPT_BOOL`, `SCRIPT_INT`, `SCRIPT_REAL` or `SCRIPT_STR`) like this:

<pre class="brush: plain; title: ; notranslate" title="">SCRIPT_STR("Some python code that accepts _arg1, _arg2, etc. and returns a single value", [someTableauField], [anotherTableauField])
</pre>

The Python code can accept one or more values from the Tableau data source and will return a single value that should match the type of the `SCRIPT_` function that you used (boolean, integer, float or string). The input values can be referred to in the Python code as \_arg1, \_arg2, etc. You can either embed all your Python code here, or define it as a function on your TabPy server and call it from Tableau.

This intermingling of Tableau and Python code can get a bit messy so I find it very helpful to use line breaks and indenting to keep them separate and readable:

<pre class="brush: plain; title: ; notranslate" title="">SCRIPT_STR("
	output = 'You passed in this value:'
	output.append(str(_arg1))
	return output
	",
	SUM([Number of Records])
	)
</pre>

Also be careful to use only single quotes in the Python code because the whole thing will be wrapped by double quotes.

Once you have TabPy working, using woothee is pretty simple. Installing TabPy will create an Anaconda environment called “Tableau-Python-Server.” Activate this environment, open a terminal session, and run:

<pre class="brush: bash; title: ; notranslate" title="">pip install woothee
</pre>

Then in a Tableau calculated field, you can pass in your `[Http User Agent]` values and call woothee’s `parse` function. I thought it would be as simple as the following, but soon encountered a few gotchas:

<pre class="brush: plain; title: ; notranslate" title="">SCRIPT_STR("
	import woothee as wt
	return wt.parse(_arg1)  # see the problem?
	",
	[Http User Agent]
	)
</pre>

## TabPy gotchas

First, I’ll mention an overall caveat about running TabPy. If you want to use it with Tableau Server, you will need to host the TabPy server somewhere that your Tableau Server can access at all times. And the TabPy team cautions that it lacks any security layer of its own so is best installed on the same box as your Tableau Server rather than a separate instance that is open to network traffic. So any Tableau Desktop users who want to author TabPy-enabled dashboards will need to install TabPy locally, rather than pointing to one central instance used by Tableau Server.

But back to the code: The most important thing to understand about TabPy calcs is that they are Table Calculations, not row-level calculations. So the inputs are expected to be aggregate data, meaning we need to wrap `[Http User Agent]` with `ATTR()`. It also means that the data passed in will be affected by the dimensions and level of aggregation in your view, so we need to design the view to group by distinct `[Http User Agent]` values, or disaggregate the measures using Analysis > Aggregate Measures > Off.

The problem with my code above is that I was assuming it was being called iteratively over individual `[Http User Agent]` values. In fact, Tableau is passing a list of values to Python, with one value for each row in the view. Fortunately, Python makes it super easy to work with lists using List Comprehensions. So rather than:

<pre class="brush: python; title: ; notranslate" title="">wt.parse(_arg1)
</pre>

We need something like:

<pre class="brush: python; title: ; notranslate" title="">[ wt.parse(s) for s in _arg1 ]
</pre>

And we should also make the code tolerant of the possibility that it will be called from a view where `ATTR([Http User Agent])` returns `'*'` instead of distinct values. So putting these changes together gives us this:

<pre class="brush: plain; title: ; notranslate" title="">SCRIPT_STR("
	import woothee as wt
	return [ wt.parse(s) if s != None else '*' for s in _arg1 ]
	",
	ATTR([Http User Agent])
	)
</pre>

But there’s still a problem here. Woothee returns JSON, for which Tableau does not have a parsing function. It would be awesome if we could return the entire JSON string in a calculation called `[Http User Agent JSON]`, and then access individual values using typical “dot notation” like `[Http User Agent JSON].[category]`. Lacking this, we have two options:

1. We could pass the whole JSON back to Tableau in one calculated field, and then create several other calculations that extract specific attributes using the various string parsing functions that Tableau does support, like `REGEXP_EXTRACT()`. For example:

<pre class="brush: plain; title: ; notranslate" title="">IF [Http User Agent JSON] != "*"
THEN REGEXP_EXTRACT([Http User Agent JSON],'"category": "([^"]+)"')
ELSE "*"
END
</pre>

However … Gotcha! … `REGEXP_EXTRACT()` does not work with Table Calculations!

2. A simpler approach would be to create a separate calculation for each Woothee attribute, and use separate Python functions to extract each attribute. For example, `[Http User Agent Category]` would be:

<pre class="brush: plain; title: ; notranslate" title="">SCRIPT_STR("
	import woothee as wt
	return [ wt.parse(s)['category'] if s != None else '*' for s in _arg1 ]
	",
	ATTR([Http User Agent])
	)
</pre>

Yes, this results in redundant calls to Woothee to parse the same http\_user\_agent, but it requires fewer calculated fields and cleaner code.

However, after all this, we hit the biggest roadblock: Table Calculations cannot be used as Tableau dimensions, which thwarts our ultimate goal of summarizing our usage data by device type. 🙁 Time to step back and re-evaluate this approach.

## What TabPy isn’t

The biggest lesson for me here was understanding the use cases for which TabPy was designed and those for which it was not. TabPy provides a way to use Python to extend Tableau’s analytical capabilities — for example to pass data through a machine-learning model that you trained outside of Tableau. But it does not cover every use case of using Python with Tableau. Python is a very versatile language. It overlaps with R in advanced data science capabilities, but also is a robust scripting and app development language, which is why it is especially popular among data scientists working on online applications vs. say, offline scientific analysis.

My use case is really data prep, not analysis. I want to apply an external function once to every row of incoming data and perform my analysis on the result. And as every Tableau veteran has heard numerous times, Tableau isn’t designed to be an end-all-be-all data prep solution. It really would be cool if Tableau provided a built-in way to run external functions on data as it is imported, but alas it does not. At present, this task is more appropriately done upstream from Tableau in your data pipeline.

## The ETL solution

So in the end, I abandoned the incremental extract and calculated field route and ended up going back to data capture solution #2: exporting the `http_request` data from the Tableau Server workgroups database and transforming it offline before importing it into Tableau. I wrote this in Python and tested it in Jupyter Notebook. The [Pandas data analysis library](http://pandas.pydata.org/) made it super easy to remove duplicate rows after multiple successive exports. I currently run it locally on an ad hoc basis, but it could easily by automated in [Alteryx](https://www.alteryx.com/), [Luigi](https://github.com/spotify/luigi) or any other Python-compatible ETL framework.

I split the process into three scripts. One that exports the current `http_requests` table to a date-stamped CSV:

<pre class="brush: python; title: ; wrap-lines: false; notranslate" title="">import pandas as pd
import os
import json
from sqlalchemy import *
from sqlalchemy.engine.url import URL

# A database credentials are in `config.json`
with open('config.json') as f:
    conf = json.load(f)
engine = create_engine(URL(**conf))

sql = "SELECT created_at, session_id, user_id, http_user_agent FROM _http_requests WHERE action = 'show'"

df = pd.read_sql_query(sql,con=engine)

# Save the data frame as a CSV in an `input` subdirectory with file name like `tableau_http_requests_YYYY-MM-DD.csv`

path = os.getcwd()
out_filename = r'tableau_http_requests_{}.csv'.format(pd.to_datetime('today').strftime("%Y-%m-%d"))
out_location = os.path.join(path, "input", out_filename)

df.to_csv(out_location, index=False, quoting=1) # 1 = QUOTE_ALL
</pre>

One that combines the latest CSV with previous exports and dedupes:

<pre class="brush: python; title: ; wrap-lines: false; notranslate" title="">import pandas as pd
import glob
import os
import datetime

headers = ['created_at', 'session_id', 'user_id', 'http_user_agent']
dtypes = {'created_at': 'str', 'session_id': 'str', 'user_id': 'str', 'http_user_agent': 'str'}
parse_dates = ['created_at']

# Get all the CSVs in the `input` subdirectory

path = os.getcwd()
all_files = glob.glob(os.path.join(path, "input", "*.csv"))

# Read into data frame
df_from_each_file = (pd.read_csv(f, sep=',', header=1, names=headers, dtype=dtypes, parse_dates=parse_dates) for f in all_files)

# Dedupe
df_dedupe = df_from_each_file.drop_duplicates()

# Save the data frame as a CSV in an `output` subdirectory with file name like `tableau_http_requests_combined.csv`

path = os.getcwd()
out_filename = r'tableau_http_requests_combined.csv'
out_location = os.path.join(path, "output", out_filename)

df_dedupe.to_csv(out_location, index=False, quoting=1) # 1 = QUOTE_ALL
</pre>

And finally one that parses the user agent string into new columns:

<pre class="brush: python; title: ; wrap-lines: false; notranslate" title="">import pandas as pd
import os
import datetime
import woothee as wt

# Set up file locations
path = os.getcwd()

in_filename = r'tableau_http_requests_combined.csv'
in_location = os.path.join(path, "output", in_filename)

out_filename = r'tableau_http_requests_parsed_{}.csv'.format(pd.to_datetime('today').strftime("%Y-%m-%d"))
out_location = os.path.join(path, "output", out_filename)

# Import the previously exported data
headers = ['created_at', 'session_id', 'user_id', 'http_user_agent']
dtypes = {'created_at': 'str', 'session_id': 'str', 'user_id': 'str', 'http_user_agent': 'str'}
parse_dates = ['created_at']
df = pd.read_csv(in_location, sep=',', header=1, names=headers, dtype=dtypes, parse_dates=parse_dates)

# Parse http user agent and append as new columns
df[['category','name','os','os_version','vendor','version']] = pd.read_json( (df.http_user_agent.apply(wt.parse)).to_json(), orient='index' )

# Save the result as CSV
df.to_csv(out_location)
</pre>

Finally, here is the resulting truer picture of our device type usage:

<div id="attachment_153" style="width: 710px" class="wp-caption alignnone">
  <a href="http://www.bawbgale.com/wp-content/uploads/2017/10/tableau_views_by_user_agent_category.png"><img class="wp-image-153 size-large" src="http://www.bawbgale.com/wp-content/uploads/2017/10/tableau_views_by_user_agent_category-1024x582.png" alt="That's much closer to the mobile usage pattern I expected" width="700" height="398" srcset="https://www.bawbgale.com/wp-content/uploads/2017/10/tableau_views_by_user_agent_category-1024x582.png 1024w, https://www.bawbgale.com/wp-content/uploads/2017/10/tableau_views_by_user_agent_category-300x170.png 300w, https://www.bawbgale.com/wp-content/uploads/2017/10/tableau_views_by_user_agent_category.png 1130w" sizes="(max-width: 700px) 100vw, 700px" /></a>
  
  <p class="wp-caption-text">
    That’s much closer to the mobile usage pattern I expected
  </p>
</div>

I was not surprised to see much lower mobile device usage than our initial analysis showed. However, I was puzzled at why any views on our internal dashboards would be categorized as “crawler,” but it turns out there is a reasonable explanation. The hits were all on a single dashboard and came from this HTTP user agent:

<pre class="brush: plain; title: ; notranslate" title="">Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko; Google Web Preview) Chrome/41.0.2272.118 Safari/537.36</pre>

This appears to be from the Google Chrome browser’s default startup page, which shows thumbnail images of your frequently accessed web pages. This means one of our users is hitting a dashboard so often that it ranks among his top visited websites, so Chrome is returning to collect a souvenir postcard to post on his virtual refrigerator! 😀

## Other approaches

Yet another route we could pursue is analyzing [Tableau Server’s log data](https://onlinehelp.tableau.com/current/server/en-us/logs_loc.htm). Tableau offers a free utility called [Logshark](https://www.tableau.com/about/blog/2016/11/introducing-logshark-analyze-your-tableau-server-log-files-tableau-62025) that parses these logs and generates Tableau workbooks. We could also feed those logs to a robust log collection and analytics platform like [Splunk](https://www.splunk.com/), which not only has its own front-end for viewing data but also has a native connector for Tableau. With these approaches, looking at device type data would be merely the tip of the iceburg. They enable not only comprehensive analysis of Tableau Server usage, but also are a great help for troubleshooting issues that users may be experiencing with Tableau Server dashboards.

Also, it’s important to keep in mind Tableau Server’s distinction between device type and screen size. While it’s good to be aware of how many of your users access your dashboards on desktops vs tablets vs phones, how you adapt your dashboards to serve those users is ultimately a function of screen size. If you use Tableau’s device layout features to adapt your dashboards for different screen sizes, Tableau Server will display the version that best fits the user’s browser window, regardless of whether the device displaying it is a desktop, tablet or smartphone.