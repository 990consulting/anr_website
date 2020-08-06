+++
author = "David Bruce Borenstein, PhD"
authorAvatar = "/uploads/david_circle_small.png"
authorFacebook = ""
authorGoogle = ""
authorImage = "/uploads/images/david-1.png"
authorInfo = "David Bruce Borenstein, Ph.D., is a partner at Applied Nonprofit Research and Chief Technology Officer at Open990. As the Director of Data Science at Charity Navigator, he was closely involved in the creation of the IRS 990 Community Concordance. David’s background is originally in computational genomics, which informs his approach to data curation."
authorTwitter = ""
date = "2020-08-04T04:00:00+00:00"
description = "During the April 2020 update, almost two thousand filings disappeared."
tags = ["dataset", "open government", "nonprofit technology", "open data", "data science"]
title = "Is the IRS deleting old e-file 990s?"

+++



## Synopsis

The IRS deleted [1,975 filings](/uploads/deleted_efiles_apr_2020.csv) from the [AWS e-file repository](https://registry.opendata.aws/irs990/) in April. However, you can [download](https://archive.org/download/IRS990-efile/irs_form990_xml.2011.tar.gz) the missing filings from the Internet Archive.

## April, 2020: Alarm bells ring

By April 2020, I was starting to get worried: the IRS, which had previously been uploading new e-files monthly, had not uploaded anything since December. Imagine my relief, therefore, when I saw that new data had finally become available.

My relief was short-lived when the [open-source data enrichment system](https://github.com/borenstein/polytropos) that powers all our datasets and APIs began reporting missing filings. Was the IRS deleting e-files?

The missing filings all seemed very old. Perhaps, I thought, the IRS was purging filings past a certain age. That was a very disturbing premise, as the older data is a goldmine for academic researchers exploring historical trends in both the tax-exempt sector and the U.S. economy more broadly.

Like basically everyone else in the US, neither Heather Kugelmass nor I had much time to look into it that month, busy as we were with rearranging our lives to cope with the COVID-19 crisis. So we worked with the data we had and planned to revisit the issue when the next update came.

## And then it stopped

The IRS finally provided a data update on or around July 28, 2020. Anticipating a huge batch of deletions, I modified my pipeline to detect and report any filings that had been deleted. I ran the code, and initially thought that I must have made a mistake: no new deletions were reported. But I had tested my code very thoroughly, and the filings deleted during the previous update were still missing.

To muddy the waters further, the missing filings turned out not to be the oldest filings on record. In fact, all of the deleted filings from April had been filed with the IRS during calendar year 2011. The oldest filings in the e-file dataset were filed in calendar year 2010. So it didn't seem to be a rolling deletion process like I initially suspected.

I resisted the urge to analyze the deleted data further, since I still had it in my own archives. However, I am curious what caused the deletion. **If anyone else wishes to perform a more thorough analysis, the list of deleted filings can be found [here](/uploads/deleted_efiles_apr_2020.csv).**

## Where to find the missing filings

[The Internet Archive](https://www.archive.org/) (of [Wayback Machine](https://archive.org/web/) fame) has been downloading and preserving the e-files since they first became available. While the AWS downloads are far faster and allow you to cherry-pick individual filings, you can get whatever you need [here](https://archive.org/download/IRS990-efile). 

All of the missing filings can be found in the [2011 archive](https://archive.org/download/IRS990-efile/irs_form990_xml.2011.tar.gz). The file is just over 1GB and will take 2+ hours to download due to bandwidth limitations at the Internet Archive.