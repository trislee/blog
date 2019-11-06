---
layout: post
title:  CFPB Analysis
date:   2019-09-25 10:32:20 +0300
description: Analysis and visualizations of Consumer Financial Protection Bureau complaint data
img: posts/2019-09-25-cfpb_sankey/sankey_cover.png
tags: [Politics, CFPB]
---

The Consumer Financial Protection Bureau (CFPB) has a dataset of all 1.3 million complaints it's received since its inception in 2011.
These complaints document instances of corporate malfeasance against consumers, including improper threats made by debt collectors, incorrect information on credit reports, and simply people struggling to pay their mortgages.
You can download the dataset [here][dataset].

I made a few interactive visualizations of the data.
The first is a series of bar charts showing the most complained-about companies by sector.
The second is a is a series of Sankey diagrams of the most complained-about companies by sector.
You can find the scripts used to generate the dashboard below [here][script].

These charts probably don't look very good on mobile, sorry about that, I'm still pretty new to web development.

{% include interactive/cfpb_both_dashboard.html %}

What stood out to me is the difference in monopoly power across the sectors.
For example, in the credit reporting sector, nearly all complaints are about the three largest companies (Equifax, Experian, and Transunion), while the debt collection sector by contrast has many smaller companies, with no single dominant monopoly.
Below is a rank-frequency plot of companies by product.
{% include interactive/cfpb_product_rank_frequency.html %}



[dataset]: https://www.consumerfinance.gov/data-research/consumer-complaints/search/?from=0&searchField=all&searchText=&size=25&sort=created_date_desc
[script]: https://raw.githubusercontent.com/trislee/cfpb/master/scripts/generate_sankey_dashboard.py
