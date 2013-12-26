---
layout: post
title: 'Using The Crunchbase API'
url: http://matrix.hackeducation.com/2013/03/16/using-the-crunchbase-api/
image: https://s3.amazonaws.com/kinlane-productions/api-evangelist/crunchbase/crunchbase-logo.png
---

<a href="http://developer.crunchbase.com/" target="_blank"><img class="c1" src="https://s3.amazonaws.com/kinlane-productions/api-evangelist/crunchbase/crunchbase-logo.png" alt="" width="200" align="right" /></a>

<a href="http://hackeducation.com/">Audrey</a> came to me last night and said she had a project that she wanted to tackle, using the <a href="http://developer.crunchbase.com/">CrunchBase API</a>. She wanted to pull a list of education startups that were founded in 2010-2012, showing their investments, CEO, founders and other related company information.

A couple weeks ago, I helped Audrey download a PHP Twitter bot, reverse engineer and make it work for an objective she had around Tweeting random responses to a certain type of tweet. I figured she was ready to see what it took to hack on an API and get the research data she needed.

Audrey wanted to pull a list of startups that were educated related. We started with the /search endpoint, using the query keyword “education”. The query parameter appears to search the title, tags and overview of each Crunchbase entry, pulling way more than what we needed, 5700 in total. Each search returned 10 startups, with a handful of fields for each company, not nearly enough information.

To prove we could get what we needed, we took the first startup returned, grabbed the name and used it to perform a company /entity search, returning a wealth of information. We had what we were looking for including date founded, amount raised, CEO, founders as well as description, logos, addresses and other vital company information.

Back to the /search endpoint. Each search only returned 10 results, when there were actually 5,700. I looked at the documentation and the only parameter listed, was “query”. No way to control how many results returned, paginate or otherwise. So I start guessing, appending page=, and max=, maxrows=, anything to get more results. Eventually I was able to change the page, so I was able to write logic to loop through each page, getting at all the results required.

<img class="c2" src="https://s3.amazonaws.com/kinlane-productions/api-evangelist/crunchbase/crunchbase-search-endpoint.png" alt="" width="500" />

For each returned result from the /search endpoint, I would take the company name and perform a company search on the /entity endpoint. However each call to the endpoint would yield a 301, redirecting me to another URL. So the search for:

<blockquote>
 <em>http://api.crunchbase.com/v/1/company/Noodle+Education.js?api_key=[key]</em>
</blockquote>

 Get a 301 returned, with the following redirect URL:

<blockquote>
 <em>http://ec2-107-21-104-179.compute-1.amazonaws.com/v/1/company/noodle-education.js</em>
</blockquote>

 So for each /entity endpoint I would have to make two separate cURL calls to get a single record for a company. An approach that is guarantee to get CrunchBase into the API billionaires club (as chatty APIs do), or at the very least generate a nice bill with their API management provider.


 Next, I needed to create a table to store the data in, I looped through the JSON object and spit out a list of fields, so I could generate a SQL script to create the table. There were a few duplicate fields, but quickly got a table up and the data inserted. The data is decent. There are a lot of issues with absence of default values, empty objects with available images, addresses, etc. But nothing I’m not used to wading through, to get what I need from an API.


 With the /search endpoint based upon a single keyword search, that spans title, overview and tags, the search returned a lot of companies that didn’t fit our search. As we prepared to enter each record into a database, we would apply further filters, making sure there companies founded in 2010 or greater, and met other criteria, before actually inserting.


 Now we had a MySQL table filled with 772 education startups. I exported as CSV, uploaded as a Google Spreadsheet, and shared it with Audrey. Game over. She had what she needed. Overall it took about 3 hours start to finish, which she felt was tedious, and feeling there was a lack of clarity regarding the whole process--something she couldn’t have done with me and my API programming skills. This is a problem CrunchBase.


 <a href="http://developer.crunchbase.com/" target="_blank"><img class="c1" src="https://s3.amazonaws.com/kinlane-productions/api-evangelist/crunchbase/startup-ecosystem-visualization.png" alt="" width="325" align="right" /></a>


 <span class="c3">Audrey is your target audience</span>. This needs to be dead simple. She shouldn’t have to blindly navigate the API endpoints, documentation, data structures and holes in the data. These are things I’m used to having to cobble together, but an analyst like herself, won’t have the time and patience.


 I can tell the CrunchBase API was created by someone who didn’t give a shit about the data, let alone use the API to build anything. I saw a job posting from CrunchBase, so I know they are looking for someone to own the API. I hope this person can become a steward for this great set of data, and make it into a meaningful API--there is a lot of potential there.


 Quick feedback for whoever takes the job, regarding some building blocks to focus on:
