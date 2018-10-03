hw03-gapminder
================

Using dplyr, ggplot2 to explore data
====================================

Initialize the data
-------------------

-   Load the gapminder and tidyverse libraries:

``` r
suppressPackageStartupMessages(library(tidyverse))
suppressPackageStartupMessages(library(gapminder))
suppressPackageStartupMessages(library(knitr))
```

-   Take a snapshot of the data to *sanity check* that the data and variables appear as we expect:

``` r
(head(gapminder))
```

    ## # A tibble: 6 x 6
    ##   country     continent  year lifeExp      pop gdpPercap
    ##   <fct>       <fct>     <int>   <dbl>    <int>     <dbl>
    ## 1 Afghanistan Asia       1952    28.8  8425333      779.
    ## 2 Afghanistan Asia       1957    30.3  9240934      821.
    ## 3 Afghanistan Asia       1962    32.0 10267083      853.
    ## 4 Afghanistan Asia       1967    34.0 11537966      836.
    ## 5 Afghanistan Asia       1972    36.1 13079460      740.
    ## 6 Afghanistan Asia       1977    38.4 14880372      786.

*Observations:* *Using the suppress messages command from cm008 blocks the library loading message from outputting.*

Task 1
------

*Get the max and min GDP per capita for all continents.*

1.  Filter the data by continent
2.  Select the GDP per capita
3.  Find the minimum and maximum values by: (i) analyzing the range; (ii) querying the min and max directly
4.  Present the data in a table
5.  Visualize the data in a graph
6.  Interpret the data \*Why would we want to look at the max and min GDP per capita for each continent?

<!-- -->

1.  Imagine we want to look at the distribution of min values and max values respectively to see if they are normal distributions.
2.  Perhaps we want to see the average min GDP and the average max GDP. Or find the countries with the biggest gap between min and max values, respectively, Or the widest range between min and max values. *Reflections:* *Finding the min and max directly is useful if you need to assign the values to individual variables, but using the range() command allows you to see both values faster with a single command. Both methods are used here because their redundancy permits error checking.* *I used Stack Overflow for some ideas* *Other resources: <https://www.math.ucla.edu/~anderson/rw1001/library/base/html/merge.html>; <https://www.sixhat.net/how-to-plot-multpile-data-series-with-ggplot.html>; <https://stackoverflow.com/questions/23635662/editing-legend-text-labels-in-ggplot*> *Might be more realistic to look only at a particular year, using filter()* *Make observations about spread, range, thresholds)* *Used a bar instead of a bar, since this feels like a more natural symbol choice for a upper and lower limit\*

``` r
# Select the necessary data and group to reduce the size; Find the max and min values per continent
min_gdp <- select(gapminder, continent, gdpPercap) %>% 
  group_by(continent) %>% 
    summarize(min_gdp = min(gdpPercap))


max_gdp <- select(gapminder, continent, gdpPercap) %>%
 group_by(continent) %>%
   summarize(max_gdp = max(gdpPercap))

#Merge data into a single table
(min_max_gdp <- merge(min_gdp, max_gdp, by.x = "continent"))
```

    ##   continent    min_gdp   max_gdp
    ## 1    Africa   241.1659  21951.21
    ## 2  Americas  1201.6372  42951.65
    ## 3      Asia   331.0000 113523.13
    ## 4    Europe   973.5332  49357.19
    ## 5   Oceania 10039.5956  34435.37

``` r
kable(min_max_gdp)
```

| continent |    min\_gdp|   max\_gdp|
|:----------|-----------:|----------:|
| Africa    |    241.1659|   21951.21|
| Americas  |   1201.6372|   42951.65|
| Asia      |    331.0000|  113523.13|
| Europe    |    973.5332|   49357.19|
| Oceania   |  10039.5956|   34435.37|

``` r
#Sanity check the value
str(min_max_gdp)
```

    ## 'data.frame':    5 obs. of  3 variables:
    ##  $ continent: Factor w/ 5 levels "Africa","Americas",..: 1 2 3 4 5
    ##  $ min_gdp  : num  241 1202 331 974 10040
    ##  $ max_gdp  : num  21951 42952 113523 49357 34435

