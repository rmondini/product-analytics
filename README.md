# Product Analytics

## Introduction

The goal of this project is to perform product analytics on a real-world messy dataset (around 4.8M rows) by tracking and measuring relevant business metrics, performing time series analysis and forecasting, extracting actionable insights, and making data-driven recommendations. The code is written in Python and contained in Jupyter notebooks.

## Dataset

The dataset `ping_dataset.json` consists of approximately 4.8M observations corresponding to ping data for an app collected in February 2016. Every time a user opens the app, a ping is recorded. Each row in the dataset corresponds to a ping and contains the following information:
- `date`: ping date in YYYY-MM-DD format
- `timestamp`: ping timestamp in YYYY-MM-DD HH:MM:SS.FFF format, where FFF represents milliseconds
- `uid`: unique ID assigned to users (purely numeric if the user is registered or alpha-numeric if the user is not registered and in this case `uid` represents a device ID)
- `isFirst`: `True` if that ping represents the first ping associated to that `uid` (for some users the first ping happened before February 2016)
- `utmSource`: traffic source that sent the user to the app.

## Data Exploration & Wrangling

Real-world datasets are messy and require extensive exploration, cleaning, and wrangling before any meaningful analysis can be carried out. In `Product_Analytics.ipynb` I used Pandas to load the dataset and perform the preliminary data exploration and wrangling (see the notebook for details). Here I list a few observations that emerged from these initial phases of the analysis:
- The dataset contains around 13,000 duplicate rows (corresponding to around 0.26% of the total number of rows). I dropped the duplicates by keeping only the first entry of each repeated row.
- Many entries (around 25%) have a different date in the `date` and `timestamp` columns (with the date in the `date` column being one day before the date in `timestamp`). This date mismatch occurs for all pings with timestamp recorded between midnight and 8am (regardless of the day and traffic source). As shown in the notebook, I believe that this is due to all times in the `timestamp` field being erroneously shifted by 8 hours (possibly because of time zone conversions and conventions?). I therefore subtracted 8 hours from all times (e.g. a timestamp such as `2016-02-02 00:30:10.001` is now `2016-02-01 18:30:10.001`) and the mismatch between `date` and `timestamp` is fixed. In a real-world scenario, it would be highly recommended to investigate this issue further. A few questions to answer are: what is the reference time zone of the dates and timestamps? Does this dataset contain pings only for users in the same time zone or in multiple time zones? If users in different time zones open the app, are the respective ping timestamps being converted to the same reference time zone when stored?
- I observed that around 7,000 timestamps appear more than one in the dataset. While two pings can happen on the same day exactly at the same time, it is very unlikely to have many such occurrences, given the recorded millisecond-level precision. I vchecked that there is no strange pattern in how milliseconds are recorded by creating an histogram of the milliseconds in the dataset and observing a uniform distribution over the range [0,999] as expected. However, simultaneous pings need additional investigation to understand whether they are genuine, errors in the dataset, or the indication of fraudulent activities (such as click spamming).
- I observed that 45 `uid`s have two instances of `isFirst=True`, with the second instance often occurring just a few minutes after the first one. This needs further investigation to understand why in those cases both pings are recorded as the first ping. One way to correct these entries in the dataset would be to set the later instance of `isFirst=True` to `False` instead. Additionally, for 44 `uid`s the timestamp to which `isFirst=True` is associated is not actually the first timestamp ever recorded. This issue should also be addressed further and resolved. Finally, for those `uid`s that do not have a `isFirst=True` ping in the dataset, it would be important to verify that their first ping was indeed recorded before February 2016.
- I observed that some traffic sources in `utmSource` are most probably the same source but appear as distinct because of minor spelling variations, so I renamed them. Specifically, `Twitter_org` as `twitter`, `facebook.com` and `Facebook_org` as `facebook`, `Blog_org` as `blog`, and `shmoop_left`, `shmoop_logo`, `shmoop_right` as `shmoop`. For other sources, additional domain knowledge is required to understand what these sources represent, whether there are errors (e.g. the source called `Sarah+Doody's+UX+Notebook` might not be the actual traffic source we are interested in), and to make sure that no duplicate sources show up as distinct.
- I noticed that `utmSource` contains approximately 35% missing values so it would be helpful to investigate why so many values are missing in this column. Moreover, while the
majority of users have pings associated with only one traffic source, 60 users have
pings associated with two distinct sources. Is this an error? How exactly are traffic sources
assigned to users? Can a missing value indicate that that user discovered the app organically? This would need to be clarified by talking to the relevant stakeholders (e.g. the marketing team or whoever is in charge of UTM tracking).

