---
layout: post
title: (im)pureXML
---

Rather than producing a native XML database like eXist or MarkLogic, IBM has chosen to add XML features to DB2, creating an interesting -- and convoluted -- hybrid that IBM has branded "pureXML".

If you've drunk the XML database koolaid, you may be thinking about how going down this path is going to (1) give you a toolchain where web app development becomes a much more declarative process, with better separation of concerns; (2) give you more flexibility than a relational database; (3) fit more naturally with a web-services model; and/or (4) get you moving toward the semantic web. 

For me, one of the attractive prospects would be to clean up the process of generating HTML. If the data coming out of your database is already XML, then its not much of a leap to use an XSLT to transform that XML into HTML. Traditional view templating schemes (such as erb) can be a mess, so maybe this world of XML databases offers a cleaner solution. 

Architecturally you've got three choices of where to apply an XSL transform -- in the database, in your application server, or in the browser. Each of these has advantages and disadvantages, but as a first pass we've been exploring having DB2 do the job. The results have been somewhat disappointing.

To get DB2 to apply an XSL stylesheet to some XML, you have to use an SQL function called XSLTRANSFORM. The XSL stylesheet has to be provided as the result of an SQL expression, which generally means that it is stored in an XML column in the database. The XML document to be transformed might also come from an XML column, or it might be the result of an XQUERY embedded in the SQL you've been forced to use as the overall structure for this operation. Given this structure it's not surprising that xsl:include declarations are not supported, but this is definitely problematic. Also disappointing is that stylesheets incorporating XQUERY statements are not supported. 

A simple example in which the XML to be transformed is coming from the xmldata column in the employees table:

{% highlight sql %}
SELECT XSLTRANSFORM(xmldata 
  USING (SELECT xsldata FROM xslts WHERE name = 'employee.xsl')) AS html 
FROM employees WHERE id = 833038373
{% endhighlight %}

It's a little more fun when the source of the XML is an embedded xquery:

{% highlight sql %}
SELECT XSLTRANSFORM (
 (SELECT XMLCAST(XMLQUERY('$data//Phones' PASSING xmldata AS "data") AS XML) 
   FROM employees WHERE id = 833038373) 
 USING (SELECT xsldata FROM xslts WHERE name = 'phones.xsl')) AS html 
FROM employees WHERE id = 833038373
{% endhighlight %}

It may look a bit strange that the WHERE id = 833038373 clause occurs twice. The second instance could be pretty much anything that returns one row. While many SQL databases will accept a simple "select 1", DB2 insists on a FROM clause, such as "select 1 from employees fetch first 1 rows only". The only really important part of the last line above (i.e. "FROM employees WHERE id = 833038373") is that it return exactly one row.

XSLTRANSFORM isn't the only case where DB2's pureXML forces you to use an unholy mixture of SQL and XQUERY. As IBM explains, "With plain XQuery you can not exploit full-text search capabilities provided by the DB2 Net Search Extender (NSE). You need to involve SQL for full-text search." This means doing something like this:

    xquery 
      <Employees> { 
        for $e in db2-fn:sqlquery(
          "SELECT xmldata FROM EMPLOYEES.EMPLOYEES 
           WHERE CONTAINS(xmldata, 
             'SECTION(""/Employee/Info/Address/LocationInfo"") ""madrid"" ') > 0"
        ) 
        let $ei := $e/Employee/Info 
        return 
          <EmployeeInfo> 
            { $ei/Name, <Departments> {$ei/DepartmentInfo/Name} </Departments> } 
          </EmployeeInfo> 
      } </Employees>

For clarity, this example has been broken across muliple lines. Don't expect DB2 to be happy with input this easy to read.

 This whole business of XQUERY embedded in SQL, and vice versa, can get amazingly messy. We've spent quite a bit of time figuring out how to do quoting and casting in order to keep both sides happy. And although DB2 supports SQL sub-queries within XQUERY, so far we've found it impossible to figure out how to quote SQL sub-queries within an XQUERY that in turn is embedded in SQL, which is necessary if you want to use XSLTRANSFORM on the result of an xquery like the one above.

Resources:

<a href="http://www.ibm.com/developerworks/db2/library/techarticle/dm-0606nicola/">pureXML in DB2 9: Which way to query your XML data?</a>  
<a href="https://publib.boulder.ibm.com/infocenter/db2luw/v9r5/index.jsp?topic=/com.ibm.db2.luw.sql.ref.doc/doc/r0050650.html">DB2 9.5 documentation</a>  
<a href="http://del.icio.us/gizmo/db2">my DB2 links on del.icio.us</a>  
<a href="http://del.icio.us/gizmo/xml">my XML links on del.icio.us</a>  
<a href="http://datadictionary.blogspot.com/search/label/XForms">on the prospects for transforming web app development with XForms, REST and XQuery</a>  
<a href="http://cafe.elharo.com/xml/the-state-of-native-xml-databases/">short reviews of various XML databases</a>  
