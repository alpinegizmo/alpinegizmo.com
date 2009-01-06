---
layout: post
title: "Cowboys and Waterfalls: Why research labs don't produce good software"
---

I spent the first twenty-plus years of my career working as a researcher in university and corporate research labs -- at Carnegie Mellon, Mitsubishi Electric (MERL), and Sun Labs. It has been my privilege to work on some great projects with some amazingly talented people. But I have to tell you that by and large, the quality of the software coming out of research labs isn't very high. I blame the cowboys and the waterfalls.

## So what goes wrong, you ask?

Most researchers are pretty good programmers, *in their domain of expertise*. Some aspect of the software they create is likely to be really well done -- the efficiency of the algorithms, the correctness of the protocols, the usability. But other aspects of the code don't get as much care. Very few researchers make a significant effort to hone their general software development skills. It's common to go straight from graduate school into a research job, never having worked in a product group. Just to give you the flavor, it's not uncommon for cvs and svn to be considered too much trouble (rather than pains in the neck that ought to be replaced by git or mercurial or darcs -- but that's another topic entirely). 

From what I've seen, most research lab directors will try to put some serious programming talent wherever it will do the most good, but corporate research labs often have considerable difficulty finding and keeping talented developers. Several factors contribute to this:

* *Vagueness*. How a research project may impact products, who the customers will be, etc, may not be clearly understood. Developers who are used to getting written specs from the marketing department, or consultants who expect the client to provide a detailed requirements document often feel at sea in this environment. (The upside is that there is plenty of room for creativity.)
* *Low probability of success*. For the researchers, success can come in many forms -- papers, patents, peer recognition. For the programmers who help them realize their ideas, success means seeing their code get used, either in an open source project or a product. The chance of this happening can be quite low.
* *Inappropriate structure and leadership*. Corporate research labs are often part of a vast enterprise. Management of development resources in these research labs is usually left either to scientists who aren't particularly knowledgeable about industry-best practices in software engineering, or to managers rotated in from some other part of the company, whose experience is in a very different context.
* *Cowboys and waterfalls*. The few really great coders in any research lab are likely to either be cowboys who can create amazing things overnight, but who lack the discipline to create code with lasting value -- or they may be believers in the "big upfront design" school of software development (aka the "[waterfall](http://en.wikipedia.org/wiki/Waterfall_model)" model). Management usually loves both the cowboys and the serious architects, because both appear to get things done, and are sometimes successful (though real long-term results are rare in most cases). Flashy demos, and grand schemes described in lengthy documents (and patents) can be impressive.

## What would be better?

Agile methods of software development (and extreme programming practices) are an excellent match for a research environment, but I've seldom seen them applied in that setting in anything better than a haphazard manner. (I am assuming you have some familiarity with the basics of these methods of going about software development -- if not, see the resources at the end of this article for some pointers to further reading.)

Here are some reasons why agile methods are a natural fit for research-driven software development:

* _Constant flow of new ideas_. It's part of the nature of research to be constantly adjusting your plan to fit with what you've just learned, and agile methods are all about adaptability. About ten years ago, while working as a researcher at Mitsubishi, I read in one of Tom Peters books that at Sony, the average time it takes to go from an idea to a prototype is one week. I was flabbergasted, but intrigued. Most research ideas fail -- your goal should be to fail faster!
* _Small teams_. The conventional wisdom is that agile methods work well in small teams, but  may be difficult to scale. Not really an issue in most research environments.
* _Variable scope is ok_. When push comes to shove and your project threatens to be late, as a manager you have three rational choices: move back the deadline, reduce the scope of work, or reduce the quality (a [death march](http://www.amazon.com/Death-March-Second-Edward-Yourdon/dp/013143635X/ref=pd_bbs_sr_1/104-3830814-3606322?ie=UTF8&s=books&qid=1180904769&sr=8-1) isn't a rational choice). 

And some reasons why there might be problems:

* _No customer (or plausible stand-in) to drive decision-making_. 
* _Lack of training and mentors_.

## Advice to research directors

Foster a climate of sustainable development. Occasional crunches to meet deadlines are natural, but don't let it get out of hand. Communicate that in general, choosing to keep quality high while reducing the scope of the work is the right choice. Putting in week after week at a punishing pace inevitably leads to poor quality.

For at least the key projects, find someone to actively participate in the role of the customer, be it a real customer, or more likely a product manager or a business development guru. Have them meet with the researchers and developers on a weekly basis to review progress and new ideas and plan the coming week or two.

Hire one or more experienced XP/agile practitioners into your lab to serve as mentors, with the goal of establishing these key practices: (1) pairing, (2) test-driven development, and (3) capturing ideas as "[stories](http://www.extremeprogramming.org/rules/userstories.html)" with estimates. Working in short iterations is also important, but should come naturally. Be encouraging, but keep it relatively low-key and trust your staff to take advantage of the opportunity to adopt new ideas if they see them working.

When I look back on the projects I worked on as a researcher, the most regrettable aspect of the code I wrote is the lack of tests. If I could do one thing differently, I'd write tests. 

## Resources

If you're not familiar with agile methods or extreme programming (XP), here are some good places to start.

[agilemanifesto.org](http://agilemanifesto.org/)  
[www.xprogramming.com/xpmag/whatisxp.htm](http://www.xprogramming.com/xpmag/whatisxp.htm)    
[wikipedia on agile software development](http://en.wikipedia.org/wiki/Agile_software_development)   
[wikipedia on extreme programming](http://en.wikipedia.org/wiki/Extreme_Programming)  

