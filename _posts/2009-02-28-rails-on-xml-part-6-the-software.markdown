---
layout: post
title: "Rails on XML, Part 6: The software"
---
[Part of the [Rails on XML](/2009/01/02/Rails-on-XML-the-series.html) series.]

I am pleased to report that Mirai España has agreed to release as free software the code we wrote for using XSLT views in Ruby on Rails applications. I have packaged this software as two separate components: the [xml_serialization gem](http://github.com/alpinegizmo/xml_serialization/tree/master) for serializing ruby objects to XML, and the [xslt_render rails plugin](http://github.com/alpinegizmo/xslt_render/tree/master).

At this point documentation is lacking. However, there isn't that much code and the tests do illustrate how the pieces are intended to be used, so the adventurous might be able to drop this into an app and get it working. I do plan future posts that will explain more of the details, which I'm sure will help. However, I know a few of you are interested in seeing the code now, so I don't want to delay in releasing it.