``` r
(range(gapminder$gdpPercap))
```

    ## [1]    241.1659 113523.1329

``` r
#Visualize the data in a graph
ggplot(min_max_gdp, aes(x = continent, y = value, color = variable)) +
  geom_point(aes(y = min_gdp, col = "min_gdp"), size=20, shape = "-") +
  geom_point(aes(y = max_gdp, col = "max_gdp"), size=20, shape = "-") +
  #Change y-axis to log-scale given the wide range in values
  scale_y_log10() +
  #Add labels
  labs(title = "Minimum & Maximum GDP per capita, all years",
  x = "Continent", y = "GDP per capita ($)", color = "Legend\n") +
  scale_color_manual(labels = c("Maximum", "Minimum"), values = c("green", "red"))
```

![](hw03-gapminder_files/figure-markdown_github/unnamed-chunk-3-1.png)

``` r
#Another way to interpret the data
# gapminder %>% 
#   select(year, continent, country, gdpPercap) %>% 
#       group_by(continent) %>% 
#         group_by(year) %>% 
#          summarize(min_gdp = min(gdpPercap))
#ggplot(min_max_gdp, aes(gdpPercap)) +
#  facet_wrap( ~ continent, scale = "free_x") +
#  geom_histogram()
```

Task 2
------

*Look at the spread of GDP per capita within the continents.*

The initial approach is similar to Task 1: *1. Filter the data by continent* *2. Select the GDP per capita* Now, let's consider the spread of the data, rather than the upper and lower limits: 3. Find the spread of the data 4. Present the data in a table 5. Visualize the data in a graph 6. Interpret the data (i) Say we want to compare the relative ranges. We could compare just the min and max, but we will get a more complete picture by looking at the distribution. (ii) Perhaps we want to know which continent has the widest spread? Narrowest spread? (iii) Referred to *Reflections:* *What different observations would I make based on this data vs. the previous? What are the adv/disadv of this analysis?* <https://www.dummies.com/programming/r/how-to-check-quantiles-in-r/> *the min and max values determined in Task 1 are also shown in the spread, but the spread also gives us more information* *Could also filter for a specific year or range of years; could also use faceting to display min/max per continent for all years*

``` r
# Select the necessary data and group to reduce the size; Find the spread of values per continent
#Evaluate the data range using statistical commands
gapminder %>% 
  select(continent, gdpPercap) %>% 
  group_by(continent) %>% 
    summarize(Mean=mean(gdpPercap), Median = median(gdpPercap), SD=sd(gdpPercap), Var=var(gdpPercap)) %>% 
     kable()
```

| continent |       Mean|     Median|         SD|        Var|
|:----------|----------:|----------:|----------:|----------:|
| Africa    |   2193.755|   1192.138|   2827.930|    7997187|
| Americas  |   7136.110|   5465.510|   6396.764|   40918591|
| Asia      |   7902.150|   2646.787|  14045.373|  197272506|
| Europe    |  14469.476|  12081.749|   9355.213|   87520020|
| Oceania   |  18621.609|  17983.304|   6358.983|   40436669|

``` r
#Visualize the data in a graph (violin plot)
gapminder %>% 
  select(continent, gdpPercap) %>% 
  group_by(continent) %>% 
    ggplot(aes(continent, gdpPercap)) +
    #Change y-axis to log-scale given the wide range in values
    scale_y_log10() +
    geom_violin(fill = "yellow") +
  #Add labels
  labs(title = "Spread of GDP per capita, all years",
  x = "Continent", y = "GDP per capita ($)")
```

![](hw03-gapminder_files/figure-markdown_github/unnamed-chunk-4-1.png)

``` r
#Visualize the data in a graph (box plot)
gapminder %>% 
  select(continent, gdpPercap) %>% 
  group_by(continent) %>% 
    ggplot(aes(continent, gdpPercap)) +
    #Change y-axis to log-scale given the wide range in values
     scale_y_log10() +
    geom_boxplot(fill = "yellow") +
  #Add labels
  labs(title = "Spread of GDP per capita, all years",
  x = "Continent", y = "GDP per capita ($)")
```

