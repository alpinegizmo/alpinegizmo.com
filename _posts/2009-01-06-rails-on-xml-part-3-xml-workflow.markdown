---
layout: post
title: "Rails on XML, Part 3: XML workflow"
---
A bit of overly simplified, sample XML. Don't get hung up on the details; the schema and data are made up. I just want to give you something concrete to look at so I can raise a few issues for you to consider. 

    <property>
      <type>office space</type>
      <area>100</area>
      <rooms>
        <room size="60" />
        <room size="30" />
        <room size="10" type="bathroom" />
      </rooms>
      <owner>
        <name>John Q. Public</name>
        <contactinfo>
          <phones>
            <phone number="1.555.555.1212" type="voice" use="contact" />
            <phone number="1.666.555.1212" type="mobile" use="contact" />
          </phones>
        </contactinfo>
      </owner>
      <location>
        <position lat="40.5000801086426" lng="-3.37295293807983" />
        <address>
          <street>123 Main St</street>
          <postalcode>28805</postalcode>
        </address>
      </location>
    </property>

## Creating, editing, and validating the XML

If you are thinking of writing an XML-based web application, where is the XML going to come from? How will it be edited? Where will you implement the business logic to validate that the data being entered makes sense?

## Changes to the structure/schema of the XML (migrations)

How will you manage the evolution of the XML schema over time -- in other words, what will take the place of Rails migrations? Will you validate XML schema changes against an XML DTD?

Will (some) users be able to make changes to the XML schema, or will this require programmer intervention?

### &nbsp;

Finding good solutions to the problems of validation and migration is a key challenge when considering Rails on XML. In our application we generated the XML from other sources, and thereby sidestepped these issues, at least for a while.
