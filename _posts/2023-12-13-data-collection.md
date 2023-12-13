---
title: "Exploring Atlanta Hawks Basketball Player Statistics"
author: Bryce Marshall
description: A quick walkthrough   
image: "/assets/images/mountains.jpg"
---

# Introduction

Basketball is not just a sport; it's a passion that brings fans together. In this blog post, we delve into the statistics of Atlanta Hawks players, seeking insights that reveal the team's dynamics over the years. Our primary motivation is to understand how player positions and shooting preferences contribute to the team's performance.

# Data Collection and Cleaning

## Scraping Player Information

To kick-start our analysis, we needed detailed information about each player. We utilized web scraping techniques to extract data from Basketball Reference. Below is a snippet of the Python code used for this process:

```
# Import necessary libraries
import pandas as pd
import requests
from bs4 import BeautifulSoup
import time

# Specify the URL
url = 'https://www.basketball-reference.com/teams/ATL/players.html'
r = requests.get(url)
soup = BeautifulSoup(r.text, 'html.parser')
links = []

# Extract relevant player links
for link in soup.find_all('a'):
    if link['href'].startswith('/players') and len(link['href']) > 10:
        links.append(link.get('href'))
```

# (Additional code for processing player information)

# Create a DataFrame
df_player_info = pd.DataFrame({'player': player, 'position': positions, 'shoots': shoots})
df_player_info['position'] = df_player_info['position'].apply(lambda x: x.split()[0])
