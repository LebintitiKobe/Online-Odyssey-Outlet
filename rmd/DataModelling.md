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
  - [Dates](#dates)
  - [Price](#price)
  - [Products](#products)
  - [Augment our data](#augment-our-data)
  - [Save the data](#save-the-data)

# Load Libraries

``` r
options(scipen = 999)
require(tidyverse)
require(here)
require(skimr)
require(ggthemes)
require(kableExtra)
require(arrow)

gc()
```

              used (Mb) gc trigger  (Mb) max used  (Mb)
    Ncells 1401921 74.9    2443152 130.5  2443152 130.5
    Vcells 2399145 18.4    8388608  64.0  5010655  38.3

# Load Data

``` r
nov_data <- read_csv(here("data/november.csv.gz"))
dec_data <- read_csv(here("data/december.csv.gz"))
```

Merge november and december data since the have the same data schema
then view the data

``` r
events_data <- bind_rows(nov_data, dec_data) %>%
  janitor::clean_names()

rm(nov_data)
rm(dec_data)
gc()
```

                used  (Mb) gc trigger   (Mb)  max used   (Mb)
    Ncells   7670356 409.7   13541342  723.2   9650509  515.4
    Vcells 107458518 819.9  190092678 1450.3 170943329 1304.2

## Clean up character data

- Convert them to the same casing
- Remove whitespaces

``` r
events_data <- events_data %>%
  mutate(across(where(is.character), ~ trimws(.) %>% str_to_lower())) %>%
  mutate(across(contains("id"), ~ as.character(.)))
```

## Convert date column into their correct format

- Convert date in character format to date format

``` r
events_data <- events_data %>%
  mutate(across(contains("time"), ~ as_datetime(.)))
```

## Validate the data

Validate the data to make sure it do not violate business logic :

### Online Stores Business logic :

User Columns :

- Each row of our data should have a session_id

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

> [!NOTE]
>
> In this case if a product has multiple brands , category code and
> category ID , we assign the most common brand , category code and ID
> in order to not have multiple values for brand and category columns .
>
> A better way to do things here is to actually assign the most recent
> brand , category code and ID to products with multiple values in these
> columns . This makes sense in online store case , so store manager can
> know which other past values were assign to the most recent , hence
> use value in those columns .

## User Columns

``` r
events_data %>%
  select(contains("user")) %>%
  skim()
```

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

``` r
events_data %>%
  filter(is.na(user_session)) %>%
  select(event_type, user_id, price, user_session) %>%
  print(n = 50) %>%
  kable()
```

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
  but it is possible , maybe the user lost connection and when they got
  reconnected they were assigned a new session id

- For now we keep all these , in order to get true conversion of all out
  items

## Events types

``` r
events_data %>%
  janitor::tabyl(event_type) %>%
  kable()
```

| event_type |       n |   percent |
|:-----------|--------:|----------:|
| cart       | 5276372 | 0.7502173 |
| purchase   | 1756753 | 0.2497827 |

- 75% of all our events are cart events , which is to be expected in an
  online store system . People can add products to cart only to change
  their mind , or only to buy only those they can afford , mater fact
  not all items in cart usually get paid for .

## Dates

``` r
events_data %>%
  select(contains("time")) %>%
  skim()
```

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

> [!NOTE]
>
> - But if we had to find a list of all business days from one date to
>   another , we use the {bizdays} and {timeDate} r package .
> - bizdays::bizseq(from,to,“weekends”,holidays) , to find all business
>   days besides weekends but this includes holidays .
> - For holidays , we use timeDate::holidays(from , to , cal = “US”) .
> - Then setdiff(bizdays_no_weekends,holidays) , to find all business
>   days not in holidays .

``` r
events_data <- events_data %>%
  mutate(event_date = as_date(event_time))

events_data %>%
  group_by(event_date) %>%
  count() %>%
  filter(n < 1) %>%
  kable()
```

| event_date |   n |
|:-----------|----:|

- No results got returned meaning there is no working date without a
  record / event , therefore we can be a little extra confident on our
  data

### Check the number of events distribution for each day of our event times

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

![](DataModelling_files/figure-commonmark/unnamed-chunk-11-1.png)

- Events are the lowest at the start of november , maybe be the time
  when the start-up launched (It actually the time the start up
  launched) .
- Events are the highest mid-november , maybe an event was held around
  this time .
- Then after we see a steady rise in events overtime , maybe correspond
  to normal business growth .

Enquire about these to figure out what was happening around these times
. Remember we need to adjust for any artificial boosting of events/sales
because these can bias products that were advertised as part of boosting
campaigns

## Price

``` r
events_data %>%
  select(price) %>%
  skim()
```

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

- We have a price range of \\0 - \\2600 and most products are prices
  below \$600 . More than 75% of the prices are below \$400

### Investigate low price products

``` r
events_data %>%
  distinct(product_id, .keep_all = T) %>%
  filter(price <= 0) %>%
  distinct(product_id, .keep_all = T)
```

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

    # A tibble: 0 × 3
    # ℹ 3 variables: product_id <chr>, n_event_type <int>, n_brands <int>

- We have around 1400 products with prices of zero , all of them do not
  have brands codes and they dont have any purchase event . These may
  products that we dropped from the catalog by some brand which was not
  selling well .
- Here we can drop the products since it wont be possible to calculate
  metrics like Revenue , Conversion , e.t.c

``` r
events_data <- events_data %>%
  filter(!product_id %in% zero_price_ids)
```

### Investigate high priced products

``` r
events_data %>%
  select(-contains("event"), -contains("user")) %>%
  filter(price > 2000) %>%
  janitor::tabyl(category_code) %>%
  arrange(desc(percent)) %>%
  kable()
```

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
  too suspicious here .

## Products

Check if each product_id is associated with one brand and category

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
  distinct(product_id, category_code, category_id, brand, .keep_all = T) %>%
  print(n = 100)
```

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
    # ℹ 39,289 more rows
    # ℹ 4 more variables: price <dbl>, user_id <chr>, user_session <chr>,
    #   event_date <date>

### If a product has two distinct brands / categories but one is NULL , replace NULL with the other brand / category

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
    brand_new = ifelse(
      length(brands) == 1 & length(brands[1]) > 0 & is.na(brand),
      brands[1],
      brand
    ),
    category_id_new = ifelse(
      length(cat_ids) == 1 & length(cat_ids[1]) > 0 & is.na(category_id),
      cat_ids[1],
      category_id
    ),
    category_code_new = ifelse(
      length(cat_codes) == 1 & length(cat_codes[1]) > 0 & is.na(category_code),
      cat_codes[1],
      category_code
    )
  ) %>%
  ungroup()

events_data <- left_join(events_data, merge_data) %>%
  mutate(
    brand = brand_new,
    category_id = category_id_new,
    category_code = category_code_new
  ) %>%
  select(-ends_with("new"), -brands, -cat_ids, -cat_codes)

events_data %>%
  filter(product_id %in% dup_ids) %>%
  arrange(product_id) %>%
  distinct(product_id, category_code, category_id, brand, .keep_all = T) %>%
  print(n = 100)
```

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
    # ℹ 39,063 more rows
    # ℹ 4 more variables: price <dbl>, user_id <chr>, user_session <chr>,
    #   event_date <date>

### Check Top Brands and Categories

``` r
events_data %>%
  janitor::tabyl(brand) %>%
  as_tibble() %>%
  arrange(desc(n)) %>%
  slice_head(n = 20) %>%
  kable(caption = "Top Brands")
```

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

``` r
events_data %>%
  janitor::tabyl(category_code) %>%
  as_tibble() %>%
  arrange(desc(n)) %>%
  slice_head(n = 20) %>%
  kable(caption = "Top Category Codes")
```

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

``` r
events_data %>%
  janitor::tabyl(category_id) %>%
  as_tibble() %>%
  arrange(desc(n)) %>%
  slice_head(n = 20) %>%
  kable(caption = "Top Category Ids")
```

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

``` r
top_brands <- events_data %>%
  count(brand) %>%
  arrange(desc(n)) %>%
  select(brand)

brand_cat_data <- events_data %>%
  count(brand, category_code) %>%
  arrange(desc(n)) %>%
  left_join(x = top_brands) %>%
  group_by(brand) %>%
  left_join(x = top_brands)

events_data %>%
  count(brand, category_code) %>%
  arrange(desc(n)) %>%
  left_join(x = top_brands) %>%
  group_by(brand) %>%
  slice_max(order_by = n, n = 3) %>%
  left_join(x = top_brands) %>%
  print(n = 100)
```

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

``` r
events_data %>%
  count(brand, category_code) %>%
  arrange(desc(n)) %>%
  left_join(x = top_brands) %>%
  group_by(brand) %>%
  slice_max(order_by = n, n = 3) %>%
  left_join(x = top_brands) %>%
  group_by(brand) %>%
  mutate(cats = list(category_code)) %>%
  rowwise() %>%
  mutate(
    cats = na.omit(cats) %>% list(),
    category_code_new = ifelse(
      cats[1] == "construction.tools.light" &
        cats[2] == "electronics.smartphone" &
        category_code == "construction.tools.light" &
        length(cats) > 1,
      cats[2],
      category_code
    )
  ) %>%
  select(-n, -cats) %>%
  print(n = 100)
```

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

``` r
brand_merge <- brand_cat_data %>%
  group_by(brand) %>%
  mutate(cats = list(category_code)) %>%
  rowwise() %>%
  mutate(
    cats = na.omit(cats) %>% list(),
    category_code_new = ifelse(
      cats[1] == "construction.tools.light" &
        cats[2] == "electronics.smartphone" &
        category_code == "construction.tools.light" &
        length(cats) > 1,
      cats[2],
      category_code
    )
  ) %>%
  select(-n, -cats)

events_data <- events_data %>%
  left_join(y = brand_merge)

rm(merge_data)
rm(dup_data)
rm(dup_ids)
```

``` r
length(events_data %>% pull(category_code) %>% na.omit())
```

    [1] 5801004

``` r
length(events_data %>% pull(category_code_new) %>% na.omit())
```

    [1] 5801004

Solutions seems to have worked just fine .

``` r
events_data <- events_data %>%
  mutate(category_code = category_code_new) %>%
  select(-ends_with("new"))
```

``` r
events_data %>%
  count(brand, category_code) %>%
  arrange(desc(n)) %>%
  left_join(x = top_brands) %>%
  na.omit() %>%
  print(n = 100)
```

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
    # ℹ 3,979 more rows

The category codes of top brand look fine , there are no suspicious
categories that need to be investigated .

``` r
events_data %>%
  count(brand, category_code, category_id) %>%
  arrange(desc(n)) %>%
  left_join(x = top_brands) %>%
  na.omit() %>%
  print(n = 100)
```

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
    # ℹ 5,284 more rows

Here , we see that there a unique brand and category combinations with
multiple category id .

Now that brands and categories are investigated , lets fix the issue
where same product has multiple categories and brands

- Lets assign top categories and brands to products products with
  multiple brand and categories , because ideally we should have one
  category and brand per product .

### Brands

Each product should be associated with one brand , category and category
ID

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
  distinct(product_id, brand, category_code, category_id, .keep_all = T) %>%
  select(product_id, brand, category_code, category_id)
```

    # A tibble: 39,014 × 4
       product_id brand      category_code               category_id        
       <chr>      <chr>      <chr>                       <chr>              
     1 100000153  respect    apparel.shoes               2053013565782098944
     2 100000153  respect    apparel.shoes               2232732098706276864
     3 100000154  respect    apparel.shoes               2053013565782098944
     4 100000154  respect    apparel.shoes               2053013556521075200
     5 100000179  parkmaster auto.accessories.parktronic 2053013560623104512
     6 100000179  parkmaster accessories.umbrella        2232732109494026752
     7 100000196  milavitsa  apparel.underwear           2070005009382113792
     8 100000196  milavitsa  electronics.telephone       2053013555573162240
     9 100000208  milavitsa  apparel.underwear           2070005009382113792
    10 100000208  milavitsa  electronics.telephone       2053013555573162240
    # ℹ 39,004 more rows

Here we see that product ids have multiple brands , category code and
IDS , lets fix this by assigning the most common values to these product
IDs

``` r
merge <- events_data %>%
  count(product_id, brand, category_code, category_id) %>%
  arrange(desc(n)) %>%
  filter(!is.na(product_id)) %>%
  group_by(product_id) %>%
  slice_head(n = 1) %>%
  rename(
    "brand_new" = brand,
    "category_code_new" = category_code,
    "category_id_new" = category_id
  ) %>%
  arrange(desc(n))

events_data <- left_join(events_data, merge) %>%
  mutate(
    brand = brand_new,
    category_code = category_code_new,
    category_id = category_id_new
  ) %>%
  select(-ends_with("new"), -n)

events_data %>%
  filter(product_id %in% dup_ids) %>%
  arrange(product_id) %>%
  distinct(product_id, brand, category_code, category_id, .keep_all = T) %>%
  select(product_id, brand, category_code, category_id)
```

    # A tibble: 19,488 × 4
       product_id brand         category_code               category_id        
       <chr>      <chr>         <chr>                       <chr>              
     1 100000153  respect       apparel.shoes               2232732098706276864
     2 100000154  respect       apparel.shoes               2053013565782098944
     3 100000179  parkmaster    auto.accessories.parktronic 2053013560623104512
     4 100000196  milavitsa     apparel.underwear           2070005009382113792
     5 100000208  milavitsa     apparel.underwear           2070005009382113792
     6 100000210  jvc           electronics.video.tv        2053013554415534336
     7 100000244  graffitisound electronics.audio.subwoofer 2232732082390433792
     8 100000263  jvc           electronics.audio.subwoofer 2053013553945772544
     9 100000273  parkmaster    auto.accessories.parktronic 2053013560623104512
    10 100000324  continent     sport.ski                   2053013551907340544
    # ℹ 19,478 more rows

``` r
events_data %>%
  group_by(product_id) %>%
  summarise(
    n_brands = n_distinct(brand),
    n_cat_id = n_distinct(category_id),
    n_cat_code = n_distinct(category_code)
  ) %>%
  filter(n_brands > 1 | n_cat_id > 1 | n_cat_code > 1)
```

    # A tibble: 0 × 4
    # ℹ 4 variables: product_id <chr>, n_brands <int>, n_cat_id <int>,
    #   n_cat_code <int>

Now we see that the product IDs now have one brand , category code and
id assigned to them .

### Category Code and ID

In online store databases , category code should be assigned to only one
category ID .

``` r
cat_data <- events_data %>%
  count(category_code, category_id) %>%
  arrange(category_code, desc(n))
cat_data
```

    # A tibble: 527 × 3
       category_code   category_id             n
       <chr>           <chr>               <int>
     1 accessories.bag 2232732108453839616  3697
     2 accessories.bag 2053013558945382912  3089
     3 accessories.bag 2053013566209917952   965
     4 accessories.bag 2232732082935693312   937
     5 accessories.bag 2053013558668559104   774
     6 accessories.bag 2232732113503781888   755
     7 accessories.bag 2139150089359196160   320
     8 accessories.bag 2232732097330544896   291
     9 accessories.bag 2055156924407612160   263
    10 accessories.bag 2232732085997535232   148
    # ℹ 517 more rows

``` r
merge <- cat_data %>%
  group_by(category_code) %>%
  slice_head(n = 1) %>%
  ungroup() %>%
  rename("category_id_new" = category_id)
merge
```

    # A tibble: 130 × 3
       category_code        category_id_new         n
       <chr>                <chr>               <int>
     1 accessories.bag      2232732108453839616  3697
     2 accessories.umbrella 2232732109494026752   160
     3 accessories.wallet   2232732097397654016 10397
     4 apparel.belt         2232732097473151488    22
     5 apparel.costume      2232732108923601664  3156
     6 apparel.dress        2146660887002349824   171
     7 apparel.glove        2159535995811266816    41
     8 apparel.jeans        2053013552469377280   321
     9 apparel.jumper       2166064855264526848   145
    10 apparel.scarf        2232732104343421440  7429
    # ℹ 120 more rows

``` r
cat_data <- cat_data %>%
  left_join(y = merge %>% select(-n)) %>%
  select(-n)
cat_data
```

    # A tibble: 527 × 3
       category_code   category_id         category_id_new    
       <chr>           <chr>               <chr>              
     1 accessories.bag 2232732108453839616 2232732108453839616
     2 accessories.bag 2053013558945382912 2232732108453839616
     3 accessories.bag 2053013566209917952 2232732108453839616
     4 accessories.bag 2232732082935693312 2232732108453839616
     5 accessories.bag 2053013558668559104 2232732108453839616
     6 accessories.bag 2232732113503781888 2232732108453839616
     7 accessories.bag 2139150089359196160 2232732108453839616
     8 accessories.bag 2232732097330544896 2232732108453839616
     9 accessories.bag 2055156924407612160 2232732108453839616
    10 accessories.bag 2232732085997535232 2232732108453839616
    # ℹ 517 more rows

Now each category_code has one ID assigned to it . Now lets merge this
with event data in order to fix category codes having multiple category
ids issue .

``` r
events_data %>%
  count(category_code, category_id) %>%
  arrange(category_code, desc(n))
```

    # A tibble: 527 × 3
       category_code   category_id             n
       <chr>           <chr>               <int>
     1 accessories.bag 2232732108453839616  3697
     2 accessories.bag 2053013558945382912  3089
     3 accessories.bag 2053013566209917952   965
     4 accessories.bag 2232732082935693312   937
     5 accessories.bag 2053013558668559104   774
     6 accessories.bag 2232732113503781888   755
     7 accessories.bag 2139150089359196160   320
     8 accessories.bag 2232732097330544896   291
     9 accessories.bag 2055156924407612160   263
    10 accessories.bag 2232732085997535232   148
    # ℹ 517 more rows

``` r
events_data %>%
  left_join(y = cat_data) %>%
  count(category_code, category_id_new) %>%
  arrange(category_code, desc(n))
```

    # A tibble: 130 × 3
       category_code        category_id_new         n
       <chr>                <chr>               <int>
     1 accessories.bag      2232732108453839616 11553
     2 accessories.umbrella 2232732109494026752   351
     3 accessories.wallet   2232732097397654016 11033
     4 apparel.belt         2232732097473151488    22
     5 apparel.costume      2232732108923601664  8091
     6 apparel.dress        2146660887002349824   171
     7 apparel.glove        2159535995811266816    73
     8 apparel.jeans        2053013552469377280   830
     9 apparel.jumper       2166064855264526848   145
    10 apparel.scarf        2232732104343421440 10931
    # ℹ 120 more rows

Our proposed solution will work as intended

``` r
events_data <- events_data %>%
  left_join(y = cat_data)

events_data %>% select(category_id) %>% na.omit() %>% nrow()
```

    [1] 5801004

``` r
events_data %>% select(category_id_new) %>% na.omit() %>% nrow()
```

    [1] 5801004

``` r
rm(cat_data)
rm(merge)
rm(dup_data)
rm(dup_ids)
gc()
```

                used  (Mb) gc trigger   (Mb)  max used   (Mb)
    Ncells   6381854 340.9   13541342  723.2  13541342  723.2
    Vcells 111822559 853.2  337813780 2577.4 337725190 2576.7

``` r
events_data <- events_data %>%
  mutate(category_id = category_id_new) %>%
  select(-ends_with("new"))
events_data
```

    # A tibble: 6,957,135 × 10
       event_time          event_type product_id category_id     category_code brand
       <dttm>              <chr>      <chr>      <chr>           <chr>         <chr>
     1 2019-11-01 00:00:14 cart       1005014    22327320930775… electronics.… sams…
     2 2019-11-01 00:00:41 purchase   13200605   <NA>            <NA>          <NA> 
     3 2019-11-01 00:01:04 purchase   1005161    22327320930775… electronics.… xiao…
     4 2019-11-01 00:03:24 cart       1801881    22327320997548… appliances.p… sams…
     5 2019-11-01 00:03:39 cart       1005115    22327320930775… electronics.… apple
     6 2019-11-01 00:04:51 purchase   1004856    22327320930775… electronics.… sams…
     7 2019-11-01 00:05:54 cart       1002542    22327320930775… electronics.… apple
     8 2019-11-01 00:06:33 purchase   1801881    22327320997548… appliances.p… sams…
     9 2019-11-01 00:06:34 purchase   5800823    22327320823904… electronics.… naka…
    10 2019-11-01 00:06:38 cart       1004856    22327320930775… electronics.… sams…
    # ℹ 6,957,125 more rows
    # ℹ 4 more variables: price <dbl>, user_id <chr>, user_session <chr>,
    #   event_date <date>

Verify that each product has one brand , category code and ID .

``` r
events_data %>%
  group_by(product_id) %>%
  summarise(
    n_brands = n_distinct(brand),
    n_cat_id = n_distinct(category_id),
    n_cat_code = n_distinct(category_code)
  ) %>%
  filter(n_brands > 1 | n_cat_id > 1 | n_cat_code > 1)
```

    # A tibble: 0 × 4
    # ℹ 4 variables: product_id <chr>, n_brands <int>, n_cat_id <int>,
    #   n_cat_code <int>

Now that is our basic data cleaning and data validation done .

## Augment our data

Here we derived extra information from existing columns which we may be
useful for us when doing actual analysis . Or we find data to add to
current data that we think will be helpful to us when doing our analysis
.

> [!NOTE]
>
> Ideas :
>
> - Finding date , day of week , hour of day and Time without date of
>   our event time column .  
> - Category code values are written in such a way that we can break
>   them down into multiple categories .
>
> These info may be useful for us when doing the actual analyses , in
> this case the second idea will definitely be useful for us .

### Event time

``` r
events_data <- events_data %>%
  mutate(
    event_hour = hour(event_time),
    event_day = wday(event_time, label = T, abbr = F),
    event_time_hms = format(event_time, "%H:%M:%S")
  )

events_data
```

    # A tibble: 6,957,135 × 13
       event_time          event_type product_id category_id     category_code brand
       <dttm>              <chr>      <chr>      <chr>           <chr>         <chr>
     1 2019-11-01 00:00:14 cart       1005014    22327320930775… electronics.… sams…
     2 2019-11-01 00:00:41 purchase   13200605   <NA>            <NA>          <NA> 
     3 2019-11-01 00:01:04 purchase   1005161    22327320930775… electronics.… xiao…
     4 2019-11-01 00:03:24 cart       1801881    22327320997548… appliances.p… sams…
     5 2019-11-01 00:03:39 cart       1005115    22327320930775… electronics.… apple
     6 2019-11-01 00:04:51 purchase   1004856    22327320930775… electronics.… sams…
     7 2019-11-01 00:05:54 cart       1002542    22327320930775… electronics.… apple
     8 2019-11-01 00:06:33 purchase   1801881    22327320997548… appliances.p… sams…
     9 2019-11-01 00:06:34 purchase   5800823    22327320823904… electronics.… naka…
    10 2019-11-01 00:06:38 cart       1004856    22327320930775… electronics.… sams…
    # ℹ 6,957,125 more rows
    # ℹ 7 more variables: price <dbl>, user_id <chr>, user_session <chr>,
    #   event_date <date>, event_hour <int>, event_day <ord>, event_time_hms <chr>

### Category Code

``` r
# events_data <- events_data %>%
#   rowwise() %>%
#   mutate(cat_splits = str_split(category_code, "\\.")) %>%
#   ungroup() %>%
#   mutate(length_split = length(cat_splits))
```

> [!NOTE]
>
> The above code is example of bad code . We applying str_split() to all
> records in our data . Meaning it will be applied to all 7m+ records ,
> we will be computationally intensive .
>
> The smart of way of doing things here is first extracting all unique
> category codes , then applying str_split() to the subset then merging
> the results back into the full dataset .

``` r
cat_code_data <- events_data %>% 
  distinct(category_code) %>%
  na.omit() %>%
  rowwise() %>%
  mutate(cat_split = str_split(category_code, "\\."),
  length_split = length(cat_split))
```

``` r
cat_code_data <- cat_code_data %>%
  ungroup() %>%
  separate(col = category_code , 
  into = paste0("cat_code_",1:max(cat_code_data$length_split)),
  sep = "\\.",
  remove = F) %>%
  select(-contains("split"))

cat_code_data 
```

    # A tibble: 129 × 5
       category_code                    cat_code_1  cat_code_2 cat_code_3 cat_code_4
       <chr>                            <chr>       <chr>      <chr>      <chr>     
     1 electronics.smartphone           electronics smartphone <NA>       <NA>      
     2 appliances.personal.massager     appliances  personal   massager   <NA>      
     3 electronics.audio.subwoofer      electronics audio      subwoofer  <NA>      
     4 construction.tools.welding       constructi… tools      welding    <NA>      
     5 sport.bicycle                    sport       bicycle    <NA>       <NA>      
     6 appliances.kitchen.refrigerators appliances  kitchen    refrigera… <NA>      
     7 computers.peripherals.printer    computers   periphera… printer    <NA>      
     8 appliances.kitchen.microwave     appliances  kitchen    microwave  <NA>      
     9 computers.desktop                computers   desktop    <NA>       <NA>      
    10 construction.tools.generator     constructi… tools      generator  <NA>      
    # ℹ 119 more rows

Now we merge the above with our data .

``` r
events_data <- events_data %>%
  left_join(y=cat_code_data)

events_data
```

    # A tibble: 6,957,135 × 17
       event_time          event_type product_id category_id     category_code brand
       <dttm>              <chr>      <chr>      <chr>           <chr>         <chr>
     1 2019-11-01 00:00:14 cart       1005014    22327320930775… electronics.… sams…
     2 2019-11-01 00:00:41 purchase   13200605   <NA>            <NA>          <NA> 
     3 2019-11-01 00:01:04 purchase   1005161    22327320930775… electronics.… xiao…
     4 2019-11-01 00:03:24 cart       1801881    22327320997548… appliances.p… sams…
     5 2019-11-01 00:03:39 cart       1005115    22327320930775… electronics.… apple
     6 2019-11-01 00:04:51 purchase   1004856    22327320930775… electronics.… sams…
     7 2019-11-01 00:05:54 cart       1002542    22327320930775… electronics.… apple
     8 2019-11-01 00:06:33 purchase   1801881    22327320997548… appliances.p… sams…
     9 2019-11-01 00:06:34 purchase   5800823    22327320823904… electronics.… naka…
    10 2019-11-01 00:06:38 cart       1004856    22327320930775… electronics.… sams…
    # ℹ 6,957,125 more rows
    # ℹ 11 more variables: price <dbl>, user_id <chr>, user_session <chr>,
    #   event_date <date>, event_hour <int>, event_day <ord>, event_time_hms <chr>,
    #   cat_code_1 <chr>, cat_code_2 <chr>, cat_code_3 <chr>, cat_code_4 <chr>

``` r
rm(cat_code_data)
gc()
```

                used   (Mb) gc trigger   (Mb)  max used   (Mb)
    Ncells   6470663  345.6   13541342  723.2  13541342  723.2
    Vcells 146786419 1119.9  337813780 2577.4 337725190 2576.7

That is our basic data cleaning done .

## Save the data

> [!NOTE]
>
> For large datasets like this , it often not advised to save the data
> as a .csv . There are other useful format designed for large data .
> Example of this is the .parquet format . This format compresses the
> data , also saves the data types of each column , so when we load the
> data again in the future we dont start by converting data to correct
> formats first .
>
> Popular r package for this format is {arrow} r package . It also
> allows us to use tidyverse / dplyr syntax when we read in the data .
>
> Use arrow::read_parquet() to read in our .parquet data . Use
> arrow::write_parquet() to save the data in .parquet format .

``` r
# write_parquet(events_data , 
# "data/events_data_model.gz.parquet",
# compression = "gzip", 
# compression_level = 5)
```