## Measurement of Key Business Metrics

We move on to the tracking and measurement of key business metrics to extract actionable insights from the data and drive product growth. For the purpose of this analysis, I decided to focus on two important metrics for an app: daily active users and daily retention. Needless to say, there are many other relevant metrics to consider when evaluating user engagement with an app and its overall success/growth. If additional data were available, it would certainly be interesting to measure metrics such as in-app purchases and revenue, user lifetime value, user acquisition cost, install and registration curves, premium subscription curves, in-app activity time, etc.

### Daily Active Users

Daily active users (DAU) is a key metric to track to understand user engagement with a product/app. I determined the number of daily active users for each day in February 2016 (a user is considered active if they had at least one ping that day). I decided to plot both the total DAUs and the breakdown of registered DAUs (numeric `uid`) vs device DAUs (alpha-numeric `uid`). The obtained plot is the following:

<img src="https://github.com/rmondini/product-analytics/blob/main/plots/DAU_plot.jpg" width="480">

We observe a very distinct weekly pattern: the number of DAUs stays constant Monday-Thursday, considerably drops on Friday and Saturday to reach the weekly minimum, and finally starts picking up again on Sunday. In addition to this weekly pattern, we observe an overall increasing trend over the month. For instance, using DAUs on Saturdays as a reference, the total DAUs on Feb 6th, 13th, 20th, and 27th are respectively 94807, 97788 (+2981), 104665 (+6877), and 112789 (+8124), which indicates an overall positive trend growing faster than linearly. These weekly and overall patterns are followed by both the registered DAU and device DAU lines. This analysis prompts us to investigate why the number of active users drops every Friday and Saturday. Is this an organic effect or are there other contributing factors? For instance, if the app is a work-related product (e.g. a productivity or project management app), reduced usage on Fridays compared to previous weekdays and greater reduced usage on Saturdays might just be a natural consequence of fewer people opening the app for work as the weekend begins. Along the same lines, a slightly increasing number of DAUs on Sundays might be explained as a number of people resuming job-related activities as the new work week approaches. In addition, if the app is used by people in different time zones and all timestamps are converted into a reference time zone (e.g. Pacific Time), it is certainly expected to observe these "weekend effects" spread out across Friday-Sunday instead of being strictly limited to Saturday and Sunday (e.g. Friday afternoon on the US West Coast corresponds to Friday night in Central Europe and Saturday in Asia, while Sunday afternoon on the West Coast corresponds to Monday morning in Asia).

We should also consider analyzing why we observe an overall positive trend over the month. Can this trend be explained solely by looking at the context in which people use the app (work, leisure, etc) or is it the result of more successful marketing and advertising campaigns over time? It could be useful to compare this positive trend with DAU plots from February of previous years and other months of the year.

Finally, the plot shows that the majority of DAUs are registered users. The percentage
of registered DAUs slightly increases over the month (80.1% on Feb 1st, 81.4% on Feb 15th,
and 81.8% on Feb 29th). It would be interesting to understand what causes this increase
and discuss strategies to further drive conversion of unregistered DAUs (device DAUs) into
registered users.

Active users can also be tracked and measured over different time intervals than daily (e.g. weekly or monthly active users) and/or expressed as ratios (e.g. the DAU to MAU ratio, or "stickiness").

### Daily Retention

Another important metric to consider is retention, i.e. the ability of the app to retain its users over time. For the purpose of this analysis, I decided to measure daily cohort-based retention curves, focusing on users who used the app for the first time on February 4th, 10th, and 14th and determining the proportion of those users who were active on each subsequent day of the month. The obtained daily retention curves are shown in the following plot:

<img src="https://github.com/rmondini/product-analytics/blob/main/plots/DR_plot.jpg" width="480">

The three curves follow a quite similar pattern, with a sharp decrease in retention in the first few days followed by a plateauing trend. The curves are also clearly modulated by the weekend effect discussed previously, which makes it more delicate to compare curves (since February 14th is a Sunday while February 4th and 10th are weekdays). The Day 2 retention is 43.9%, 50.7%, and 55.0%, while the Day 7 retention is 43.2%, 48.3%, and 47.4% for February 4th, 10th, and 14th respectively. The overall higher retention for the February 10th and 14th curves compared to the February 4th curve correlates with the fact that the number of DAUs has an overall increasing trend over the course of the month. The Day 2 retention shows that approximately half of the users stop using the app after only two days. Can we analyze additional data and identify what causes such quick churn? In the longer run (e.g. Day 14), the retention seems to be plateauing around 38-44% for the three dates. Do the relevant stakeholders find this acceptable?