![](hw03-gapminder_files/figure-markdown_github/unnamed-chunk-4-2.png)

Task 3
------

*Compute a trimmed mean of life expectancy for different years. Or a weighted mean, weighting by population. Just try something other than the plain vanilla mean.* \* I will examine the weighted mean (by population) of life expectancy for different years for Canada

1.  Select the year, continent, population and life expectancy
2.  Evaluate the traditional mean, weighted mean by population and trimmed mean (trimming 5 values on either end of the data)
3.  Present the data in a table
4.  Visualize the data in a graph
5.  Interpret the data *Reflections:* *there's way too much data to view it for all years in table format; graph format is better for this* *table format shows the exact values, which is a nice benefit*

``` r
(mean_comp <- gapminder) %>% 
  select(year, continent, lifeExp, pop) %>% 
      group_by(year, continent) %>% 
       summarize(mean_lifeExp = mean(lifeExp), w_mean_lifeExp = weighted.mean(lifeExp, pop), t_mean_lifeExp = mean(lifeExp, trim = 0.3)) %>% 
        kable()
```

|  year| continent |  mean\_lifeExp|  w\_mean\_lifeExp|  t\_mean\_lifeExp|
|-----:|:----------|--------------:|-----------------:|-----------------:|
|  1952| Africa    |       39.13550|          38.79973|          39.15482|
|  1952| Americas  |       53.27984|          60.23599|          53.31764|
|  1952| Asia      |       46.31439|          42.94114|          45.12860|
|  1952| Europe    |       64.40850|          64.90540|          65.54583|
|  1952| Oceania   |       69.25500|          69.17040|          69.25500|
|  1957| Africa    |       41.26635|          40.94031|          41.01205|
|  1957| Americas  |       55.96028|          62.01806|          56.55545|
|  1957| Asia      |       49.31854|          47.28835|          48.39533|
|  1957| Europe    |       66.70307|          66.89364|          67.70667|
|  1957| Oceania   |       70.29500|          70.31693|          70.29500|
|  1962| Africa    |       43.31944|          43.09925|          42.87036|
|  1962| Americas  |       58.39876|          63.43706|          59.32182|
|  1962| Asia      |       51.56322|          46.57369|          50.54327|
|  1962| Europe    |       68.53923|          68.45957|          69.50750|
|  1962| Oceania   |       71.08500|          70.98808|          71.08500|
|  1967| Africa    |       45.33454|          45.17721|          44.92855|
|  1967| Americas  |       60.41092|          64.50630|          61.36200|
|  1967| Asia      |       54.66364|          53.88261|          54.28301|
|  1967| Europe    |       69.73760|          69.54963|          70.47833|
|  1967| Oceania   |       71.31000|          71.17848|          71.31000|
|  1972| Africa    |       47.45094|          47.21229|          47.03082|
|  1972| Americas  |       62.39492|          65.70490|          63.27855|
|  1972| Asia      |       57.31927|          57.52159|          57.72693|
|  1972| Europe    |       70.77503|          70.46884|          71.03717|
|  1972| Oceania   |       71.91000|          71.92273|          71.91000|
|  1977| Africa    |       49.58042|          49.20883|          49.21564|
|  1977| Americas  |       64.39156|          67.60591|          65.43445|
|  1977| Asia      |       59.61056|          59.55648|          60.74836|
|  1977| Europe    |       71.93777|          71.53989|          72.04050|
|  1977| Oceania   |       72.85500|          73.25684|          72.85500|
|  1982| Africa    |       51.59287|          51.01744|          51.10986|
|  1982| Americas  |       66.22884|          69.19264|          67.33682|
|  1982| Asia      |       62.61794|          61.57472|          63.42100|
|  1982| Europe    |       72.80640|          72.56247|          73.13033|
|  1982| Oceania   |       74.29000|          74.58291|          74.29000|
|  1987| Africa    |       53.34479|          52.82479|          52.59827|
|  1987| Americas  |       68.09072|          70.35814|          68.81500|
|  1987| Asia      |       64.85118|          63.53710|          65.93267|
|  1987| Europe    |       73.64217|          73.44717|          74.07408|
|  1987| Oceania   |       75.32000|          75.98107|          75.32000|
|  1992| Africa    |       53.62958|          53.37292|          53.35745|
|  1992| Americas  |       69.56836|          71.72177|          70.03055|
|  1992| Asia      |       66.53721|          65.14874|          67.72627|
|  1992| Europe    |       74.44010|          74.44273|          75.02908|
|  1992| Oceania   |       76.94500|          77.35788|          76.94500|
|  1997| Africa    |       53.59827|          53.28327|          52.57077|
|  1997| Americas  |       71.15048|          73.19154|          71.46118|
|  1997| Asia      |       68.02052|          66.77092|          69.21373|
|  1997| Europe    |       75.50517|          75.70849|          76.09958|
|  1997| Oceania   |       78.19000|          78.61843|          78.19000|
|  2002| Africa    |       53.32523|          53.30314|          51.56086|
|  2002| Americas  |       72.42204|          74.24736|          72.55145|
|  2002| Asia      |       69.23388|          68.13732|          70.45633|
|  2002| Europe    |       76.70060|          77.02232|          77.24142|
|  2002| Oceania   |       79.74000|          80.16006|          79.74000|
|  2007| Africa    |       54.80604|          54.56441|          53.37859|
|  2007| Americas  |       73.60812|          75.35668|          73.69555|
|  2007| Asia      |       70.72848|          69.44386|          71.78233|
|  2007| Europe    |       77.64860|          77.89057|          78.24717|
|  2007| Oceania   |       80.71950|          81.06215|          80.71950|

