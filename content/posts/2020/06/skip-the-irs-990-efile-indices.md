+++
author = "David Bruce Borenstein, PhD"
authorAvatar = "/uploads/david_circle_small.png"
authorFacebook = ""
authorGoogle = ""
authorImage = "/uploads/images/david-1.png"
authorInfo = "David Bruce Borenstein, Ph.D., is a partner at Applied Nonprofit Research and Chief Technology Officer at Open990. As the Director of Data Science at Charity Navigator, he was closely involved in the creation of the IRS 990 Community Concordance. David’s background is originally in computational genomics, which informs his approach to data curation."
authorTwitter = ""
date = "2020-06-26T04:00:00+00:00"
description = "They are often wrong, and there's a better way."
tags = ["dataset", "open government", "nonprofit technology", "open data", "data science"]
title = "Don't use the IRS 990 e-file indices"

+++

## Synopsis

The IRS-provided indices of available e-files are unreliable. This post concludes with a Python script to generate your own index much faster than would be possible using the AWS command line.

## A little context

If you're reading this, you probably already know that [most nonprofits must submit detailed returns about their activities to the IRS](https://www.irs.gov/charities-non-profits/exempt-organization-annual-filing-requirements-overview), and that the IRS [publishes](https://registry.opendata.aws/irs990/) all electronically filed returns in XML format. You probably also know that working with these data can be quite challenging.

