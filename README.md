# Online Odyssey Outlet Analysis Report
Lebintiti Kobe

- [Problem Statement](#problem-statement)
- [Results driven approach](#results-driven-approach)
  - [1. Understanding the problem :](#1-understanding-the-problem-)
  - [Assessing the chosen metric](#assessing-the-chosen-metric)
- [Data Structure](#data-structure)
- [Data Limitations](#data-limitations)
- [Executive Summary](#executive-summary)
  - [Insight deep dive](#insight-deep-dive)
- [Know data quality issues](#know-data-quality-issues)
- [Potential further analysis improvement
  .](#potential-further-analysis-improvement-)

# Problem Statement

------------------------------------------------------------------------

> You are a junior analyst working for an e-commerce startup, Online
> Odyssey Outlet.
>
> The date is January 2020, and the startup has just been through its
> first winter sale. Senior stakeholders are interested in knowing which
> products performed best during the Christmas period so that they can
> streamline the products they advertise in future sale periods.

------------------------------------------------------------------------

> You will do this by analyzing up to two months of events data,
> presented as two separate data sources with identical structures.
> Events refer not just to sales but to more general customer actions,
> such as viewing a product on the website or placing it into their
> virtual cart.

------------------------------------------------------------------------

> The challenge is that, when pressed, stakeholders aren’t exactly sure
> what they mean by “best.” In an initial brainstorming session, they
> highlight some aspects they care about when ranking their products:
>
> - Volume of sales
> - Total revenue from a single product
> - Popularity, measured by the number of unique customers who bought a
>   product
> - Conversion, meaning the percentage of time a product is bought once
>   it has been placed in the virtual shopping cart
> - Products with increased performance from November to December

# Results driven approach

------------------------------------------------------------------------

## 1. Understanding the problem :

------------------------------------------------------------------------

Senior stakeholders are interested in knowing which products performed
best during the Christmas period so that they can streamline the
products they advertise in future sale periods.

### What are our stakeholders hoping to achieve if they advertise the best performing products ? :

------------------------------------------------------------------------

**It may be one of the following or something else (The purpose of
analysis) :**

- To attract and retain new customers
- To improve sales
- To increase revenue
- To increase brand awareness and presence
- e.t.c

### “Product” ? :

------------------------------------------------------------------------

**What do we mean by product ? :**

- Here product can mean an individual product , product brands , product
  categories , e.t.c .

- In our analysis we will focus on individual products .

### “best performing product over christmas period” ? :

------------------------------------------------------------------------

**Based on potential purposes of analysis :**

- Product that had an increase appeal over christmas period (Conversion
  rates)
- Product that attracted most customers over christmas period (Number of
  unique customers)
- Product that retained most customers over christmas period (Repeat
  sales maybe)
- Product with most increased revenue and sales over christmas period
  (Revenue and Volume)
- e.t.c

### Implications of different metrics

------------------------------------------------------------------------

**“Units Sold / Volume” :**

- a product can have high volume but only be bought by a few customers ,
  have the worst conversion (appeal) , have the worst customer rating /
  reviews , have lowest profit , e.t.c
- So to use Volume we need to remove these NOISE and BIASED products

**“Conversion Rates” :**

- A product can have a high conversion , but lowest units sold … meaning
  these product are unreliable (conversion is based on small sample) ,
  NOISE , we should cut these noise if we using this metric .
- Also issues with UNITS SOLD Carry over to this metric too .

**“Profits” :**

- Two products can have the same total profit but different costs ,
  therefore we should couple this with profit margin (%) , high margin
  implies low cost and high profit … meaning we should filter out
  products with high profit but low margins , they are NOISE in this
  case .

------------------------------------------------------------------------

Here my instinct is to use number of units sold to find the best
performing products , but let us think of implications of this metric ,
assess and think of ways we can improve it .

## Assessing the chosen metric

------------------------------------------------------------------------

### **Implications**

*A product can have high volume but only bought by a few customers ,
have the worst conversion (appeal) , have the worst customer rating /
reviews , have lowest profit , e.t.c . So to use Volume we need to
remove these NOISE and BIASED products*

- A product with high volume but few unique buyers is bad for
  advertisement , it will attract only those few customers . Set a
  threshold for number of unique buyers in order to remove the
  unrealiable products , meaning products we choose will have solid
  business value

- A product with high volume but bad ratings are also bad for bad for
  business , low ratings mean our customers are unhappy therefore more
  likely to churn . Set a threshold for product rating to make sure we
  pick products that will ensure long term business success .

- A product with high volume , high total profit is actually bad if its
  margin is low , meaning the products need high sales volume to
  generate the high profit . It will require more inventory space and to
  outsell itself in order to make significant profit , this product can
  be unreliable / unpractical . Set a threshold for profit margin in
  order to discard these unreliable products .

### Systems Thinking assessment

------------------------------------------------------------------------

***Open the full system thinking assessment here :
[Systems_Thinking.md](scripts/Understanding%20the%20problem/Systems%20Thinking.md)***

------------------------------------------------------------------------

**Using systems thinking , in order for Units sold to work as a metric
:**

- Adjust for return , stock-outs , discounts , ratings/reviews , Product
  appeal .

- For product appeal , use conversion rates . But a product with high
  conversion but a few views / sales is bad for business . Set a
  threshold for units sold .

- For returns , we subtract product returns volumes from units sold to
  adjust for returns

- For stock-outs/ discounts , drop sales records that occurred during
  discounts / stock-out periods in order to make product comparisons
  without any biases , because these period usually boosts advertised
  products , not all products .

------------------------------------------------------------------------

### Final Metrics set

- Rank our products by Net Units sold , with threshold set for
  Rating/Reviews , Number of unique buyers , profit margin and excluding
  records during product stock-outs and discounts periods .

# Data Structure

------------------------------------------------------------------------

- We have online store events data for november and december data ,
  which combined have 7M+ records .
- Each record in the data either corresponds to a cart or a purchase
  event .

------------------------------------------------------------------------

*The both november and december data follow the following schema*

``` mermaid

erDiagram

    EVENTS_DATA {

        string event_type
        int user_id
        int user_session
        int product_id
        string brand
        string category_id
        string category_code
        dbl price
    }
```

# Data Limitations

------------------------------------------------------------------------

Our data is limited . There is no information on returns ,
ratings/reviews and for now we got no information on
sales/discounts/advertising periods .

Metrics we will be able to calculate from our data are :

- ***Units Solds*** = number of product purchase events for each month
- ***Number of unique buyers*** = number of unique customers ids for
  product purchase events
- ***Revenue*** = sum of prices for product purchase events
- ***Conversion rates*** = number of cart events over number of purchase
  events for each month .

------------------------------------------------------------------------

For units sold to work as a metric , we need to set a threshold for
number of unique buyers and a threshold for conversion rates , to make
sure we choose reliable products as best performing products . Revenue
in this case is not that useful , we needed profit and cost metrics
which we do not have for now .

After that we calculate growth rates from november to december for these
metrics . Also set growth rates threshold to ensure product chosen had
significant growth .

Our final results or minimum viable answer is a table of Products rank
by our growth rates of Units sold , with all necessary adjustments by
other supporting metrics , from November to December .

# Executive Summary

------------------------------------------------------------------------

**These are the products we recommend should be advertised in future
sales periods .**

| Product ID | Product Category Code | Brand | December Conversion Rate | December Units Sold | December Unique Customers | Conversion Growth Rate | Unit Sales Growth Rate | Customer Growth Rate |
|---:|:---|:---|---:|---:|---:|:---|:---|:---|
| 1004903 | Electronics Smartphone | Huawei | 0.3927416 | 6558 | 4656 | 14.90% | 324.19% | 288.97% |
| 1004723 | Electronics Smartphone | Huawei | 0.3816435 | 1867 | 1427 | 10.52% | 304.99% | 282.57% |
| 1005273 | Electronics Smartphone | Huawei | 0.4183751 | 551 | 441 | 19.81% | 257.79% | 212.77% |
| 4804497 | Sport Bicycle | Jbl | 0.4232673 | 171 | 146 | 11.77% | 167.19% | 170.37% |
| 1307136 | NA | NA | 0.4331927 | 616 | 485 | 29.77% | 162.13% | 155.26% |
| 5100505 | Electronics Clocks | Apple | 0.4390805 | 191 | 134 | 26.31% | 161.64% | 119.67% |
| 1002542 | Electronics Smartphone | Apple | 0.4036113 | 760 | 597 | 11.60% | 155.03% | 137.85% |
| 1307589 | Electronics Audio Headphone | Lenovo | 0.4020101 | 1200 | 951 | 15.76% | 153.70% | 145.10% |
| 1005245 | Electronics Smartphone | Apple | 0.4737864 | 244 | 175 | 30.77% | 144.00% | 113.41% |
| 5801655 | Electronics Audio Subwoofer | Element | 0.4805014 | 345 | 257 | 34.14% | 139.58% | 107.26% |
| 1005177 | Electronics Smartphone | Samsung | 0.5131399 | 4120 | 2698 | 26.88% | 132.90% | 117.06% |
| 1005100 | Electronics Smartphone | Samsung | 0.4108056 | 23457 | 15279 | 20.81% | 131.51% | 112.86% |
| 1002547 | Electronics Smartphone | Apple | 0.4267223 | 3066 | 2167 | 17.08% | 131.05% | 113.29% |
| 1004748 | Electronics Smartphone | Huawei | 0.3963565 | 805 | 646 | 19.35% | 126.76% | 128.27% |
| 1004857 | Electronics Smartphone | Samsung | 0.4176780 | 3889 | 2997 | 17.29% | 112.51% | 101.14% |
| 21401209 | Electronics Clocks | Casio | 0.4303406 | 139 | 117 | 29.10% | 101.45% | 108.93% |

Top Performing Products

## Insight deep dive

------------------------------------------------------------------------

***Found 50+ products with every metric greater than average of each
metric for all products , then further trimmed these to find :***

- Products that sold twice as much in december as compared to november ,
  units growth rate of 100% or more.
- Products with positive conversion growth , more than 5% growth .
- Products that sold more than 50 units in november .
- Products that had more than 50 unique customers in in november .
- Products that had twice as much unique customers in december .

**To end up with 16 best performing products .**

| Product ID | Product Category Code | Brand | December Conversion Rate | December Units Sold | December Unique Customers | Conversion Growth Rate | Unit Sales Growth Rate | Customer Growth Rate |
|---:|:---|:---|---:|---:|---:|:---|:---|:---|
| 1004903 | Electronics Smartphone | Huawei | 0.3927416 | 6558 | 4656 | 14.90% | 324.19% | 288.97% |
| 1004723 | Electronics Smartphone | Huawei | 0.3816435 | 1867 | 1427 | 10.52% | 304.99% | 282.57% |
| 1005273 | Electronics Smartphone | Huawei | 0.4183751 | 551 | 441 | 19.81% | 257.79% | 212.77% |
| 4804497 | Sport Bicycle | Jbl | 0.4232673 | 171 | 146 | 11.77% | 167.19% | 170.37% |
| 1307136 | NA | NA | 0.4331927 | 616 | 485 | 29.77% | 162.13% | 155.26% |
| 5100505 | Electronics Clocks | Apple | 0.4390805 | 191 | 134 | 26.31% | 161.64% | 119.67% |
| 1002542 | Electronics Smartphone | Apple | 0.4036113 | 760 | 597 | 11.60% | 155.03% | 137.85% |
| 1307589 | Electronics Audio Headphone | Lenovo | 0.4020101 | 1200 | 951 | 15.76% | 153.70% | 145.10% |
| 1005245 | Electronics Smartphone | Apple | 0.4737864 | 244 | 175 | 30.77% | 144.00% | 113.41% |
| 5801655 | Electronics Audio Subwoofer | Element | 0.4805014 | 345 | 257 | 34.14% | 139.58% | 107.26% |
| 1005177 | Electronics Smartphone | Samsung | 0.5131399 | 4120 | 2698 | 26.88% | 132.90% | 117.06% |
| 1005100 | Electronics Smartphone | Samsung | 0.4108056 | 23457 | 15279 | 20.81% | 131.51% | 112.86% |
| 1002547 | Electronics Smartphone | Apple | 0.4267223 | 3066 | 2167 | 17.08% | 131.05% | 113.29% |
| 1004748 | Electronics Smartphone | Huawei | 0.3963565 | 805 | 646 | 19.35% | 126.76% | 128.27% |
| 1004857 | Electronics Smartphone | Samsung | 0.4176780 | 3889 | 2997 | 17.29% | 112.51% | 101.14% |
| 21401209 | Electronics Clocks | Casio | 0.4303406 | 139 | 117 | 29.10% | 101.45% | 108.93% |

Top Performing Products

# Know data quality issues

------------------------------------------------------------------------

- *Around 1400 products with prices of zero , all of them do not have
  brands codes and they dont have any purchase event . These products
  that we dropped from the catalog by some brand which was not selling
  well . These were dropped because they dont affect our analysis at all
  .*

- *Over 19K+ products with multiple brands , category code and/or
  category IDs . Here we replaced the multiple brands , category codes
  and/or category IDs with the most common ones . Our result may contain
  products with wrong categories and brands due to our solution to this
  problem .*

- *All brands with SMARTPHONE as their second best category all have
  construction.tools.light as their top categories , including Brands
  like Samsung , Apple , Huawei , e.t.c which all specialize in
  smartphone , so construction.tools.light is suspicious . Replace
  construction.tools.light with smartphones , but we do this for brands
  that have smartphones as their second best category .*

- *Found category codes with multiple category IDs . Replaced the
  multiple category ID with the most common one for each category code
  .*

# Potential further analysis improvement .

------------------------------------------------------------------------

1.  **Metric Selection shortfall**

Our data is limited . There is no information on returns ,
ratings/reviews and on sales/discounts/advertising periods . And sales
volume , conversion and number of unique customers arent enough to cover
all the concept that makes a product best performing in our business
concept .

- ***A product can have high sales volume by a high number of unique
  customers and high appeal conversion , but if it has bad reviews /
  returns then the product is no any good to our business since it will
  make customer no trust us and churn .***

- ***A product can have high sales volume by a high number of unique
  customers and high appeal conversion , but this may be due to the fact
  it was the product that was pushed / boosted , so its sales are
  artificial , and if we dont adust for these boosting periods then our
  result will be biased toward these products .***

  ------------------------------------------------------------------------

2.  **Solutions to product with multiple brand and categories , category
    codes with multiple category IDs**

- ***Instead of just replacing product multiple brands and categories
  with the most common ones , we could replace them with the most recent
  one , then also keep a list of other brands / categories associated
  with the product , for data lineage and trust .***
- ***Same goes for category codes with multiple category IDs .***
