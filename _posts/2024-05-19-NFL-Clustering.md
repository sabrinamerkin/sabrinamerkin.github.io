---
title: "Upload in progress..."
date: 2024-5-19
tags: [Python, K-means Clustering]
excerpt: "Uploading..."
---

## Background
The NFL has players of all shapes and sizes. With diversity in player build, we might expect particular player types to excel in different NFL combine tests...

We have been asked to investigate trends in service user behavior regarding chat attempts, wait times, and successful chats from queues. First, we will use SQL and Python to join and format the two datasets. After that, we will visualize our findings in Tableau with interactive dashboards. All Python code will be executed in an IPython kernel using Jupyter Notebook.

**Key Notes**

- 'Queues' are synonymous for waits to enter a Kooth chatroom.
- Not all successful chats require queues. Users can book chat appointments ahead of time.
- Not all users have a successful chat after initiating a queue.
- Users can be removed from their queues by a practitioner if they are not going to get a chat. They may also exit queues themselves.
- Chatroom availability opens at 9:00am and closes at 9:00pm.

## Data Analysis
For this analysis, we will be executing MySQL queries within a Python environment using the *pandasql* Python package. Before we begin, we must install this package on our desktop. We can then import all required packages for analysis.

```python
