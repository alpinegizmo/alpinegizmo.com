---
layout: post
title: "Rails on XML, Part 2: Data modeling"
---
[Part of the [Rails on XML](/2009/01/02/Rails-on-XML-the-series.html) series.]

Among the first design decisions to be made with any web application are how to represent the data, and where to store it. The assumption behind this set of articles is that you are going to have XML data, probably in some sort of XML database. But why might someone come to this conclusion, and what are the alternatives?

For many applications, modeling the data for a relational database is pretty straightforward, especially after you've done it many times before. Customer records, addresses, phone numbers, orders, login-ids and passwords -- we all know what to do with this stuff. Moreover, there are good tools available, and there are lots of talented people around who know how to use them. Both MySQL and PostgreSQL have their weak points, but are nevertheless pretty well-behaved and a small team isn't going to waste much time fighting with either one of them. Rails, with ActiveRecord, migrations, and validations, provides a good foundation for keeping complex applications relatively sane.

By contrast, choosing an XML data representation is much riskier. The tools are less mature, and few there be that know how to use them. Don't do it unless you have a good reason.

One obvious domain for applying the XML web application toolchain is for working with an existing set of XML documents. I suspect this is a relatively rare case, however.

As a motivating example, imagine being asked to build a commercial real estate site that will have detailed listings for business properties for rent and for sale: farms, office space, retail space, restaurants, hardware stores, etc. New types of properties will be added to the site on a regular basis -- six months from now you may encounter the first listing for a marina, or a mine. The web site is expected to have rich editing and search interfaces, making it possible, for example, to search for bars for sale with seating for more than 50 customers, or for a 2-room office with an on-street entrance and its own bathroom.

In this case, modeling each property as an XML document seems like an interesting idea. Attributes that many or all of the properties have can be managed uniformly (eg listing agent, asking price, size in square-meters, taxes), and more unique characteristics (eg number of berths of various lengths in a marina) can also be represented, edited, and searched.

Once you have decided to use XML for at least some of your data, you have to decide if you are going to be a purist and use XML for everything, or not. The cleanliness of a pure approach appealed to us, but didn't seem practical. We already had a login system we liked, based on restful authentication and open-id; we planned to use attachment-fu for uploaded images; and in general we wanted to leave the door open for any and all Rails plugins. Our client had already chosen to use DB2, which supports hybrid use of both XML and SQL, and we decided to take advantage of that.

Another interesting choice that I've been playing with, but haven't used for anything real, would be to use one of the new document-oriented data stores, such as [CouchDB](http://couchdb.apache.org/). It is my belief -- but I can't completely justify it -- that if you are planning to exploit the hierarchical structure that an XML document allows you to use, then XQuery is going to win over what you can do with CouchDB.
