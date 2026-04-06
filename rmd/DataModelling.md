# Online Odyssey Outlet - Data Modelling Script
Lebintiti Kobe

- [Load Libraries](#load-libraries)
- [Load Data](#load-data)
  - [Clean up character data](#clean-up-character-data)
  - [Convert date column into their correct
    format](#convert-date-column-into-their-correct-format)
  - [Validate the data](#validate-the-data)
  - [User Columns](#user-columns)
  - [Events types](#events-types)
  - [Events Times](#events-times)
  - [Price](#price)
  - [Products](#products)
  - [Augment our data](#augment-our-data)
  - [Save our data model](#save-our-data-model)

# Load Libraries

<details class="code-fold">
<summary>Code</summary>

``` r
options(scipen = 999)
require(tidyverse)
require(here)
require(skimr)
require(pander)
require(ggthemes)
require(kableExtra)
require(bizdays)
require(timeDate)
```

</details>

# Load Data

<details class="code-fold">
<summary>Code</summary>

``` r
nov_data <- read_csv(here("data/november.csv.gz"))
dec_data <- read_csv(here("data/december.csv.gz"))
```

</details>

Merge november and december data since the have the same data schema
then view the data

<details class="code-fold">
<summary>Code</summary>

``` r
events_data <- bind_rows(nov_data, dec_data)
```

</details>

## Clean up character data

- Convert them to the same casing
- Remove whitespaces

<details class="code-fold">
<summary>Code</summary>

``` r
events_data <- events_data %>%
  mutate(across(where(is.character), ~ trimws(.) %>% str_to_lower())) %>%
  mutate(across(contains("id"),~as.character(.)))
```

</details>

## Convert date column into their correct format

- Convert date in character format to date format

<details class="code-fold">
<summary>Code</summary>

``` r
events_data <- events_data %>%
  mutate(across(contains("time"), ~ as_datetime(.))) %>%
  janitor::clean_names()
```

</details>

## Validate the data

Validate the data to make sure it do not violate business logic :

### Online Stores Business logic :

User Columns :

- Each row of our data should have a session_id .

Event types :

- Usually we should have more product impressions than cart actions ,
  more cart actions than checkouts and so forth .

Date Columns :

- Data should be available on all business days dates .
- Times of order should always be before delivery times / return times ,
  and so on .

Price Columns :

- Prices can be less than zero .
- Investigate prices that are too high or too low .

Products/Brands/Categories :

- Each product ID should be unique to each product .
- Each product ID should have one brand/category and so forth .

Of course , there is so much we can validate for online store data , but
for now we focus on concepts that can be found in the data we have ,
nothing more nothing less .

## User Columns

<details class="code-fold">
<summary>Code</summary>

``` r
events_data %>%
  select(contains("user")) %>% 
  skim()
```

</details>

|                                                  |            |
|:-------------------------------------------------|:-----------|
| Name                                             | Piped data |
| Number of rows                                   | 7033125    |
| Number of columns                                | 2          |
| \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_   |            |
| Column type frequency:                           |            |
| character                                        | 2          |
| \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ |            |
| Group variables                                  | None       |

Data summary

**Variable type: character**

| skim_variable | n_missing | complete_rate | min | max | empty | n_unique | whitespace |
|:--------------|----------:|--------------:|----:|----:|------:|---------:|-----------:|
| user_id       |         0 |             1 |   8 |   9 |     0 |  1333980 |          0 |
| user_session  |        27 |             1 |  36 |  36 |     0 |  3169946 |          0 |

- We have 27 events records without any user_id . Lets investing these

<details class="code-fold">
<summary>Code</summary>

``` r
events_data %>%
  filter(is.na(user_session)) %>%
  select(event_type, user_id, price, user_session) %>%
  print(n = 50) %>%
  kable()
```

</details>

    # A tibble: 27 × 4
       event_type user_id    price user_session
       <chr>      <chr>      <dbl> <chr>       
     1 cart       568843390  566.  <NA>        
     2 cart       570411102   97.8 <NA>        
     3 cart       570878749  244.  <NA>        
     4 cart       573722572  360.  <NA>        
     5 cart       576301354  181.  <NA>        
     6 cart       576935861  335.  <NA>        
     7 cart       579123407 1001.  <NA>        
     8 cart       580714819  708.  <NA>        
     9 cart       519513506   10.2 <NA>        
    10 cart       584193732   63.1 <NA>        
    11 cart       585526753  231.  <NA>        
    12 cart       585953165   97.8 <NA>        
    13 cart       586879399  226.  <NA>        
    14 cart       574084617   38.4 <NA>        
    15 cart       587428481  111.  <NA>        
    16 cart       589172852 1335.  <NA>        
    17 cart       590035230  327.  <NA>        
    18 cart       591290578  103.  <NA>        
    19 cart       591580955  452.  <NA>        
    20 cart       535383117  128.  <NA>        
    21 cart       592696948   63.1 <NA>        
    22 cart       592718043   22.3 <NA>        
    23 cart       570391628  893.  <NA>        
    24 cart       541641713  463.  <NA>        
    25 cart       594002694  106.  <NA>        
    26 cart       532482951  274.  <NA>        
    27 cart       595269952  446.  <NA>        

| event_type | user_id   |   price | user_session |
|:-----------|:----------|--------:|:-------------|
| cart       | 568843390 |  566.27 | NA           |
| cart       | 570411102 |   97.81 | NA           |
| cart       | 570878749 |  243.51 | NA           |
| cart       | 573722572 |  360.09 | NA           |
| cart       | 576301354 |  181.47 | NA           |
| cart       | 576935861 |  334.58 | NA           |
| cart       | 579123407 | 1001.42 | NA           |
| cart       | 580714819 |  707.84 | NA           |
| cart       | 519513506 |   10.17 | NA           |
| cart       | 584193732 |   63.06 | NA           |
| cart       | 585526753 |  231.38 | NA           |
| cart       | 585953165 |   97.79 | NA           |
| cart       | 586879399 |  226.30 | NA           |
| cart       | 574084617 |   38.35 | NA           |
| cart       | 587428481 |  110.65 | NA           |
| cart       | 589172852 | 1334.65 | NA           |
| cart       | 590035230 |  326.88 | NA           |
| cart       | 591290578 |  102.68 | NA           |
| cart       | 591580955 |  452.17 | NA           |
| cart       | 535383117 |  128.32 | NA           |
| cart       | 592696948 |   63.06 | NA           |
| cart       | 592718043 |   22.33 | NA           |
| cart       | 570391628 |  892.94 | NA           |
| cart       | 541641713 |  463.31 | NA           |
| cart       | 594002694 |  106.03 | NA           |
| cart       | 532482951 |  273.74 | NA           |
| cart       | 595269952 |  445.98 | NA           |

- These are all Cart events . These should normally have session_ids ,
  but it is possible that maybe the user lost connection and when they
  got reconnected they were assigned a new seesion id . Ask stakeholders
  about these .

- For now we keep all these , in order to get true conversion of all out
  items

## Events types

<details class="code-fold">
<summary>Code</summary>

``` r
events_data %>%
  select(event_type) %>%
  skim()
```

</details>

|                                                  |            |
|:-------------------------------------------------|:-----------|
| Name                                             | Piped data |
| Number of rows                                   | 7033125    |
| Number of columns                                | 1          |
| \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_   |            |
| Column type frequency:                           |            |
| character                                        | 1          |
| \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ |            |
| Group variables                                  | None       |

Data summary

**Variable type: character**

| skim_variable | n_missing | complete_rate | min | max | empty | n_unique | whitespace |
|:--------------|----------:|--------------:|----:|----:|------:|---------:|-----------:|
| event_type    |         0 |             1 |   4 |   8 |     0 |        2 |          0 |

<details class="code-fold">
<summary>Code</summary>

``` r
events_data %>%
  janitor::tabyl(event_type) %>%
  kableExtra::kable()
```

</details>

| event_type |       n |   percent |
|:-----------|--------:|----------:|
| cart       | 5276372 | 0.7502173 |
| purchase   | 1756753 | 0.2497827 |

- 75% of all our events are cart events , which is to be expected in an
  online store system . People can add products to cart only to change
  their mind , or only to buy only those they can afford , matter fact
  not all items in cart usually get paid for .

## Events Times

<details class="code-fold">
<summary>Code</summary>

``` r
events_data %>% 
  select(contains("time"))%>%
  skim()
```

</details>

|                                                  |            |
|:-------------------------------------------------|:-----------|
| Name                                             | Piped data |
| Number of rows                                   | 7033125    |
| Number of columns                                | 1          |
| \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_   |            |
| Column type frequency:                           |            |
| POSIXct                                          | 1          |
| \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ |            |
| Group variables                                  | None       |

Data summary

**Variable type: POSIXct**

| skim_variable | n_missing | complete_rate | min | max | median | n_unique |
|:---|---:|---:|:---|:---|:---|---:|
| event_time | 0 | 1 | 2019-11-01 00:00:14 | 2019-12-31 23:59:09 | 2019-12-07 16:10:17 | 3033668 |

- Our dates range from 2019-11-01 to 2019-12-31 . Which reassures us
  that our data was merged correctly .

### Verify if each working day has a record

- Unlike physical stores , online store can run even on non-working days
  .

> - But if we had to find a list of all business days from one date to
>   another , we use the {bizdays} together with {timeDate} r packages .
> - business_days \<- bizdays::bizseq(from,to,“weekends”,holidays) .
> - holidays \<- For holidays , we use timeDate::holidays(from , to ,
>   cal = “US”) and so forth .
> - Then full_bizdays \<- setdiff(business_days , holidays)

<details class="code-fold">
<summary>Code</summary>

``` r
events_data <- events_data %>%
  mutate(event_date = as_date(event_time))

events_data %>%
  group_by(event_date) %>%
  count() %>%
  filter(n < 1) %>%
  kable()
```

</details>

| event_date |   n |
|:-----------|----:|

- No results got returned meaning there is no working date without a
  record / event , therefore we can be a little extra confident on our
  data

### Check the number of events distribution for each day of our event times

<details class="code-fold">
<summary>Code</summary>

``` r
events_data %>%
  group_by(event_date) %>%
  summarise(n = n()) %>%
  ggplot(aes(x = event_date, y = n, fill = n)) +
  geom_col(show.legend = F) +
  labs(title = "Distribution of events per day", x = "Event Date", y = "") +
  scale_y_continuous(
    n.breaks = 10,
    labels = scales::label_number(scale = 0.001, suffix = "K")
  ) +
  theme_excel_new()
```

</details>

![](DataModelling_files/figure-commonmark/unnamed-chunk-11-1.png)

- Events are the lowest at the start of november , may be the time when
  the start-up launched (It actually the time the start up launched) .
- Events are the highest mid-november , maybe an event was held around
  this time . Ask stakeholders about this spike so we can be able to
  adjust for artificial increases in sales , to reduce bias when
  selecting best performing products .
- Then after we see a steady rise in events overtime , maybe correspond
  to normal business growth .

Enquire about these to figure out what was happening around these times
. Remember we need to adjust for any artificial boosting of events/sales
because these can bias products that were advertised as part of boosting
campaigns

## Price

<details class="code-fold">
<summary>Code</summary>

``` r
events_data %>%
  select(price) %>%
  skim()
```

</details>

|                                                  |            |
|:-------------------------------------------------|:-----------|
| Name                                             | Piped data |
| Number of rows                                   | 7033125    |
| Number of columns                                | 1          |
| \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_   |            |
| Column type frequency:                           |            |
| numeric                                          | 1          |
| \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ |            |
| Group variables                                  | None       |

Data summary

**Variable type: numeric**

| skim_variable | n_missing | complete_rate | mean | sd | p0 | p25 | p50 | p75 | p100 | hist |
|:---|---:|---:|---:|---:|---:|---:|---:|---:|---:|:---|
| price | 0 | 1 | 318.68 | 343.81 | 0 | 104.25 | 189.18 | 385.83 | 2574.07 | ▇▂▁▁▁ |

- We have a price range of \$0 - \$2600 and most products are prices
  below \$600 . More than 75% of the prices are below \$400

### Investigate low price products

<details class="code-fold">
<summary>Code</summary>

``` r
events_data %>%
  distinct(product_id, .keep_all = T) %>%
  filter(price <= 0) %>%
  distinct(product_id, .keep_all = T)
```

</details>

    # A tibble: 1,365 × 10
       event_time          event_type product_id category_id     category_code brand
       <dttm>              <chr>      <chr>      <chr>           <chr>         <chr>
     1 2019-11-08 05:55:09 cart       5800296    20530135539457… electronics.… <NA> 
     2 2019-11-08 06:36:20 cart       19200239   20530135562023… construction… <NA> 
     3 2019-11-08 08:54:07 cart       13500452   20530135570998… furniture.be… <NA> 
     4 2019-11-08 09:51:42 cart       5701421    20530135539709… auto.accesso… <NA> 
     5 2019-11-08 11:47:28 cart       28722404   20530135657820… apparel.shoes <NA> 
     6 2019-11-08 13:06:49 cart       1802136    20530135544155… electronics.… <NA> 
     7 2019-11-08 17:01:14 cart       1005283    20530135556318… electronics.… <NA> 
     8 2019-11-08 17:07:48 cart       28722414   20530135656394… apparel.shoes <NA> 
     9 2019-11-08 17:12:31 cart       17200939   20530135597926… furniture.li… <NA> 
    10 2019-11-08 17:37:17 cart       50600107   21349050448336… auto.accesso… <NA> 
    # ℹ 1,355 more rows
    # ℹ 4 more variables: price <dbl>, user_id <chr>, user_session <chr>,
    #   event_date <date>

<details class="code-fold">
<summary>Code</summary>

``` r
zero_price_ids <- events_data %>%
  distinct(product_id, .keep_all = T) %>%
  filter(price <= 0) %>%
  distinct(product_id, .keep_all = T) %>%
  pull(product_id)

events_data %>%
  filter(product_id %in% zero_price_ids) %>%
  arrange(product_id) %>%
  group_by(product_id) %>%
  summarise(
    n_event_type = length(list(unique(event_type))),
    n_brands = length(list(unique(brand)))
  ) %>%
  filter(n_brands > 1 | n_event_type > 1)
```

</details>

    # A tibble: 0 × 3
    # ℹ 3 variables: product_id <chr>, n_event_type <int>, n_brands <int>

- We have around 1400 products with prices of zero , all of them do not
  have brands codes and they dont have any purchase event . These may
  products that we dropped from the catalog by some brand which was not
  selling well . Or products the store was planning to sell but did not
  the budget to actually stock them .
- Here we can drop the products since it wont be possible to calculate
  metrics like Revenue , Conversion , e.t.c

<details class="code-fold">
<summary>Code</summary>

``` r
events_data <- events_data %>%
  filter(!product_id %in% zero_price_ids)
```

</details>

### Investigate high priced products

<details class="code-fold">
<summary>Code</summary>

``` r
events_data %>%
  select(-contains("event"), -contains("user")) %>%
  filter(price > 2000) %>%
  janitor::tabyl(category_code) %>%
  arrange(desc(percent)) %>%
  kable()
```

</details>

| category_code                          |    n |   percent |
|:---------------------------------------|-----:|----------:|
| electronics.smartphone                 | 2334 | 0.2336570 |
| construction.tools.light               | 1903 | 0.1905096 |
| computers.notebook                     | 1378 | 0.1379517 |
| electronics.video.tv                   |  881 | 0.0881970 |
| appliances.personal.massager           |  608 | 0.0608670 |
| electronics.audio.headphone            |  446 | 0.0446491 |
| computers.desktop                      |  428 | 0.0428471 |
| electronics.clocks                     |  363 | 0.0363400 |
| furniture.bedroom.blanket              |  301 | 0.0301331 |
| appliances.kitchen.refrigerators       |  266 | 0.0266293 |
| computers.peripherals.printer          |  161 | 0.0161177 |
| appliances.sewing_machine              |  128 | 0.0128141 |
| apparel.underwear                      |   60 | 0.0060066 |
| electronics.camera.photo               |   58 | 0.0058064 |
| sport.bicycle                          |   53 | 0.0053058 |
| electronics.audio.microphone           |   46 | 0.0046051 |
| furniture.living_room.sofa             |   45 | 0.0045050 |
| appliances.kitchen.washer              |   43 | 0.0043047 |
| sport.trainer                          |   38 | 0.0038042 |
| computers.components.cooler            |   35 | 0.0035039 |
| apparel.shoes.moccasins                |   32 | 0.0032035 |
| electronics.audio.acoustic             |   29 | 0.0029032 |
| kids.skates                            |   28 | 0.0028031 |
| appliances.kitchen.coffee_machine      |   27 | 0.0027030 |
| construction.tools.drill               |   23 | 0.0023025 |
| auto.accessories.compressor            |   21 | 0.0021023 |
| sport.tennis                           |   21 | 0.0021023 |
| apparel.shirt                          |   19 | 0.0019021 |
| apparel.shorts                         |   19 | 0.0019021 |
| appliances.iron                        |   19 | 0.0019021 |
| furniture.bathroom.bath                |   16 | 0.0016018 |
| apparel.shoes.keds                     |   15 | 0.0015017 |
| electronics.audio.music_tools.piano    |   15 | 0.0015017 |
| appliances.kitchen.toster              |   13 | 0.0013014 |
| computers.components.motherboard       |    9 | 0.0009010 |
| construction.components.faucet         |    9 | 0.0009010 |
| apparel.sock                           |    8 | 0.0008009 |
| construction.tools.saw                 |    8 | 0.0008009 |
| construction.tools.screw               |    8 | 0.0008009 |
| kids.toys                              |    8 | 0.0008009 |
| electronics.camera.video               |    7 | 0.0007008 |
| apparel.costume                        |    6 | 0.0006007 |
| apparel.jumper                         |    6 | 0.0006007 |
| kids.carriage                          |    5 | 0.0005006 |
| sport.ski                              |    5 | 0.0005006 |
| computers.components.videocards        |    4 | 0.0004004 |
| appliances.kitchen.mixer               |    3 | 0.0003003 |
| construction.tools.generator           |    3 | 0.0003003 |
| electronics.telephone                  |    3 | 0.0003003 |
| furniture.bedroom.bed                  |    3 | 0.0003003 |
| sport.snowboard                        |    3 | 0.0003003 |
| apparel.trousers                       |    2 | 0.0002002 |
| appliances.kitchen.hood                |    2 | 0.0002002 |
| appliances.kitchen.oven                |    2 | 0.0002002 |
| country_yard.lawn_mower                |    2 | 0.0002002 |
| electronics.video.projector            |    2 | 0.0002002 |
| furniture.kitchen.table                |    2 | 0.0002002 |
| furniture.living_room.cabinet          |    2 | 0.0002002 |
| apparel.shoes.espadrilles              |    1 | 0.0001001 |
| appliances.environment.air_conditioner |    1 | 0.0001001 |
| appliances.kitchen.steam_cooker        |    1 | 0.0001001 |
| computers.peripherals.monitor          |    1 | 0.0001001 |
| construction.tools.welding             |    1 | 0.0001001 |

- Looking at top categories of high priced items by number of events ,
  the categories listed here are usually high prices , there is nothing
  too suspicious here

## Products

Check if each product_id is associated with one brand and category

<details class="code-fold">
<summary>Code</summary>

``` r
dup_data <- events_data %>%
  group_by(product_id) %>%
  summarise(
    n_brands = n_distinct(brand),
    n_cat_id = n_distinct(category_id),
    n_cat_code = n_distinct(category_code)
  )

dup_ids <- dup_data %>%
  filter(n_brands > 1 | n_cat_id > 1 | n_cat_code > 1) %>%
  pull(product_id)

events_data %>%
  filter(product_id %in% dup_ids) %>%
  arrange(product_id) %>%
  distinct(product_id, category_code,category_id,brand ,.keep_all = T) %>% print(n=300)
```

</details>

    # A tibble: 39,389 × 10
        event_time          event_type product_id category_id    category_code brand
        <dttm>              <chr>      <chr>      <chr>          <chr>         <chr>
      1 2019-11-17 11:03:35 cart       100000153  2053013565782… apparel.shoes resp…
      2 2019-12-03 17:14:57 cart       100000153  2232732098706… apparel.shoes resp…
      3 2019-11-14 18:01:29 cart       100000154  2053013565782… apparel.shoes resp…
      4 2019-12-04 13:50:49 cart       100000154  2053013556521… apparel.shoes resp…
      5 2019-11-14 20:04:11 cart       100000179  2053013560623… auto.accesso… park…
      6 2019-12-22 19:02:56 cart       100000179  2232732109494… accessories.… park…
      7 2019-11-16 11:11:03 cart       100000196  2070005009382… apparel.unde… mila…
      8 2019-12-22 17:19:22 cart       100000196  2053013555573… electronics.… mila…
      9 2019-11-15 09:03:23 cart       100000208  2070005009382… apparel.unde… mila…
     10 2019-12-22 16:17:00 cart       100000208  2053013555573… electronics.… mila…
     11 2019-11-16 04:42:47 cart       100000210  2053013553970… auto.accesso… jvc  
     12 2019-12-02 13:43:37 cart       100000210  2053013554415… electronics.… jvc  
     13 2019-11-14 06:25:28 cart       100000244  2053013553945… electronics.… graf…
     14 2019-12-01 19:10:34 cart       100000244  2232732082390… electronics.… graf…
     15 2019-11-11 09:14:57 cart       100000263  2053013553945… electronics.… jvc  
     16 2019-12-02 16:50:12 cart       100000263  2232732082390… electronics.… jvc  
     17 2019-11-16 05:58:14 cart       100000273  2053013560623… auto.accesso… park…
     18 2019-12-29 21:13:39 cart       100000273  2232732109494… accessories.… park…
     19 2019-11-13 06:07:07 cart       100000324  2053013558945… accessories.… cont…
     20 2019-12-07 14:14:07 cart       100000324  2053013551907… sport.ski     cont…
     21 2019-11-21 13:24:44 cart       100000368  2053013552326… appliances.e… deli…
     22 2019-12-20 14:15:07 cart       100000368  2053013557452… electronics.… deli…
     23 2019-11-17 05:19:23 cart       100000783  2053013561780… furniture.be… adam…
     24 2019-12-23 07:44:16 cart       100000783  2053013553375… computers.no… adam…
     25 2019-11-11 18:09:48 cart       100000811  2053013552293… appliances.e… resa…
     26 2019-12-28 14:56:14 cart       100000811  2232732091961… appliances.e… resa…
     27 2019-11-22 21:03:38 cart       100000912  2070005009382… apparel.unde… mila…
     28 2019-12-07 18:31:51 cart       100000912  2053013555573… electronics.… mila…
     29 2019-11-18 14:11:03 cart       100000923  2053013555262… appliances.k… deli…
     30 2019-12-05 09:59:02 cart       100000923  2232732091391… appliances.k… deli…
     31 2019-11-13 15:13:29 cart       100001169  2053013561579… electronics.… skag…
     32 2019-12-06 09:53:31 cart       100001169  2232732082063… electronics.… skag…
     33 2019-11-14 23:08:08 cart       100001222  2053013561579… electronics.… zepp…
     34 2019-12-17 12:31:53 cart       100001222  2232732082063… electronics.… zepp…
     35 2019-11-13 10:37:16 cart       100001331  2053013561579… electronics.… arma…
     36 2019-12-13 15:18:09 cart       100001331  2232732082063… electronics.… arma…
     37 2019-11-17 15:11:16 cart       100001368  2053013561579… electronics.… mich…
     38 2019-12-11 18:57:21 cart       100001368  2232732082063… electronics.… mich…
     39 2019-11-26 03:25:21 cart       100001372  2053013561579… electronics.… mich…
     40 2019-12-19 10:24:21 cart       100001372  2232732082063… electronics.… mich…
     41 2019-11-23 19:08:46 cart       100001428  2053013561780… furniture.be… dorm…
     42 2019-12-26 15:38:28 cart       100001428  2053013553375… computers.no… dorm…
     43 2019-11-16 16:16:05 cart       100001429  2053013561579… electronics.… dkny 
     44 2019-12-30 11:27:23 cart       100001429  2232732082063… electronics.… dkny 
     45 2019-11-17 17:56:35 cart       100001505  2053013561579… electronics.… foss…
     46 2019-12-24 09:32:23 cart       100001505  2232732082063… electronics.… foss…
     47 2019-11-14 13:59:33 cart       100001529  2053013552293… appliances.e… rovus
     48 2019-12-08 05:24:11 cart       100001529  2232732091961… appliances.e… rovus
     49 2019-11-14 05:19:44 cart       100001549  2086471240800… apparel.trou… puma 
     50 2019-12-02 14:52:35 cart       100001549  2053013558978… sport.bicycle puma 
     51 2019-11-16 07:00:42 cart       100001564  2146660887002… apparel.dress haya…
     52 2019-12-03 12:16:33 cart       100001564  2053013557645… electronics.… haya…
     53 2019-11-16 14:02:30 cart       100001577  2053013561579… electronics.… skag…
     54 2019-12-15 07:47:25 cart       100001577  2232732082063… electronics.… skag…
     55 2019-11-14 09:19:19 cart       100001674  2053013561579… electronics.… skag…
     56 2019-12-01 06:03:04 cart       100001674  2232732082063… electronics.… skag…
     57 2019-11-30 13:22:24 cart       100001718  2053013561420… accessories.… amen 
     58 2019-12-04 14:24:37 cart       100001718  2232732089369… accessories.… amen 
     59 2019-11-27 08:15:21 cart       100001796  2053013561420… accessories.… amen 
     60 2019-12-12 13:45:54 cart       100001796  2232732110643… country_yard… amen 
     61 2019-11-27 08:14:40 cart       100001800  2053013561420… accessories.… amen 
     62 2019-12-26 19:18:50 cart       100001800  2232732110643… country_yard… amen 
     63 2019-11-09 13:53:58 cart       100001820  2053013559792… furniture.li… pins…
     64 2019-12-18 09:35:25 cart       100001820  2232732090980… furniture.li… pins…
     65 2019-11-27 17:43:31 cart       100001852  2053013559792… furniture.li… pins…
     66 2019-12-08 07:44:53 cart       100001852  2232732090980… furniture.li… pins…
     67 2019-11-15 10:17:32 cart       100001925  2053013561420… accessories.… amen 
     68 2019-12-23 06:17:42 cart       100001925  2232732110643… country_yard… amen 
     69 2019-11-17 14:02:54 cart       100001927  2053013561420… accessories.… amen 
     70 2019-12-02 03:02:02 cart       100001927  2232732110643… country_yard… amen 
     71 2019-11-18 02:20:44 cart       100001983  2053013561847… furniture.be… dorm…
     72 2019-12-26 06:00:51 cart       100001983  2232732089302… furniture.be… dorm…
     73 2019-11-12 12:40:37 cart       100001984  2053013557670… electronics.… cara…
     74 2019-12-02 10:26:59 cart       100001984  2232732082734… electronics.… cara…
     75 2019-11-10 08:09:52 cart       100001985  2053013557670… electronics.… cara…
     76 2019-12-20 08:49:53 cart       100001985  2232732082734… electronics.… cara…
     77 2019-11-16 03:30:38 cart       100001986  2053013561847… furniture.be… dorm…
     78 2019-12-19 13:34:45 cart       100001986  2232732089302… furniture.be… dorm…
     79 2019-11-15 08:58:52 cart       100001987  2053013557670… electronics.… cara…
     80 2019-12-20 12:45:01 cart       100001987  2232732082734… electronics.… cara…
     81 2019-11-15 00:42:31 cart       100001988  2053013557670… electronics.… adag…
     82 2019-12-01 17:31:47 cart       100001988  2232732107883… electronics.… adag…
     83 2019-11-15 09:41:26 cart       100001989  2053013561847… furniture.be… dorm…
     84 2019-12-07 13:27:22 cart       100001989  2232732089302… furniture.be… dorm…
     85 2019-11-29 10:07:15 cart       100001990  2053013557670… electronics.… devi…
     86 2019-12-15 17:34:07 cart       100001990  2232732107883… electronics.… devi…
     87 2019-11-17 19:54:17 cart       100001991  2053013561847… furniture.be… dorm…
     88 2019-12-17 15:22:22 cart       100001991  2232732089302… furniture.be… dorm…
     89 2019-11-14 19:04:10 cart       100001992  2053013557670… electronics.… joker
     90 2019-12-16 13:44:24 cart       100001992  2232732082734… electronics.… joker
     91 2019-11-15 18:54:15 cart       100001993  2053013557670… electronics.… adag…
     92 2019-12-06 05:22:02 cart       100001993  2232732107883… electronics.… adag…
     93 2019-11-25 10:37:00 cart       100001999  2053013566176… appliances.i… rovus
     94 2019-12-08 11:08:10 cart       100001999  2053013557477… furniture.ba… rovus
     95 2019-11-15 18:09:58 cart       100002006  2053013557192… furniture.be… orto…
     96 2019-12-01 18:55:53 cart       100002006  2232732061804… furniture.be… orto…
     97 2019-11-14 18:01:41 cart       100002007  2053013557192… furniture.be… orto…
     98 2019-12-18 06:19:20 cart       100002007  2232732061804… furniture.be… orto…
     99 2019-11-15 06:19:24 cart       100002008  2053013557192… furniture.be… orto…
    100 2019-12-02 01:35:29 cart       100002008  2232732061804… furniture.be… orto…
    101 2019-11-16 12:15:12 cart       100002024  2053013565941… appliances.p… well…
    102 2019-12-08 04:54:31 cart       100002024  2232732114611… construction… well…
    103 2019-11-16 17:17:14 cart       100002031  2053013565941… appliances.p… well…
    104 2019-12-09 16:23:48 cart       100002031  2232732114611… construction… well…
    105 2019-11-16 12:12:06 cart       100002034  2053013565941… appliances.p… well…
    106 2019-12-13 08:41:55 cart       100002034  2232732114611… construction… well…
    107 2019-11-13 15:46:07 cart       100002043  2053013558920… computers.no… acer 
    108 2019-12-02 06:07:09 cart       100002043  2053013554658… electronics.… acer 
    109 2019-11-12 09:41:46 cart       100002087  2053013558920… computers.no… leno…
    110 2019-12-01 08:37:17 cart       100002087  2053013554658… electronics.… leno…
    111 2019-11-13 04:40:41 cart       100002113  2053013558920… computers.no… leno…
    112 2019-12-01 06:34:28 cart       100002113  2053013554658… electronics.… leno…
    113 2019-11-10 10:04:01 cart       100002174  2053013558920… computers.no… leno…
    114 2019-12-12 08:37:10 cart       100002174  2053013554658… electronics.… leno…
    115 2019-11-10 06:30:02 cart       100002201  2053013558920… computers.no… leno…
    116 2019-12-19 03:58:42 cart       100002201  2053013554658… electronics.… leno…
    117 2019-11-13 08:44:37 cart       100002263  2053013558920… computers.no… leno…
    118 2019-12-01 07:11:10 cart       100002263  2053013554658… electronics.… leno…
    119 2019-11-16 09:12:40 cart       100002283  2146660887002… apparel.dress omero
    120 2019-12-14 09:33:31 cart       100002283  2053013557645… electronics.… omero
    121 2019-11-12 10:01:33 cart       100002292  2053013554658… electronics.… xiao…
    122 2019-12-01 10:06:51 cart       100002292  2232732079706… sport.bicycle xiao…
    123 2019-11-16 05:49:33 cart       100002304  2090228413380… apparel.unde… visa…
    124 2019-12-04 05:37:34 cart       100002304  2053013561302… apparel.shoe… visa…
    125 2019-11-30 19:45:39 cart       100002308  2090228413380… apparel.unde… visa…
    126 2019-12-03 07:41:00 cart       100002308  2053013561302… apparel.shoe… visa…
    127 2019-11-18 15:00:27 cart       100002310  2053013559868… computers.de… aero…
    128 2019-12-03 16:29:50 cart       100002310  2232732086257… electronics.… aero…
    129 2019-11-16 16:10:19 cart       100002365  2090228413380… apparel.unde… visa…
    130 2019-12-08 03:28:31 cart       100002365  2053013561302… apparel.shoe… visa…
    131 2019-11-14 06:30:14 cart       100002367  2053013554121… computers.co… amd  
    132 2019-12-04 08:18:46 cart       100002367  2232732103294… apparel.shoe… amd  
    133 2019-11-12 05:37:18 cart       100002371  2053013558920… computers.no… leno…
    134 2019-12-03 06:04:25 cart       100002371  2053013554658… electronics.… leno…
    135 2019-11-16 17:14:48 cart       100002385  2090228413380… apparel.unde… visa…
    136 2019-12-02 12:34:29 cart       100002385  2053013561302… apparel.shoe… visa…
    137 2019-11-17 08:43:43 cart       100002386  2090228413380… apparel.unde… visa…
    138 2019-12-15 08:24:53 cart       100002386  2053013561302… apparel.shoe… visa…
    139 2019-11-17 04:04:07 cart       100002409  2090228413380… apparel.unde… visa…
    140 2019-12-21 17:26:33 cart       100002409  2053013561302… apparel.shoe… visa…
    141 2019-11-13 03:02:45 cart       100002460  2053013553945… electronics.… <NA> 
    142 2019-12-01 08:57:29 cart       100002460  2232732082390… electronics.… <NA> 
    143 2019-11-15 19:19:37 cart       100002461  2053013553945… electronics.… <NA> 
    144 2019-12-01 19:02:19 cart       100002461  2232732082390… electronics.… <NA> 
    145 2019-11-12 17:42:02 cart       100002462  2053013553945… electronics.… <NA> 
    146 2019-12-01 08:36:47 cart       100002462  2232732082390… electronics.… <NA> 
    147 2019-12-03 05:06:45 cart       100002527  2232732084546… furniture.li… <NA> 
    148 2019-12-04 17:03:31 cart       100002527  2232732084546… furniture.li… lego 
    149 2019-12-02 05:54:57 cart       100002530  2232732084546… furniture.li… <NA> 
    150 2019-12-10 11:14:28 cart       100002530  2232732084546… furniture.li… lego 
    151 2019-11-16 12:06:57 cart       100002560  2053013560807… auto.accesso… star…
    152 2019-12-02 22:18:25 cart       100002560  2232732103546… apparel.shoes star…
    153 2019-11-15 05:08:18 cart       100002716  2053013556311… construction… resa…
    154 2019-12-15 10:24:51 cart       100002716  2053013563743… appliances.k… resa…
    155 2019-11-14 18:59:20 cart       100002720  2053013566209… accessories.… tony…
    156 2019-12-02 16:04:21 cart       100002720  2232732082935… accessories.… tony…
    157 2019-11-21 16:03:24 cart       100002723  2053013566209… accessories.… tony…
    158 2019-12-15 19:34:18 cart       100002723  2232732082935… accessories.… tony…
    159 2019-11-17 10:45:14 cart       100002726  2053013556311… construction… resa…
    160 2019-12-10 07:11:15 cart       100002726  2053013563743… appliances.k… resa…
    161 2019-11-15 17:42:10 cart       100002728  2053013566209… accessories.… tony…
    162 2019-12-08 21:12:26 cart       100002728  2232732082935… accessories.… tony…
    163 2019-11-30 06:53:07 cart       100002729  2053013556311… construction… resa…
    164 2019-12-15 10:28:42 cart       100002729  2053013563743… appliances.k… resa…
    165 2019-11-27 18:20:19 cart       100002730  2053013556311… construction… resa…
    166 2019-12-20 06:10:39 cart       100002730  2053013563743… appliances.k… resa…
    167 2019-11-17 17:58:58 cart       100002747  2053013566209… accessories.… tony…
    168 2019-12-01 17:30:46 cart       100002747  2232732082935… accessories.… tony…
    169 2019-11-16 08:16:14 cart       100002842  2053013556311… construction… resa…
    170 2019-12-29 06:30:20 cart       100002842  2053013563743… appliances.k… resa…
    171 2019-11-25 04:25:42 cart       100002867  2127425436764… construction… ener…
    172 2019-12-09 11:41:00 cart       100002867  2232732095074… construction… ener…
    173 2019-11-17 17:43:58 cart       100002972  2116907524488… apparel.shoe… garv…
    174 2019-12-24 16:37:39 cart       100002972  2053013558643… apparel.shoe… garv…
    175 2019-11-13 17:16:27 cart       100003280  2055156924407… accessories.… <NA> 
    176 2019-12-23 16:28:43 cart       100003280  2053013560623… auto.accesso… luck…
    177 2019-11-12 05:00:38 cart       100003313  2053013552888… computers.pe… cany…
    178 2019-12-12 03:34:30 cart       100003313  2232732104066… furniture.ki… cany…
    179 2019-11-15 15:10:15 cart       100003314  2053013552955… computers.pe… canon
    180 2019-12-02 19:54:33 cart       100003314  2232732071460… kids.toys     canon
    181 2019-12-10 12:59:36 cart       100003314  2232732071460… kids.toys     pant…
    182 2019-11-14 21:11:27 cart       100003319  2053013552888… computers.pe… cany…
    183 2019-12-03 09:57:19 cart       100003319  2232732104066… furniture.ki… cany…
    184 2019-11-30 15:21:38 cart       100003320  2053013552888… computers.pe… leno…
    185 2019-12-02 17:59:10 cart       100003320  2232732104066… furniture.ki… leno…
    186 2019-11-27 17:34:13 cart       100003321  2053013552888… computers.pe… coug…
    187 2019-12-03 10:53:32 cart       100003321  2232732104066… furniture.ki… coug…
    188 2019-11-30 18:39:46 cart       100003322  2053013552888… computers.pe… leno…
    189 2019-12-12 16:06:20 cart       100003322  2232732104066… furniture.ki… leno…
    190 2019-11-12 19:21:08 cart       100003338  2053013558920… computers.no… asus 
    191 2019-12-01 04:46:49 cart       100003338  2053013554658… electronics.… asus 
    192 2019-11-15 21:36:05 cart       100003343  2053013552293… appliances.e… sibr…
    193 2019-12-11 11:26:31 cart       100003343  2232732091961… appliances.e… sibr…
    194 2019-11-15 13:52:35 cart       100003348  2053013552293… appliances.e… sibr…
    195 2019-12-05 19:25:51 cart       100003348  2232732091961… appliances.e… sibr…
    196 2019-11-14 12:55:25 cart       100003358  2053013558920… computers.no… leno…
    197 2019-12-07 07:14:44 cart       100003358  2053013554658… electronics.… leno…
    198 2019-11-12 16:02:09 cart       100003363  2127425436764… construction… ener…
    199 2019-12-27 11:26:36 cart       100003363  2232732095074… construction… ener…
    200 2019-11-14 20:22:22 cart       100003365  2053013561579… electronics.… mich…
    201 2019-12-04 07:57:05 cart       100003365  2232732082063… electronics.… mich…
    202 2019-11-17 03:20:56 cart       100003376  2053013566209… accessories.… tony…
    203 2019-12-03 14:53:14 cart       100003376  2232732082935… accessories.… tony…
    204 2019-11-14 17:35:03 cart       100003378  2053013566209… accessories.… tony…
    205 2019-12-02 08:43:13 cart       100003378  2232732082935… accessories.… tony…
    206 2019-11-14 09:28:55 cart       100003390  2053013558920… computers.no… leno…
    207 2019-12-25 09:06:27 cart       100003390  2053013554658… electronics.… leno…
    208 2019-11-14 10:25:55 cart       100003423  2053013561579… electronics.… arma…
    209 2019-12-21 12:52:34 cart       100003423  2232732082063… electronics.… arma…
    210 2019-11-17 06:00:33 cart       100003441  2053013561579… electronics.… mich…
    211 2019-12-30 10:05:17 cart       100003441  2232732082063… electronics.… mich…
    212 2019-11-19 10:48:27 cart       100003454  2053013561579… electronics.… skag…
    213 2019-12-15 13:27:58 cart       100003454  2232732082063… electronics.… skag…
    214 2019-11-16 08:57:43 cart       100003477  2053013561579… electronics.… mich…
    215 2019-12-03 04:16:04 cart       100003477  2232732082063… electronics.… mich…
    216 2019-11-16 09:43:53 cart       100003482  2053013561579… electronics.… mich…
    217 2019-12-03 03:08:23 cart       100003482  2232732082063… electronics.… mich…
    218 2019-11-17 10:44:52 cart       100003493  2053013561579… electronics.… roya…
    219 2019-12-22 07:26:58 cart       100003493  2232732082063… electronics.… roya…
    220 2019-11-12 07:41:51 cart       100003496  2053013554415… electronics.… lg   
    221 2019-12-01 04:59:01 cart       100003496  2232732099754… appliances.p… lg   
    222 2019-11-17 04:34:00 cart       100003522  2053013561579… electronics.… foss…
    223 2019-12-16 11:10:32 cart       100003522  2232732082063… electronics.… foss…
    224 2019-11-17 06:58:24 cart       100003541  2053013561579… electronics.… skag…
    225 2019-12-16 11:11:43 cart       100003541  2232732082063… electronics.… skag…
    226 2019-11-12 18:58:12 cart       100003580  2053013563835… appliances.k… opti…
    227 2019-12-02 17:50:38 cart       100003580  2053013563970… appliances.k… opti…
    228 2019-11-16 16:52:14 cart       100003596  2053013561579… electronics.… loui…
    229 2019-12-16 08:22:30 cart       100003596  2232732082063… electronics.… loui…
    230 2019-11-15 08:58:05 cart       100003608  2116907524572… apparel.shoes bart…
    231 2019-12-29 09:51:37 cart       100003608  2053013554247… computers.co… bart…
    232 2019-11-16 12:18:54 cart       100003620  2116907524572… apparel.shoes bart…
    233 2019-12-13 05:55:37 cart       100003620  2053013553786… apparel.shoes bart…
    234 2019-11-12 15:03:01 cart       100003668  2053013558920… computers.no… asus 
    235 2019-12-01 17:54:09 cart       100003668  2053013554658… electronics.… asus 
    236 2019-11-16 15:01:11 cart       100003680  2053013561579… electronics.… oris 
    237 2019-12-20 18:18:22 cart       100003680  2232732082063… electronics.… oris 
    238 2019-11-12 18:57:07 cart       100003716  2053013554658… electronics.… beats
    239 2019-12-01 14:04:33 cart       100003716  2232732079706… sport.bicycle beats
    240 2019-11-17 16:43:03 cart       100003750  2053013560346… kids.carriage <NA> 
    241 2019-12-22 11:14:05 cart       100003750  2232732079009… kids.skates   <NA> 
    242 2019-11-15 11:59:50 cart       100003766  2070005009382… apparel.unde… mila…
    243 2019-12-19 07:20:00 cart       100003766  2053013555573… electronics.… mila…
    244 2019-11-16 01:56:08 cart       100003768  2070005009382… apparel.unde… mila…
    245 2019-12-08 09:57:10 cart       100003768  2053013555573… electronics.… mila…
    246 2019-11-11 20:05:04 cart       100003841  2053013563835… appliances.k… midea
    247 2019-12-01 06:23:56 cart       100003841  2232732091307… appliances.k… midea
    248 2019-11-29 16:44:00 cart       100003856  2116907524572… apparel.shoes garv…
    249 2019-12-06 05:48:52 cart       100003856  2053013554247… computers.co… garv…
    250 2019-11-16 05:39:41 cart       100003857  2116907524572… apparel.shoes garv…
    251 2019-12-13 11:01:12 cart       100003857  2053013554247… computers.co… garv…
    252 2019-11-14 18:25:46 cart       100003858  2116907524572… apparel.shoes garv…
    253 2019-12-13 11:02:55 cart       100003858  2053013554247… computers.co… garv…
    254 2019-11-15 22:17:21 cart       100003944  2053013565639… apparel.shoes resp…
    255 2019-12-30 09:09:33 cart       100003944  2232732116137… apparel.unde… resp…
    256 2019-11-15 03:37:01 cart       100004021  2053013563877… appliances.k… dsp  
    257 2019-12-04 09:18:31 cart       100004021  2232732102749… appliances.k… dsp  
    258 2019-11-16 15:10:50 cart       100004066  2053013565639… apparel.shoes baden
    259 2019-12-22 16:26:49 cart       100004066  2232732116137… apparel.unde… baden
    260 2019-11-15 02:53:01 cart       100004147  2053013556487… furniture.ki… skil…
    261 2019-12-04 05:03:21 cart       100004147  2232732097775… apparel.shoe… skil…
    262 2019-11-17 17:00:30 purchase   100004152  2053013565639… apparel.shoes baden
    263 2019-12-09 17:48:12 cart       100004152  2232732116137… apparel.unde… baden
    264 2019-11-16 09:29:25 purchase   100004153  2116907524572… apparel.shoes biom…
    265 2019-12-30 08:03:36 cart       100004153  2053013554247… computers.co… biom…
    266 2019-11-25 04:05:28 cart       100004158  2116907524572… apparel.shoes biom…
    267 2019-12-16 06:37:29 cart       100004158  2053013554247… computers.co… biom…
    268 2019-11-19 02:40:48 cart       100004161  2116907524572… apparel.shoes biom…
    269 2019-12-22 18:34:49 cart       100004161  2053013554247… computers.co… biom…
    270 2019-11-15 05:13:42 cart       100004162  2053013566176… appliances.i… tefal
    271 2019-12-16 10:12:55 cart       100004162  2053013557477… furniture.ba… tefal
    272 2019-11-16 21:47:28 cart       100004165  2053013565639… apparel.shoes baden
    273 2019-12-05 09:15:28 cart       100004165  2232732103764… apparel.shoes baden
    274 2019-11-16 05:44:39 cart       100004168  2053013565639… apparel.shoes baden
    275 2019-12-12 08:47:50 cart       100004168  2232732116137… apparel.unde… baden
    276 2019-11-16 08:18:19 cart       100004189  2053013555573… electronics.… texet
    277 2019-12-02 18:46:30 cart       100004189  2053013560530… electronics.… texet
    278 2019-11-12 14:52:52 cart       100004192  2053013565228… apparel.shoes resp…
    279 2019-12-01 15:41:52 cart       100004192  2053013557024… apparel.shoe… resp…
    280 2019-11-19 13:33:17 cart       100004194  2053013565228… apparel.shoes resp…
    281 2019-12-04 11:49:23 cart       100004194  2053013557024… apparel.shoe… resp…
    282 2019-11-18 02:05:32 cart       100004196  2171876313638… furniture.li… ikea 
    283 2019-12-16 13:17:58 cart       100004196  2053013554323… electronics.… ikea 
    284 2019-11-14 18:53:14 cart       100004208  2053013560346… kids.carriage wise…
    285 2019-12-04 17:09:38 cart       100004208  2053013551966… kids.carriage wise…
    286 2019-11-13 16:10:30 cart       100004229  2053013566176… appliances.i… tefal
    287 2019-12-07 02:52:02 cart       100004229  2053013557477… furniture.ba… tefal
    288 2019-12-01 12:29:49 cart       100004246  2232732110089… apparel.trou… <NA> 
    289 2019-12-04 19:42:41 cart       100004246  2232732110089… apparel.trou… nika 
    290 2019-11-12 14:41:28 cart       100004287  2053013559792… furniture.li… asm  
    291 2019-12-01 17:15:21 cart       100004287  2232732090980… furniture.li… asm  
    292 2019-11-15 08:53:29 cart       100004310  2053013555262… appliances.k… moul…
    293 2019-12-01 07:50:27 cart       100004310  2232732091391… appliances.k… moul…
    294 2019-11-12 09:07:07 cart       100004326  2053013552326… appliances.e… hube…
    295 2019-12-03 14:42:37 cart       100004326  2053013557452… electronics.… hube…
    296 2019-11-20 03:31:33 cart       100004345  2171876313638… furniture.li… ikea 
    297 2019-12-01 07:14:38 cart       100004345  2053013554323… electronics.… ikea 
    298 2019-11-21 11:40:13 cart       100004352  2147123662858… auto.accesso… rave…
    299 2019-12-01 05:59:27 cart       100004352  2053013566100… appliances.s… rave…
    300 2019-11-21 21:23:13 cart       100004367  2147123662858… auto.accesso… rave…
    # ℹ 39,089 more rows
    # ℹ 4 more variables: price <dbl>, user_id <chr>, user_session <chr>,
    #   event_date <date>

### If a product has two distinct brands / categories but one is NULL , replace NULL with the other brand / category

<details class="code-fold">
<summary>Code</summary>

``` r
merge_data <- events_data %>%
  filter(product_id %in% dup_ids) %>%
  arrange(product_id) %>%
  distinct(product_id, brand, category_id, category_code) %>%
  group_by(product_id) %>%
  mutate(
    brands = list(brand),
    cat_ids = list(category_id),
    cat_codes = list(category_code)
  ) %>%
  rowwise() %>%
  mutate(
    brands = unique(brands) %>% na.omit() %>% list(),
    cat_ids = unique(brands) %>% na.omit() %>% list(),
    cat_codes = unique(cat_codes) %>% na.omit() %>% list(),
    brand_new = ifelse(length(brands) == 1 & length(brands[1]) > 0 & is.na(brand) , brands[1] , brand),
    category_id_new = ifelse(length(cat_ids) == 1 & length(cat_ids[1]) > 0 & is.na(category_id) , cat_ids[1], category_id),
    category_code_new = ifelse(length(cat_codes) == 1 & length(cat_codes[1])>0  & is.na(category_code), cat_codes[1] , category_code)
  ) %>%
  ungroup()

events_data <- left_join(events_data, merge_data) %>%
  mutate(
    brand = brand_new,
    category_id = category_id_new,
    category_code = category_code_new
  ) %>%
  select(-ends_with("new"), -brands , -cat_ids , -cat_codes)

events_data %>%
  filter(product_id %in% dup_ids) %>%
  arrange(product_id) %>%
  distinct(product_id, category_code,category_id,brand ,.keep_all = T) %>% print(n=300)
```

</details>

    # A tibble: 39,163 × 10
        event_time          event_type product_id category_id    category_code brand
        <dttm>              <chr>      <chr>      <chr>          <chr>         <chr>
      1 2019-11-17 11:03:35 cart       100000153  2053013565782… apparel.shoes resp…
      2 2019-12-03 17:14:57 cart       100000153  2232732098706… apparel.shoes resp…
      3 2019-11-14 18:01:29 cart       100000154  2053013565782… apparel.shoes resp…
      4 2019-12-04 13:50:49 cart       100000154  2053013556521… apparel.shoes resp…
      5 2019-11-14 20:04:11 cart       100000179  2053013560623… auto.accesso… park…
      6 2019-12-22 19:02:56 cart       100000179  2232732109494… accessories.… park…
      7 2019-11-16 11:11:03 cart       100000196  2070005009382… apparel.unde… mila…
      8 2019-12-22 17:19:22 cart       100000196  2053013555573… electronics.… mila…
      9 2019-11-15 09:03:23 cart       100000208  2070005009382… apparel.unde… mila…
     10 2019-12-22 16:17:00 cart       100000208  2053013555573… electronics.… mila…
     11 2019-11-16 04:42:47 cart       100000210  2053013553970… auto.accesso… jvc  
     12 2019-12-02 13:43:37 cart       100000210  2053013554415… electronics.… jvc  
     13 2019-11-14 06:25:28 cart       100000244  2053013553945… electronics.… graf…
     14 2019-12-01 19:10:34 cart       100000244  2232732082390… electronics.… graf…
     15 2019-11-11 09:14:57 cart       100000263  2053013553945… electronics.… jvc  
     16 2019-12-02 16:50:12 cart       100000263  2232732082390… electronics.… jvc  
     17 2019-11-16 05:58:14 cart       100000273  2053013560623… auto.accesso… park…
     18 2019-12-29 21:13:39 cart       100000273  2232732109494… accessories.… park…
     19 2019-11-13 06:07:07 cart       100000324  2053013558945… accessories.… cont…
     20 2019-12-07 14:14:07 cart       100000324  2053013551907… sport.ski     cont…
     21 2019-11-21 13:24:44 cart       100000368  2053013552326… appliances.e… deli…
     22 2019-12-20 14:15:07 cart       100000368  2053013557452… electronics.… deli…
     23 2019-11-17 05:19:23 cart       100000783  2053013561780… furniture.be… adam…
     24 2019-12-23 07:44:16 cart       100000783  2053013553375… computers.no… adam…
     25 2019-11-11 18:09:48 cart       100000811  2053013552293… appliances.e… resa…
     26 2019-12-28 14:56:14 cart       100000811  2232732091961… appliances.e… resa…
     27 2019-11-22 21:03:38 cart       100000912  2070005009382… apparel.unde… mila…
     28 2019-12-07 18:31:51 cart       100000912  2053013555573… electronics.… mila…
     29 2019-11-18 14:11:03 cart       100000923  2053013555262… appliances.k… deli…
     30 2019-12-05 09:59:02 cart       100000923  2232732091391… appliances.k… deli…
     31 2019-11-13 15:13:29 cart       100001169  2053013561579… electronics.… skag…
     32 2019-12-06 09:53:31 cart       100001169  2232732082063… electronics.… skag…
     33 2019-11-14 23:08:08 cart       100001222  2053013561579… electronics.… zepp…
     34 2019-12-17 12:31:53 cart       100001222  2232732082063… electronics.… zepp…
     35 2019-11-13 10:37:16 cart       100001331  2053013561579… electronics.… arma…
     36 2019-12-13 15:18:09 cart       100001331  2232732082063… electronics.… arma…
     37 2019-11-17 15:11:16 cart       100001368  2053013561579… electronics.… mich…
     38 2019-12-11 18:57:21 cart       100001368  2232732082063… electronics.… mich…
     39 2019-11-26 03:25:21 cart       100001372  2053013561579… electronics.… mich…
     40 2019-12-19 10:24:21 cart       100001372  2232732082063… electronics.… mich…
     41 2019-11-23 19:08:46 cart       100001428  2053013561780… furniture.be… dorm…
     42 2019-12-26 15:38:28 cart       100001428  2053013553375… computers.no… dorm…
     43 2019-11-16 16:16:05 cart       100001429  2053013561579… electronics.… dkny 
     44 2019-12-30 11:27:23 cart       100001429  2232732082063… electronics.… dkny 
     45 2019-11-17 17:56:35 cart       100001505  2053013561579… electronics.… foss…
     46 2019-12-24 09:32:23 cart       100001505  2232732082063… electronics.… foss…
     47 2019-11-14 13:59:33 cart       100001529  2053013552293… appliances.e… rovus
     48 2019-12-08 05:24:11 cart       100001529  2232732091961… appliances.e… rovus
     49 2019-11-14 05:19:44 cart       100001549  2086471240800… apparel.trou… puma 
     50 2019-12-02 14:52:35 cart       100001549  2053013558978… sport.bicycle puma 
     51 2019-11-16 07:00:42 cart       100001564  2146660887002… apparel.dress haya…
     52 2019-12-03 12:16:33 cart       100001564  2053013557645… electronics.… haya…
     53 2019-11-16 14:02:30 cart       100001577  2053013561579… electronics.… skag…
     54 2019-12-15 07:47:25 cart       100001577  2232732082063… electronics.… skag…
     55 2019-11-14 09:19:19 cart       100001674  2053013561579… electronics.… skag…
     56 2019-12-01 06:03:04 cart       100001674  2232732082063… electronics.… skag…
     57 2019-11-30 13:22:24 cart       100001718  2053013561420… accessories.… amen 
     58 2019-12-04 14:24:37 cart       100001718  2232732089369… accessories.… amen 
     59 2019-11-27 08:15:21 cart       100001796  2053013561420… accessories.… amen 
     60 2019-12-12 13:45:54 cart       100001796  2232732110643… country_yard… amen 
     61 2019-11-27 08:14:40 cart       100001800  2053013561420… accessories.… amen 
     62 2019-12-26 19:18:50 cart       100001800  2232732110643… country_yard… amen 
     63 2019-11-09 13:53:58 cart       100001820  2053013559792… furniture.li… pins…
     64 2019-12-18 09:35:25 cart       100001820  2232732090980… furniture.li… pins…
     65 2019-11-27 17:43:31 cart       100001852  2053013559792… furniture.li… pins…
     66 2019-12-08 07:44:53 cart       100001852  2232732090980… furniture.li… pins…
     67 2019-11-15 10:17:32 cart       100001925  2053013561420… accessories.… amen 
     68 2019-12-23 06:17:42 cart       100001925  2232732110643… country_yard… amen 
     69 2019-11-17 14:02:54 cart       100001927  2053013561420… accessories.… amen 
     70 2019-12-02 03:02:02 cart       100001927  2232732110643… country_yard… amen 
     71 2019-11-18 02:20:44 cart       100001983  2053013561847… furniture.be… dorm…
     72 2019-12-26 06:00:51 cart       100001983  2232732089302… furniture.be… dorm…
     73 2019-11-12 12:40:37 cart       100001984  2053013557670… electronics.… cara…
     74 2019-12-02 10:26:59 cart       100001984  2232732082734… electronics.… cara…
     75 2019-11-10 08:09:52 cart       100001985  2053013557670… electronics.… cara…
     76 2019-12-20 08:49:53 cart       100001985  2232732082734… electronics.… cara…
     77 2019-11-16 03:30:38 cart       100001986  2053013561847… furniture.be… dorm…
     78 2019-12-19 13:34:45 cart       100001986  2232732089302… furniture.be… dorm…
     79 2019-11-15 08:58:52 cart       100001987  2053013557670… electronics.… cara…
     80 2019-12-20 12:45:01 cart       100001987  2232732082734… electronics.… cara…
     81 2019-11-15 00:42:31 cart       100001988  2053013557670… electronics.… adag…
     82 2019-12-01 17:31:47 cart       100001988  2232732107883… electronics.… adag…
     83 2019-11-15 09:41:26 cart       100001989  2053013561847… furniture.be… dorm…
     84 2019-12-07 13:27:22 cart       100001989  2232732089302… furniture.be… dorm…
     85 2019-11-29 10:07:15 cart       100001990  2053013557670… electronics.… devi…
     86 2019-12-15 17:34:07 cart       100001990  2232732107883… electronics.… devi…
     87 2019-11-17 19:54:17 cart       100001991  2053013561847… furniture.be… dorm…
     88 2019-12-17 15:22:22 cart       100001991  2232732089302… furniture.be… dorm…
     89 2019-11-14 19:04:10 cart       100001992  2053013557670… electronics.… joker
     90 2019-12-16 13:44:24 cart       100001992  2232732082734… electronics.… joker
     91 2019-11-15 18:54:15 cart       100001993  2053013557670… electronics.… adag…
     92 2019-12-06 05:22:02 cart       100001993  2232732107883… electronics.… adag…
     93 2019-11-25 10:37:00 cart       100001999  2053013566176… appliances.i… rovus
     94 2019-12-08 11:08:10 cart       100001999  2053013557477… furniture.ba… rovus
     95 2019-11-15 18:09:58 cart       100002006  2053013557192… furniture.be… orto…
     96 2019-12-01 18:55:53 cart       100002006  2232732061804… furniture.be… orto…
     97 2019-11-14 18:01:41 cart       100002007  2053013557192… furniture.be… orto…
     98 2019-12-18 06:19:20 cart       100002007  2232732061804… furniture.be… orto…
     99 2019-11-15 06:19:24 cart       100002008  2053013557192… furniture.be… orto…
    100 2019-12-02 01:35:29 cart       100002008  2232732061804… furniture.be… orto…
    101 2019-11-16 12:15:12 cart       100002024  2053013565941… appliances.p… well…
    102 2019-12-08 04:54:31 cart       100002024  2232732114611… construction… well…
    103 2019-11-16 17:17:14 cart       100002031  2053013565941… appliances.p… well…
    104 2019-12-09 16:23:48 cart       100002031  2232732114611… construction… well…
    105 2019-11-16 12:12:06 cart       100002034  2053013565941… appliances.p… well…
    106 2019-12-13 08:41:55 cart       100002034  2232732114611… construction… well…
    107 2019-11-13 15:46:07 cart       100002043  2053013558920… computers.no… acer 
    108 2019-12-02 06:07:09 cart       100002043  2053013554658… electronics.… acer 
    109 2019-11-12 09:41:46 cart       100002087  2053013558920… computers.no… leno…
    110 2019-12-01 08:37:17 cart       100002087  2053013554658… electronics.… leno…
    111 2019-11-13 04:40:41 cart       100002113  2053013558920… computers.no… leno…
    112 2019-12-01 06:34:28 cart       100002113  2053013554658… electronics.… leno…
    113 2019-11-10 10:04:01 cart       100002174  2053013558920… computers.no… leno…
    114 2019-12-12 08:37:10 cart       100002174  2053013554658… electronics.… leno…
    115 2019-11-10 06:30:02 cart       100002201  2053013558920… computers.no… leno…
    116 2019-12-19 03:58:42 cart       100002201  2053013554658… electronics.… leno…
    117 2019-11-13 08:44:37 cart       100002263  2053013558920… computers.no… leno…
    118 2019-12-01 07:11:10 cart       100002263  2053013554658… electronics.… leno…
    119 2019-11-16 09:12:40 cart       100002283  2146660887002… apparel.dress omero
    120 2019-12-14 09:33:31 cart       100002283  2053013557645… electronics.… omero
    121 2019-11-12 10:01:33 cart       100002292  2053013554658… electronics.… xiao…
    122 2019-12-01 10:06:51 cart       100002292  2232732079706… sport.bicycle xiao…
    123 2019-11-16 05:49:33 cart       100002304  2090228413380… apparel.unde… visa…
    124 2019-12-04 05:37:34 cart       100002304  2053013561302… apparel.shoe… visa…
    125 2019-11-30 19:45:39 cart       100002308  2090228413380… apparel.unde… visa…
    126 2019-12-03 07:41:00 cart       100002308  2053013561302… apparel.shoe… visa…
    127 2019-11-18 15:00:27 cart       100002310  2053013559868… computers.de… aero…
    128 2019-12-03 16:29:50 cart       100002310  2232732086257… electronics.… aero…
    129 2019-11-16 16:10:19 cart       100002365  2090228413380… apparel.unde… visa…
    130 2019-12-08 03:28:31 cart       100002365  2053013561302… apparel.shoe… visa…
    131 2019-11-14 06:30:14 cart       100002367  2053013554121… computers.co… amd  
    132 2019-12-04 08:18:46 cart       100002367  2232732103294… apparel.shoe… amd  
    133 2019-11-12 05:37:18 cart       100002371  2053013558920… computers.no… leno…
    134 2019-12-03 06:04:25 cart       100002371  2053013554658… electronics.… leno…
    135 2019-11-16 17:14:48 cart       100002385  2090228413380… apparel.unde… visa…
    136 2019-12-02 12:34:29 cart       100002385  2053013561302… apparel.shoe… visa…
    137 2019-11-17 08:43:43 cart       100002386  2090228413380… apparel.unde… visa…
    138 2019-12-15 08:24:53 cart       100002386  2053013561302… apparel.shoe… visa…
    139 2019-11-17 04:04:07 cart       100002409  2090228413380… apparel.unde… visa…
    140 2019-12-21 17:26:33 cart       100002409  2053013561302… apparel.shoe… visa…
    141 2019-11-13 03:02:45 cart       100002460  2053013553945… electronics.… <NA> 
    142 2019-12-01 08:57:29 cart       100002460  2232732082390… electronics.… <NA> 
    143 2019-11-15 19:19:37 cart       100002461  2053013553945… electronics.… <NA> 
    144 2019-12-01 19:02:19 cart       100002461  2232732082390… electronics.… <NA> 
    145 2019-11-12 17:42:02 cart       100002462  2053013553945… electronics.… <NA> 
    146 2019-12-01 08:36:47 cart       100002462  2232732082390… electronics.… <NA> 
    147 2019-12-03 05:06:45 cart       100002527  2232732084546… furniture.li… lego 
    148 2019-12-02 05:54:57 cart       100002530  2232732084546… furniture.li… lego 
    149 2019-11-16 12:06:57 cart       100002560  2053013560807… auto.accesso… star…
    150 2019-12-02 22:18:25 cart       100002560  2232732103546… apparel.shoes star…
    151 2019-11-15 05:08:18 cart       100002716  2053013556311… construction… resa…
    152 2019-12-15 10:24:51 cart       100002716  2053013563743… appliances.k… resa…
    153 2019-11-14 18:59:20 cart       100002720  2053013566209… accessories.… tony…
    154 2019-12-02 16:04:21 cart       100002720  2232732082935… accessories.… tony…
    155 2019-11-21 16:03:24 cart       100002723  2053013566209… accessories.… tony…
    156 2019-12-15 19:34:18 cart       100002723  2232732082935… accessories.… tony…
    157 2019-11-17 10:45:14 cart       100002726  2053013556311… construction… resa…
    158 2019-12-10 07:11:15 cart       100002726  2053013563743… appliances.k… resa…
    159 2019-11-15 17:42:10 cart       100002728  2053013566209… accessories.… tony…
    160 2019-12-08 21:12:26 cart       100002728  2232732082935… accessories.… tony…
    161 2019-11-30 06:53:07 cart       100002729  2053013556311… construction… resa…
    162 2019-12-15 10:28:42 cart       100002729  2053013563743… appliances.k… resa…
    163 2019-11-27 18:20:19 cart       100002730  2053013556311… construction… resa…
    164 2019-12-20 06:10:39 cart       100002730  2053013563743… appliances.k… resa…
    165 2019-11-17 17:58:58 cart       100002747  2053013566209… accessories.… tony…
    166 2019-12-01 17:30:46 cart       100002747  2232732082935… accessories.… tony…
    167 2019-11-16 08:16:14 cart       100002842  2053013556311… construction… resa…
    168 2019-12-29 06:30:20 cart       100002842  2053013563743… appliances.k… resa…
    169 2019-11-25 04:25:42 cart       100002867  2127425436764… construction… ener…
    170 2019-12-09 11:41:00 cart       100002867  2232732095074… construction… ener…
    171 2019-11-17 17:43:58 cart       100002972  2116907524488… apparel.shoe… garv…
    172 2019-12-24 16:37:39 cart       100002972  2053013558643… apparel.shoe… garv…
    173 2019-11-13 17:16:27 cart       100003280  2055156924407… accessories.… luck…
    174 2019-12-23 16:28:43 cart       100003280  2053013560623… auto.accesso… luck…
    175 2019-11-12 05:00:38 cart       100003313  2053013552888… computers.pe… cany…
    176 2019-12-12 03:34:30 cart       100003313  2232732104066… furniture.ki… cany…
    177 2019-11-15 15:10:15 cart       100003314  2053013552955… computers.pe… canon
    178 2019-12-02 19:54:33 cart       100003314  2232732071460… kids.toys     canon
    179 2019-12-10 12:59:36 cart       100003314  2232732071460… kids.toys     pant…
    180 2019-11-14 21:11:27 cart       100003319  2053013552888… computers.pe… cany…
    181 2019-12-03 09:57:19 cart       100003319  2232732104066… furniture.ki… cany…
    182 2019-11-30 15:21:38 cart       100003320  2053013552888… computers.pe… leno…
    183 2019-12-02 17:59:10 cart       100003320  2232732104066… furniture.ki… leno…
    184 2019-11-27 17:34:13 cart       100003321  2053013552888… computers.pe… coug…
    185 2019-12-03 10:53:32 cart       100003321  2232732104066… furniture.ki… coug…
    186 2019-11-30 18:39:46 cart       100003322  2053013552888… computers.pe… leno…
    187 2019-12-12 16:06:20 cart       100003322  2232732104066… furniture.ki… leno…
    188 2019-11-12 19:21:08 cart       100003338  2053013558920… computers.no… asus 
    189 2019-12-01 04:46:49 cart       100003338  2053013554658… electronics.… asus 
    190 2019-11-15 21:36:05 cart       100003343  2053013552293… appliances.e… sibr…
    191 2019-12-11 11:26:31 cart       100003343  2232732091961… appliances.e… sibr…
    192 2019-11-15 13:52:35 cart       100003348  2053013552293… appliances.e… sibr…
    193 2019-12-05 19:25:51 cart       100003348  2232732091961… appliances.e… sibr…
    194 2019-11-14 12:55:25 cart       100003358  2053013558920… computers.no… leno…
    195 2019-12-07 07:14:44 cart       100003358  2053013554658… electronics.… leno…
    196 2019-11-12 16:02:09 cart       100003363  2127425436764… construction… ener…
    197 2019-12-27 11:26:36 cart       100003363  2232732095074… construction… ener…
    198 2019-11-14 20:22:22 cart       100003365  2053013561579… electronics.… mich…
    199 2019-12-04 07:57:05 cart       100003365  2232732082063… electronics.… mich…
    200 2019-11-17 03:20:56 cart       100003376  2053013566209… accessories.… tony…
    201 2019-12-03 14:53:14 cart       100003376  2232732082935… accessories.… tony…
    202 2019-11-14 17:35:03 cart       100003378  2053013566209… accessories.… tony…
    203 2019-12-02 08:43:13 cart       100003378  2232732082935… accessories.… tony…
    204 2019-11-14 09:28:55 cart       100003390  2053013558920… computers.no… leno…
    205 2019-12-25 09:06:27 cart       100003390  2053013554658… electronics.… leno…
    206 2019-11-14 10:25:55 cart       100003423  2053013561579… electronics.… arma…
    207 2019-12-21 12:52:34 cart       100003423  2232732082063… electronics.… arma…
    208 2019-11-17 06:00:33 cart       100003441  2053013561579… electronics.… mich…
    209 2019-12-30 10:05:17 cart       100003441  2232732082063… electronics.… mich…
    210 2019-11-19 10:48:27 cart       100003454  2053013561579… electronics.… skag…
    211 2019-12-15 13:27:58 cart       100003454  2232732082063… electronics.… skag…
    212 2019-11-16 08:57:43 cart       100003477  2053013561579… electronics.… mich…
    213 2019-12-03 04:16:04 cart       100003477  2232732082063… electronics.… mich…
    214 2019-11-16 09:43:53 cart       100003482  2053013561579… electronics.… mich…
    215 2019-12-03 03:08:23 cart       100003482  2232732082063… electronics.… mich…
    216 2019-11-17 10:44:52 cart       100003493  2053013561579… electronics.… roya…
    217 2019-12-22 07:26:58 cart       100003493  2232732082063… electronics.… roya…
    218 2019-11-12 07:41:51 cart       100003496  2053013554415… electronics.… lg   
    219 2019-12-01 04:59:01 cart       100003496  2232732099754… appliances.p… lg   
    220 2019-11-17 04:34:00 cart       100003522  2053013561579… electronics.… foss…
    221 2019-12-16 11:10:32 cart       100003522  2232732082063… electronics.… foss…
    222 2019-11-17 06:58:24 cart       100003541  2053013561579… electronics.… skag…
    223 2019-12-16 11:11:43 cart       100003541  2232732082063… electronics.… skag…
    224 2019-11-12 18:58:12 cart       100003580  2053013563835… appliances.k… opti…
    225 2019-12-02 17:50:38 cart       100003580  2053013563970… appliances.k… opti…
    226 2019-11-16 16:52:14 cart       100003596  2053013561579… electronics.… loui…
    227 2019-12-16 08:22:30 cart       100003596  2232732082063… electronics.… loui…
    228 2019-11-15 08:58:05 cart       100003608  2116907524572… apparel.shoes bart…
    229 2019-12-29 09:51:37 cart       100003608  2053013554247… computers.co… bart…
    230 2019-11-16 12:18:54 cart       100003620  2116907524572… apparel.shoes bart…
    231 2019-12-13 05:55:37 cart       100003620  2053013553786… apparel.shoes bart…
    232 2019-11-12 15:03:01 cart       100003668  2053013558920… computers.no… asus 
    233 2019-12-01 17:54:09 cart       100003668  2053013554658… electronics.… asus 
    234 2019-11-16 15:01:11 cart       100003680  2053013561579… electronics.… oris 
    235 2019-12-20 18:18:22 cart       100003680  2232732082063… electronics.… oris 
    236 2019-11-12 18:57:07 cart       100003716  2053013554658… electronics.… beats
    237 2019-12-01 14:04:33 cart       100003716  2232732079706… sport.bicycle beats
    238 2019-11-17 16:43:03 cart       100003750  2053013560346… kids.carriage <NA> 
    239 2019-12-22 11:14:05 cart       100003750  2232732079009… kids.skates   <NA> 
    240 2019-11-15 11:59:50 cart       100003766  2070005009382… apparel.unde… mila…
    241 2019-12-19 07:20:00 cart       100003766  2053013555573… electronics.… mila…
    242 2019-11-16 01:56:08 cart       100003768  2070005009382… apparel.unde… mila…
    243 2019-12-08 09:57:10 cart       100003768  2053013555573… electronics.… mila…
    244 2019-11-11 20:05:04 cart       100003841  2053013563835… appliances.k… midea
    245 2019-12-01 06:23:56 cart       100003841  2232732091307… appliances.k… midea
    246 2019-11-29 16:44:00 cart       100003856  2116907524572… apparel.shoes garv…
    247 2019-12-06 05:48:52 cart       100003856  2053013554247… computers.co… garv…
    248 2019-11-16 05:39:41 cart       100003857  2116907524572… apparel.shoes garv…
    249 2019-12-13 11:01:12 cart       100003857  2053013554247… computers.co… garv…
    250 2019-11-14 18:25:46 cart       100003858  2116907524572… apparel.shoes garv…
    251 2019-12-13 11:02:55 cart       100003858  2053013554247… computers.co… garv…
    252 2019-11-15 22:17:21 cart       100003944  2053013565639… apparel.shoes resp…
    253 2019-12-30 09:09:33 cart       100003944  2232732116137… apparel.unde… resp…
    254 2019-11-15 03:37:01 cart       100004021  2053013563877… appliances.k… dsp  
    255 2019-12-04 09:18:31 cart       100004021  2232732102749… appliances.k… dsp  
    256 2019-11-16 15:10:50 cart       100004066  2053013565639… apparel.shoes baden
    257 2019-12-22 16:26:49 cart       100004066  2232732116137… apparel.unde… baden
    258 2019-11-15 02:53:01 cart       100004147  2053013556487… furniture.ki… skil…
    259 2019-12-04 05:03:21 cart       100004147  2232732097775… apparel.shoe… skil…
    260 2019-11-17 17:00:30 purchase   100004152  2053013565639… apparel.shoes baden
    261 2019-12-09 17:48:12 cart       100004152  2232732116137… apparel.unde… baden
    262 2019-11-16 09:29:25 purchase   100004153  2116907524572… apparel.shoes biom…
    263 2019-12-30 08:03:36 cart       100004153  2053013554247… computers.co… biom…
    264 2019-11-25 04:05:28 cart       100004158  2116907524572… apparel.shoes biom…
    265 2019-12-16 06:37:29 cart       100004158  2053013554247… computers.co… biom…
    266 2019-11-19 02:40:48 cart       100004161  2116907524572… apparel.shoes biom…
    267 2019-12-22 18:34:49 cart       100004161  2053013554247… computers.co… biom…
    268 2019-11-15 05:13:42 cart       100004162  2053013566176… appliances.i… tefal
    269 2019-12-16 10:12:55 cart       100004162  2053013557477… furniture.ba… tefal
    270 2019-11-16 21:47:28 cart       100004165  2053013565639… apparel.shoes baden
    271 2019-12-05 09:15:28 cart       100004165  2232732103764… apparel.shoes baden
    272 2019-11-16 05:44:39 cart       100004168  2053013565639… apparel.shoes baden
    273 2019-12-12 08:47:50 cart       100004168  2232732116137… apparel.unde… baden
    274 2019-11-16 08:18:19 cart       100004189  2053013555573… electronics.… texet
    275 2019-12-02 18:46:30 cart       100004189  2053013560530… electronics.… texet
    276 2019-11-12 14:52:52 cart       100004192  2053013565228… apparel.shoes resp…
    277 2019-12-01 15:41:52 cart       100004192  2053013557024… apparel.shoe… resp…
    278 2019-11-19 13:33:17 cart       100004194  2053013565228… apparel.shoes resp…
    279 2019-12-04 11:49:23 cart       100004194  2053013557024… apparel.shoe… resp…
    280 2019-11-18 02:05:32 cart       100004196  2171876313638… furniture.li… ikea 
    281 2019-12-16 13:17:58 cart       100004196  2053013554323… electronics.… ikea 
    282 2019-11-14 18:53:14 cart       100004208  2053013560346… kids.carriage wise…
    283 2019-12-04 17:09:38 cart       100004208  2053013551966… kids.carriage wise…
    284 2019-11-13 16:10:30 cart       100004229  2053013566176… appliances.i… tefal
    285 2019-12-07 02:52:02 cart       100004229  2053013557477… furniture.ba… tefal
    286 2019-12-01 12:29:49 cart       100004246  2232732110089… apparel.trou… nika 
    287 2019-11-12 14:41:28 cart       100004287  2053013559792… furniture.li… asm  
    288 2019-12-01 17:15:21 cart       100004287  2232732090980… furniture.li… asm  
    289 2019-11-15 08:53:29 cart       100004310  2053013555262… appliances.k… moul…
    290 2019-12-01 07:50:27 cart       100004310  2232732091391… appliances.k… moul…
    291 2019-11-12 09:07:07 cart       100004326  2053013552326… appliances.e… hube…
    292 2019-12-03 14:42:37 cart       100004326  2053013557452… electronics.… hube…
    293 2019-11-20 03:31:33 cart       100004345  2171876313638… furniture.li… ikea 
    294 2019-12-01 07:14:38 cart       100004345  2053013554323… electronics.… ikea 
    295 2019-11-21 11:40:13 cart       100004352  2147123662858… auto.accesso… rave…
    296 2019-12-01 05:59:27 cart       100004352  2053013566100… appliances.s… rave…
    297 2019-11-21 21:23:13 cart       100004367  2147123662858… auto.accesso… rave…
    298 2019-12-01 16:33:18 cart       100004367  2053013566100… appliances.s… rave…
    299 2019-11-30 11:02:14 cart       100004377  2147123662858… auto.accesso… rave…
    300 2019-12-06 11:33:56 cart       100004377  2053013566100… appliances.s… rave…
    # ℹ 38,863 more rows
    # ℹ 4 more variables: price <dbl>, user_id <chr>, user_session <chr>,
    #   event_date <date>

### Check Top Brands and Categories

<details class="code-fold">
<summary>Code</summary>

``` r
events_data %>%
  janitor::tabyl(brand) %>%
  as_tibble()%>%
  arrange(desc(n))%>%
  slice_head(n=20) %>%
  kable(caption = "Top Brands")
```

</details>

| brand   |       n |   percent | valid_percent |
|:--------|--------:|----------:|--------------:|
| samsung | 1731844 | 0.2489306 |     0.3075536 |
| NA      | 1326103 | 0.1906105 |            NA |
| apple   | 1306775 | 0.1878323 |     0.2320667 |
| xiaomi  |  609927 | 0.0876693 |     0.1083153 |
| huawei  |  255420 | 0.0367134 |     0.0453594 |
| oppo    |  127180 | 0.0182805 |     0.0225856 |
| lg      |  113219 | 0.0162738 |     0.0201063 |
| artel   |   69756 | 0.0100265 |     0.0123878 |
| bosch   |   48179 | 0.0069251 |     0.0085560 |
| lenovo  |   46460 | 0.0066780 |     0.0082507 |
| indesit |   42358 | 0.0060884 |     0.0075222 |
| acer    |   38380 | 0.0055166 |     0.0068158 |
| haier   |   34015 | 0.0048892 |     0.0060406 |
| beko    |   32954 | 0.0047367 |     0.0058522 |
| midea   |   32432 | 0.0046617 |     0.0057595 |
| casio   |   28305 | 0.0040685 |     0.0050266 |
| hp      |   28276 | 0.0040643 |     0.0050215 |
| tefal   |   27065 | 0.0038903 |     0.0048064 |
| sony    |   27004 | 0.0038815 |     0.0047956 |
| vitek   |   26061 | 0.0037459 |     0.0046281 |

Top Brands

<details class="code-fold">
<summary>Code</summary>

``` r
events_data %>%
  janitor::tabyl(category_code) %>%
  as_tibble()%>%
  arrange(desc(n)) %>%
  slice_head(n=20) %>%
  kable(caption = "Top Category Codes")
```

</details>

| category_code                     |       n |   percent | valid_percent |
|:----------------------------------|--------:|----------:|--------------:|
| construction.tools.light          | 1791115 | 0.2574501 |     0.3087595 |
| electronics.smartphone            | 1490413 | 0.2142280 |     0.2569233 |
| NA                                | 1156131 | 0.1661792 |            NA |
| electronics.audio.headphone       |  222754 | 0.0320181 |     0.0383992 |
| sport.bicycle                     |  219328 | 0.0315256 |     0.0378086 |
| electronics.clocks                |  200052 | 0.0287549 |     0.0344858 |
| appliances.kitchen.refrigerators  |  177944 | 0.0255772 |     0.0306747 |
| appliances.personal.massager      |  167144 | 0.0240248 |     0.0288129 |
| appliances.kitchen.washer         |  165089 | 0.0237295 |     0.0284587 |
| appliances.environment.vacuum     |  155555 | 0.0223591 |     0.0268152 |
| electronics.video.tv              |  142194 | 0.0204386 |     0.0245120 |
| computers.notebook                |   62652 | 0.0090054 |     0.0108002 |
| apparel.shoes.slipons             |   43535 | 0.0062576 |     0.0075047 |
| apparel.shoes                     |   36241 | 0.0052092 |     0.0062474 |
| appliances.kitchen.blender        |   36112 | 0.0051906 |     0.0062251 |
| appliances.kitchen.oven           |   33536 | 0.0048204 |     0.0057811 |
| electronics.tablet                |   31743 | 0.0045627 |     0.0054720 |
| appliances.environment.air_heater |   29713 | 0.0042709 |     0.0051220 |
| furniture.bedroom.blanket         |   28883 | 0.0041516 |     0.0049790 |
| electronics.audio.subwoofer       |   27410 | 0.0039398 |     0.0047250 |

Top Category Codes

<details class="code-fold">
<summary>Code</summary>

``` r
events_data %>%
  janitor::tabyl(category_id) %>%
  as_tibble()%>%
  arrange(desc(n))%>%
  slice_head(n=20) %>%
  kable(caption = "Top Category Ids")
```

</details>

| category_id         |       n |   percent | valid_percent |
|:--------------------|--------:|----------:|--------------:|
| 2232732093077520640 | 1780665 | 0.2559480 |     0.3069581 |
| 2053013555631882496 | 1490390 | 0.2142247 |     0.2569193 |
| NA                  | 1156131 | 0.1661792 |            NA |
| 2053013554658804224 |  222754 | 0.0320181 |     0.0383992 |
| 2232732079706079232 |  217453 | 0.0312561 |     0.0374854 |
| 2232732099754852864 |  154057 | 0.0221437 |     0.0265570 |
| 2053013554415534336 |  141771 | 0.0203778 |     0.0244390 |
| 2053013563810776064 |   87950 | 0.0126417 |     0.0151612 |
| 2232732103101907456 |   79205 | 0.0113847 |     0.0136537 |
| 2232732092297380096 |   76076 | 0.0109350 |     0.0131143 |
| 2053013565983425536 |   75975 | 0.0109204 |     0.0130969 |
| 2232732101063475712 |   74056 | 0.0106446 |     0.0127661 |
| 2053013563835941888 |   73084 | 0.0105049 |     0.0125985 |
| 2053013553341792768 |   64209 | 0.0092292 |     0.0110686 |
| 2053013558920217344 |   58954 | 0.0084739 |     0.0101627 |
| 2053013563911439360 |   40830 | 0.0058688 |     0.0070384 |
| 2232732101407408640 |   37656 | 0.0054126 |     0.0064913 |
| 2232732091718566144 |   37535 | 0.0053952 |     0.0064704 |
| 2232732082063278080 |   25520 | 0.0036682 |     0.0043992 |
| 2172371436436455680 |   23799 | 0.0034208 |     0.0041026 |

Top Category Ids

For top brands , find their top categories

<details class="code-fold">
<summary>Code</summary>

``` r
top_brands <- events_data %>%
  count(brand) %>%
  arrange(desc(n)) %>%
  select(brand) 

brand_cat_data <- events_data %>%
  count(brand , category_code) %>%
  arrange(desc(n))%>%
  left_join(x=top_brands)%>%
  group_by(brand) %>%
  left_join(x=top_brands)

events_data %>%
  count(brand , category_code) %>%
  arrange(desc(n))%>%
  left_join(x=top_brands)%>%
  group_by(brand) %>%
  slice_max(order_by = n , n = 3) %>%
  left_join(x=top_brands) %>%
  print(n=100)
```

</details>

    # A tibble: 2,538 × 3
        brand    category_code                             n
        <chr>    <chr>                                 <int>
      1 samsung  construction.tools.light             727041
      2 samsung  electronics.smartphone               619504
      3 samsung  appliances.personal.massager          60790
      4 <NA>     <NA>                                1156131
      5 <NA>     appliances.kitchen.refrigerators      27699
      6 <NA>     electronics.clocks                    11022
      7 apple    construction.tools.light             521023
      8 apple    electronics.smartphone               468818
      9 apple    sport.bicycle                        116251
     10 xiaomi   construction.tools.light             282132
     11 xiaomi   electronics.smartphone               226867
     12 xiaomi   sport.bicycle                         41652
     13 huawei   construction.tools.light             147424
     14 huawei   electronics.smartphone                87854
     15 huawei   sport.bicycle                          4553
     16 oppo     construction.tools.light              68326
     17 oppo     electronics.smartphone                58714
     18 oppo     appliances.kitchen.refrigerators        140
     19 lg       appliances.kitchen.washer             37316
     20 lg       appliances.personal.massager          24062
     21 lg       electronics.video.tv                  19190
     22 artel    appliances.personal.massager          25513
     23 artel    electronics.video.tv                  23044
     24 artel    appliances.kitchen.washer              8642
     25 bosch    appliances.environment.vacuum         17210
     26 bosch    construction.tools.drill               3502
     27 bosch    appliances.kitchen.hood                2996
     28 lenovo   electronics.audio.headphone           21170
     29 lenovo   computers.notebook                    19618
     30 lenovo   apparel.shoes.slipons                  2181
     31 indesit  appliances.kitchen.refrigerators      21269
     32 indesit  appliances.kitchen.washer             20796
     33 indesit  appliances.kitchen.dishwasher           146
     34 acer     computers.notebook                    19539
     35 acer     electronics.audio.headphone           13989
     36 acer     appliances.personal.massager           1296
     37 haier    appliances.personal.massager          15175
     38 haier    electronics.video.tv                   9425
     39 haier    appliances.kitchen.refrigerators       4430
     40 beko     appliances.kitchen.washer             19143
     41 beko     appliances.kitchen.refrigerators      10699
     42 beko     appliances.kitchen.dishwasher          1163
     43 midea    appliances.kitchen.washer             14927
     44 midea    appliances.kitchen.refrigerators       5087
     45 midea    furniture.bedroom.blanket              3952
     46 casio    electronics.clocks                    27678
     47 casio    apparel.underwear                       474
     48 casio    electronics.audio.music_tools.piano     153
     49 hp       electronics.audio.headphone            9426
     50 hp       computers.notebook                     8416
     51 hp       computers.peripherals.printer          3864
     52 tefal    appliances.environment.vacuum         10394
     53 tefal    appliances.iron                        3572
     54 tefal    furniture.bathroom.bath                3508
     55 sony     appliances.personal.massager           5964
     56 sony     sport.bicycle                          5543
     57 sony     electronics.video.tv                   4745
     58 vitek    appliances.environment.vacuum          9734
     59 vitek    appliances.kitchen.blender             4811
     60 vitek    appliances.kitchen.mixer               2087
     61 redmond  appliances.kitchen.blender             6204
     62 redmond  electronics.camera.photo               3931
     63 redmond  appliances.kitchen.kettle              2703
     64 dauscher appliances.kitchen.refrigerators       5258
     65 dauscher appliances.environment.vacuum          3156
     66 dauscher furniture.bedroom.blanket              2340
     67 nokia    electronics.camera.video               9659
     68 nokia    electronics.telephone                  8021
     69 nokia    construction.tools.light               2847
     70 philips  appliances.environment.vacuum          4450
     71 philips  appliances.personal.hair_cutter        2833
     72 philips  appliances.iron                        2780
     73 arg      furniture.bedroom.blanket              6804
     74 arg      appliances.kitchen.microwave           4999
     75 arg      appliances.kitchen.refrigerators       3307
     76 starline auto.accessories.alarm                12682
     77 starline apparel.shoes                          7101
     78 polaris  appliances.kitchen.blender             4843
     79 polaris  appliances.environment.vacuum          2561
     80 polaris  appliances.kitchen.mixer               2133
     81 jbl      sport.bicycle                         10752
     82 jbl      electronics.audio.headphone            7197
     83 jbl      electronics.audio.subwoofer             733
     84 braun    appliances.kitchen.blender             9399
     85 braun    appliances.personal.hair_cutter        2877
     86 braun    furniture.bathroom.bath                1758
     87 elenberg furniture.bathroom.bath                3234
     88 elenberg appliances.iron                        2374
     89 elenberg appliances.kitchen.oven                2308
     90 vivo     construction.tools.light              10379
     91 vivo     electronics.smartphone                 6945
     92 vivo     appliances.kitchen.refrigerators        553
     93 asus     computers.notebook                     7541
     94 asus     electronics.audio.headphone            6832
     95 asus     appliances.environment.air_heater       831
     96 janome   appliances.sewing_machine             16784
     97 janome   apparel.scarf                            20
     98 meizu    construction.tools.light               9028
     99 meizu    electronics.smartphone                 7321
    100 meizu    electronics.audio.headphone             142
    # ℹ 2,438 more rows

We see a pattern here , All brands with SMARTPHONE as their second best
category all have construction.tools.light as their top categories

We know Brands like Samsung , Apple , Huawei , e.t.c all specialize in
smartphone , so construction.tools.light is wrong . We should replace it
with smartphones . But we do this for brands that have smartphones as
their second best category .

<details class="code-fold">
<summary>Code</summary>

``` r
events_data %>%
  count(brand , category_code) %>%
  arrange(desc(n))%>%
  left_join(x=top_brands)%>%
  group_by(brand) %>%
  slice_max(order_by = n , n = 3) %>%
  left_join(x=top_brands) %>%
  group_by(brand) %>%
  mutate(cats = list(category_code)) %>%
  rowwise() %>%
  mutate(
    cats = na.omit(cats) %>% list() , 
    category_code_new = ifelse(cats[1] == "construction.tools.light" & cats[2] == "electronics.smartphone" & category_code =="construction.tools.light" & length(cats) > 1,
      cats[2],category_code)) %>%
  select(-n,-cats)%>%
  print(n=100) 
```

</details>

    # A tibble: 2,538 × 3
    # Rowwise:  brand
        brand    category_code                       category_code_new              
        <chr>    <chr>                               <chr>                          
      1 samsung  construction.tools.light            electronics.smartphone         
      2 samsung  electronics.smartphone              electronics.smartphone         
      3 samsung  appliances.personal.massager        appliances.personal.massager   
      4 <NA>     <NA>                                <NA>                           
      5 <NA>     appliances.kitchen.refrigerators    appliances.kitchen.refrigerato…
      6 <NA>     electronics.clocks                  electronics.clocks             
      7 apple    construction.tools.light            electronics.smartphone         
      8 apple    electronics.smartphone              electronics.smartphone         
      9 apple    sport.bicycle                       sport.bicycle                  
     10 xiaomi   construction.tools.light            electronics.smartphone         
     11 xiaomi   electronics.smartphone              electronics.smartphone         
     12 xiaomi   sport.bicycle                       sport.bicycle                  
     13 huawei   construction.tools.light            electronics.smartphone         
     14 huawei   electronics.smartphone              electronics.smartphone         
     15 huawei   sport.bicycle                       sport.bicycle                  
     16 oppo     construction.tools.light            electronics.smartphone         
     17 oppo     electronics.smartphone              electronics.smartphone         
     18 oppo     appliances.kitchen.refrigerators    appliances.kitchen.refrigerato…
     19 lg       appliances.kitchen.washer           appliances.kitchen.washer      
     20 lg       appliances.personal.massager        appliances.personal.massager   
     21 lg       electronics.video.tv                electronics.video.tv           
     22 artel    appliances.personal.massager        appliances.personal.massager   
     23 artel    electronics.video.tv                electronics.video.tv           
     24 artel    appliances.kitchen.washer           appliances.kitchen.washer      
     25 bosch    appliances.environment.vacuum       appliances.environment.vacuum  
     26 bosch    construction.tools.drill            construction.tools.drill       
     27 bosch    appliances.kitchen.hood             appliances.kitchen.hood        
     28 lenovo   electronics.audio.headphone         electronics.audio.headphone    
     29 lenovo   computers.notebook                  computers.notebook             
     30 lenovo   apparel.shoes.slipons               apparel.shoes.slipons          
     31 indesit  appliances.kitchen.refrigerators    appliances.kitchen.refrigerato…
     32 indesit  appliances.kitchen.washer           appliances.kitchen.washer      
     33 indesit  appliances.kitchen.dishwasher       appliances.kitchen.dishwasher  
     34 acer     computers.notebook                  computers.notebook             
     35 acer     electronics.audio.headphone         electronics.audio.headphone    
     36 acer     appliances.personal.massager        appliances.personal.massager   
     37 haier    appliances.personal.massager        appliances.personal.massager   
     38 haier    electronics.video.tv                electronics.video.tv           
     39 haier    appliances.kitchen.refrigerators    appliances.kitchen.refrigerato…
     40 beko     appliances.kitchen.washer           appliances.kitchen.washer      
     41 beko     appliances.kitchen.refrigerators    appliances.kitchen.refrigerato…
     42 beko     appliances.kitchen.dishwasher       appliances.kitchen.dishwasher  
     43 midea    appliances.kitchen.washer           appliances.kitchen.washer      
     44 midea    appliances.kitchen.refrigerators    appliances.kitchen.refrigerato…
     45 midea    furniture.bedroom.blanket           furniture.bedroom.blanket      
     46 casio    electronics.clocks                  electronics.clocks             
     47 casio    apparel.underwear                   apparel.underwear              
     48 casio    electronics.audio.music_tools.piano electronics.audio.music_tools.…
     49 hp       electronics.audio.headphone         electronics.audio.headphone    
     50 hp       computers.notebook                  computers.notebook             
     51 hp       computers.peripherals.printer       computers.peripherals.printer  
     52 tefal    appliances.environment.vacuum       appliances.environment.vacuum  
     53 tefal    appliances.iron                     appliances.iron                
     54 tefal    furniture.bathroom.bath             furniture.bathroom.bath        
     55 sony     appliances.personal.massager        appliances.personal.massager   
     56 sony     sport.bicycle                       sport.bicycle                  
     57 sony     electronics.video.tv                electronics.video.tv           
     58 vitek    appliances.environment.vacuum       appliances.environment.vacuum  
     59 vitek    appliances.kitchen.blender          appliances.kitchen.blender     
     60 vitek    appliances.kitchen.mixer            appliances.kitchen.mixer       
     61 redmond  appliances.kitchen.blender          appliances.kitchen.blender     
     62 redmond  electronics.camera.photo            electronics.camera.photo       
     63 redmond  appliances.kitchen.kettle           appliances.kitchen.kettle      
     64 dauscher appliances.kitchen.refrigerators    appliances.kitchen.refrigerato…
     65 dauscher appliances.environment.vacuum       appliances.environment.vacuum  
     66 dauscher furniture.bedroom.blanket           furniture.bedroom.blanket      
     67 nokia    electronics.camera.video            electronics.camera.video       
     68 nokia    electronics.telephone               electronics.telephone          
     69 nokia    construction.tools.light            construction.tools.light       
     70 philips  appliances.environment.vacuum       appliances.environment.vacuum  
     71 philips  appliances.personal.hair_cutter     appliances.personal.hair_cutter
     72 philips  appliances.iron                     appliances.iron                
     73 arg      furniture.bedroom.blanket           furniture.bedroom.blanket      
     74 arg      appliances.kitchen.microwave        appliances.kitchen.microwave   
     75 arg      appliances.kitchen.refrigerators    appliances.kitchen.refrigerato…
     76 starline auto.accessories.alarm              auto.accessories.alarm         
     77 starline apparel.shoes                       apparel.shoes                  
     78 polaris  appliances.kitchen.blender          appliances.kitchen.blender     
     79 polaris  appliances.environment.vacuum       appliances.environment.vacuum  
     80 polaris  appliances.kitchen.mixer            appliances.kitchen.mixer       
     81 jbl      sport.bicycle                       sport.bicycle                  
     82 jbl      electronics.audio.headphone         electronics.audio.headphone    
     83 jbl      electronics.audio.subwoofer         electronics.audio.subwoofer    
     84 braun    appliances.kitchen.blender          appliances.kitchen.blender     
     85 braun    appliances.personal.hair_cutter     appliances.personal.hair_cutter
     86 braun    furniture.bathroom.bath             furniture.bathroom.bath        
     87 elenberg furniture.bathroom.bath             furniture.bathroom.bath        
     88 elenberg appliances.iron                     appliances.iron                
     89 elenberg appliances.kitchen.oven             appliances.kitchen.oven        
     90 vivo     construction.tools.light            electronics.smartphone         
     91 vivo     electronics.smartphone              electronics.smartphone         
     92 vivo     appliances.kitchen.refrigerators    appliances.kitchen.refrigerato…
     93 asus     computers.notebook                  computers.notebook             
     94 asus     electronics.audio.headphone         electronics.audio.headphone    
     95 asus     appliances.environment.air_heater   appliances.environment.air_hea…
     96 janome   appliances.sewing_machine           appliances.sewing_machine      
     97 janome   apparel.scarf                       apparel.scarf                  
     98 meizu    construction.tools.light            electronics.smartphone         
     99 meizu    electronics.smartphone              electronics.smartphone         
    100 meizu    electronics.audio.headphone         electronics.audio.headphone    
    # ℹ 2,438 more rows

Here we see that our fix will work as intended .

<details class="code-fold">
<summary>Code</summary>

``` r
brand_merge <- brand_cat_data %>%
  group_by(brand) %>%
  mutate(cats = list(category_code)) %>%
  rowwise() %>%
  mutate(
    cats = na.omit(cats) %>% list() , 
    category_code_new = ifelse(cats[1] == "construction.tools.light" & cats[2] == "electronics.smartphone" & category_code =="construction.tools.light" & length(cats) > 1,
      cats[2],category_code)) %>%
  select(-n,-cats)

events_data <- events_data %>%
  left_join(y=brand_merge)

length(events_data %>% pull(category_code) %>% na.omit())
```

</details>

    [1] 5801004

<details class="code-fold">
<summary>Code</summary>

``` r
length(events_data %>% pull(category_code_new) %>% na.omit())
```

</details>

    [1] 5801004

Solutions seems to have worked just fine .

<details class="code-fold">
<summary>Code</summary>

``` r
events_data <- events_data %>%
  mutate(category_code = category_code_new) %>%
  select(-ends_with("new"))
```

</details>

<details class="code-fold">
<summary>Code</summary>

``` r
events_data %>%
  count(brand , category_code) %>%
  arrange(desc(n))%>%
  left_join(x=top_brands) %>%
  na.omit() %>%
  print(n=300)
```

</details>

    # A tibble: 4,079 × 3
        brand   category_code                                n
        <chr>   <chr>                                    <int>
      1 samsung electronics.smartphone                 1346545
      2 samsung appliances.personal.massager             60790
      3 samsung appliances.environment.vacuum            55993
      4 samsung electronics.video.tv                     52846
      5 samsung appliances.kitchen.washer                49766
      6 samsung appliances.kitchen.refrigerators         49311
      7 samsung electronics.clocks                       30688
      8 samsung apparel.shoes.slipons                    21938
      9 samsung sport.bicycle                            19269
     10 samsung electronics.tablet                       14649
     11 samsung electronics.audio.headphone              12679
     12 samsung appliances.kitchen.microwave              5625
     13 samsung furniture.bedroom.blanket                 4225
     14 samsung computers.peripherals.monitor             1948
     15 samsung appliances.kitchen.oven                    738
     16 samsung appliances.kitchen.hob                     714
     17 samsung appliances.kitchen.juicer                  645
     18 samsung computers.peripherals.printer              599
     19 samsung computers.components.hdd                   570
     20 samsung appliances.kitchen.hood                    450
     21 samsung appliances.environment.air_conditioner     440
     22 samsung appliances.kitchen.dishwasher              381
     23 samsung apparel.costume                            373
     24 samsung kids.toys                                  286
     25 samsung electronics.audio.acoustic                 175
     26 samsung construction.tools.welding                  68
     27 samsung stationery.cartrige                         68
     28 samsung electronics.camera.video                    36
     29 samsung appliances.iron                             13
     30 samsung computers.components.memory                 10
     31 samsung computers.ebooks                             6
     32 apple   electronics.smartphone                  989841
     33 apple   sport.bicycle                           116251
     34 apple   electronics.audio.headphone             106647
     35 apple   electronics.clocks                       77742
     36 apple   apparel.shoes.slipons                     6134
     37 apple   computers.notebook                        4846
     38 apple   electronics.tablet                        3635
     39 apple   appliances.kitchen.refrigerators           689
     40 apple   computers.desktop                          344
     41 apple   computers.ebooks                           142
     42 apple   furniture.kitchen.table                    139
     43 apple   computers.peripherals.mouse                111
     44 apple   accessories.bag                             82
     45 apple   apparel.scarf                               81
     46 apple   computers.peripherals.keyboard              65
     47 apple   apparel.sock                                17
     48 apple   sport.ski                                    9
     49 xiaomi  electronics.smartphone                  508999
     50 xiaomi  sport.bicycle                            41652
     51 xiaomi  electronics.audio.headphone              25573
     52 xiaomi  appliances.environment.vacuum            12339
     53 xiaomi  electronics.clocks                        7102
     54 xiaomi  appliances.kitchen.refrigerators          2181
     55 xiaomi  appliances.personal.massager              1856
     56 xiaomi  kids.swing                                1276
     57 xiaomi  accessories.wallet                        1193
     58 xiaomi  appliances.personal.scales                1175
     59 xiaomi  auto.accessories.videoregister            1170
     60 xiaomi  kids.skates                                677
     61 xiaomi  electronics.camera.photo                   563
     62 xiaomi  electronics.video.tv                       563
     63 xiaomi  accessories.bag                            551
     64 xiaomi  appliances.kitchen.mixer                   542
     65 xiaomi  sport.ski                                  448
     66 xiaomi  computers.notebook                         383
     67 xiaomi  appliances.kitchen.kettle                  362
     68 xiaomi  furniture.kitchen.table                    203
     69 xiaomi  apparel.shoes.keds                         131
     70 xiaomi  apparel.shoes.slipons                      117
     71 xiaomi  auto.accessories.compressor                116
     72 xiaomi  electronics.camera.video                   106
     73 xiaomi  computers.peripherals.mouse                104
     74 xiaomi  electronics.tablet                          86
     75 xiaomi  apparel.tshirt                              76
     76 xiaomi  appliances.environment.air_heater           65
     77 xiaomi  appliances.kitchen.blender                  64
     78 xiaomi  accessories.umbrella                        48
     79 xiaomi  appliances.kitchen.juicer                   40
     80 xiaomi  computers.desktop                           33
     81 xiaomi  apparel.sock                                28
     82 xiaomi  computers.ebooks                            26
     83 xiaomi  electronics.telephone                       15
     84 xiaomi  electronics.video.projector                 15
     85 xiaomi  furniture.bedroom.blanket                   13
     86 xiaomi  furniture.living_room.cabinet               12
     87 xiaomi  country_yard.furniture.hammok                9
     88 xiaomi  kids.toys                                    9
     89 xiaomi  furniture.bedroom.pillow                     4
     90 xiaomi  construction.tools.welding                   2
     91 huawei  electronics.smartphone                  235278
     92 huawei  sport.bicycle                             4553
     93 huawei  electronics.audio.headphone               3570
     94 huawei  apparel.shoes.slipons                     3121
     95 huawei  electronics.clocks                        2989
     96 huawei  appliances.kitchen.refrigerators          2871
     97 huawei  electronics.tablet                        2102
     98 huawei  kids.swing                                 649
     99 huawei  appliances.personal.scales                 268
    100 huawei  apparel.scarf                               15
    101 huawei  computers.peripherals.camera                 4
    102 oppo    electronics.smartphone                  127040
    103 oppo    appliances.kitchen.refrigerators           140
    104 lg      appliances.kitchen.washer                37316
    105 lg      appliances.personal.massager             24062
    106 lg      electronics.video.tv                     19190
    107 lg      appliances.kitchen.refrigerators         16064
    108 lg      appliances.environment.vacuum             9628
    109 lg      appliances.kitchen.microwave              2518
    110 lg      furniture.bedroom.blanket                 2112
    111 lg      apparel.costume                            564
    112 lg      computers.peripherals.monitor              508
    113 lg      electronics.audio.acoustic                 475
    114 lg      electronics.smartphone                     280
    115 lg      appliances.environment.air_conditioner     190
    116 lg      construction.tools.light                   170
    117 lg      computers.peripherals.printer              113
    118 lg      electronics.clocks                          21
    119 lg      electronics.camera.video                     6
    120 lg      sport.bicycle                                2
    121 artel   appliances.personal.massager             25513
    122 artel   electronics.video.tv                     23044
    123 artel   appliances.kitchen.washer                 8642
    124 artel   appliances.kitchen.oven                   7105
    125 artel   appliances.environment.vacuum             1951
    126 artel   electronics.clocks                        1506
    127 artel   appliances.environment.water_heater        961
    128 artel   appliances.kitchen.hood                    531
    129 artel   appliances.kitchen.refrigerators           315
    130 artel   apparel.shoes.keds                          69
    131 artel   appliances.environment.air_conditioner      65
    132 artel   appliances.kitchen.meat_grinder             35
    133 artel   electronics.camera.photo                    11
    134 artel   appliances.kitchen.kettle                    8
    135 bosch   appliances.environment.vacuum            17210
    136 bosch   construction.tools.drill                  3502
    137 bosch   appliances.kitchen.hood                   2996
    138 bosch   appliances.kitchen.washer                 2672
    139 bosch   appliances.kitchen.refrigerators          2431
    140 bosch   appliances.kitchen.dishwasher             2392
    141 bosch   appliances.kitchen.oven                   2039
    142 bosch   appliances.kitchen.meat_grinder           1898
    143 bosch   appliances.kitchen.blender                1656
    144 bosch   apparel.shoes.keds                        1285
    145 bosch   appliances.kitchen.hob                    1248
    146 bosch   appliances.kitchen.mixer                  1099
    147 bosch   appliances.kitchen.grill                   815
    148 bosch   electronics.audio.microphone               796
    149 bosch   appliances.kitchen.juicer                  792
    150 bosch   appliances.personal.massager               695
    151 bosch   electronics.camera.photo                   635
    152 bosch   construction.tools.saw                     634
    153 bosch   appliances.kitchen.toster                  558
    154 bosch   furniture.bedroom.blanket                  545
    155 bosch   appliances.kitchen.kettle                  484
    156 bosch   construction.tools.light                   309
    157 bosch   appliances.iron                            256
    158 bosch   apparel.shoes.moccasins                    223
    159 bosch   accessories.bag                            155
    160 bosch   country_yard.lawn_mower                    113
    161 bosch   appliances.kitchen.microwave                99
    162 bosch   construction.tools.painting                 99
    163 bosch   sport.trainer                               99
    164 bosch   appliances.kitchen.coffee_grinder           94
    165 bosch   apparel.shoes                               93
    166 bosch   furniture.bathroom.bath                     57
    167 bosch   apparel.shoes.sandals                       53
    168 bosch   apparel.shirt                               49
    169 bosch   electronics.video.tv                        39
    170 bosch   appliances.ironing_board                    32
    171 bosch   computers.components.cooler                 17
    172 bosch   appliances.kitchen.coffee_machine            8
    173 bosch   construction.tools.screw                     2
    174 lenovo  electronics.audio.headphone              21170
    175 lenovo  computers.notebook                       19618
    176 lenovo  apparel.shoes.slipons                     2181
    177 lenovo  electronics.tablet                        1430
    178 lenovo  accessories.bag                            458
    179 lenovo  sport.ski                                  380
    180 lenovo  electronics.clocks                         244
    181 lenovo  appliances.kitchen.refrigerators           212
    182 lenovo  computers.desktop                          176
    183 lenovo  furniture.living_room.cabinet              155
    184 lenovo  computers.peripherals.mouse                136
    185 lenovo  furniture.kitchen.table                    125
    186 lenovo  sport.bicycle                               63
    187 lenovo  apparel.scarf                               41
    188 lenovo  computers.peripherals.monitor               29
    189 lenovo  appliances.personal.massager                21
    190 lenovo  computers.peripherals.keyboard              21
    191 indesit appliances.kitchen.refrigerators         21269
    192 indesit appliances.kitchen.washer                20796
    193 indesit appliances.kitchen.dishwasher              146
    194 indesit appliances.kitchen.oven                     58
    195 indesit appliances.kitchen.hob                      54
    196 indesit appliances.kitchen.hood                     31
    197 indesit appliances.personal.massager                 4
    198 acer    computers.notebook                       19539
    199 acer    electronics.audio.headphone              13989
    200 acer    appliances.personal.massager              1296
    201 acer    computers.peripherals.monitor             1250
    202 acer    computers.desktop                         1104
    203 acer    appliances.kitchen.refrigerators           883
    204 acer    sport.ski                                  115
    205 acer    accessories.bag                             85
    206 acer    furniture.kitchen.table                     41
    207 acer    electronics.clocks                          35
    208 acer    electronics.tablet                          20
    209 acer    apparel.shoes.slipons                       19
    210 acer    computers.peripherals.mouse                  2
    211 acer    sport.bicycle                                2
    212 haier   appliances.personal.massager             15175
    213 haier   electronics.video.tv                      9425
    214 haier   appliances.kitchen.refrigerators          4430
    215 haier   appliances.kitchen.washer                 3404
    216 haier   appliances.environment.water_heater        777
    217 haier   electronics.clocks                         530
    218 haier   construction.tools.light                   147
    219 haier   electronics.smartphone                      97
    220 haier   computers.peripherals.printer               13
    221 haier   appliances.kitchen.oven                      9
    222 haier   appliances.kitchen.steam_cooker              8
    223 beko    appliances.kitchen.washer                19143
    224 beko    appliances.kitchen.refrigerators         10699
    225 beko    appliances.kitchen.dishwasher             1163
    226 beko    appliances.kitchen.oven                   1089
    227 beko    appliances.kitchen.hob                     539
    228 beko    appliances.environment.air_conditioner     211
    229 beko    appliances.kitchen.hood                     60
    230 beko    appliances.kitchen.steam_cooker             35
    231 beko    appliances.personal.massager                15
    232 midea   appliances.kitchen.washer                14927
    233 midea   appliances.kitchen.refrigerators          5087
    234 midea   furniture.bedroom.blanket                 3952
    235 midea   appliances.kitchen.hob                    2041
    236 midea   appliances.kitchen.microwave              1569
    237 midea   appliances.environment.air_conditioner     745
    238 midea   appliances.kitchen.hood                    735
    239 midea   appliances.environment.air_heater          685
    240 midea   appliances.kitchen.oven                    650
    241 midea   appliances.personal.massager               526
    242 midea   appliances.kitchen.dishwasher              418
    243 midea   computers.components.cooler                409
    244 midea   apparel.tshirt                             327
    245 midea   appliances.environment.water_heater        170
    246 midea   construction.tools.welding                  66
    247 midea   auto.accessories.videoregister              49
    248 midea   construction.tools.generator                33
    249 midea   electronics.clocks                          27
    250 midea   appliances.iron                              9
    251 midea   apparel.belt                                 7
    252 casio   electronics.clocks                       27678
    253 casio   apparel.underwear                          474
    254 casio   electronics.audio.music_tools.piano        153
    255 hp      electronics.audio.headphone               9426
    256 hp      computers.notebook                        8416
    257 hp      computers.peripherals.printer             3864
    258 hp      computers.peripherals.monitor              942
    259 hp      appliances.personal.massager               884
    260 hp      kids.toys                                  877
    261 hp      sport.ski                                  658
    262 hp      accessories.bag                            605
    263 hp      appliances.kitchen.refrigerators           546
    264 hp      computers.desktop                          427
    265 hp      stationery.cartrige                        412
    266 hp      kids.carriage                              327
    267 hp      computers.peripherals.mouse                262
    268 hp      furniture.kitchen.table                    199
    269 hp      apparel.sock                               159
    270 hp      apparel.scarf                               85
    271 hp      electronics.audio.subwoofer                 54
    272 hp      sport.bicycle                               53
    273 hp      computers.peripherals.keyboard              44
    274 hp      furniture.living_room.cabinet               22
    275 hp      appliances.kitchen.hob                       8
    276 hp      kids.skates                                  3
    277 hp      appliances.kitchen.kettle                    1
    278 hp      computers.peripherals.camera                 1
    279 hp      construction.tools.drill                     1
    280 tefal   appliances.environment.vacuum            10394
    281 tefal   appliances.iron                           3572
    282 tefal   furniture.bathroom.bath                   3508
    283 tefal   construction.tools.generator              2306
    284 tefal   electronics.camera.photo                  1751
    285 tefal   construction.tools.welding                1466
    286 tefal   appliances.kitchen.kettle                 1464
    287 tefal   appliances.kitchen.grill                  1309
    288 tefal   appliances.environment.air_heater          318
    289 tefal   appliances.kitchen.steam_cooker            283
    290 tefal   apparel.shoes.slipons                      239
    291 tefal   apparel.shoes                              134
    292 tefal   kids.swing                                 125
    293 tefal   appliances.personal.scales                  91
    294 tefal   appliances.ironing_board                    29
    295 tefal   apparel.shirt                               24
    296 tefal   appliances.kitchen.blender                  24
    297 tefal   furniture.bedroom.blanket                   23
    298 tefal   appliances.kitchen.toster                    5
    299 sony    appliances.personal.massager              5964
    300 sony    sport.bicycle                             5543
    # ℹ 3,779 more rows

The category codes of top brand look fine , there are no suspicious
categories that need to be investigated .

<details class="code-fold">
<summary>Code</summary>

``` r
events_data %>%
  count(brand , category_code,category_id) %>%
  arrange(desc(n))%>%
  left_join(x=top_brands) %>%
  na.omit() %>%
  print(n=300)
```

</details>

    # A tibble: 5,384 × 4
        brand   category_code                          category_id              n
        <chr>   <chr>                                  <chr>                <int>
      1 samsung electronics.smartphone                 2232732093077520640 727041
      2 samsung electronics.smartphone                 2053013555631882496 619501
      3 samsung appliances.personal.massager           2232732099754852864  58562
      4 samsung electronics.video.tv                   2053013554415534336  52837
      5 samsung appliances.kitchen.refrigerators       2053013563835941888  41110
      6 samsung appliances.kitchen.washer              2053013563810776064  28691
      7 samsung appliances.environment.vacuum          2053013565983425536  28000
      8 samsung appliances.environment.vacuum          2232732101063475712  27993
      9 samsung apparel.shoes.slipons                  2232732101407408640  21931
     10 samsung appliances.kitchen.washer              2232732092297380096  21075
     11 samsung sport.bicycle                          2232732079706079232  19269
     12 samsung electronics.clocks                     2232732103101907456  17374
     13 samsung electronics.tablet                     2172371436436455680  14649
     14 samsung electronics.clocks                     2053013553341792768  13314
     15 samsung electronics.audio.headphone            2053013554658804224  12679
     16 samsung appliances.kitchen.microwave           2053013554776244480   5625
     17 samsung appliances.kitchen.refrigerators       2053013563911439360   4565
     18 samsung furniture.bedroom.blanket              2232732102103663104   4225
     19 samsung appliances.kitchen.refrigerators       2232732091718566144   3636
     20 samsung appliances.personal.massager           2232732099981345280   1976
     21 samsung computers.peripherals.monitor          2053013553031414016   1948
     22 samsung appliances.kitchen.juicer              2053013555220840960    645
     23 samsung computers.components.hdd               2053013554222596608    570
     24 samsung appliances.kitchen.hood                2053013563743667200    450
     25 samsung appliances.kitchen.hob                 2053013563877884928    417
     26 samsung computers.peripherals.printer          2053013552955916544    402
     27 samsung appliances.kitchen.oven                2053013564003714048    396
     28 samsung apparel.costume                        2232732109049430784    373
     29 samsung appliances.kitchen.oven                2232732092565815808    342
     30 samsung appliances.kitchen.hob                 2232732102749585920    297
     31 samsung kids.toys                              2232732071460078336    286
     32 samsung appliances.environment.air_conditioner 2053013552351936512    260
     33 samsung appliances.personal.massager           2232732100769874432    252
     34 samsung appliances.kitchen.dishwasher          2053013557385101824    207
     35 samsung appliances.environment.air_conditioner 2053013557695480320    180
     36 samsung electronics.audio.acoustic             2053013554499420416    175
     37 samsung appliances.kitchen.dishwasher          2053013563944993536    174
     38 samsung computers.peripherals.printer          2053013553056579840    147
     39 samsung construction.tools.welding             2232732099436085760     68
     40 samsung stationery.cartrige                    2053013552737812736     55
     41 samsung computers.peripherals.printer          2232732086358245376     50
     42 samsung electronics.camera.video               2053013554323259904     36
     43 samsung appliances.iron                        2053013566176363520     13
     44 samsung stationery.cartrige                    2232732107019387392     13
     45 samsung computers.components.memory            2053013554189041920     10
     46 samsung electronics.video.tv                   2053013552603594752      9
     47 samsung apparel.shoes.slipons                  2053013552662315264      7
     48 samsung computers.ebooks                       2053013553316626432      6
     49 samsung electronics.smartphone                 2053013558525952512      3
     50 apple   electronics.smartphone                 2232732093077520640 521023
     51 apple   electronics.smartphone                 2053013555631882496 468818
     52 apple   sport.bicycle                          2232732079706079232 116251
     53 apple   electronics.audio.headphone            2053013554658804224 106647
     54 apple   electronics.clocks                     2232732103101907456  41387
     55 apple   electronics.clocks                     2053013553341792768  36355
     56 apple   apparel.shoes.slipons                  2232732101407408640   6134
     57 apple   computers.notebook                     2053013558920217344   4826
     58 apple   electronics.tablet                     2172371436436455680   3591
     59 apple   appliances.kitchen.refrigerators       2053013563835941888    689
     60 apple   computers.desktop                      2053013561092866816    344
     61 apple   computers.ebooks                       2053013553316626432    142
     62 apple   furniture.kitchen.table                2232732104066597376    139
     63 apple   computers.peripherals.mouse            2053013552888807680    111
     64 apple   apparel.scarf                          2232732104343421440     81
     65 apple   computers.peripherals.keyboard         2053013552913973504     65
     66 apple   accessories.bag                        2232732085997535232     53
     67 apple   electronics.tablet                     2053013555178897920     44
     68 apple   accessories.bag                        2053013558945382912     29
     69 apple   computers.notebook                     2053013553375346944     20
     70 apple   apparel.sock                           2053013553283072256     17
     71 apple   sport.ski                              2053013551907340544      9
     72 xiaomi  electronics.smartphone                 2232732093077520640 282116
     73 xiaomi  electronics.smartphone                 2053013555631882496 226867
     74 xiaomi  sport.bicycle                          2232732079706079232  41614
     75 xiaomi  electronics.audio.headphone            2053013554658804224  25573
     76 xiaomi  appliances.environment.vacuum          2053013565983425536   6175
     77 xiaomi  appliances.environment.vacuum          2232732101063475712   6164
     78 xiaomi  electronics.clocks                     2053013553341792768   5772
     79 xiaomi  appliances.kitchen.refrigerators       2053013563835941888   2181
     80 xiaomi  appliances.personal.massager           2232732099754852864   1856
     81 xiaomi  electronics.clocks                     2232732103101907456   1320
     82 xiaomi  kids.swing                             2232732105635267072   1276
     83 xiaomi  accessories.wallet                     2232732097397654016   1193
     84 xiaomi  appliances.personal.scales             2053013562946749184   1175
     85 xiaomi  auto.accessories.videoregister         2053013560899928576   1170
     86 xiaomi  kids.skates                            2053013555816432128    623
     87 xiaomi  electronics.camera.photo               2232732086928670976    563
     88 xiaomi  electronics.video.tv                   2053013554415534336    563
     89 xiaomi  appliances.kitchen.mixer               2053013555397001728    542
     90 xiaomi  accessories.bag                        2053013558945382912    515
     91 xiaomi  sport.ski                              2053013551907340544    448
     92 xiaomi  appliances.kitchen.kettle              2053013554834964736    362
     93 xiaomi  computers.notebook                     2053013558920217344    321
     94 xiaomi  furniture.kitchen.table                2232732104066597376    203
     95 xiaomi  apparel.shoes.keds                     2232732102640534016    131
     96 xiaomi  apparel.shoes.slipons                  2232732101407408640    117
     97 xiaomi  auto.accessories.compressor            2165087460176953600    116
     98 xiaomi  electronics.camera.video               2053013560530830080    106
     99 xiaomi  computers.peripherals.mouse            2053013552888807680    102
    100 xiaomi  electronics.tablet                     2172371436436455680     86
    101 xiaomi  apparel.tshirt                         2232732112211936000     76
    102 xiaomi  appliances.environment.air_heater      2053013552293216256     62
    103 xiaomi  computers.notebook                     2053013553375346944     62
    104 xiaomi  appliances.kitchen.blender             2053013555262784000     55
    105 xiaomi  kids.skates                            2053013555757711616     54
    106 xiaomi  appliances.kitchen.juicer              2053013555220840960     40
    107 xiaomi  sport.bicycle                          2116907525235278080     38
    108 xiaomi  computers.desktop                      2232732085150286080     33
    109 xiaomi  accessories.bag                        2232732085997535232     31
    110 xiaomi  apparel.sock                           2053013553283072256     28
    111 xiaomi  accessories.umbrella                   2053013561185141504     27
    112 xiaomi  computers.ebooks                       2053013553316626432     26
    113 xiaomi  accessories.umbrella                   2094006249627582720     21
    114 xiaomi  electronics.smartphone                 2053013562116277248     15
    115 xiaomi  electronics.telephone                  2053013555413778944     15
    116 xiaomi  electronics.video.projector            2053013552570040576     15
    117 xiaomi  furniture.bedroom.blanket              2232732093421453824     13
    118 xiaomi  furniture.living_room.cabinet          2232732110861370368     12
    119 xiaomi  electronics.clocks                     2053013561554240256     10
    120 xiaomi  appliances.kitchen.blender             2232732091391410688      9
    121 xiaomi  country_yard.furniture.hammok          2232732110643266304      9
    122 xiaomi  kids.toys                              2053013553090134528      9
    123 xiaomi  accessories.bag                        2053013561420022272      5
    124 xiaomi  furniture.bedroom.pillow               2053013561780732928      4
    125 xiaomi  appliances.environment.air_heater      2232732091961835776      3
    126 xiaomi  computers.peripherals.mouse            2053013552637149440      2
    127 xiaomi  construction.tools.welding             2232732099436085760      2
    128 xiaomi  electronics.smartphone                 2110937143172923904      1
    129 huawei  electronics.smartphone                 2232732093077520640 147424
    130 huawei  electronics.smartphone                 2053013555631882496  87854
    131 huawei  sport.bicycle                          2232732079706079232   4553
    132 huawei  electronics.audio.headphone            2053013554658804224   3570
    133 huawei  apparel.shoes.slipons                  2232732101407408640   3121
    134 huawei  appliances.kitchen.refrigerators       2053013563835941888   2871
    135 huawei  electronics.tablet                     2172371436436455680   1980
    136 huawei  electronics.clocks                     2053013553341792768   1621
    137 huawei  electronics.clocks                     2232732103101907456   1368
    138 huawei  kids.swing                             2232732105635267072    649
    139 huawei  appliances.personal.scales             2053013562946749184    268
    140 huawei  electronics.tablet                     2053013555178897920    122
    141 huawei  apparel.scarf                          2232732104175649280     15
    142 huawei  computers.peripherals.camera           2053013552502931968      4
    143 oppo    electronics.smartphone                 2232732093077520640  68326
    144 oppo    electronics.smartphone                 2053013555631882496  58714
    145 oppo    appliances.kitchen.refrigerators       2053013563835941888    140
    146 lg      appliances.personal.massager           2232732099754852864  23478
    147 lg      electronics.video.tv                   2053013554415534336  18974
    148 lg      appliances.kitchen.washer              2053013563810776064  18854
    149 lg      appliances.kitchen.washer              2232732092297380096  18187
    150 lg      appliances.kitchen.refrigerators       2053013563911439360   8368
    151 lg      appliances.kitchen.refrigerators       2232732091718566144   7555
    152 lg      appliances.environment.vacuum          2053013565983425536   5864
    153 lg      appliances.environment.vacuum          2232732101063475712   3764
    154 lg      appliances.kitchen.microwave           2053013554776244480   2518
    155 lg      furniture.bedroom.blanket              2232732102103663104   1904
    156 lg      appliances.personal.massager           2232732099981345280    584
    157 lg      apparel.costume                        2232732109049430784    564
    158 lg      computers.peripherals.monitor          2053013553031414016    497
    159 lg      electronics.audio.acoustic             2053013554499420416    475
    160 lg      electronics.smartphone                 2053013555631882496    280
    161 lg      appliances.kitchen.washer              2232732062048060160    275
    162 lg      electronics.video.tv                   2053013552603594752    216
    163 lg      furniture.bedroom.blanket              2232732102204326400    208
    164 lg      construction.tools.light               2232732093077520640    170
    165 lg      appliances.kitchen.refrigerators       2053013563835941888    141
    166 lg      computers.peripherals.printer          2053013552955916544    113
    167 lg      appliances.environment.air_conditioner 2053013552351936512     98
    168 lg      appliances.environment.air_conditioner 2053013557695480320     92
    169 lg      electronics.clocks                     2053013561579406080     20
    170 lg      computers.peripherals.monitor          2232732061737681664     11
    171 lg      electronics.camera.video               2053013554323259904      6
    172 lg      sport.bicycle                          2116907525176557824      2
    173 lg      electronics.clocks                     2232732082063278080      1
    174 artel   appliances.personal.massager           2232732099754852864  24977
    175 artel   electronics.video.tv                   2053013554415534336  23044
    176 artel   appliances.kitchen.washer              2232732092297380096   4505
    177 artel   appliances.kitchen.oven                2232732092565815808   4201
    178 artel   appliances.kitchen.washer              2053013563810776064   4137
    179 artel   appliances.kitchen.oven                2053013564003714048   2904
    180 artel   electronics.clocks                     2053013557452210688   1506
    181 artel   appliances.environment.vacuum          2232732101063475712   1136
    182 artel   appliances.environment.water_heater    2053013552326770944    961
    183 artel   appliances.environment.vacuum          2053013565983425536    815
    184 artel   appliances.personal.massager           2232732100769874432    536
    185 artel   appliances.kitchen.hood                2053013563743667200    531
    186 artel   appliances.kitchen.refrigerators       2232732091718566144    183
    187 artel   appliances.kitchen.refrigerators       2053013563911439360    132
    188 artel   apparel.shoes.keds                     2232732102019777024     69
    189 artel   appliances.environment.air_conditioner 2053013557695480320     38
    190 artel   appliances.kitchen.meat_grinder        2053013555321504000     35
    191 artel   appliances.environment.air_conditioner 2053013552351936512     27
    192 artel   electronics.camera.photo               2232732086928670976     11
    193 artel   appliances.kitchen.kettle              2053013554834964736      8
    194 bosch   appliances.environment.vacuum          2053013565983425536   8543
    195 bosch   appliances.environment.vacuum          2232732101063475712   8383
    196 bosch   appliances.kitchen.hood                2053013563743667200   2996
    197 bosch   construction.tools.drill               2053013556311360000   2248
    198 bosch   appliances.kitchen.meat_grinder        2053013555321504000   1898
    199 bosch   appliances.kitchen.washer              2053013563810776064   1722
    200 bosch   appliances.kitchen.refrigerators       2053013563911439360   1445
    201 bosch   appliances.kitchen.dishwasher          2053013563944993536   1304
    202 bosch   apparel.shoes.keds                     2232732102019777024   1285
    203 bosch   appliances.kitchen.oven                2053013564003714048   1211
    204 bosch   appliances.kitchen.dishwasher          2053013557385101824   1082
    205 bosch   appliances.kitchen.refrigerators       2232732091718566144    986
    206 bosch   appliances.kitchen.blender             2053013555262784000    962
    207 bosch   appliances.kitchen.washer              2232732092297380096    950
    208 bosch   construction.tools.drill               2053013556252639744    853
    209 bosch   appliances.kitchen.oven                2232732092565815808    828
    210 bosch   electronics.audio.microphone           2232732092087664896    796
    211 bosch   appliances.kitchen.juicer              2053013555220840960    787
    212 bosch   appliances.kitchen.hob                 2053013563877884928    725
    213 bosch   appliances.kitchen.mixer               2232732105912091136    716
    214 bosch   appliances.personal.massager           2232732100769874432    695
    215 bosch   appliances.kitchen.blender             2232732091391410688    694
    216 bosch   electronics.camera.photo               2232732086928670976    635
    217 bosch   appliances.kitchen.grill               2232732105811427840    617
    218 bosch   appliances.kitchen.hob                 2232732102749585920    523
    219 bosch   appliances.kitchen.kettle              2053013554834964736    484
    220 bosch   furniture.bedroom.blanket              2232732102858637824    465
    221 bosch   appliances.kitchen.toster              2053013554960793856    407
    222 bosch   construction.tools.drill               2053013551932506112    401
    223 bosch   appliances.kitchen.mixer               2053013555069845760    383
    224 bosch   construction.tools.saw                 2053013556227473920    317
    225 bosch   construction.tools.light               2232732093345956096    309
    226 bosch   appliances.iron                        2053013566176363520    256
    227 bosch   apparel.shoes.moccasins                2232732098077131264    223
    228 bosch   appliances.environment.vacuum          2134905044766557184    199
    229 bosch   appliances.kitchen.grill               2053013554751078656    198
    230 bosch   construction.tools.saw                 2053013556202308096    153
    231 bosch   appliances.kitchen.toster              2232732107111662080    151
    232 bosch   construction.tools.saw                 2232732092028944640    125
    233 bosch   country_yard.lawn_mower                2053013560061067776    113
    234 bosch   accessories.bag                        2053013558668559104    109
    235 bosch   appliances.kitchen.microwave           2053013554776244480     99
    236 bosch   construction.tools.painting            2127425434894205440     99
    237 bosch   sport.trainer                          2232732095568937216     99
    238 bosch   appliances.kitchen.coffee_grinder      2053013554725913088     94
    239 bosch   furniture.bedroom.blanket              2232732102103663104     80
    240 bosch   appliances.environment.vacuum          2232732101164139008     58
    241 bosch   furniture.bathroom.bath                2053013557477376512     57
    242 bosch   apparel.shoes.sandals                  2232732098614002176     53
    243 bosch   apparel.shoes                          2232732127554699520     51
    244 bosch   apparel.shirt                          2232732109418529536     49
    245 bosch   accessories.bag                        2139150089359196160     46
    246 bosch   apparel.shoes                          2053013565782098944     42
    247 bosch   construction.tools.saw                 2144916515806248960     39
    248 bosch   electronics.video.tv                   2053013552603594752     39
    249 bosch   appliances.ironing_board               2053013566033757184     32
    250 bosch   appliances.environment.vacuum          2053013559960404480     27
    251 bosch   computers.components.cooler            2053013557787755008     17
    252 bosch   appliances.kitchen.coffee_machine      2053013555095011840      8
    253 bosch   appliances.kitchen.dishwasher          2232732064673694720      6
    254 bosch   appliances.kitchen.juicer              2232732065311228928      5
    255 bosch   construction.tools.screw               2187707789055361280      2
    256 lenovo  electronics.audio.headphone            2053013554658804224  21170
    257 lenovo  computers.notebook                     2053013558920217344  19010
    258 lenovo  apparel.shoes.slipons                  2232732101407408640   2181
    259 lenovo  electronics.tablet                     2172371436436455680   1381
    260 lenovo  computers.notebook                     2053013553375346944    608
    261 lenovo  accessories.bag                        2053013558945382912    458
    262 lenovo  sport.ski                              2053013551907340544    380
    263 lenovo  electronics.clocks                     2053013553341792768    244
    264 lenovo  appliances.kitchen.refrigerators       2053013563835941888    212
    265 lenovo  computers.desktop                      2053013561092866816    176
    266 lenovo  furniture.living_room.cabinet          2232732110861370368    155
    267 lenovo  furniture.kitchen.table                2232732104066597376    125
    268 lenovo  computers.peripherals.mouse            2053013552637149440     90
    269 lenovo  sport.bicycle                          2232732079706079232     63
    270 lenovo  computers.peripherals.mouse            2053013552888807680     46
    271 lenovo  apparel.scarf                          2232732104343421440     41
    272 lenovo  electronics.tablet                     2053013555178897920     40
    273 lenovo  computers.peripherals.monitor          2053013553031414016     29
    274 lenovo  appliances.personal.massager           2232732099981345280     21
    275 lenovo  computers.peripherals.keyboard         2053013552913973504     21
    276 lenovo  electronics.tablet                     2053013561059312384      9
    277 indesit appliances.kitchen.washer              2053013563810776064  11847
    278 indesit appliances.kitchen.refrigerators       2053013563911439360   9495
    279 indesit appliances.kitchen.washer              2232732092297380096   8901
    280 indesit appliances.kitchen.refrigerators       2232732091718566144   8237
    281 indesit appliances.kitchen.refrigerators       2053013563835941888   1847
    282 indesit appliances.kitchen.refrigerators       2232732091307524608   1690
    283 indesit appliances.kitchen.dishwasher          2053013563944993536     96
    284 indesit appliances.kitchen.washer              2232732062048060160     48
    285 indesit appliances.kitchen.oven                2053013564003714048     38
    286 indesit appliances.kitchen.dishwasher          2053013557385101824     34
    287 indesit appliances.kitchen.hood                2053013563743667200     31
    288 indesit appliances.kitchen.hob                 2053013563877884928     29
    289 indesit appliances.kitchen.hob                 2232732102749585920     25
    290 indesit appliances.kitchen.oven                2232732092565815808     20
    291 indesit appliances.kitchen.dishwasher          2232732064673694720     16
    292 indesit appliances.personal.massager           2232732100769874432      4
    293 acer    computers.notebook                     2053013558920217344  18908
    294 acer    electronics.audio.headphone            2053013554658804224  13989
    295 acer    appliances.personal.massager           2232732099981345280   1296
    296 acer    computers.peripherals.monitor          2053013553031414016   1217
    297 acer    computers.desktop                      2053013561092866816   1104
    298 acer    appliances.kitchen.refrigerators       2053013563835941888    883
    299 acer    computers.notebook                     2053013553375346944    631
    300 acer    sport.ski                              2053013551907340544    115
    # ℹ 5,084 more rows

Here , we see that there a unique brand and category combinations with
multiple category id .

Now that brands and categories are investigated , lets fix the issue
where same product has multiple categories and brands

- Lets assign top categories and brands to products products with
  multiple brand and categories , because ideally we should have one
  category and brand per product .

### Assign each product to one brand , cateogry code and ID

Each product should be associated with one brand , category and category
ID

<details class="code-fold">
<summary>Code</summary>

``` r
dup_data <- events_data %>%
  group_by(product_id) %>%
  summarise(
    n_brands = n_distinct(brand),
    n_cat_id = n_distinct(category_id),
    n_cat_code = n_distinct(category_code)
  )

dup_ids <- dup_data %>%
  filter(n_brands > 1 | n_cat_id > 1 | n_cat_code > 1) %>%
  pull(product_id)

events_data %>%
  filter(product_id %in% dup_ids) %>%
  arrange(product_id)%>%
  distinct(product_id ,brand,category_code ,category_id , .keep_all = T) %>%
  select(product_id , brand , category_code , category_id)%>%
  head(n=20) %>% kable()
```

</details>

| product_id | brand         | category_code               | category_id         |
|:-----------|:--------------|:----------------------------|:--------------------|
| 100000153  | respect       | apparel.shoes               | 2053013565782098944 |
| 100000153  | respect       | apparel.shoes               | 2232732098706276864 |
| 100000154  | respect       | apparel.shoes               | 2053013565782098944 |
| 100000154  | respect       | apparel.shoes               | 2053013556521075200 |
| 100000179  | parkmaster    | auto.accessories.parktronic | 2053013560623104512 |
| 100000179  | parkmaster    | accessories.umbrella        | 2232732109494026752 |
| 100000196  | milavitsa     | apparel.underwear           | 2070005009382113792 |
| 100000196  | milavitsa     | electronics.telephone       | 2053013555573162240 |
| 100000208  | milavitsa     | apparel.underwear           | 2070005009382113792 |
| 100000208  | milavitsa     | electronics.telephone       | 2053013555573162240 |
| 100000210  | jvc           | auto.accessories.player     | 2053013553970938368 |
| 100000210  | jvc           | electronics.video.tv        | 2053013554415534336 |
| 100000244  | graffitisound | electronics.audio.subwoofer | 2053013553945772544 |
| 100000244  | graffitisound | electronics.audio.subwoofer | 2232732082390433792 |
| 100000263  | jvc           | electronics.audio.subwoofer | 2053013553945772544 |
| 100000263  | jvc           | electronics.audio.subwoofer | 2232732082390433792 |
| 100000273  | parkmaster    | auto.accessories.parktronic | 2053013560623104512 |
| 100000273  | parkmaster    | accessories.umbrella        | 2232732109494026752 |
| 100000324  | continent     | accessories.bag             | 2053013558945382912 |
| 100000324  | continent     | sport.ski                   | 2053013551907340544 |

Here we see that product ids have multiple brands ,category code and IDS
, lets fix this by assigning the most common brand , category code and
ID combination to these product IDs

<details class="code-fold">
<summary>Code</summary>

``` r
merge <- events_data %>%
  count(product_id,brand , category_code , category_id) %>%
  arrange(desc(n))%>%
  filter(!is.na(product_id)) %>%
  group_by(product_id)%>%
  slice_head(n=1)%>%
  rename("brand_new" = brand , 
         "category_code_new" = category_code ,
         "category_id_new" = category_id) %>%
  arrange(desc(n))

events_data <- left_join(events_data , merge)%>%
  mutate(brand = brand_new , 
         category_code = category_code_new , 
         category_id = category_id_new) %>%
  select(-ends_with("new"),-n)

events_data %>%
  filter(product_id %in% dup_ids) %>%
  arrange(product_id)%>%
  distinct(product_id ,brand,category_code ,category_id , .keep_all = T) %>%
  select(product_id , brand , category_code , category_id)%>%
  head(n=20) %>% kable()
```

</details>

| product_id | brand | category_code | category_id |
|:---|:---|:---|:---|
| 100000153 | respect | apparel.shoes | 2232732098706276864 |
| 100000154 | respect | apparel.shoes | 2053013565782098944 |
| 100000179 | parkmaster | auto.accessories.parktronic | 2053013560623104512 |
| 100000196 | milavitsa | apparel.underwear | 2070005009382113792 |
| 100000208 | milavitsa | apparel.underwear | 2070005009382113792 |
| 100000210 | jvc | electronics.video.tv | 2053013554415534336 |
| 100000244 | graffitisound | electronics.audio.subwoofer | 2232732082390433792 |
| 100000263 | jvc | electronics.audio.subwoofer | 2053013553945772544 |
| 100000273 | parkmaster | auto.accessories.parktronic | 2053013560623104512 |
| 100000324 | continent | sport.ski | 2053013551907340544 |
| 100000368 | delimano | electronics.clocks | 2053013557452210688 |
| 100000783 | adamas | computers.notebook | 2053013553375346944 |
| 100000811 | resanta | appliances.environment.air_heater | 2053013552293216256 |
| 100000912 | milavitsa | apparel.underwear | 2070005009382113792 |
| 100000923 | delimano | appliances.kitchen.blender | 2232732091391410688 |
| 100001169 | skagen | electronics.clocks | 2232732082063278080 |
| 100001222 | zeppelin | electronics.clocks | 2232732082063278080 |
| 100001331 | armani | electronics.clocks | 2232732082063278080 |
| 100001368 | michaelkors | electronics.clocks | 2053013561579406080 |
| 100001372 | michaelkors | electronics.clocks | 2232732082063278080 |

<details class="code-fold">
<summary>Code</summary>

``` r
events_data %>%
  group_by(product_id) %>%
  summarise(
    n_brands = n_distinct(brand),
    n_cat_id = n_distinct(category_id),
    n_cat_code = n_distinct(category_code)
  ) %>%
  filter(n_brands > 1 | n_cat_id > 1 | n_cat_code > 1)%>%
  head(n=20) %>% kable()
```

</details>

| product_id | n_brands | n_cat_id | n_cat_code |
|:-----------|---------:|---------:|-----------:|

Now we see that the product IDs now have one brand , category code and
id assigned to them .

> - Instead of assigning the most common brand , category code and ID
>   combination to each product ID with multiple combinations , we could
>   have assign the last used combination instead , meaning order
>   combinations by their event time / date and assign the latest
>   combination to our product IDs

### Category Code and ID

In online store databases , category code should be assigned to only one
category ID

<details class="code-fold">
<summary>Code</summary>

``` r
cat_data <- events_data %>%
  count(category_code , category_id)%>%
  arrange(category_code,desc(n))
cat_data%>%
  head(n=20) %>% kable()
```

</details>

| category_code        | category_id         |     n |
|:---------------------|:--------------------|------:|
| accessories.bag      | 2232732108453839616 |  3697 |
| accessories.bag      | 2053013558945382912 |  3089 |
| accessories.bag      | 2053013566209917952 |   965 |
| accessories.bag      | 2232732082935693312 |   937 |
| accessories.bag      | 2053013558668559104 |   774 |
| accessories.bag      | 2232732113503781888 |   755 |
| accessories.bag      | 2139150089359196160 |   320 |
| accessories.bag      | 2232732097330544896 |   291 |
| accessories.bag      | 2055156924407612160 |   263 |
| accessories.bag      | 2232732085997535232 |   148 |
| accessories.bag      | 2232732097255047680 |   116 |
| accessories.bag      | 2126679654801604864 |    74 |
| accessories.bag      | 2053013561420022272 |    61 |
| accessories.bag      | 2110937189033444096 |    59 |
| accessories.bag      | 2232732113596056320 |     4 |
| accessories.umbrella | 2232732109494026752 |   160 |
| accessories.umbrella | 2053013561185141504 |   112 |
| accessories.umbrella | 2094006249627582720 |    79 |
| accessories.wallet   | 2232732097397654016 | 10397 |
| accessories.wallet   | 2053013566243472384 |   636 |

<details class="code-fold">
<summary>Code</summary>

``` r
merge <- cat_data %>%
  group_by(category_code) %>%
  slice_head(n=1) %>%
  ungroup()%>%
  rename("category_id_new" = category_id)

merge%>%
  head(n=20) %>% kable()
```

</details>

| category_code              | category_id_new     |     n |
|:---------------------------|:--------------------|------:|
| accessories.bag            | 2232732108453839616 |  3697 |
| accessories.umbrella       | 2232732109494026752 |   160 |
| accessories.wallet         | 2232732097397654016 | 10397 |
| apparel.belt               | 2232732097473151488 |    22 |
| apparel.costume            | 2232732108923601664 |  3156 |
| apparel.dress              | 2146660887002349824 |   171 |
| apparel.glove              | 2159535995811266816 |    41 |
| apparel.jeans              | 2053013552469377280 |   321 |
| apparel.jumper             | 2166064855264526848 |   145 |
| apparel.scarf              | 2232732104343421440 |  7429 |
| apparel.shirt              | 2232732128041238784 |  3602 |
| apparel.shoes              | 2053013565639492608 |  4771 |
| apparel.shoes.ballet_shoes | 2232732103613612544 |  4597 |
| apparel.shoes.espadrilles  | 2232732098303623680 |  3634 |
| apparel.shoes.keds         | 2232732102019777024 | 14950 |
| apparel.shoes.moccasins    | 2053013557024391680 |  1397 |
| apparel.shoes.sandals      | 2232732098446230016 |   800 |
| apparel.shoes.slipons      | 2232732101407408640 | 46095 |
| apparel.shoes.step_ins     | 2232732103294845440 |  3025 |
| apparel.shorts             | 2232732107312988672 |   271 |

<details class="code-fold">
<summary>Code</summary>

``` r
cat_data <- cat_data %>%
  left_join(y=merge %>% select(-n)) %>% select(-n)
cat_data%>%
  head(n=20) %>% kable()
```

</details>

| category_code        | category_id         | category_id_new     |
|:---------------------|:--------------------|:--------------------|
| accessories.bag      | 2232732108453839616 | 2232732108453839616 |
| accessories.bag      | 2053013558945382912 | 2232732108453839616 |
| accessories.bag      | 2053013566209917952 | 2232732108453839616 |
| accessories.bag      | 2232732082935693312 | 2232732108453839616 |
| accessories.bag      | 2053013558668559104 | 2232732108453839616 |
| accessories.bag      | 2232732113503781888 | 2232732108453839616 |
| accessories.bag      | 2139150089359196160 | 2232732108453839616 |
| accessories.bag      | 2232732097330544896 | 2232732108453839616 |
| accessories.bag      | 2055156924407612160 | 2232732108453839616 |
| accessories.bag      | 2232732085997535232 | 2232732108453839616 |
| accessories.bag      | 2232732097255047680 | 2232732108453839616 |
| accessories.bag      | 2126679654801604864 | 2232732108453839616 |
| accessories.bag      | 2053013561420022272 | 2232732108453839616 |
| accessories.bag      | 2110937189033444096 | 2232732108453839616 |
| accessories.bag      | 2232732113596056320 | 2232732108453839616 |
| accessories.umbrella | 2232732109494026752 | 2232732109494026752 |
| accessories.umbrella | 2053013561185141504 | 2232732109494026752 |
| accessories.umbrella | 2094006249627582720 | 2232732109494026752 |
| accessories.wallet   | 2232732097397654016 | 2232732097397654016 |
| accessories.wallet   | 2053013566243472384 | 2232732097397654016 |

Here we see that different category codes have multiple category IDs .
Lets fix this by assign the most common category ID to each category
codes with multiple category ID .

> - Better solution is by assigning last used category ID to each
>   category code . Meaning finding the category ID of each use with the
>   most recent event time and assigning it to the category code .

Now each category_code has one ID assigned to it . Now lets merge this
with event data in order to fix category codes having multiple category
ids issue .

<details class="code-fold">
<summary>Code</summary>

``` r
events_data %>%
  group_by(category_code) %>%
  summarise(n_dist_cat_id = n_distinct(category_id)) %>%
  arrange(desc(n_dist_cat_id)) %>% ungroup() %>% 
  select(category_code) -> cat_codes

events_data %>%
  group_by(category_code) %>%
  summarise(n_dist_cat_id = n_distinct(category_id)) %>%
  arrange(desc(n_dist_cat_id))%>%
  head(n=20) %>% kable()
```

</details>

| category_code                    | n_dist_cat_id |
|:---------------------------------|--------------:|
| apparel.shoes                    |            48 |
| apparel.shirt                    |            16 |
| accessories.bag                  |            15 |
| apparel.shoes.keds               |            15 |
| apparel.costume                  |            12 |
| apparel.shoes.moccasins          |            10 |
| construction.tools.light         |            10 |
| furniture.bedroom.blanket        |            10 |
| apparel.shoes.sandals            |             8 |
| appliances.kitchen.refrigerators |             8 |
| furniture.living_room.cabinet    |             8 |
| sport.ski                        |             8 |
| sport.trainer                    |             8 |
| appliances.kitchen.grill         |             7 |
| computers.desktop                |             7 |
| construction.tools.saw           |             7 |
| electronics.clocks               |             7 |
| furniture.bedroom.bed            |             7 |
| furniture.kitchen.table          |             7 |
| kids.skates                      |             7 |

We see that before assigning one category ID to each category code , we
have same category IDs with more than one distinct category ID . Example
here is category code “apparel.shoes” with 48 distinct category IDs

<details class="code-fold">
<summary>Code</summary>

``` r
events_data %>%
  left_join(y=cat_data) %>%
  group_by(category_code) %>%
  summarise(n_dist_cat_id = n_distinct(category_id_new)) %>%
  left_join(x=cat_codes) %>%
  head(n=20) %>% kable()
```

</details>

| category_code                    | n_dist_cat_id |
|:---------------------------------|--------------:|
| apparel.shoes                    |             1 |
| apparel.shirt                    |             1 |
| accessories.bag                  |             1 |
| apparel.shoes.keds               |             1 |
| apparel.costume                  |             1 |
| apparel.shoes.moccasins          |             1 |
| construction.tools.light         |             1 |
| furniture.bedroom.blanket        |             1 |
| apparel.shoes.sandals            |             1 |
| appliances.kitchen.refrigerators |             1 |
| furniture.living_room.cabinet    |             1 |
| sport.ski                        |             1 |
| sport.trainer                    |             1 |
| appliances.kitchen.grill         |             1 |
| computers.desktop                |             1 |
| construction.tools.saw           |             1 |
| electronics.clocks               |             1 |
| furniture.bedroom.bed            |             1 |
| furniture.kitchen.table          |             1 |
| kids.skates                      |             1 |

Here we see that if we assign one category ID to each category code ,
the category codes will have one distinct category ID . Example
apparel.shoes will now have one distinct category ID instead of the 48
distinct category IDs

<details class="code-fold">
<summary>Code</summary>

``` r
events_data <- events_data %>%
  left_join(y=cat_data) %>%
  mutate(category_id = category_id_new) %>%
  select(-ends_with("new"))
```

</details>

Now lets verify that each product ID has one brand , category code and
category ID .

<details class="code-fold">
<summary>Code</summary>

``` r
events_data %>%
  group_by(product_id) %>%
  summarise(
    n_brands = n_distinct(brand),
    n_cat_id = n_distinct(category_id),
    n_cat_code = n_distinct(category_code)
  ) %>%
  filter(n_brands >1 | n_cat_id >1 | n_cat_code >1) %>%
  head() %>% kable()
```

</details>

| product_id | n_brands | n_cat_id | n_cat_code |
|:-----------|---------:|---------:|-----------:|

<details class="code-fold">
<summary>Code</summary>

``` r
events_data %>% head(n=10) %>% kable()
```

</details>

| event_time | event_type | product_id | category_id | category_code | brand | price | user_id | user_session | event_date |
|:---|:---|:---|:---|:---|:---|---:|:---|:---|:---|
| 2019-11-01 00:00:14 | cart | 1005014 | 2232732093077520640 | electronics.smartphone | samsung | 503.09 | 533326659 | 6b928be2-2bce-4640-8296-0efdf2fda22a | 2019-11-01 |
| 2019-11-01 00:00:41 | purchase | 13200605 | NA | NA | NA | 566.30 | 559368633 | d6034fa2-41fb-4ac0-9051-55ea9fc9147a | 2019-11-01 |
| 2019-11-01 00:01:04 | purchase | 1005161 | 2232732093077520640 | electronics.smartphone | xiaomi | 211.92 | 513351129 | e6b7ce9b-1938-4e20-976c-8b4163aea11d | 2019-11-01 |
| 2019-11-01 00:03:24 | cart | 1801881 | 2232732099754852864 | appliances.personal.massager | samsung | 488.80 | 557746614 | 4d76d6d3-fff5-4880-8327-e9e57b618e0e | 2019-11-01 |
| 2019-11-01 00:03:39 | cart | 1005115 | 2232732093077520640 | electronics.smartphone | apple | 949.47 | 565865924 | fd4bd6d4-bd14-4fdc-9aff-bd41a594f82e | 2019-11-01 |
| 2019-11-01 00:04:51 | purchase | 1004856 | 2232732093077520640 | electronics.smartphone | samsung | 128.42 | 562958505 | 0f039697-fedc-40fa-8830-39c1a024351d | 2019-11-01 |
| 2019-11-01 00:05:54 | cart | 1002542 | 2232732093077520640 | electronics.smartphone | apple | 486.80 | 549256216 | dcbdc6e4-cd49-4ee8-95c5-e85f3c618fa1 | 2019-11-01 |
| 2019-11-01 00:06:33 | purchase | 1801881 | 2232732099754852864 | appliances.personal.massager | samsung | 488.80 | 557746614 | 4d76d6d3-fff5-4880-8327-e9e57b618e0e | 2019-11-01 |
| 2019-11-01 00:06:34 | purchase | 5800823 | 2232732082390433792 | electronics.audio.subwoofer | nakamichi | 123.56 | 514166940 | 8ef5214a-86ad-4d0b-8df3-4280dd411b47 | 2019-11-01 |
| 2019-11-01 00:06:38 | cart | 1004856 | 2232732093077520640 | electronics.smartphone | samsung | 128.42 | 513645631 | 61ceaf50-820a-4858-9a68-bab804d47a22 | 2019-11-01 |

Thats is our basic cleaning and data validation done .

## Augment our data

Use existing columns to find other columns that could be useful to our
analysis

<details class="code-fold">
<summary>Code</summary>

``` r
events_data %>% head(n=10) %>% kable()
```

</details>

| event_time | event_type | product_id | category_id | category_code | brand | price | user_id | user_session | event_date |
|:---|:---|:---|:---|:---|:---|---:|:---|:---|:---|
| 2019-11-01 00:00:14 | cart | 1005014 | 2232732093077520640 | electronics.smartphone | samsung | 503.09 | 533326659 | 6b928be2-2bce-4640-8296-0efdf2fda22a | 2019-11-01 |
| 2019-11-01 00:00:41 | purchase | 13200605 | NA | NA | NA | 566.30 | 559368633 | d6034fa2-41fb-4ac0-9051-55ea9fc9147a | 2019-11-01 |
| 2019-11-01 00:01:04 | purchase | 1005161 | 2232732093077520640 | electronics.smartphone | xiaomi | 211.92 | 513351129 | e6b7ce9b-1938-4e20-976c-8b4163aea11d | 2019-11-01 |
| 2019-11-01 00:03:24 | cart | 1801881 | 2232732099754852864 | appliances.personal.massager | samsung | 488.80 | 557746614 | 4d76d6d3-fff5-4880-8327-e9e57b618e0e | 2019-11-01 |
| 2019-11-01 00:03:39 | cart | 1005115 | 2232732093077520640 | electronics.smartphone | apple | 949.47 | 565865924 | fd4bd6d4-bd14-4fdc-9aff-bd41a594f82e | 2019-11-01 |
| 2019-11-01 00:04:51 | purchase | 1004856 | 2232732093077520640 | electronics.smartphone | samsung | 128.42 | 562958505 | 0f039697-fedc-40fa-8830-39c1a024351d | 2019-11-01 |
| 2019-11-01 00:05:54 | cart | 1002542 | 2232732093077520640 | electronics.smartphone | apple | 486.80 | 549256216 | dcbdc6e4-cd49-4ee8-95c5-e85f3c618fa1 | 2019-11-01 |
| 2019-11-01 00:06:33 | purchase | 1801881 | 2232732099754852864 | appliances.personal.massager | samsung | 488.80 | 557746614 | 4d76d6d3-fff5-4880-8327-e9e57b618e0e | 2019-11-01 |
| 2019-11-01 00:06:34 | purchase | 5800823 | 2232732082390433792 | electronics.audio.subwoofer | nakamichi | 123.56 | 514166940 | 8ef5214a-86ad-4d0b-8df3-4280dd411b47 | 2019-11-01 |
| 2019-11-01 00:06:38 | cart | 1004856 | 2232732093077520640 | electronics.smartphone | samsung | 128.42 | 513645631 | 61ceaf50-820a-4858-9a68-bab804d47a22 | 2019-11-01 |

Ideas :

- Find time , hour , day and date of each event time .
- Split category code into further / broad category codes . Example is
  electronics.smartphone could be split into electronics and smartphone
  which are broader category of the category code

Thats all i have for now .

### Event times

<details class="code-fold">
<summary>Code</summary>

``` r
events_data <- events_data %>%
  mutate(event_hour = hour(event_time) ,
         event_day = wday(event_time,label = T,abbr = F),
         event_time_hms = format(event_time ,"%H:%M:%S"))
```

</details>

Above code takes too long to run . So instead of splitting all
individual category codes of over 7 million records , lets find our
distinct category codes , split those then left join the results with
our dataset .

<details class="code-fold">
<summary>Code</summary>

``` r
category_codes <- events_data %>%
  distinct(category_code)

category_codes <- category_codes %>%
  rowwise() %>%
  mutate(cat_splits = str_split(category_code,"\\."),
         length_split = length(cat_splits))

category_codes <- category_codes %>%
  separate(category_code,into = paste0("cat_code_",1:max(category_codes$length_split)),sep = "\\.",remove = F) %>%
  select(-contains("split"))

category_codes %>%
  head(n=10) %>% kable()
```

</details>

| category_code | cat_code_1 | cat_code_2 | cat_code_3 | cat_code_4 |
|:---|:---|:---|:---|:---|
| electronics.smartphone | electronics | smartphone | NA | NA |
| NA | NA | NA | NA | NA |
| appliances.personal.massager | appliances | personal | massager | NA |
| electronics.audio.subwoofer | electronics | audio | subwoofer | NA |
| construction.tools.welding | construction | tools | welding | NA |
| sport.bicycle | sport | bicycle | NA | NA |
| appliances.kitchen.refrigerators | appliances | kitchen | refrigerators | NA |
| computers.peripherals.printer | computers | peripherals | printer | NA |
| appliances.kitchen.microwave | appliances | kitchen | microwave | NA |
| computers.desktop | computers | desktop | NA | NA |

Now we merge these to our events dataset

<details class="code-fold">
<summary>Code</summary>

``` r
events_data <- events_data %>%
  left_join(y=category_codes)

events_data %>%
  head() %>% kable()
```

</details>

| event_time | event_type | product_id | category_id | category_code | brand | price | user_id | user_session | event_date | event_hour | event_day | event_time_hms | cat_code_1 | cat_code_2 | cat_code_3 | cat_code_4 |
|:---|:---|:---|:---|:---|:---|---:|:---|:---|:---|---:|:---|:---|:---|:---|:---|:---|
| 2019-11-01 00:00:14 | cart | 1005014 | 2232732093077520640 | electronics.smartphone | samsung | 503.09 | 533326659 | 6b928be2-2bce-4640-8296-0efdf2fda22a | 2019-11-01 | 0 | Friday | 00:00:14 | electronics | smartphone | NA | NA |
| 2019-11-01 00:00:41 | purchase | 13200605 | NA | NA | NA | 566.30 | 559368633 | d6034fa2-41fb-4ac0-9051-55ea9fc9147a | 2019-11-01 | 0 | Friday | 00:00:41 | NA | NA | NA | NA |
| 2019-11-01 00:01:04 | purchase | 1005161 | 2232732093077520640 | electronics.smartphone | xiaomi | 211.92 | 513351129 | e6b7ce9b-1938-4e20-976c-8b4163aea11d | 2019-11-01 | 0 | Friday | 00:01:04 | electronics | smartphone | NA | NA |
| 2019-11-01 00:03:24 | cart | 1801881 | 2232732099754852864 | appliances.personal.massager | samsung | 488.80 | 557746614 | 4d76d6d3-fff5-4880-8327-e9e57b618e0e | 2019-11-01 | 0 | Friday | 00:03:24 | appliances | personal | massager | NA |
| 2019-11-01 00:03:39 | cart | 1005115 | 2232732093077520640 | electronics.smartphone | apple | 949.47 | 565865924 | fd4bd6d4-bd14-4fdc-9aff-bd41a594f82e | 2019-11-01 | 0 | Friday | 00:03:39 | electronics | smartphone | NA | NA |
| 2019-11-01 00:04:51 | purchase | 1004856 | 2232732093077520640 | electronics.smartphone | samsung | 128.42 | 562958505 | 0f039697-fedc-40fa-8830-39c1a024351d | 2019-11-01 | 0 | Friday | 00:04:51 | electronics | smartphone | NA | NA |

## Save our data model

- 

<details class="code-fold">
<summary>Code</summary>

``` r
# arrow::write_parquet(events_data , "data/events_data_model.parquet")
```

</details>
