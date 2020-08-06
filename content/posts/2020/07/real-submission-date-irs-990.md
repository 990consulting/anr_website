+++
author = "David Bruce Borenstein, PhD"
authorAvatar = "/uploads/david_circle_small.png"
authorFacebook = ""
authorGoogle = ""
authorImage = "/uploads/images/david-1.png"
authorInfo = "David Bruce Borenstein, Ph.D., is a partner at Applied Nonprofit Research and Chief Technology Officer at Open990. As the Director of Data Science at Charity Navigator, he was closely involved in the creation of the IRS 990 Community Concordance. David’s background is originally in computational genomics, which informs his approach to data curation."
authorTwitter = ""
date = "2020-07-06T04:00:00+00:00"
description = "To get the final story, we asked an e-filer's accountant."
tags = ["dataset", "open government", "nonprofit technology", "open data", "data science"]
title = "Identifying the real submission date of an e-filed IRS Form 990"

+++



## Synopsis

When parsing e-filed returns from tax-exempt entities, use the value of the XML path  `/Return/ReturnHeader/ReturnTs` for date submitted.



## So many timestamps, so little documentation...

The IRS provides [.csv and .json index files](https://docs.opendata.aws/irs-990/readme.html) for 990 e-files [available via AWS](https://registry.opendata.aws/irs990/). In our [previous post](/posts/2020/06/skip-the-irs-990-efile-indices/), we noted that the.json index contains a `date_submitted` field. We also noted that it is probably wrong: the date listed in the index does not appear anywhere else in the entire return.

There are many dates listed inside each return. In particular each return contains the following:

### The date that the Form 990 / 990-EZ / 990-PF was filled out:

This field is only completed when a professional tax preparer fills out the form for the entity, and refers to the date that the preparer completed that task. 

* Pre-2013 returns: `/Return/ReturnHeader/Preparer/DatePrepared`
* 2013+ returns: `/Return/ReturnHeader/PreparerPersonGrp/PreparationDt`

### The date that the filing was signed by an officer

On the 990, 990-EZ, and 990-PF, there is an area where an officer of the entity must sign (or electronically affirm) the following statement:

> Under penalties of perjury, I declare that I have examined this return, including accompanying schedules and statements, and to the best of my knowledge and belief, it is true, correct, and complete. Declaration of preparer (other than officer) is based on all information of which preparer has any knowledge.

The date of this e-signature, however, need not be the same as the date upon which the return is actually transmitted to the IRS. In fact, there exists a separate form, [Form 8879-EO](https://www.irs.gov/pub/irs-pdf/f8879eo.pdf) that authorizes a paid tax preparer to enter an officer's signature into an electronic filing. 

The preparer would have to receive this form before they could legally transmit the return that they prepared. The accountants that I know also typically require payment before submitting the filing, and they may find it convenient to submit many filings at once.

* Pre-2013 returns: `/Return/ReturnHeader/Officer/DateSigned`
* 2013+ returns: `/Return/ReturnHeader/BusinessOfficerGrp/SignatureDt`

### The others

After these relatively straightforward dates, there are a couple more that are not really explained:

- `/Return/ReturnHeader/ReturnTs`: "The date and time when the return was created." The term "created" is not defined.
- `/Return/ReturnHeader/FilingSecurityInformation/FederalOriginalSubmissionIdDt`: "Federal Original Submission Identification Date" (2016+)
- `/Return/ReturnHeader/BuildTs`: Not defined anywhere, but it doesn't match `ReturnTs`
- The `date_submitted` column in the .json index: Also not defined anywhere

Which one, if any, represents the date upon which a nonprofit's own accountant believes that the filing was filed?

## Ask, and it will be given

It occurred to me that any organization that e-files a 990 could put this question to rest. I started with ProPublica, which, as of this writing, is using the `date_submitted` field on [its profile of itself](https://projects.propublica.org/nonprofits/organizations/142007220). Although they thanked me for pointing out the likely mistake, they did not respond to my follow up.

So I checked in with a senior officer of another nonprofit. This person was kind enough to consult with the organization's accountant, who told us that **the date reported in `ReturnTs` is consistent with their own records.**

This is now the date used throughout Open990.org, and that we will use in all of the datasets provided by Applied Nonprofit Research going forward.

### What do the others mean?

I'll be honest: I still have no idea what the other timestamps mean. My guess is that the .json `date_submitted` has to do with some kind of bulk transaction between a tax services software provider and the IRS itself. If anyone has information on the others, please [e-mail me](mailto:david@appliednonprofitresearch.com).