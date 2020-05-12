+++
author = "David Bruce Borenstein, PhD"
authorAvatar = "/uploads/david_circle_small.png"
authorFacebook = ""
authorGoogle = ""
authorImage = "/uploads/images/david-1.png"
authorInfo = "David Bruce Borenstein, Ph.D., is a partner at Applied Nonprofit Research and Chief Technology Officer at Open990. As the Director of Data Science at Charity Navigator, he was closely involved in the creation of the IRS 990 Community Concordance. David’s background is originally in computational genomics, which informs his approach to data curation."
authorTwitter = ""
date = "2020-02-25T04:00:00+00:00"
description = "A sermon on the evils of treating data extraction as anything other than a traditional software task."
tags = ["dataset", "open government", "nonprofit technology", "open data", "data science"]
thumbnail = "/uploads/efile_xml.png"
title = "Smelt, don't mine: robust ETL through iterative enrichment"
draft = true
+++

## Prelude: Of Lakes and vistas

In the beginning, there was the [data warehouse](https://www.talend.com/resources/what-is-data-warehouse/): a carefully planned relational storehouse of structured data, built from arbitrarily many sources. Data warehouses required up-front understanding about what you might want to do with your data. In return, you got structure and reliability. And Microsoft SQL server said that it was good.

{{< figure src="/uploads/smelt/data_warehouse_toolkit.png" class="figureWithCaption" caption="The 1996 edition of the [Data Warehouse Toolkit](https://www.abebooks.com/products/isbn/9780471153375?cm_sp=bdp-_-ISBN10-_-PLP). CD-ROM included!" >}}

[Cover of first edition of The Data Warehouse Toolkit -- subtitle: "The original Data Warehouse Toolkit. CD-ROM included!"]

Except it wasn't. The need to anticipate use cases limited the potential for iteration and novel perspectives. Have some totally unanticipated question? If it can't be shoehorned into the contours of [the schema](https://www.red-gate.com/simple-talk/sql/learn-sql-server/sql-server-data-warehouse-cribsheet/#fifth), you need to extend or redesign your warehouse. 

Enter the [data lake](https://aws.amazon.com/big-data/datalakes-and-analytics/what-is-a-data-lake/). The data lake is everything a data warehouse is not. Your data lives in its native format, ready to be analyzed ad hoc at a moment's notice. New data source? Dump it in the lake! Cheap computing power and easy parallelism meant that any question could be answered, no matter how unexpected. 

The big cloud providers would love you to believe that you can safely and meaningfully gain insight straight from your data lake. Tools like [AWS Glue](https://aws.amazon.com/glue/), which is based on [Apache Hive](https://hive.apache.org/) make data lakes behave like databases. If you need to get fancier, you could write a [Spark](https://spark.apache.org/) pipeline With a few lines of [R](https://spark.rstudio.com/) or [Python](https://spark.apache.org/docs/latest/api/python/index.html), and start scanning petabytes on by-the-second spot rentals for the cost of a nice lunch.

## But ETL software is still software

Let's forget about ETL for a second. Would you write software that is critical to your business without any tests? How about as a [monolithic block of procedural code](https://www.oreilly.com/library/view/working-effectively-with/0131177052/ch22.html)? Of course not. You know you'd regret it later, either when you went to change it or when you encountered a subtle but disastrous bug.

It is [notoriously hard](https://blog.dominodatalab.com/unit-testing-data-science/) to test code written for ad hoc data analysis. If you're using a black box like Hive, you *can't* write tests. And anyway, this is just a one-off query, right? I'm just answering a "quick question." I don't need to go crazy on formalities.

And yet consider: when you are querying your data, it's usually to drive business decisions. That is, *these queries constitute software that is critical to your business.*
 
Every query represents a set of assumptions about what's in your data lake and how it's structured. But there aren't really any guarantees, even if some data dictionary says there ought to be. Intuitively, you may know this. So you do due diligence. You look a good hard look at the output every time you run your pipeline. 

But one day, you happen to look at line 724 of the resulting spreadsheet, and something weird happened on line 13,276. And now you're up all night--or worse, you're eating crow because you botched your forecast after application X added a decimal point to the third column of some CSV dump in the latest minor update.

{{< figure src="https://imgs.xkcd.com/comics/data_pipeline.png" class="figureWithCaption" caption="Reproduced with permission from [xkcd](https://xkcd.com/2054/)." >}}
