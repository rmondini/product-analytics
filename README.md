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

## Measurement of key business metrics

We move on to the tracking and measurement of key business metrics to extract actionable insights and drive product growth.

### Daily Active Users

Daily active users (DAU) is a key metric to track to understand how a product/app is used. I determined the number of daily active users for each day in February 2016 (a user is considered active if they had at least one ping that day). I decided to plot both the total DAUs and the breakdown of registered DAUs (numeric `uid`) vs device DAUs (alpha-numeric `uid`). The obtained plot is the following:

![alt_text](https://github.com/rmondini/product-analytics/blob/main/plots/DAU_plot.jpg?raw=true)

We observe a very distinct weekly pattern: the number of DAUs stays constant Monday-Thursday, considerably drops on Friday and Saturday to reach the weekly minimum, and finally starts picking up again on Sunday. In addition to this weekly pattern, we observe an overall increasing trend over the month. For instance, using DAUs on Saturdays as a reference, the total DAUs on Feb 6th, 13th, 20th, and 27th are respectively 94807, 97788 (+2981), 104665 (+6877), and 112789 (+8124), which indicates an overall positive trend growing faster than linearly. These weekly and overall patterns are followed by both the registered DAU and device DAU lines. This analysis prompts us to investigate why the number of active users drops every Friday and Saturday. Is this an organic effect or are there other contributing factors? For instance, if the app is a work-related product (e.g. a productivity or project management app), reduced usage on Fridays compared to previous weekdays and greater reduced usage on Saturdays might just be a natural consequence of fewer people opening the app for work as the weekend begins. Along the same lines, a slightly increasing number of DAUs on Sundays might be explained as a number of people resuming job-related activities as the new work week approaches. In addition, if the app is used by people in different time zones and all timestamps are converted into a reference time zone (e.g. Pacific Time), it is certainly expected to observe these "weekend effects" spread out across Friday-Sunday instead of being strictly limited to Saturday and Sunday (e.g. Friday afternoon on the US West Coast corresponds to Friday night in Central Europe and Saturday in Asia, while Sunday afternoon on the West Coast corresponds to Monday morning in Asia).

We should also consider analyzing why we observe an overall positive trend over the month. Can this trend be explained solely by looking at the context in which people use the app or is it the result of more successful marketing and advertising campaigns over time? It could be useful to compare this positive trend with DAU plots from February of previous years and other months of the year.

Finally, the plot shows that the majority of DAUs are registered users. The percentage
of registered DAUs slightly increases over the month (80.1% on Feb 1st, 81.4% on Feb 15th,
and 81.8% on Feb 29th). It would be interesting to understand what causes this increase
and discuss strategies to further drive conversion of unregistered DAUs (device DAUs) into
registered users.




