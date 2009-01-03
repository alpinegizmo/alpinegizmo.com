---
layout: post
title: "Rails on XML, Part 1: Background"
---

I'm sure the first question in many readers' minds is "Why
bother?". The intersection between the XML community and the Ruby
community is very small. A rubyist *might* use XML to implement a
RESTful web service, but that's going far enough, thank you very
much. However, if you've been paying attention to the pure XML
workflow for creating web applications, it has some interesting ideas,
and some exuberant followers. I'm not sure I'm tapped into the most
forceful propaganda for this way of thinking, but you can sample it
here, in this quote from 2006:

> A profound change is likely about to shake up your world if you’re a
  web developer, one that I suspect will make the recent efforts in the
  AJAX space pale in comparison as far as its effect. Very quietly, over
  the last few weeks, the Mozilla team has been upgrading their XForms
  capabilities ...
  
> In point of fact, you can create some incredibly rich “web
  experience” applications that are mixed XHTML and XForms, and can do
  so in a remarkably short amount of time. These are not simply empty
  text fields that you lay out as you do with HTML input elements --
  XForms bears about as much resemblance to HTML forms as a Bengal
  tiger has to a ferret. [http://www.oreillynet.com/xml/blog/2006/03/why_xforms_matter_revisited.html](http://www.oreillynet.com/xml/blog/2006/03/why_xforms_matter_revisited.html)
  
and in this quote from 2007:

> [E]nlightened innovators ... have seen the real world benefits of
  dumping object middle-tier stacks and relational databases and going
  with a pure declarative approach to solving business problems based
  around XForms/REST and XQuery. These [are] the people that are going
  to lead innovations in application development. ...

> At one of the XForms sessions a young Ruby advocate stated that he
  thought his Ruby code was "beautiful" but he did not himself think
  that XForms code was "beautiful". Most of the people in the audience
  agreed that XForms was beautiful but like any new language, it takes
  getting used to. ...

> XForms/REST/XQuery to me is indeed beautiful. Its beauty comes from
  its ability to quickly map real-world requirements into working
  systems with high fidelity. No other system in the world approaches
  its elegant architecture. [http://datadictionary.blogspot.com/2007/12/impressions-of-xml-2007-in-boston.html](http://datadictionary.blogspot.com/2007/12/impressions-of-xml-2007-in-boston.html)

To put these remarks in context, I need to backup and give an outline
of the entire XML web application toolchain.

* XML database: your application data lives here, in XML
* XQuery: database query language, it takes the place of SQL
* XSLT: transforms XML to HTML (can be applied in the DB, your app server, or in the browser)
* XForms: forms that post XML back to the database

Note that if you buy into all of this, you don't really need an
application server any more, be it implemented in Rails, or not. (If you want to play with some examples of this pure XML web app architecture, I think the sample apps distributed with [eXist](http://exist.sourceforge.net/), an open source, native XML database, are the best place to start.)

Of course it is now 2009, and we've yet to see the XForms/REST/XQuery approach gain traction in the way you might've expected, if you bought into what was being said a couple of years ago. Yes, in some ways this XML architecture for  web application development is truly elegant -- but some pieces of the puzzle are rather hideous. 

I'll have more to say about what's good, and bad, in future postings. I'll also talk about various approaches we explored for combining these tools with Rails.