A couple years ago, I wrote a [how-to guide](https://appliednonprofitresearch.com/posts/2018/06/the-irs-990-e-file-dataset-getting-to-the-chocolatey-center-of-data-deliciousness/) explaining how to work with these data. In it, I said that you should depend on the `.json` and `.csv` indices that the IRS publishes when they perform a data update. **This is no longer good advice.**

In this post, I will explain why I am moving away from the IRS' published indices, and why you should, too.

## Why the indices seem like a good idea

There are millions of objects in the official repository containing e-filed data, which is hosted on Amazon Web Services' S3 platform. The filings are named using apparently random "object IDs," so you can't programatically predict the URLs. 

As an example, the TY2016 filing for an organization whose tax identification number (EIN) is 04-2662873 has the object ID [201612439349300006](https://s3.amazonaws.com/irs-form-990/201612439349300006_public.xml). Hey, at least it starts with 2016!

Now, if you want to find a particular filing for a particular organization, just [search by EIN at Open990.org](https://www.open990.org/org/) and we'll hook you up. But if you're reading this, chances are you want to look at many or all of the filings at once.

Aside from the index files, you do have the option of using the [AWS command-line interface](https://aws.amazon.com/cli/) or a language-specific library like [boto3](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html) to list everything, but it will take upwards of an hour. That is itself an improvement: back in 2016, the requests simply timed out. For the morbidly curious, the CLI command is:

```
aws s3 ls s3://irs-form-990/
```

## Why the indices are bad

The indices are wrong. They will break your heart. Let me count the ways.

### Lesson 1: The indices contain filings that aren't there

My first inkling that there was something wrong with the indices came when I first built an automated data processing platform for e-filed 990s. For a few object IDs reported in the index, there was no corresponding file on S3. 

But I was young and naive, and I figured there was some reason why the IRS had held these particular filings back, so I just skipped over them. It was at about this time that my partner, [Heather Kugelmass](https://www.open990.org/contact/), noticed that the "date submitted" in the metadata files didn't match up with the date submitted in the e-file. But more on that in a minute.

### Lesson 2: The indices aren't always updated together

The alarm bells really started ringing when the IRS  posted data for 2020 after almost five months of waiting. (Prior to this, the IRS was updating its dataset approximately monthly.) My automated ETL pipeline didn't notice the update at first, because the `.json` index was empty, and my code was using the `.json` index.

Then it occurred to me to check the `.csv` index. There were hundreds of thousands of filings. Whoops!

Well, I went right in and manually forced my pipeline to read the `.csv` index for 2020, and got my data update out to Open990.org and to all my clients. Phew! Problem solved.

### Lesson 3: The indices can be buggy

Until a technically sophisticated client pointed out that I was missing tens of thousands of filings from 2019. How could this be? Well, it turned out that the [2019 `.json` index](https://s3.amazonaws.com/irs-form-990/index_2019.json) had far fewer filings than the [corresponding `.csv` index](https://s3.amazonaws.com/irs-form-990/index_2019.csv). 

Spot checking a few of the missing records, I noticed that--at least for the ones I checked--these were organizations that had submitted returns for two different tax periods during the same calendar year. The `.csv` index had both, and `.json` file only had the first one. Whoopsy-daisy, IRS!

### Lesson 4: The indices contain misleading information

From the very beginning, Heather had her concerns about the `DateSubmitted` field provided in the `.json` index. It didn't match the date stamps on ProPublica, or the timestamp inside the filing. The IRS doesn't provide a data dictionary for these indices, so I didn't have any way of **really** knowing what this field meant. But it was a convenient indicator for figuring out which filing to use if there were several for the same tax period and EIN: obviously, the most recently submitted one is the one I should be using, right?

Maybe, maybe not. The `DateSubmitted` for the return mentioned above is `2017-01-04`. But `2017` doesn't appear *anywhere in the entire return.* ([Search it yourself](https://s3.amazonaws.com/irs-form-990/201612439349300006_public.xml).)

Now, there are plenty of things that you might reasonably call DateSubmitted in this return. Here are a few examples:

* `/Return/ReturnHeader/ReturnTs` (2016-08-30T09:15:46-05:00)
* `/Return/ReturnHeader/BusinessOfficerGroup/SignatureDt` (2016-08-15)
* `/Return/ReturnHeader/DPreparerPersonGrp/PreparationDt` (2016-08-24)
* `/Return/ReturnHeader/BuildTS` (2016-12-15 16:53:06Z)

Maybe it's the date uploaded to S3? Nope, they have a field for that, `LastUpdated`, and that does seem to correspond to the S3 timestamp. The only sensible conclusion is that `DateSubmitted` refers to some internal IRS process, and **not** to anything that the filer did.

### Lesson 5: Everything that *is* useful can be obtained elsewhere

Most of the information in the indices comes from either the return itself or from the S3 object metadata, which can be retrieved as follows:

```
aws s3api head-object --bucket irs-form-990 --key 201612439349300006_public.xml
```

The only exceptions are the aforementioned `DateSubmitted`, along with two more identifiers called `DLN` (which appears in both indices) and `RETURN_ID` (which appears in the `.csv` index only). I have never, in four years of working with these data, come across any use for either of these identifiers. 

## How to list the filings directly, fast

The tricky thing is that the S3 API does not allow you to say "start from position X" or "report the total number of pages" or "report the total number of items." It has pagination, but the pagination doesn't lend itself to parallel processing. As such, it can be hard to work with large buckets unless they have a good "prefix" (directory) structure.

The e-file bucket certainly doesn't have that; everything is just dumped into the root prefix. But we do know that each key starts with a year followed by some numeric digits. And that's all we need.

In [a rather clever blog post](https://joeray.me/gnu-parallel-how-to-list-millions-objects.html), developer Joe Ray notices that if your keys are reasonably random, you can chunk up your listing task into one chunk for each possible prefix. Our records all start with a year, so there are only a few possible values for the first four characters. Then you have numbers that, while not random, are at least spread out enough to be treated the same way.

The numbers are not uniformly distributed; they seem to increment up. But if you use the first two numerals after the year, you get 100 chunks per year, i.e.

```
200900
200901
200902
...
202097
202098
202099
```

Most of those will be empty, but the non-empty ones will be a pretty good size for parallelizing: not so small that you lose all efficiency to overhead, but small enough that no one job runs for more than a minute or so.

Using the AWS library for Python (`boto3`) and the amazing `concurrent.futures` package, I was able to get a listing of all 3,220,237 filings in **179.3 seconds** on my plain ol' desktop computer. Here's the code. May it help you finish your project speedily.

```
import time
import datetime
from collections import deque
from typing import List, Deque, Iterable, Dict
import logging
import boto3
from concurrent.futures import ProcessPoolExecutor, ThreadPoolExecutor, as_completed, Future

logging.basicConfig(format='%(asctime)s : %(levelname)s : %(message)s', level=logging.INFO)

BUCKET: str = "irs-form-990"
EARLIEST_YEAR: int = 2009
cur_year: int = datetime.datetime.now().year

first_prefix: int = EARLIEST_YEAR * 100
last_prefix: int = (cur_year + 1) * 100

def get_keys_for_prefix(prefix: str) -> Iterable[str]:
		"""Return a collection of all key names starting with the specified prefix."""
    client = boto3.client('s3')
    
    # See https://boto3.amazonaws.com/v1/documentation/api/latest/guide/paginators.html
    paginator = client.get_paginator('list_objects_v2')
    page_iterator = paginator.paginate(Bucket=BUCKET, Prefix=prefix)

		# A deque is a collection with O(1) appends and O(n) iteration
    results: Deque[str] = deque()
    i = 0
    for i, page in enumerate(page_iterator):
        if "Contents" not in page:
            continue
        
        # You could also capture, e.g., the timestamp or checksum here
        page_keys: Iterable = (element["Key"] for element in page["Contents"])
        results.extend(page_keys)
    logging.info("Scanned {} page(s) with prefix {}.".format(i+1, prefix))
    return results

start: float = time.time()

# ProcessPoolExecutor starts a completely separate copy of Python for each worker
with ProcessPoolExecutor() as executor:
    futures: Deque[Future] = deque()
    for prefix in range(first_prefix, last_prefix):
        future: Future = executor.submit(get_keys_for_prefix, str(prefix))
        futures.append(future)
    
n = 0

# as_completed ignores submission order to prevent unnecessary waiting
for future in as_completed(futures):
    keys: Iterable = future.result()
    for key in keys:
    		# Do your analysis here
        n += 1
elapsed: float = time.time() - start
logging.info("Discovered {:,} keys in {:,.1f} seconds.".format(n, elapsed))
```