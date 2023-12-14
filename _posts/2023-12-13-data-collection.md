---
title: "Exploring Atlanta Hawks Basketball Player Statistics"
author: Bryce Marshall
description: Data Collection and Cleaning  
image: "/assets/images/mountains.jpg"
---

# Introduction

Basketball is not just a sport; it's a passion that brings fans together. In this blog post, we delve into the statistics of Atlanta Hawks players, seeking insights that reveal the team's dynamics over the years. Our primary motivation is to understand how player positions and shooting preferences contribute to the team's performance. The goal is also to see how this has changed over the years.

# Data Collection and Cleaning

## Scraping Player Information

To kick-start our analysis, I needed detailed information about each player. I utilized web scraping techniques to extract data from Basketball Reference. I started using pandas to pull a table from Basketball Reference about nearly six hundred players that have played for the Atlanta Hawks over the course of the team’s history. The table includes several numeric statistics about each player. Now since some of these players were playing back in the 1950's some data is missing for some players. Below is the python code to pull the basic table from Basketball Reference:

```
import pandas as pd

# URL for the Atlanta Hawks players page
team_url = 'https://www.basketball-reference.com/teams/ATL/players.html'

# Read the HTML table into a DataFrame
df_player_data = pd.read_html(team_url)[0]

# Display the resulting DataFrame
print(df_player_data.head())

# Save the DataFrame to a CSV file
df_player_data.to_csv('basketball_player_data.csv', index=False)

```
I created a csv file for this data frame, and the plan will be to create one for a second data frame and then merge the two into a final csv file with all the information. Now the second data frame is focused on going through each player link on the page and pulling two categorical values (position and shooting hand) for each player. To begin, I used beautiful soup and set up an empty list for a variable called links in order to run a for loop to go through each individual player:

```
import pandas as pd
import requests
from bs4 import BeautifulSoup
import time

url = 'https://www.basketball-reference.com/teams/ATL/players.html'
r = requests.get(url)
soup = BeautifulSoup(r.text, 'html.parser')
links = []

for link in soup.find_all('a'):
   if link['href'].startswith('/players') and len(link['href']) > 10:
       links.append(link.get('href'))

links = links[:-19]

```
Now the links variable also included players outside the table so I just excluded the last nineteen rows so that the number of rows will match up with the first data frame. I will now set up the data to include a column for player name, position, and shooting hand. To do this I will have to clean up the code to exclude newline characters and unnecessary whitespace.

```
base_url = 'https://www.basketball-reference.com'
search_word = 'Position:'

player = []
positions = []
shoots = []

for web in links:
   time.sleep(2)
   r = requests.get(base_url + web)

   if r.ok:
       soup = BeautifulSoup(r.text, 'html.parser')
       name = soup.find('h1').text.strip()  # Remove leading and trailing whitespaces, including '\n'
       p_tag = soup.find(lambda tag: tag.name == 'p' and search_word in tag.text)

```
Since the information I need is all in text form in a paragraph, I now need to go through the information and separate it out based on keywords. I will then put them into the three columns.

```
if p_tag:
           info_text = p_tag.text
           info_cleaned = ' '.join(info_text.replace(search_word, '').split())  # Remove newline characters
           info_list = [info.strip() for info in info_cleaned.split(',')]

           # Extracting the entire position
           position = info_list[0].strip()

           # Extracting the 'Right' or 'Left' value for shoots
           shoots_value = next((info.strip().split()[-1] for info in info_list if 'Shoots' in info), 'No info')

           player.append(name)
           positions.append(position)
           shoots.append(shoots_value)
       else:
           player.append(name)
           positions.append('No info')
           shoots.append('No info')

```
This code is painting out what to look for in the paragraph and to separate it based on position and shooting hand. Now that this is done, we can create the second data frame. However, I was experiencing troubles with pulling the position and shooting hand data since not every single player had the same format. As a result, I had to only pull the first word for the position so it all came out correctly. This will explain why you will see “Point” and “Shooting” instead of “Point Guard” and “Shooting Guard” in the data and in graphs. This is the same for the forward positions. You will also notice a combination of positions like “Guard/Forward” and so on for a few players. These were for players in the early 1950’s. Here is the code for creating the data frame:

```
# Creating DataFrame
df = pd.DataFrame({'player': player, 'position': positions, 'shoots': shoots})
df['position'] = df['position'].apply(lambda x: x.split()[0])  # Extract only the first word from the position column

df.to_csv('basketball_player_info2.csv', index=False)

```

## Merging the Two Data Frames

Next is to merge the two data frames together. To do this we first will have to change a column name in the first data frame so that it matches the player column in the second data frame. 

```
df_player_data = pd.read_csv('basketball_player_data.csv')

# Rename the 'Unnamed: 1_level_0' column to 'player'
df_player_data = df_player_data.rename(columns={'Unnamed: 1_level_0': 'player'})

```

Now we just save the second variable as reference to the second csv file created and merge the two and create a new and final combined csv with this code:

```
df = pd.read_csv('basketball_player_info2.csv')

# Merge the two DataFrames on the 'player' column
merged_df = pd.merge(df_player_data, df, on='player', how='left')

# Save the merged DataFrame to a CSV file
merged_df.to_csv('merged_basketball_data.csv', index=False)
```

# Ethical Considerations

As responsible data practitioners, ethical considerations are paramount. In our web scraping process, we adhered to ethical guidelines by incorporating  a time delay between requests to avoid overloading the website’s server. Additionally, we respected the terms of use outlined by Basketball Reference.

# Conclusion 

Our journey into Atlanta Hawks player stats began with digging into Basketball Reference data through careful web scraping. We collected info on almost six hundred players, focusing not just on numbers but also on player positions and shooting preferences.

We kept things ethical, spacing out our requests and respecting Basketball Reference's rules. The magic happened when we merged the player info with the stats. Now, our dataset is a one-stop-shop, showing both the numbers and the stories behind Atlanta Hawks players.

For those curious minds, the full code and data are up for grabs [here](https://github.com/bryce11m/semester-project). Feel free to dive in, explore, and maybe even add your touch to the story.

