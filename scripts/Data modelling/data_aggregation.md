# Online Odyssey Outlet - Data Modelling Script
Lebintiti Kobe

- [Problem Statement](#problem-statement)
- [Results driven approach](#results-driven-approach)
  - [1. Understanding the problem :](#1-understanding-the-problem-)
  - [Assess Volume(Units Sold) A
    Metric](#assess-volumeunits-sold-a-metric)
  - [Final Metrics set](#final-metrics-set)
  - [Data Limitations](#data-limitations)
- [Aggregate our data into useful
  format](#aggregate-our-data-into-useful-format)
  - [What format / schema should our data be
    in](#what-format--schema-should-our-data-be-in)
  - [Load the data](#load-the-data)
  - [Save final results table](#save-final-results-table)

# Problem Statement

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

## 1. Understanding the problem :

Senior stakeholders are interested in knowing which products performed
best during the Christmas period so that they can streamline the
products they advertise in future sale periods.

### What are our hoping stakeholders to achieve if they advertise the best performing products :

- To attract and retain new customers
- To improve sales
- To increase revenue
- To increase brand awareness and presence
- e.t.c

### “Product” ? :

What do we mean by product ? :

- Here product can mean an individual product , product brands , product
  categories , e.t.c . Choose any level of granularity .

### Therefore “best performing product over christmas period” ? :

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

**“Units Sold / Volume” :**

- a product can have high volume but only bought by a few customers ,
  have the worst conversion (appeal) , have the worst customer rating /
  reviews , have lowest profit , e.t.c
- So to use Volume we need to remove these NOISE and BIASED products

**“Conversion Rates” :**

- A product can have a high conversion , but lowest units sold … meaning
  these product are unreliable , NOISE , we should cut these noise if we
  using this metric . Also issues with UNITS SOLD Carry over to this
  metric too .

**“Profits” :**

- Two products can have the same total profit but different costs ,
  therefore we should couple this with profit margin (%) , high margin
  implies low cost and high profit … meaning we should filter out
  products with high profit but low margins , they are NOISE in this
  case .

Here my instinct is to use number of units sold to find the best
performing products , but lets think of implications of this metric ,
assess and think of ways we can improve it .

## Assess Volume(Units Sold) A Metric

### Implications

A product can have high volume but only bought by a few customers , have
the worst conversion (appeal) , have the worst customer rating / reviews
, have lowest profit , e.t.c . So to use Volume we need to remove these
NOISE and BIASED products

- A product with high volume but few unique buyers is bad for
  advertisement , it will attract only those few customers . Set a
  threshold for number of unique buyers in order to remove the bias ,
  meaning products we choose will have solid business value .

- A product with high volume but bad ratings are also bad for bad for
  business , low ratings mean our customers are unhappy therefore more
  likely to churn . Set a threshold for product rating to make sure we
  pick products that will ensure long term business success .

- A product with high volume , high total profit is actually bad if its
  margin is low . It will require more inventory space and to outsell
  itself in order to make significant profit , this product can be
  unreliable / practical . Set a threshold for profit margin in order to
  discard these unreliable products

### Systems Thinking assessment

In order for Units sold to work as a metric :

- Adjust for return , stock-outs , discounts , ratings/reviews , Product
  appeal .

- For product appeal , use conversion rates . But a product with high
  conversion but a few views / sales is bad for business . Set a
  threshold for units sold .

- For returns , we subtract product returns volumes from units sold to
  adjust for returns

- For stock-outs/ discounts , drop sales records that occurred during
  discounts / stock-out periods in order to make product comparisons
  without any biases .

## Final Metrics set

- Rank our products by Net Units sold , with threshold set for
  Rating/Reviews , Number of unique buyers , profit margin and excluding
  records during product stock-outs and discounts periods .

## Data Limitations

Our data is limited . There is no info on returns , ratings/reviews and
for now we got no information on sales periods .

Metrics we will be able to calculate from our data :

Units Solds = number of product purchase events for each month Number of
unique buyers = number of unique customers ids for product purchase
events Revenue = sum of prices for product purchase events Conversion
rates = number of cart events over number of purchase events for each
month .

For units sold to work , we need to set a threshold for number of unique
buyers , a threshold for conversion rates . Revenue in this case is not
that useful , we needed profit and cost metrics which we do not have for
now .

That is about it . We want growth rates from november to december for
these metrics .

Our final results or minimum viable answer will be a table of Products /
Product brands / Categories rank by our growth rates of Units sold ,
with all necessary adjustments , from November to December .

# Aggregate our data into useful format

## What format / schema should our data be in

Our main metric here should be

## Load the data

``` r
 require(arrow)
 require(tidyverse)
 require(kableExtra)
 require(here)
 require(janitor)

 events_data <- read_parquet(here("data/events_data_model.gz.parquet"))

 aggregate_data <- events_data %>%
    group_by(product_id , event_month) %>%
    summarise(
    number_cart = sum(event_type == "cart"),
    number_purchase = sum(event_type == "purchase"),
    conversion_rate = number_purchase / number_cart ,
    number_units_sold = number_purchase) %>%
    ungroup() %>%  
    select(-number_cart , -number_purchase) 

  merge <- events_data %>%
    group_by(product_id , event_month) %>%
    filter(event_type == "purchase") %>%
    summarise(number_unique_customers = n_distinct(user_id))

  
  aggregate_data <- aggregate_data %>%
  left_join(y=merge)

  rm(merge)
  gc()  
```

               used  (Mb) gc trigger  (Mb) max used  (Mb)
    Ncells  1623503  86.8    6956024 371.5  8695030 464.4
    Vcells 36521160 278.7  119613974 912.6 99606827 760.0

``` r
  aggregate_data 
```

    # A tibble: 106,340 × 5
       product_id event_month conversion_rate number_units_sold
       <chr>      <ord>                 <dbl>             <int>
     1 100000000  December              1                     1
     2 100000002  December              1                     1
     3 100000013  December              0                     0
     4 100000016  December              0.333                 1
     5 100000021  December              0.5                   1
     6 100000024  December              0.357                10
     7 100000042  December              0                     0
     8 100000043  December              0.25                  2
     9 100000044  December              0.5                   1
    10 100000046  December              0.5                   4
    # ℹ 106,330 more rows
    # ℹ 1 more variable: number_unique_customers <int>

``` r
  aggregate_data <- aggregate_data %>% na.omit() %>% filter(conversion_rate < 1)
```

``` r
 dec_data <- aggregate_data %>%
  filter(event_month == "December") %>%
  select(-event_month)

dec_avgs <- dec_data %>%
  filter(conversion_rate < 1) %>%
  summarise(mean_unq = mean(number_unique_customers),
  mean_units = mean(number_units_sold),
  mean_conv = mean(conversion_rate))

dec_data <- dec_data %>%
  filter(conversion_rate > dec_avgs$mean_conv , 
  number_units_sold > dec_avgs$mean_units,
  number_unique_customers > dec_avgs$mean_unq)

nov_data <- aggregate_data %>%
  filter(event_month == "November")%>%
  select(-event_month)

nov_avgs <- nov_data %>%
  filter(conversion_rate < 1) %>%
  summarise(mean_unq = mean(number_unique_customers),
  mean_units = mean(number_units_sold),
  mean_conv = mean(conversion_rate))  

nov_data <- nov_data %>%
  filter(conversion_rate > nov_avgs$mean_conv , 
  number_units_sold > nov_avgs$mean_units,
  number_unique_customers > nov_avgs$mean_unq)
```

``` r
nov_data <- nov_data %>%
  rename_with(.cols = ,~paste0("nov_",.x))%>%
  rename("product_id" = nov_product_id) 

dec_data <- dec_data %>%
  rename_with(.cols = ,~paste0("dec_",.x))%>%
  rename("product_id" = dec_product_id) 


aggregate_data <- nov_data %>%
  full_join(y=dec_data)
```

``` r
aggregate_data <- aggregate_data %>%
  mutate(conv_growth_rate = ((dec_conversion_rate-nov_conversion_rate)/nov_conversion_rate)*100,
  unit_growth_rate = ((dec_number_units_sold-nov_number_units_sold)/nov_number_units_sold)*100,
  unq_cust_growth_rate = ((dec_number_unique_customers-nov_number_unique_customers)/nov_number_unique_customers)*100)


growth_avgs <- aggregate_data %>%
  summarise(mean_conv_growth = mean(conv_growth_rate,na.rm = T),
  mean_unq_cust_growth = mean(unq_cust_growth_rate,na.rm = T),
  mean_units_growth = mean(unit_growth_rate,na.rm = T))

top_products <- aggregate_data %>%
  filter(conv_growth_rate > growth_avgs$mean_conv_growth,
  unit_growth_rate > growth_avgs$mean_units_growth ,
  unq_cust_growth_rate > growth_avgs$mean_unq_cust_growth) %>%
    arrange(desc(unit_growth_rate),desc(unq_cust_growth_rate))
top_products
```

    # A tibble: 59 × 10
       product_id nov_conversion_rate nov_number_units_sold nov_number_unique_cust…¹
       <chr>                    <dbl>                 <int>                    <int>
     1 1004903                  0.342                  1546                     1197
     2 1004723                  0.345                   461                      373
     3 1005273                  0.349                   154                      141
     4 100003338                0.349                    37                       29
     5 4804497                  0.379                    64                       54
     6 1307136                  0.334                   235                      190
     7 5100505                  0.348                    73                       61
     8 1002542                  0.362                   298                      251
     9 1307589                  0.347                   473                      388
    10 1005245                  0.362                   100                       82
    # ℹ 49 more rows
    # ℹ abbreviated name: ¹​nov_number_unique_customers
    # ℹ 6 more variables: dec_conversion_rate <dbl>, dec_number_units_sold <int>,
    #   dec_number_unique_customers <int>, conv_growth_rate <dbl>,
    #   unit_growth_rate <dbl>, unq_cust_growth_rate <dbl>

``` r
best_prod_id <- top_products %>% pull(product_id)

merge <- events_data %>%
  filter(product_id %in% best_prod_id) %>%
  distinct(product_id,.keep_all = T) %>%
  select(product_id , contains("code") , brand , category_id)

top_products <- top_products %>%
  left_join(x=merge) %>%
  select(-cat_code_4)

rm(events_data)
rm(merge)
gc()
```

              used (Mb) gc trigger  (Mb) max used  (Mb)
    Ncells 1588414 84.9    5564820 297.2  8695030 464.4
    Vcells 4867372 37.2   95691180 730.1 99606827 760.0

So we have 50+ products with every variable greater than average of each
variable for all products .

But lets further trim these products :

- We want products that sold twice as much in december as compared to
  november , units growth rate of 100% or more.
- We want products that sold more than 50 units in november .
- We want products that had more than 50 unique customers in in november
  .
- We want products that had twice as much unique customers in december .
- The conversion rates and conversion growths seem fine to me , but we
  would like products with positive conversion growth , more than 5%.

``` r
top_products <- top_products %>%
  filter(
  nov_number_units_sold > 50 ,
  nov_number_unique_customers > 50 ,
  unit_growth_rate > 100 , 
  unq_cust_growth_rate > 100,
  conv_growth_rate > 5) %>%
    arrange(desc(unit_growth_rate),desc(unq_cust_growth_rate))

top_products %>%
  kable()
```

| product_id | category_code | cat_code_1 | cat_code_2 | cat_code_3 | brand | category_id | nov_conversion_rate | nov_number_units_sold | nov_number_unique_customers | dec_conversion_rate | dec_number_units_sold | dec_number_unique_customers | conv_growth_rate | unit_growth_rate | unq_cust_growth_rate |
|:---|:---|:---|:---|:---|:---|:---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| 1004903 | electronics.smartphone | electronics | smartphone | NA | huawei | 2232732093077520640 | 0.3418085 | 1546 | 1197 | 0.3927416 | 6558 | 4656 | 14.90106 | 324.1915 | 288.9724 |
| 1004723 | electronics.smartphone | electronics | smartphone | NA | huawei | 2232732093077520640 | 0.3453184 | 461 | 373 | 0.3816435 | 1867 | 1427 | 10.51932 | 304.9892 | 282.5737 |
| 1005273 | electronics.smartphone | electronics | smartphone | NA | huawei | 2232732093077520640 | 0.3492063 | 154 | 141 | 0.4183751 | 551 | 441 | 19.80741 | 257.7922 | 212.7660 |
| 4804497 | sport.bicycle | sport | bicycle | NA | jbl | 2232732079706079232 | 0.3786982 | 64 | 54 | 0.4232673 | 171 | 146 | 11.76903 | 167.1875 | 170.3704 |
| 1307136 | NA | NA | NA | NA | NA | NA | 0.3338068 | 235 | 190 | 0.4331927 | 616 | 485 | 29.77347 | 162.1277 | 155.2632 |
| 5100505 | electronics.clocks | electronics | clocks | NA | apple | 2232732103101907456 | 0.3476190 | 73 | 61 | 0.4390805 | 191 | 134 | 26.31082 | 161.6438 | 119.6721 |
| 1002542 | electronics.smartphone | electronics | smartphone | NA | apple | 2232732093077520640 | 0.3616505 | 298 | 251 | 0.4036113 | 760 | 597 | 11.60258 | 155.0336 | 137.8486 |
| 1307589 | electronics.audio.headphone | electronics | audio | headphone | lenovo | 2053013554658804224 | 0.3472834 | 473 | 388 | 0.4020101 | 1200 | 951 | 15.75850 | 153.6998 | 145.1031 |
| 1005245 | electronics.smartphone | electronics | smartphone | NA | apple | 2232732093077520640 | 0.3623188 | 100 | 82 | 0.4737864 | 244 | 175 | 30.76505 | 144.0000 | 113.4146 |
| 5801655 | electronics.audio.subwoofer | electronics | audio | subwoofer | element | 2232732082390433792 | 0.3582090 | 144 | 124 | 0.4805014 | 345 | 257 | 34.13997 | 139.5833 | 107.2581 |
| 1005177 | electronics.smartphone | electronics | smartphone | NA | samsung | 2232732093077520640 | 0.4044353 | 1769 | 1243 | 0.5131399 | 4120 | 2698 | 26.87811 | 132.8999 | 117.0555 |
| 1005100 | electronics.smartphone | electronics | smartphone | NA | samsung | 2232732093077520640 | 0.3400456 | 10132 | 7178 | 0.4108056 | 23457 | 15279 | 20.80896 | 131.5140 | 112.8587 |
| 1002547 | electronics.smartphone | electronics | smartphone | NA | apple | 2232732093077520640 | 0.3644603 | 1327 | 1016 | 0.4267223 | 3066 | 2167 | 17.08335 | 131.0475 | 113.2874 |
| 1004748 | electronics.smartphone | electronics | smartphone | NA | huawei | 2232732093077520640 | 0.3320861 | 355 | 283 | 0.3963565 | 805 | 646 | 19.35354 | 126.7606 | 128.2686 |
| 1004857 | electronics.smartphone | electronics | smartphone | NA | samsung | 2232732093077520640 | 0.3561004 | 1830 | 1490 | 0.4176780 | 3889 | 2997 | 17.29220 | 112.5137 | 101.1409 |
| 21401209 | electronics.clocks | electronics | clocks | NA | casio | 2232732103101907456 | 0.3333333 | 69 | 56 | 0.4303406 | 139 | 117 | 29.10217 | 101.4493 | 108.9286 |

We have a total of 16 products . Then we have two potential errors in
our final results . A sport bicycle by JBL do not seem right at all .
Then we have one product with NULLs for brand and categories .

But no worries since this is just our minumum viable solution . We will
iterate once we clarified a few things with our stakeholders .

## Save final results table

``` r
# write_csv(x = top_products , file = "data/top_products.csv")
```
