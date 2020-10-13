# Product Analytics

## Introduction

The goal of this project is to work with a quite large dataset (around 4.8M rows) to measure relevant business metrics, perform time series forecasting, extract actionable insights, and make data-driven recommendations. The code is written in Python and contained in Jupyter notebooks.

## Dataset

The dataset `ping_dataset.json` consists of approximately 4.8M observations corresponding to ping data for an app collected in February 2016. Every time a user opens the app, a ping is recorded. Each row in the dataset corresponds to a ping and contains the following information:
- `date`: ping date in YYYY-MM-DD format (based on Pacific time)
- `timestamp`: ping timestamp in YYYY-MM-DD HH:MM:SS.FFF format, where FFF represents milliseconds and timestamps are based on Pacific time
- `uid`: unique ID assigned to users (purely numeric if the user is registered or alpha-numeric if the user is not registered and in this case `uid` represents a device ID)
- `isFirst`: `True` if that ping represents the first ping associated to that `uid` (for some users the first ping happened before February 2016)
- `utmSource`: traffic source which first redirected the user to the app.

## Data Exploration
