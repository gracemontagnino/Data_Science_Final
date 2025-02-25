Government Spending on Children
================
Shreya Chowdhary and Grace Montagnino
2020-12-14

-   [Setup](#setup)
-   [General Background](#general-background)
-   [Results and Analysis](#results-and-analysis)
    -   [Examining State Spending
        Patterns](#examining-state-spending-patterns)
    -   [Predicting Median Income through
        Spending](#predicting-median-income-through-spending)
-   [Conclusions and Further
    Questions](#conclusions-and-further-questions)

Setup
=====

    library(dplyr)

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

    library(tidyverse)

    ## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.0 ──

    ## ✓ ggplot2 3.3.2     ✓ purrr   0.3.4
    ## ✓ tibble  3.0.3     ✓ stringr 1.4.0
    ## ✓ tidyr   1.1.2     ✓ forcats 0.5.0
    ## ✓ readr   1.3.1

    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

    library(broom)
    library(readxl)
    library(modelr)

    ## 
    ## Attaching package: 'modelr'

    ## The following object is masked from 'package:broom':
    ## 
    ##     bootstrap

    library(rsample)
    library("gridExtra")

    ## 
    ## Attaching package: 'gridExtra'

    ## The following object is masked from 'package:dplyr':
    ## 
    ##     combine

    library(usmap)
    library(ggplot2)

    # load in datasets
    kids <- readr::read_csv('https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2020/2020-09-15/kids.csv')

    ## Parsed with column specification:
    ## cols(
    ##   state = col_character(),
    ##   variable = col_character(),
    ##   year = col_double(),
    ##   raw = col_double(),
    ##   inf_adj = col_double(),
    ##   inf_adj_perchild = col_double()
    ## )

    df_income <- read_csv('https://raw.githubusercontent.com/rbyirdaw/US-median-household-income/master/median-household-income-by-state-master.csv')

    ## Parsed with column specification:
    ## cols(
    ##   year = col_double(),
    ##   state = col_character(),
    ##   `median income` = col_number(),
    ##   se = col_number(),
    ##   footnote = col_character()
    ## )

General Background
==================

Our central questions were:

-   How does state spending on kids vary state to state and over time?

-   Does government spending in money relate to the economic wellbeing
    of these states, measured in median income?

We were interested in studying these questions as investment in children
is an investment in the future of our society. Recently, government
spending on children (especially on the federal level) has been
[decreasing a
lot](https://www.urban.org/features/shortchanging-future-generations).
This lack of investment in children is [reflected
in](https://www.urban.org/research/publication/kids-share-2019-report-federal-expenditures-children-through-2018-and-future-projections/view/full_report)
the United States’ higher povery rates, lower birth weights, preschoole
enrollment rates, percentages of 15-19 year olds participating in
education, employment, or training, and composite measure of child
well-being, compared to other countries. As federal spending decreases,
states need to scramble to try to figure out how they can invest all the
necessary funds in children to support society’s future as a whole.

To answer these questions, we utilized two datasets: one that details
how each state has spent money on kids over time, and another that
documents median household income in each state overtime. The dataset
with information about spending on kids was formulated in a
collaboration between Brown University and Urban Institute Russell Sage
Foundation Grant, and a Eunice Kennedy Shriver National Institute of
Child Health and Human Development and the National Institutes of Health
grant, to isolate the gaps in class and public spending on child
development. The dataset includes the year, state, raw amount spent on
children (`raw`), inflation adjusted amount spent on children
(`inf_adj`), and inflation adjusted amount spent per child
(`inf_adj_perchild`), as well as a variable (named `variable`)
indicating where the money specifically was spent. The variable options
are: `addCC`, `CTC`, `edservs`, `edsubs`, `fedEITC`, `fedSSI`, `HCD`,
`HeadStartPriv`, `highered`, `lib`, `Medicaid_CHIP`, `other_health`,
`othercashserv`, `parkrec`, `pell`, `PK12ed`, `pubhealth`, `SNAP`,
`socsec`, `stateEITC`, `TANFbasic`, `unemp`, `wcomp`. Because there are
so many variables, we are not going to explain what each of those
spending areas means, but they are all explained here:
<a href="https://jrosen48.github.io/tidykids/articles/tidykids-codebook.html" class="uri">https://jrosen48.github.io/tidykids/articles/tidykids-codebook.html</a>,
and we will explain the variables we are noting the effects of, most of
these are exactly what the names sound like! The second dataset held
household median income in each state from 1984-2014.

Results and Analysis
====================

Examining State Spending Patterns
---------------------------------

The first question we had for our data was: How does state spending on
kids vary state to state and over time?

We started our analysis by first looking at the differences in which
categories states allocate the most money to. We began by looking at a
few categories that seemed especially relevant:

-   `highered`, or state spending on higher education

-   `HeadStartPriv`, or state spending on the Head Start program

-   `PK12ed`, or state spending on K12 education

-   `SNAP`, or state spending on the SNAP program (food stamps)

-   `HCD`, or state spending on housing and community development

-   `socsec`, or state spending on Social Security

-   `CTC`, or state spending on Child Tax Credit

-   `lib`, or state spending on libraries

-   `parkrec`, or state spending on parks and recreation

-   `pubhealth`, or state spending on public health

All of these categories feel like direct reflections of the amount of
money being invested into children.

    kids %>%
      group_by(state, variable) %>%
      filter(variable == c("highered","HeadStartPriv", "PK12ed", "SNAP", "HCD", "socsec", "CTC", "lib", "parkrec", "pubhealth")) %>%
      summarize(spent_pc = sum(inf_adj_perchild)) %>%
      ggplot() +
      geom_col(aes(state, spent_pc, fill = variable), position = "stack") +
      theme_minimal() +
      theme(axis.text.x = element_text(angle = 90)) +
      ggtitle("Money spent by each state per category")

    ## `summarise()` regrouping output by 'state' (override with `.groups` argument)

![](Final_Project_Write_up_files/figure-gfm/state-category-spending-1.png)<!-- -->

Through this graph, we can see that K12 education is the highest
spending category in every state, though there is a lot of variety in
how much each state spends. The second highest category varies across
states, though typically it seems to be either public health or higher
education (though in some states, like Delaware, HCD is also a high
category of spending). In all the states, SNAP, social security, CTC,
HEad Start, and libraries are lower spending categories.

Because different states have different spending budgets both within
spending on children, and across all categories of state expenditures,
we wanted a way to normalize what was being spent on children in
different categories. Therefore, we decided to use a graph of the United
States to visualize what percentage of expenditures that go towards
children were spent on different categories and programs in different
years. The darker the state, the higher percentage of child expenditures
spent on a specific category.

To do this, we made a dataframe filtered for each year we were
interested in, that dictated the totals spent in each state on children
that year. Then we create a second data frame that filters the full kids
dataset for that same specific year, and a spending category. We then
join these two dataframes, and calculate the proportion of spending on
part to whole with a simple ratio calculation.

We did not use a quantitative method of uncertainty in this section
because we were neither predicting, nor creating new data, we were
simply graphically representing what was given in the dataset. That
being said, we would still like to give a qualitative assessment of the
trustworthiness of our results. We have a very high level of confidence
in the “kids” dataset because it was endorsed and created by several
different trustworthy organizations and universities. However, we feel
less confident about the accuracy of the median income dataset because
it does not come with error/uncertainty assessments, and the source of
the data is a bit unclear, as it came from a github repo with minimal
citation of data sources. With the combo of those two datasets we
believe that our results are trustworthy enough to give us a general
idea of trends and relationships, but probably not to give us exact
numerical and statistical results. Below are our results for the
variables we found most interesting.

    # Filter total spending for 1998
    total_year_state_98 <-
    kids %>%
      group_by(state, year) %>%
      summarize(total = sum(inf_adj_perchild)) %>%
      filter(year == "1998")

    ## `summarise()` regrouping output by 'state' (override with `.groups` argument)

    # Determine proportional spending for 1998
    prop_98<-
    kids %>%
      filter(variable == "PK12ed") %>%
      filter(year == "1998") %>%
      group_by(state, year)%>%
      left_join(total_year_state_98, by = c("state", "year")) %>%
      mutate(prop = inf_adj_perchild/total)

    # Plot map for 1998
    p1<- plot_usmap(data = prop_98, values = "prop", color = "red") + 
      scale_fill_continuous(
        low = "white", high = "red", 
        name = "Prop of Total Money Spent", 
        label = scales::comma
      ) + 
      theme(legend.position = "right") + 
      ggtitle("Prop of money for kids spent on K12ed in 1998")

    # Filter total spending for 2005
    total_year_state_05 <-
    kids %>%
      group_by(state, year) %>%
      summarize(total = sum(inf_adj_perchild)) %>%
      filter(year == "2005")

    ## `summarise()` regrouping output by 'state' (override with `.groups` argument)

    # Determine proportional spending for 2005
    prop_05<-
    kids %>%
      filter(variable == "PK12ed") %>%
      filter(year == "2005") %>%
      group_by(state, year)%>%
      left_join(total_year_state_05, by = c("state", "year")) %>%
      mutate(prop = inf_adj_perchild/total)

    # Plot map for 2005
    p2<- plot_usmap(data = prop_05, values = "prop", color = "red") + 
      scale_fill_continuous(
        low = "white", high = "red", 
        name = "Prop of Total Money Spent", 
        label = scales::comma
      ) + 
      theme(legend.position = "right") + 
      ggtitle("Prop of money for kids spent on K12ed in 2005")

    # Filter total spending for 2012
    total_year_state_12 <-
    kids %>%
      group_by(state, year) %>%
      summarize(total = sum(inf_adj_perchild)) %>%
      filter(year == "2012")

    ## `summarise()` regrouping output by 'state' (override with `.groups` argument)

    # Determine proportional spendng for 2012
    prop_12<-
    kids %>%
      filter(variable == "PK12ed") %>%
      filter(year == "2012") %>%
      group_by(state, year)%>%
      left_join(total_year_state_12, by = c("state", "year")) %>%
      mutate(prop = inf_adj_perchild/total)

    # Plot map for 2012
    p3<-plot_usmap(data = prop_12, values = "prop", color = "red") + 
      scale_fill_continuous(
        low = "white", high = "red", 
        name = "Prop of Total Money Spent", label = scales::comma
      ) + 
      theme(legend.position = "right") + 
      ggtitle("Prop of money for kids spent on K12ed in 2012")

    # Filter total spending for 2016
    total_year_state_16 <-
    kids %>%
      group_by(state, year) %>%
      summarize(total = sum(inf_adj_perchild)) %>%
      filter(year == "2016")

    ## `summarise()` regrouping output by 'state' (override with `.groups` argument)

    # Determine proportional spending for 2016
    prop_16<-
    kids %>%
      filter(variable == "PK12ed") %>%
      filter(year == "2016") %>%
      group_by(state, year)%>%
      left_join(total_year_state_16, by = c("state", "year")) %>%
      mutate(prop = inf_adj_perchild/total)

    # Plot map for 2016
    p4<-plot_usmap(data = prop_16, values = "prop", color = "red") + 
      scale_fill_continuous(
        low = "white", high = "red", 
        name = "Prop of Total Money Spent", 
        label = scales::comma
      ) + 
      theme(legend.position = "right") + 
      ggtitle("Prop of money for kids spent on K12ed in 2016")

    # Show plots for K12 spending across the 4 years
    grid.arrange(p1,p2,p3,p4, nrow=2, ncol =2) 

![](Final_Project_Write_up_files/figure-gfm/k12-year-comps-1.png)<!-- -->

Unfortunately, we were not surprised to see that generally, over time
spending on K12 education (K12ed) has decreased in priority for states.
Comparing the 1998 map to the 2016 map, there is no doubt that the
densities have lessened. This fits with the shrinking amount of national
finance spent on public education that we have seen since 1998.
Currently, when we talk about public school education, it is often
riddled with pleads for improvement, but no is followed with little to
no increase in resources. One surprising note here, was that there was a
slight increase in density of proportional spending between 2012 and
2016. However, this is only in \~3 states. Because K12 education is by
far the largest spending category in all states, we were discouraged to
see that that had lost some resources over time because as the largest
category it seems indicative of spending on children as a whole: a
movement away from spending on our youth.

The next variable we will investigate is CTC, or the Child Tax Credit.
CTC is a tax benefit given to medium to low income families with
dependent children. With this, qualifying families are able to pay a bit
less in taxes.

    # Calculate proportions for 1998
    prop_98<-
    kids %>%
      filter(variable == "CTC") %>%
      filter(year == "1998") %>%
      group_by(state, year)%>%
      left_join(total_year_state_98, by = c("state", "year")) %>%
      mutate(prop = inf_adj_perchild/total)

    # Create plot for 1998
    p1<- plot_usmap(data = prop_98, values = "prop", color = "blue") + 
      scale_fill_continuous(
        low = "white", high = "blue", 
        name = "Prop of Total Money Spent", 
        label = scales::comma
      ) + 
      theme(legend.position = "right") + 
      ggtitle("Prop of money for kids spent on CTC in 1998")

    # Calculate proportions for 2005
    prop_05<-
    kids %>%
      filter(variable == "CTC") %>%
      filter(year == "2005") %>%
      group_by(state, year)%>%
      left_join(total_year_state_05, by = c("state", "year")) %>%
      mutate(prop = inf_adj_perchild/total)

    # Create plot for 2005
    p2<- plot_usmap(data = prop_05, values = "prop", color = "blue") + 
      scale_fill_continuous(
        low = "white", high = "blue", 
        name = "Prop of Total Money Spent", 
        label = scales::comma
      ) + 
      theme(legend.position = "right") + 
      ggtitle("Prop of money for kids spent on CTC in 2005")

    # Calculate proportions for 2012
    prop_12<-
    kids %>%
      filter(variable == "CTC") %>%
      filter(year == "2012") %>%
      group_by(state, year)%>%
      left_join(total_year_state_12, by = c("state", "year")) %>%
      mutate(prop = inf_adj_perchild/total)

    # Create plot for 2012
    p3<-plot_usmap(data = prop_12, values = "prop", color = "blue") + 
      scale_fill_continuous(
        low = "white", high = "blue", 
        name = "Prop of Total Money Spent", 
        label = scales::comma
      ) + 
      theme(legend.position = "right") + 
      ggtitle("Prop of money for kids spent on CTC in 2012")

    # Calculate proportions for 2016
    prop_16<-
    kids %>%
      filter(variable == "CTC") %>%
      filter(year == "2016") %>%
      group_by(state, year)%>%
      left_join(total_year_state_16, by = c("state", "year")) %>%
      mutate(prop = inf_adj_perchild/total)

    # Create plot for 2016
    p4<-plot_usmap(data = prop_16, values = "prop", color = "blue") + 
      scale_fill_continuous(
        low = "white", high = "blue", 
        name = "Prop of Total Money Spent", 
        label = scales::comma
      ) + 
      theme(legend.position = "right") + 
      ggtitle("Prop of money for kids spent on CTC in 2016")

    # Plot CTC graphs over 4 years
    grid.arrange(p1,p2,p3,p4, nrow=2, ncol =2) 

![](Final_Project_Write_up_files/figure-gfm/ctc-year-comps-1.png)<!-- -->

Following a similar tendency to what we saw with K12 education, the
proportion of spending devoted to CTC has decreased noticeably since
1998. In 1998, the average proportion going to CTC appears to be roughly
\~.025, but in 2016, that same proportion appears to be more like
\~.018. The general decrease in CTC spending proportion indicates to us
that either fewer families are qualifying for this tax credit, or state
governments are becoming more stringent about which families qualify and
for how much.

We followed the same process to get insights on how SNAP spending on
children has changed over time. SNAP or the Supplemental Nutrition
Assistance Program, is also known as “food stamps,” in this case the
total spending here has been readjusted to not include adults on SNAP,
just children.

    # Calculate proportions for 1998
    prop_98<-
      kids %>%
        filter(variable == "SNAP") %>%
        filter(year == "1998") %>%
        group_by(state, year) %>%
        left_join(total_year_state_98, by = c("state", "year")) %>%
        mutate(prop = inf_adj_perchild/total)

    # Create plot for 1998
    p1<- plot_usmap(data = prop_98, values = "prop", color = "purple") + 
      scale_fill_continuous(
        low = "white", high = "purple", 
        name = "Prop of Total Money Spent", 
        label = scales::comma
      ) + 
      theme(legend.position = "right") + 
      ggtitle("Prop of money for kids spent on SNAP in 1998")

    # Calculate proportions for 2005
    prop_05<-
    kids %>%
      filter(variable == "SNAP") %>%
      filter(year == "2005") %>%
      group_by(state, year)%>%
      left_join(total_year_state_05, by = c("state", "year")) %>%
      mutate(prop = inf_adj_perchild/total)

    # Create plot for 2005
    p2<- plot_usmap(data = prop_05, values = "prop", color = "purple") + 
      scale_fill_continuous(
        low = "white", high = "purple", 
        name = "Prop of Total Money Spent", 
        label = scales::comma
      ) + 
      theme(legend.position = "right") + 
      ggtitle("Prop of money for kids spent on SNAP in 2005")

    # Calculate proportions for 2012
    prop_12<-
    kids %>%
      filter(variable == "SNAP") %>%
      filter(year == "2012") %>%
      group_by(state, year) %>%
      left_join(total_year_state_12, by = c("state", "year")) %>%
      mutate(prop = inf_adj_perchild/total)

    # Create plot for 2012
    p3<-plot_usmap(data = prop_12, values = "prop", color = "purple") + 
      scale_fill_continuous(
        low = "white", high = "purple", 
        name = "Prop of Total Money Spent", 
        label = scales::comma
      ) + 
      theme(legend.position = "right") + 
      ggtitle("Prop of money for kids spent on SNAP in 2012")

    # Calculate proportions for 2016
    prop_16<-
    kids %>%
      filter(variable == "SNAP") %>%
      filter(year == "2016") %>%
      group_by(state, year)%>%
      left_join(total_year_state_16, by = c("state", "year")) %>%
      mutate(prop = inf_adj_perchild/total)

    # Create plot for 2016
    p4<-plot_usmap(data = prop_16, values = "prop", color = "purple") + 
      scale_fill_continuous(
        low = "white", high = "purple", 
        name = "Prop of Total Money Spent", 
        label = scales::comma
      ) + 
      theme(legend.position = "right") + 
      ggtitle("Prop of money for kids spent on SNAP in 2016")

    # Plot SNAP graphs for all four years
    grid.arrange(p1,p2,p3,p4, nrow=2, ncol =2) 

![](Final_Project_Write_up_files/figure-gfm/snap-year-comps-1.png)<!-- -->

We found these variable proportion comparisons particularly interesting
because of their regional connection. Between 1998 and 2016 there is a
definite increase in spending proportions in the southern half of the
United States. In other words, the proportion of state spending
dedicated to SNAP for children increased. 2012 seems to have been the
peak for SNAP spending on children based on the graphs above, but
generally there is still an increasing trend of southern states needing
to increase SNAP funding (more dramatically than non-southern states).
Specifically looking at New Mexico, we can see an increase in SNAP
spending ratio from \~.015 to &lt;\~.025 in 2012. While we find the
geographical connection alarming, it does fit with our understanding of
the economic disparity between southern and northern states in the US,
as the southern states tend to be less city based and urban. At the same
time, we were glad to see that spending had increased proportionally
because it indicates that something is being done to help feed children
without other options. Obviously, this does not fix the problem of
hunger for every child in the least, but seeing an increase in spending
on child nutrition is somewhat encouraging.

While there are about 20 spending categories in total to explore, we
found similar map progressions in all of them except for SNAP.
Generally, they all show a decreased proportion in spending on that
category, with the exception of a few outliers. SNAP was the only
section to overtly break this pattern. Overall, in regards to the
initial question, we saw that different states choose to spend their
money quite differently when in regards to expenditures on children, but
generally, overtime, there has been less and less spending on child
related ventures.

Predicting Median Income through Spending
-----------------------------------------

Next, to understand the relationship between median income and spending
on kids, we created a simple linear model based on all the spending
categories to determine how well they can predict the median income. We
split the data at the year 2010, as this is around when the impacts of
the financial crisis of 2008 could be observed.

    # Separate the spending categories and data into different columns
    kids_separated <-
    kids %>%
      pivot_wider(
        names_from = c("variable"),
        names_sep = "_",
        values_from = c("raw", "inf_adj", "inf_adj_perchild")
      )

    # Combine with income data set
    df_combo <- 
      df_income %>%
      left_join(kids_separated, by = c("year","state")) %>%
      filter(state!= "United States") %>%
      rename(median_income = "median income") %>%
      filter(year >= 1997)

    # Split dataset
    split_year <- 2010
    df_train = df_combo %>% filter(year <= split_year)
    df_validate = df_combo %>% filter(year > split_year & year <= 2014)

    # Create simple linear model
    fit_inf_adj_per_child <- 
      df_train %>%
      lm(formula = median_income ~ inf_adj_perchild_PK12ed + inf_adj_perchild_highered + inf_adj_perchild_edservs + inf_adj_perchild_pell + inf_adj_perchild_HeadStartPriv + inf_adj_perchild_TANFbasic + inf_adj_perchild_othercashserv + inf_adj_perchild_SNAP + inf_adj_perchild_socsec + inf_adj_perchild_fedEITC + inf_adj_perchild_pubhealth + inf_adj_perchild_HCD + inf_adj_perchild_lib + inf_adj_perchild_parkrec)

    # Show coefficients
    fit_inf_adj_per_child %>% tidy(conf.int = TRUE)

    ## # A tibble: 15 x 7
    ##    term                estimate std.error statistic   p.value conf.low conf.high
    ##    <chr>                  <dbl>     <dbl>     <dbl>     <dbl>    <dbl>     <dbl>
    ##  1 (Intercept)          56490.      1768.   31.9    7.60e-137   53017.    59962.
    ##  2 inf_adj_perchild_P…   2411.       213.   11.3    2.11e- 27    1994.     2828.
    ##  3 inf_adj_perchild_h…    -29.3      510.   -0.0574 9.54e-  1   -1032.      973.
    ##  4 inf_adj_perchild_e…  -6962.      1594.   -4.37   1.45e-  5  -10092.    -3833.
    ##  5 inf_adj_perchild_p… -11373.      3267.   -3.48   5.31e-  4  -17787.    -4959.
    ##  6 inf_adj_perchild_H… -21635.      7965.   -2.72   6.77e-  3  -37274.    -5997.
    ##  7 inf_adj_perchild_T…   -351.      1986.   -0.177  8.60e-  1   -4251.     3548.
    ##  8 inf_adj_perchild_o…  -3869.       638.   -6.06   2.20e-  9   -5121.    -2616.
    ##  9 inf_adj_perchild_S…  -7699.      4054.   -1.90   5.80e-  2  -15659.      262.
    ## 10 inf_adj_perchild_s…  -6049.      6831.   -0.886  3.76e-  1  -19461.     7363.
    ## 11 inf_adj_perchild_f… -15779.      2011.   -7.84   1.72e- 14  -19728.   -11829.
    ## 12 inf_adj_perchild_p…   1906.       609.    3.13   1.83e-  3     710.     3103.
    ## 13 inf_adj_perchild_H…   3375.       777.    4.34   1.64e-  5    1849.     4902.
    ## 14 inf_adj_perchild_l…  12701.      6027.    2.11   3.55e-  2     867.    24535.
    ## 15 inf_adj_perchild_p…  10139.      1643.    6.17   1.17e-  9    6913.    13365.

Initially, we developed a linear model to predict median income based
off the inflation-adjusted spending per child in all the categories
(excluding only those where many states did not allocate any money).
This model has a few possibly redundant variables, though, which we
could tell by looking at the confidence intervals for the coefficient
estimates. Some of the variables, like `ed_servs`, `TANFbasic`, `SNAP`,
and `socsec`, have confidence intervals that include 0 as a possible
value for the coefficient. This means that we cannot confidently say
that these variables are predictors of median income. To confirm this,
we constructed a leaner version of this model which excluded variables
that might have no correlation to median income, which is shown below.

    # Create leaner model
    fit_leaner <- 
      df_train %>%
      lm(formula = median_income ~ inf_adj_perchild_PK12ed + inf_adj_perchild_highered + inf_adj_perchild_pell + inf_adj_perchild_HeadStartPriv +  inf_adj_perchild_othercashserv + inf_adj_perchild_fedEITC + inf_adj_perchild_pubhealth + inf_adj_perchild_HCD + inf_adj_perchild_lib + inf_adj_perchild_parkrec)

    # Show coefficients
    fit_leaner %>% tidy(conf.int = TRUE)

    ## # A tibble: 11 x 7
    ##    term                estimate std.error statistic   p.value conf.low conf.high
    ##    <chr>                  <dbl>     <dbl>     <dbl>     <dbl>    <dbl>     <dbl>
    ##  1 (Intercept)          54660.      1418.    38.6   9.46e-173   51877.    57444.
    ##  2 inf_adj_perchild_P…   2229.       204.    10.9   1.15e- 25    1828.     2630.
    ##  3 inf_adj_perchild_h…     96.9      513.     0.189 8.50e-  1    -911.     1104.
    ##  4 inf_adj_perchild_p… -15581.      2476.    -6.29  5.59e- 10  -20442.   -10720.
    ##  5 inf_adj_perchild_H… -28271.      7572.    -3.73  2.05e-  4  -43138.   -13403.
    ##  6 inf_adj_perchild_o…  -3633.       639.    -5.68  1.95e-  8   -4888.    -2378.
    ##  7 inf_adj_perchild_f… -17481.      1540.   -11.3   1.91e- 27  -20506.   -14457.
    ##  8 inf_adj_perchild_p…   1290.       586.     2.20  2.79e-  2     140.     2440.
    ##  9 inf_adj_perchild_H…   2208.       743.     2.97  3.05e-  3     750.     3666.
    ## 10 inf_adj_perchild_l…  17270.      6022.     2.87  4.26e-  3    5446.    29094.
    ## 11 inf_adj_perchild_p…  11720.      1615.     7.26  1.08e- 12    8549.    14891.

In our leaner model, we can make the following observations:

-   Spending on K12 education is strongly positively correlated with
    median income.

-   We cannot confidently determine if spending on higher education is
    correlated with median income as the confidence interval for the
    confidence interval for the confidence interval includes 0.

-   Spending on Pell grants, Head Start, other cash assistance services,
    and EITC is negatively correlated with median income.

-   Spending on public health, HCD (housing and community development),
    libraries, and parks and recreation is positively correlated with
    median income.

Interestingly, we observe that the large majority of cash assistance
programs or other such social services are either not confidently
correlated with median income or negatively correlated. There are two
hypotheses that could explain these trends. First, we hypothesize that
more affluent communities have greater resources to invest in
infrastructural resources and less demand for cash assistance programs,
whereas poorer communities have fewer resources to invest in
infrastructure and more of their community members need cash assistance.
A second hypothesis could be that more affluent communities are more
affluent because they have invested so much in social infrastructure,
whereas poorer communities are poorer because they have not been able to
invest as much in social infrastructure. Because we didn’t run a
controlled experiment, we cannot conclude the directionality of the
association and can only say that median income and spending in these
categories are clearly associated.

We also looked at the accuracy of each of these models to determine how
much we could trust their results.

    # Show Rsquared values for the full model
    rsquare(fit_inf_adj_per_child, df_train)

    ## [1] 0.6677366

    rsquare(fit_inf_adj_per_child, df_validate)

    ## [1] 0.6703908

    # Show Rsquared values for the leaner model
    rsquare(fit_leaner, df_train)

    ## [1] 0.6522968

    rsquare(fit_leaner, df_validate)

    ## [1] 0.6324106

The model packed with more variables is 67% accurate for both the
training and validation data. The leaner model is 65% accurate for the
training data and 63% for the validation data, which is comparable to
the fuller model. Both models have high accuracy given that median
income is influenced by so many other conditions that are not captured
in the data.

Conclusions and Further Questions
=================================

In conclusion, in most categories the spending proportions have
decreased over time, with the exception of SNAP, which has seen a
dramatic increase in spending in the southern half of the US. From our
model, we were able to conclude that spending and income are associated,
at least for some categories – such that it is possible with some
accuracy to predict the median income of a state given their spending on
children statistics. Generally, social infrastructure related spending
categories (libraries, parks and rec, public health, HCD) tend to be
more positively correlated with a state’s median income, whereas cash
assistance programs (CTC, Pell grants, EITC) are negatively correlated,
or no correlation is found. These conclusions roughly match our
hypotheses – there has been a general decrease in spending on children
in states, and there is some connection between state spending on
children and median income.

We have a number of further questions we would like to explore:

-   We wonder if it is possible to predict spending on kids based on the
    median income, as currently our model predicts income based off of
    spending on kids. We didn’t attempt to do this because we reasoned
    that spending on kids is dependent on other data about how
    governments are choosing to allocate all of their funds.

-   Relatedly, we’re curious where the money that is not going to kids
    is being allocated instead.

-   We had trouble finding total expenditures over time, and would’ve
    liked to use that as a normalizer to determine if richer states are
    spending a proportional amount of money on children as poorer
    states.

-   We also had a number of questions about how demographic
    characteristic of states relate to these trends. For example, what
    is the effect of the population of children in each state on state
    spending? How do race and gender distributions of kids, and of the
    state in general, affect state spending on kids? Is there a
    connection between the foster care population and state spending on
    kids?

-   Finally, we were curious what trends we would be observe if we used
    other metrics of children’s success beyond the median income in each
    state. For example, we could’ve used the number of young adults
    participating in either education, employment, or training, or the
    child poverty rate, or birthweight (all factors listed earlier in
    this report).