``` r
#Visualize the data in a graph

w_mean <- gapminder %>%
  group_by(year,continent) %>%
   summarize(w_mean = weighted.mean(lifeExp, pop))

spread_w_mean <- spread(w_mean, key = "year", value = "w_mean")
kable(spread_w_mean)
```

| continent |      1952|      1957|      1962|      1967|      1972|      1977|      1982|      1987|      1992|      1997|      2002|      2007|
|:----------|---------:|---------:|---------:|---------:|---------:|---------:|---------:|---------:|---------:|---------:|---------:|---------:|
| Africa    |  38.79973|  40.94031|  43.09925|  45.17721|  47.21229|  49.20883|  51.01744|  52.82479|  53.37292|  53.28327|  53.30314|  54.56441|
| Americas  |  60.23599|  62.01806|  63.43706|  64.50630|  65.70490|  67.60591|  69.19264|  70.35814|  71.72177|  73.19154|  74.24736|  75.35668|
| Asia      |  42.94114|  47.28835|  46.57369|  53.88261|  57.52159|  59.55648|  61.57472|  63.53710|  65.14874|  66.77092|  68.13732|  69.44386|
| Europe    |  64.90540|  66.89364|  68.45957|  69.54963|  70.46884|  71.53989|  72.56247|  73.44717|  74.44273|  75.70849|  77.02232|  77.89057|
| Oceania   |  69.17040|  70.31693|  70.98808|  71.17848|  71.92273|  73.25684|  74.58291|  75.98107|  77.35788|  78.61843|  80.16006|  81.06215|

``` r
w_mean %>%
  ggplot(aes(year, w_mean)) +
  geom_point(aes(color = continent)) +
  geom_line(method = 'lm', aes(color = continent)) +
  #Add labels
  labs(title = "Weighted Mean Life Expectancy over Time (Weighted by population)",
    x = "Year", y = "Life Expectancy")
```

    ## Warning: Ignoring unknown parameters: method

![](hw03-gapminder_files/figure-markdown_github/unnamed-chunk-5-1.png)

Task 4
------

*How is life expectancy changing over time on different continents?*

1.  Filter the data by continent
2.  Select the GDP per capita
3.  Find the minimum and maximum values by: (i) analyzing the range; (ii) querying the min and max directly
4.  Present the data in a table
5.  Visualize the data in a graph
6.  Interpret the data *Reflections:*
