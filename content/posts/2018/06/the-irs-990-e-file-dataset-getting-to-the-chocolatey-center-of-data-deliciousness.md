+++
author = "David Bruce Borenstein, PhD"
authorAvatar = "/uploads/david_circle_small.png"
authorFacebook = ""
authorGoogle = ""
authorImage = "/uploads/images/david-1.png"
authorInfo = "David Bruce Borenstein, Ph.D., is a partner at Applied Nonprofit Research and Chief Technology Officer at Open990. As the Director of Data Science at Charity Navigator, he was closely involved in the creation of the IRS 990 Community Concordance. David’s background is originally in computational genomics, which informs his approach to data curation."
authorTwitter = ""
date = "2018-06-04T04:00:00+00:00"
description = "Learn to extract, transform, and interpret data from the IRS 990 e-file dataset, a corpus of over two million tax returns from hundreds of thousands of nonprofits since 2010."
tags = ["dataset", "open government", "nonprofit technology", "open data", "data science"]
thumbnail = "/uploads/efile_xml.png"
title = "The IRS 990 e-file dataset: getting to the chocolatey center of data deliciousness"

+++

**Updated June 26, 2020**

## Background, or “What the hell am I looking at?”

In June 2016, the IRS [unleashed a torrent](https://www.irs.gov/newsroom/irs-makes-electronically-filed-form-990-data-available-in-new-format) of electronic data on the nonprofit sector. Blandly dubbed the “[IRS 990 filings](https://registry.opendata.aws/irs990/),” these millions of files each contain hundreds of details about a nonprofit’s spending, revenue, compensation, fundraising and more.

{{< figure src="/uploads/nfl_990.png" class="figureWithCaption" caption="The beginning of the [2015 IRS Form 990 filing](https://projects.propublica.org/nonprofits/download-filing?path=2016_07_EO%2F13-1922622_990O_201506.pdf) for the [National Football League](https://www.nfl.com), which is exempt from income tax under [Section 501(c)(6)](https://www.irs.gov/charities-non-profits/other-non-profits/business-leagues) of the United States [Internal Revenue Code](https://en.wikipedia.org/wiki/Internal_Revenue_Code)." >}}

The data itself wasn’t new: the IRS has been collecting these disclosures [since 1940](https://en.wikipedia.org/wiki/Form_990). Nor had the public been denied access: scans of the paper records have been [sold by the IRS](https://www.irs.gov/charities-non-profits/copies-of-scanned-eo-returns-available) for years, and are available for free from [archive.org](https://archive.org/details/IRS990).

What was new was the availability of electronic files in their _native format_ of [XML](https://www.w3schools.com/xml/xml_whatis.asp). This meant that, for the first time, researchers could analyze the data directly, without a costly image-to-text ([OCR](https://en.wikipedia.org/wiki/Optical_character_recognition)) step — or worse, manual data entry.

A couple months before the data release, I had just started a job at [Charity Navigator](https://www.charitynavigator.org/), which rates nonprofits based on nonprofit tax filings. (I now work at[ 990 Consulting](https://www.990consulting.com/), which hosts the free service [Open990](https://www.open990.com/).) Charity Navigator has a hardworking [team](https://www.charitynavigator.org/index.cfm?bay=content.view&cpid=19) of data analysts, who historically typed in [critical 990 data](https://www.charitynavigator.org/index.cfm?bay=content.view&cpid=33) by hand before crunching the numbers. Needless to say, the upcoming release was a Very Big Deal.

{{< figure src="/uploads/datathon.jpg" class="figureWithCaption" caption="The original [990 “datathon”](https://aws.amazon.com/blogs/publicsector/how-the-nonprofit-open-data-collective-came-together-to-work-on-irs-990-data-in-the-cloud/) event at the [Aspen Institute](https://www.aspeninstitute.org/blog-posts/aspen-institutes-program-philanthropy-social-innovation-psi-hosts-nonprofit-datathon/) in Washington, D.C. Volunteers spent two days creating “mappings” between the paper form and the electronic version. A follow-up validation event was called — you guessed it — the [990 validatathon](https://www.aspeninstitute.org/blog-posts/aspen-hosts-990-vali-datathon-part-philanthropys-data-revolution/)." >}}

Yet it took more than a year — and [two](https://www.aspeninstitute.org/blog-posts/aspen-institutes-program-philanthropy-social-innovation-psi-hosts-nonprofit-datathon/) [rounds](https://www.aspeninstitute.org/blog-posts/aspen-hosts-990-vali-datathon-part-philanthropys-data-revolution/) of cross-organizational barn-raising— before nearly anyone was able to do much of anything with these data. There are several reasons for this. In this article, I will go over some of the challenges a would-be user would face, and how we’ve solved them at [Open990](https://www.open990.com/).

## What's inside the box

Although the dataset is called “IRS 990 filings,” there are three separate tax forms contained inside it: the 990, the 990EZ, and the 990PF.

{{< figure src="/uploads/990_comparison.png" class="figureWithCaption" caption="The first page of (left) the IRS 990, which is filed for tax-exempt organizations with significant cash flows or specially regulated functions; (center) the IRS 990EZ, for organizations with small — but not extremely small — cash flows; and (right) the IRS 990PF, which is filed by all private foundations. All three kinds of filing appear in the “IRS 990 filings” dataset." >}}

Most tax-exempt organizations are required to annually file a [Form 990](https://www.irs.gov/forms-pubs/about-form-990)(“Return of Organization Exempt from Income Tax”) with the IRS. If you’re a small-ish organization that isn’t subject to special regulations (as are schools and hospitals), you can file the [IRS Form 990EZ](https://www.irs.gov/forms-pubs/about-form-990ez), which is the short form version of the 990. But the EZ is not just an excerpt of the full form: the numbering is different, the descriptions are different, the order is different, and some items have been rolled together in ways that are definitely not designed to be easily compared.

If you’re really small, you don’t file a return at all. Instead, you file an electronic notice (“e-Postcard”), also called the Form 990N. This is a [separate dataset](https://www.irs.gov/charities-non-profits/tax-exempt-organization-search-bulk-data-downloads), and it contains very little information — basically a name and an address.

Private foundations — regardless of size — file a different document called the [Form 990PF](https://www.irs.gov/forms-pubs/about-form-990pf). The distinction between public charities (which comprise a large fraction of Form 990s) and private foundations is [not always cut and dried](https://www.irs.gov/charities-non-profits/eo-operational-requirements-private-foundations-and-public-charities), and organizations sometimes switch from one type to the other.

While more than half of the dataset consists of IRS 990 filings, the EZs and the PFs are not to be ignored.

{{< figure src="/uploads/filings_by_year.png" class="figureWithCaption" caption="The 990 dataset is not just for full Form 990s: it contains hundreds of thousands of filings from smaller organizations (which file the 990EZ) and from private foundations (which file the 990PF). Many of these e-filings also contain various schedules, which add details about specific aspects of the organization." >}}

There are over 120 thousand organizations for which we have seven years’ worth of filings, making it a treasure trove for comparative data. ([Open990](https://www.open990.com/), my social enterprise, specializes in providing such comparative data.)

{{< figure src="/uploads/filing_count_histo.png" class="figureWithCaption" caption="Filings nominally date back to 2011, but there are numerous filings from earlier than that, typically because of amended returns. There are over 260,000 organizations with at least five years of e-filed data, making the 990 dataset a goldmine for longitudinal analysis." >}}

## Understanding schema versions

Stop me if you’ve heard this one: you just spent hours (or days) staring at 990 filings in XML format, finding the specific locations (or “[Xpaths](https://www.w3schools.com/Xml/xpath_intro.asp)”) for the data you want. Then you go to run your import script, and…it doesn’t work on half the filings. Why? I’ll tell you why.

{{< figure src="/uploads/efile_xml.png" class="figureWithCaption" caption="The beginning of [a 990 filing](https://s3.amazonaws.com/irs-form-990/201541349349307794_public.xml), as viewed in Mozilla Firefox. The “returnVersion” inside the “return” tag indicates what version of the schema is being used. In general, the first four digits should indicate the form year, though 2017 filings have largely been using 2016v3.0 and 2016v3.1." >}}

Look closely at the XML above. See the very first tag, called <Return>? It has an attribute, returnVersion, that you may reasonably have decided to ignore. Don’t.

Every time the U.S. Congress changes the federal tax code, IRS forms have to change. On top of that, the electronic format is an imperfect map from the paper to the electronic worlds, and the IRS is always trying to improve it. As a side effect, it is also always changing the format of the data.

Most forms had a major revision in mid-2013. If you can live with post-2012 data, drop all versions prior to 2013v3.0 and you should be good. (In this case, you may also be able to use an off-the-shelf tool, [IRSx](https://github.com/jsfenfen/990-xml-reader). Read on.)

If you need to look at all the versions, see if you can get what you need from the [Community Concordance](https://github.com/Nonprofit-Open-Data-Collective/irs-efile-master-concordance-file). This work was painstakingly assembled by volunteers at two events in Washington, DC, and is reasonably reliable. Once you find what fields you need, you can download a long-form data file containing the value of that field for each 990 from [Open990](https://www.open990.com/).

If you still can’t find what you need, then you’ll need to start getting into the weeds. The IRS used to make schema files available [on its website](https://www.irs.gov/e-file-providers/current-valid-xml-schemas-and-business-rules-for-exempt-organizations-modernized-e-file). They have since been taken down, but Charity Navigator [still hosts the old ones on GitHub](https://github.com/CharityNavigator/990_metadata/tree/master/schema). New ones are only available from the IRS upon request.

The same GitHub repository contains a Python program that scans all of the returns, counts up the occurrences of each Xpath by version, and produces a report. Combining this with the metadata from the schemas, you can pretty much figure out everything you need. This tool was definitely a first draft — it depends needlessly on Apache Spark and doesn’t find everything in the schema. But it’s something.

## Is the 990 dataset really “big” data?

[Apache Spark](https://spark.apache.org/), part of the [Hadoop](http://www.bmc.com/guides/hadoop-ecosystem.html) ecosystem, is used for [unimaginably huge datasets](https://databricks.com/session/netflix-productionizing-spark-on-yarn-for-etl-at-petabyte-scale). By comparison, the entire 990 dataset can fit on the hard disk of most laptops. As of this writing, there are around 2.4 million filings, most of which have only a few hundred data points. That’s not nothing, especially as those data points are deeply and inconsistently nested. But it’s not that big.

{{< figure src="/uploads/data_points_per_filing.png" class="figureWithCaption" caption="Most returns have only a few hundred data points, though a very small number of returns have many thousands. The largest returns do not fit into a single MongoDB record, even after converting to JSON." >}}
So why was I using it for the Charity Navigator 990 projects? Inexperience, for starters: I had never seen a dataset like this. But it was also because finding what you want in millions of XML files takes a really long time.

There are XML-specific databases that you can use, of course: [eXist](http://exist-db.org/exist/apps/homepage/index.html) and [BaseX](http://basex.org/)are among the better known. This might be the right way to go for some use cases, and if yours is one of them, [I’d love to hear about it](https://www.open990.com/contact/). For most people, though, I think this is all missing the point.

**The 990 dataset is nothing but hierarchical data. In 2018, developers use**[**JSON**](https://www.w3schools.com/js/js_json_intro.asp) **for hierarchical data, and store JSON in** [**NoSQL databases**](https://www.infoworld.com/article/3240644/nosql/what-is-nosql-nosql-databases-explained.html)**.**

My very first step in any 990 analysis is now to ditch the XML files. There are [lots of nice formats](http://wiki.open311.org/JSON_and_XML_Conversion/) for representing XML as JSON, so choose one and move on with your life.

This will speed up your analysis by an order of magnitude, but now you’re left with a new problem: without XML, how will I handle all those Xpaths? If you’re working in Python, the answer turns out to be a new library called [glom](https://github.com/mahmoud/glom). This package lets you write Xpath-like queries against arbitrarily nested Python data structures. Neat!

## A note on S3

The IRS’ [documentation concerning the AWS dataset](https://docs.opendata.aws/irs-990/readme.html) provides instructions for transferring the 990 filings individually by HTTPS. Doing that requires an [HTTPS handshake](https://robertheaton.com/2014/03/27/how-does-https-actually-work/) for every single filing. To speed things up, copy the filings using the [S3 protocol](https://aws.amazon.com/documentation/s3/), either via the [AWS command line interface](https://aws.amazon.com/cli/) or a library like [boto3](http://boto3.readthedocs.io/en/latest/).

**UPDATE 2020-06-26: I no longer recommend using the IRS indices. Instead, I recommend listing the filings directly, using parallelism to speed things up. Read my why and how [here](/posts/2020/06/skip-the-irs-990-efile-indices/).**

~~Either way, please don’t do anything that forces S3 to give you a directory listing, such as the “sync” or “ls” commands. There are millions of little files in a single directory.* This is not a good design, and that is going to be reflected in the performance of these commands. Instead, download an index file (in [JSON](https://s3.amazonaws.com/irs-form-990/index_2011.json) or [CSV](https://s3.amazonaws.com/irs-form-990/index_2011.csv)) from the IRS and then get each file (or “object”) explicitly.~~

_* Technically, S3 doesn’t have files or folders. In practical terms, of course it does._

## Lists, tables, and inconsistent field names

You’ve gotten all your data into an efficient local database somewhere. You’ve wrapped your head around the schema, created some crosswalk for you to find the stuff you’re looking for, and you’re ready to party. Then you actually start trying to use the data — and that’s when you discover that it’s really, really messy.

These returns were generated via one of at least [20 different programs](https://www.irs.gov/e-file-providers/tax-year-2017-exempt-organizations-efile-providers), each of which must follow a [230-page specification](https://www.irs.gov/pub/irs-pdf/p4164.pdf). That specification includes some stuff that’s clearly designed to mirror what appears on the paper, and other stuff that’s nothing like what’s on paper.

There’s no shortcut for this problem: you’re just going to have to make thoughtful decisions about what it is you’re seeing.

If you don’t feel like doing that, you can always [hire us](https://www.open990.com/pro/)! But if you’re reading this, you are probably the DIY sort. For those intrepid souls, here are some cases that you’ll probably run into. (Credit goes to my co-founder [Heather Kugelmass](https://www.open990.com/contact/) for figuring these out.)

**Separate _parent_ Xpaths for each item in a list**

Part III of the IRS 990 is a list of the top three programs offered by a public charity. To a data scientist, this is a list of similar things. To the IRS, each of these things is a separate line on a form (4a, 4b, 4c). So they each deserve a unique Xpath, right? Well, probably not, but that’s what you get. Also, there’s a catch-all for “other” that may or may not have as much data associated with it as the other three, even though it looks like it shouldn’t on the paper version.

When you see this, you must normalize it away by treating each of these Xpaths as equivalent roots for the same data point. If you are using Python, the previously mentioned [glom](https://github.com/mahmoud/glom) package can make this a bit easier with its lovely [Coalesce](https://glom.readthedocs.io/en/latest/tutorial.html#changing-requirements) function.

**Same parent, different child Xpath for each item in a list**

If you are working with the 990PF, prepare for an extra-large helping of sadness as you attempt to work with Part IX-A. Here, the IRS stuffed all four rows of this table into the same root node, then gave each of them xpaths with the line number buried inside: “Description3”, “Expenses2,” etc. You’re just going to have to hard code this one.

**Two “lines” for the same field**

Business names can be long. The IRS knows this, so they (sometimes) supply two lines for them on the paper form. XML schemas for tax documents should resemble the paper form. The IRS knows this, too, so they (sometimes) give you a separate field for each “line” that appears on the paper form. Finally, schemas should be simple. The IRS knows this as well, so _whenever_ you have a business name, you _always_ have the potential for a second “line.”

What is the distinction between the first line and the second line? Nothing, usually, but stuff might appear in the second line anyway, so you’d better be ready to handle it. Glue the two lines together and move on. Note that addresses also have two lines, and in that case, there may be a meaning to it.

**Two different Xpaths for the same thing**

Part VII of the IRS 990 contains compensation data for key personnel, making it a perennial target of almost any 990-based research. For the vast majority of filings, each person’s name is identified by a <PersonNm> tag (or similar, depending on the version.) For a small minority of filings, however, the person is identified by a “business name.” And you guessed it: that means that their name can appear on two lines.

You can anticipate the potential for this kind of silliness by examining the schema files. If you see that the definition for the field you’re interested in contains a <choice>, each of the elements under that <choice> could potentially appear (and probably do).

_FYI:_ [_Open990_](https://www.open990.com/) _tracks compensation of individuals across years — for free — so that you don’t have to think about this kind of thing._

**Text in ALL CAPS**

Many IRS forms contain specific instructions about how to fill out paper forms: use blue or black ink, write in block letters, type if possible. Only [occasionally](https://www.irs.gov/instructions/i5300) do they require you to write in all caps, but it appears to be a convention — [perhaps to facilitate optical character recognition](http://answers.google.com/answers/threadview/id/529005.html).

Obviously, you don’t need to worry about how your XML will photocopy or whether a computer will be able to read it as a printout. But accountants are people and they have ways that things have always been done, and one of those is capitalizing returns.

Unfortunately, it’s very hard to correct the capitalization of English text. [Part-of-speech tagging](https://stackoverflow.com/questions/7706696/how-can-i-best-determine-the-correct-capitalization-for-a-word?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa) can help by identifying proper nouns. However, part-of-speech tagging depends in part on capitalization, so your mileage may vary. At Open990, we tried correcting capitalization, and decided that the cure was worse than the disease.

And my all-time favorite: **a field whose definition is different on the paper form.**

Yes, this actually happens at least once, and that means it probably happens more than once.

Schedule J is one of the more popular schedules because it contains compensation data that supplements — and partially overlaps with — Part VII. (Side note: [Open990](https://www.open990.com/) links the two sources of compensation data by person.) In the e-file schema, and the stylesheet used to render e-files into eFile graphic prints, Column F of Part II is described as “Compensation _reported in prior_ Form 990 or Form 990-EZ.” In the paper form, it is described as “Compensation reported _as deferred_ in prior Form 990.”

Solution? Head over to the [IRS’s instructions](https://www.irs.gov/pub/irs-prior/i990sj--2017.pdf) for clarification — and hope that tax preparers consulted the instructions too. If you are working with longitudinal data, be sure to also check the instructions pre-2013 for discrepancies.

## Off-the-shelf solutions

There are a number of tools available to help you get started with 990 data. Charity Navigator got the ball rolling with two parsing libraries: [990 Decoder](https://github.com/CharityNavigator/irs990), which extracts the fields actually used by Charity Navigator, and [990_long](https://github.com/CharityNavigator/990_long), which generates a vast set of key-value pairs.

We don’t recommend these legacy libraries, though, because more recently, [Jacob Fenton](http://www.jacobfenton.com/) developed an extraction package called [IRSx](https://github.com/jsfenfen/990-xml-reader), which is available on an MIT license via GitHub. IRSx lets you specify a list of fields to extract from some or all 990s. It works with 2013+ filings and generally expects you to find your own solutions to pathological cases like the ones above.

The Nonprofit Open Data Collective, headed up by Jesse Lecy, provides a bunch of useful repos on GitHub to deal with 990-related issues. Chief among them is the [community concordance](https://github.com/Nonprofit-Open-Data-Collective/irs-efile-master-concordance-file), which is a volunteer-maintained .csv file mapping field names to Xpaths.

So is it worthwhile to roll your own? Ultimately, this is your own call. It’s usually a good idea to try using an existing tool, and then estimating the difficulty of extending it to meet your needs.

### In conclusion

We’ve only begun to scratch the surface of the 990 dataset. In the first half of 2018, there has been a growing trickle of rich investigative reporting based on the IRS Form 990. New websites like [hospitalfinances.org](http://www.hospitalfinances.org/) are emerging to facilitate 990-based journalism, and nonprofit boards are beginning to wonder how their organization stacks up. I hope this article has brought you a little closer towards unlocking the next big use case. I’d [love to hear from you](https://www.open990.com/contact/)with questions, thoughts, and ideas.
