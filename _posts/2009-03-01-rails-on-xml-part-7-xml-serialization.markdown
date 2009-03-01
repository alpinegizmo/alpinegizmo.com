---
layout: post
title: "Rails on XML, Part 7: XML serialization (a gem)"
---
[Part of the [Rails on XML](/2009/01/02/Rails-on-XML-the-series.html) series.]

An essential part of working with XML and Ruby is being able to serialize ruby objects to XML. There are several approaches out there for doing this; the one that appeals to me most is the one implemented by Rails in ActiveSupport. It provides a nice to_xml method for your ActiveRecord objects, and deals with arrays and hashes that contain ActiveRecord models. For example, here's what you might get from a really simple blog engine in response to `Page.find(:first).to_xml`:

    <?xml version="1.0" encoding="UTF-8"?>
    <page>
      <allow-comments type="boolean" nil="true"></allow-comments>
      <body>blah blah blah</body>
      <id type="integer">1</id>
      <title>my first post</title>
      <created-at type="datetime">2008-01-08T17:52:54+01:00</created-at>
      <updated-at type="datetime">2008-11-26T17:01:51+01:00</updated-at>
      <user-id type="integer">1</user-id>
    </page>

There may be more information here than we will need for XSLT rendering (eg it probably doesn't matter that id is an integer), but I like the thoroughness of this approach, and the way they handle arrays and hashes is quite nice.

For our purposes we need to be able to serialize to XML anything we might put in a controller instance variable. The [xml_serialization gem](http://github.com/alpinegizmo/xml_serialization/tree/master) adds support for integers, symbols, and strings, and arrays and hashes that contain them.

For example, `{:numbers => [1, 2], :strings => ['one', 'two']}.to_xml`
    
    <?xml version="1.0" encoding="UTF-8"?>
    <hash>
      <numbers type="array">
        <number>1</number>
        <number>2</number>
      </numbers>
      <strings type="array">
        <string>one</string>
        <string>two</string>
      </strings>
    </hash>

We also cover the important case where you already have XML (eg from a database or web service) that you want to pass through unmolested. 

    @data = RawXML.new '<tag>content</tag>'

In this case, @data.to_xml returns the same string that went in, i.e. '&lt;tag>content&lt;/tag>'.