Finally, the number of new users on February 4th (Thursday), 10th (Wednesday), and
14th (Sunday) is respectively 2761, 3129, and 2735. The lower number on February 14th
(despite being a later day in the month than the other two dates) is due to the fact that
on Sundays the number of users is lower than previous weekdays. The number of users on
Wednesdays and Thursdays is usually comparable, and therefore as expected the number
of new users on the 10th is higher than on the 4th due to the overall monthly increasing
trend.

## Analysis of Traffic Sources

Tracking sources of traffic to the app allows us to extract insights about what traffic sources redirect the app's best (and worst) users. These insights are extremely valuable for many reasons, for instance in the evaluation of the effectiveness of specific marketing and advertising campaigns, in the creation of ad spending budgets on specific networks and platforms, in the segmentation of users for targeted actions (promotions, recommendations, etc) and so on. I therefore decided to analyze the traffic sources in the dataset to answer the question: "From which sources does the app get its best users?".

As mentioned earlier, the `utmSource` column contains approximately 35% missing values. While it is absolutely crucial to understand the meaning and the cause of these missing values in a real-world scenario, without additional information at my disposal I had to exclude all the rows with missing traffic source from the analysis of traffic sources and quality of users. If we are interested in determining from which traffic source the app
gets its best and worst users, we first need to define who good users are. For this analysis, I decided to study traffic sources according to two different definitions of "good users": 1. users who open the app a lot (large total number of pings over the month), and 2. users who open the app at least once on many different days (many "active" days during the month). This is not an exhaustive list of definitions of good users. In the presence of additional data (e.g. time spent on the app per session, in-app purchases, premium subscription, etc), many other definitions could be given.

Using the first definition, I was interested in computing the median number of pings
per user in February 2016 for each traffic source and then ranking sources from best to
worst according to this metric. I decided to use the median instead of the mean to
limit the contribution of outliers (e.g. users who pinged an incredibly high number of times
possibly due to fraudulent activities such as click spamming). In order to construct the
desired metric, I first determined the number of pings for each user by traffic source and the number of distinct users coming from each source. I noticed that some sources redirected
to the app a very small number of users during the whole month. For this reason, I decided
to focus only on sources with at least 100 distinct users. The median numbers of pings
per user for the selected sources are shown in the following table:

<img src="https://github.com/rmondini/product-analytics/blob/main/plots/MNPPU_Table.png" width="480">

We observe that, according to this metric and definition of good users, the app gets its best users from `tapjoy` (users from this source pinged 29 times over the month as a median), while the worst users come from the source called `answers` (15 median pings per user). While using the median makes this metric rather robust, it is still not optimal, as a source could have a high median number of pings per user due to users who pinged a lot only on one specific day and then stopped using the app. In this situation, it is not desirable to consider these as good users. For this reason, I moved to the second definition presented earlier. In this case, I chose the median number of active days per user as the relevant metric (with an active day defined as a day with at least one ping). In using this definition to determine the best and worst sources, I implicitly assumed that all sources could redirect users to the app from the first day of the month (if a source started redirecting traffic only from February 20th, its median number of active days per user could be at most 10 and a comparison with other sources might not be meaningful). This assumption would need to be explicitly checked in a real-world scenario. As in the previous analysis, I reported results only for sources with at least 100 distinct users. The median numbers of active days per user are shown in the following table:

<img src="https://github.com/rmondini/product-analytics/blob/main/plots/MADPU_Table.png" width="480">

Also in this case, `tapjoy` redirects to the app the best users, who had a median of 24 active days, while the worst users (13 median active days) come again from `answers`. We also observe that the top 5 sources according to the two metrics/definitions are the same: `contenthub`, `Grub+Street`, `salesmanago`, `tapjoy`, and `youtube`. As mentioned earlier, this analysis could be used in conjunction with an analysis of how the advertising budget is spent across all sources to inform future marketing and advertising campaigns. For instance, since these identified five sources seem to provide the app with its most engaged users, the recommendation could be to increase ad spending on those sources to further drive the number of good users redirected to the app. On the other hand, any advertising budget spent on sources like `answers` does not seem to be extremely fruitful, as this source redirects users that are considerably less engaged according to either metric.







