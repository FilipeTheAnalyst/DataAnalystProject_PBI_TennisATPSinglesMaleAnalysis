# Tennis ATP Singles Male Analysis
In this project I collected data from 2001 till 2021 (last 20 years) from Tennis ATP Singles Male competition showing insights about total matches played, tournaments, players, tournament winners, grand slams, player stats and head-to-head stats.

![tennis](https://user-images.githubusercontent.com/61323876/135621642-bc35d4f2-3752-4282-ab8c-60686a85771b.jpg)

## Data Source
The data is collected from JeffSackmann repository that you can check [here](https://github.com/JeffSackmann/tennis_atp) in csv format and loaded directly into Power BI.

## Data Collection
From the repository I used the following files to get the data that I needed:

* atp_matches_2001 to atp_matches_2021
* atp_players
* atp_rankings_00 to atp_rankings_current

I also created support data into new csv files to help me organize and visualize the data.
These files can be checked in my [repository](https://github.com/FilipeTheAnalyst/DataAnalystProject_PBI_TennisATPSinglesMaleAnalysis)
* __player_face__: table with the player id, name and image with the face of each player to use in visualizations
* __tourney_logos__: table with the tournament name and image with the logo of each Grand Slam tournament (Australian Open, Roland Garros, Wimbledon, US Open) to use in visualizations
* __round__: table to sort order of each round in tennis tournaments by relevance (ex: Final, Semi-Final)
* __tourney_level__: table to sort order of each tournament level by importance (ex: Grand Slam, ATP Masters, ATP Tour)

## Data Model
Using the data stated above as a starting point for my analysis and data visualization, I created a couple more tables to achieve my goal.
Here is a brief explanation for each table included in the Power BI model:
* __Date__: I created a Date table to manage date related values for all other tables on the model
* __Measures Table__: table with all calculations defined in Power BI
* __ATP Ranking 2001-2021__: table with the data from ATP Rankings 
* __ATP Matches 2001-2021__: table with the data from ATP Matches
* __ATP All Matches 2001-2021__: Auxiliar table created from ATP Matches table that duplicates each match from the winner and loser point of view.
* __player_face__: table with the player id, name and image with the face of each player to use in visualizations
* __player_face_opponent__: same as above for the opponent player
* __tourney_logos__: table with the tournament name and image with the logo of each Grand Slam tournament (Australian Open, Roland Garros, Wimbledon, US Open) to use in visualizations
*  __Last Refresh__: Table with the date/time record of the last update of data

<img width="794" alt="DataModel" src="https://user-images.githubusercontent.com/61323876/135585027-733d1292-2b04-4717-b827-b502f714fb5d.png">

## Calculations
The following calculations were created in the Power BI reports using DAX (Data Analysis Expressions).
I'm not going to show here all the calculations created in this model because there are above 50 with a lot of them regarding head-to-head statistics.

If you're interested in checking those you can access the pbix file in [here](https://github.com/FilipeTheAnalyst/DataAnalystProject_PBI_TennisATPSinglesMaleAnalysis/blob/main/Tennis%20Singles%20Male%20Analysis.pbix)
Below I state the global calculations created.
To lessen the extent of coding, the re-use of measures (measure branching) was emphasized:
```
# Matches = 
COUNTROWS('ATP Matches 2001-2021')+0

# Matches Lost = 
CALCULATE(
    COUNTROWS(
        'ATP Matches 2001-2021'
    ),
    USERELATIONSHIP(
        'ATP Matches 2001-2021'[loser_id],'ATP Players'[player_id]
    )
)+0

# Players = 
var winning_player = DISTINCT('ATP Matches 2001-2021'[winner_id])
var losing_player = DISTINCT('ATP Matches 2001-2021'[loser_id])
RETURN
COUNTROWS(
    DISTINCT(
        UNION(
            winning_player, losing_player
        )
    )
) 

# Players Win Tournament = 
CALCULATE(
    COUNTROWS(
        DISTINCT('ATP Matches 2001-2021'[winner_id]
        )
    ),
    'ATP Matches 2001-2021'[round] = "Final"
)+0

# Total Matches = 
[# Matches] + [# Matches Lost]

# Tournaments = 
CALCULATE(
    COUNTROWS(
        DISTINCT('ATP Matches 2001-2021'[tourney_id])
    )
)

# Tournaments Final Lost = 
CALCULATE(
    COUNTROWS('ATP Matches 2001-2021'),
    DISTINCT('ATP Matches 2001-2021'[tourney_id]),
    DISTINCT('ATP Matches 2001-2021'[loser_id]),
    'ATP Matches 2001-2021'[round] = "Final",
    USERELATIONSHIP('ATP Matches 2001-2021'[loser_id],'ATP Players'[player_id])
)

# Tournaments Win = 
COUNTROWS(
    FILTER(
        GROUPBY(
            'ATP Matches 2001-2021',
            'ATP Matches 2001-2021'[tourney_id],
            'ATP Matches 2001-2021'[winner_id],
            'ATP Matches 2001-2021'[round]
        ),
        'ATP Matches 2001-2021'[round] = "Final"
    )
)

% Matches Lost = 
[# Matches Lost] / [# Total Matches]

% Matches Win = 
[# Matches] / [# Total Matches]

-- Players selected (1 and 2) in Head-to-Head page dashboard
Player Selected 1 = 
IF(
    ISFILTERED(player_face[Name]),
    FIRSTNONBLANK(player_face[Name],1),
    "No Player Selected"
)

Player Selected 2 = 
IF(
    ISFILTERED(player_face_opponent[Name]),
    FIRSTNONBLANK(player_face_opponent[Name],1),
    "No Player Selected"
)

--Tournament selected in Grand Slam filter showed in Grand Slams page
Tournament Selected = 
IF(
    ISFILTERED('ATP Matches 2001-2021'[tourney_name]),
    FIRSTNONBLANK('ATP Matches 2001-2021'[tourney_name],1),
    "No Tournament Selected"
)

```
## Dashboards
The finished dashboard consists of visualizations and filters that gives an easy option for the end users to navigate through the data of Tennis ATP Singles Male from the last 20 years. 

There are 4 pages with specific data (Global Numbers, Grand Slams, Player Records, Head to Head Stats) and additional pages defined as tooltips to provide more contextual information and detail on the different visuals.
Below there are a couple of screenshots walking through the different visuals that you can access.

__Click [here](https://app.powerbi.com/view?r=eyJrIjoiMDc5NzI0MjgtOWNmOC00YjVkLThiYWQtOTBmNzJhNGFhYjNlIiwidCI6IjBiZmE4NTAwLWIxZjItNDU2Ni1iYWYxLTZmNTkzNzA4OTNlNyIsImMiOjh9) to open the dashboard and try it out!__

#### Global Numbers Dashboard
<img width="671" alt="GlobalNumbers1" src="https://user-images.githubusercontent.com/61323876/135620151-dc8e20f6-29ed-4182-87df-efa15781b056.png">

#### Global Numbers - Tournaments by Category Tooltip

<img width="687" alt="GlobalNumbersTooltip1" src="https://user-images.githubusercontent.com/61323876/135620194-5e860d9b-a4cc-4efa-a347-71e6a6628e56.png">

#### Global Numbers - Players by Country Tooltip

<img width="671" alt="GlobalNumbersTooltip2" src="https://user-images.githubusercontent.com/61323876/135620319-f742c46a-0a61-43ee-817f-2bfe3685a8cd.png">

#### Global Numbers - Players Win Tournament by Country Tooltip

<img width="671" alt="GlobalNumbersTooltip3" src="https://user-images.githubusercontent.com/61323876/135620328-5d315fc5-55ec-488e-a88d-84563ab84435.png">

#### Global Numbers - Tournaments Win by Country Tooltip

<img width="669" alt="GlobalNumbersTooltip4" src="https://user-images.githubusercontent.com/61323876/135620341-3675c443-cc58-4f27-b5d0-fe4bdd728fc1.png">

#### Global Numbers - Tournaments Win by Player Tooltip

<img width="669" alt="GlobalNumbersTooltip5" src="https://user-images.githubusercontent.com/61323876/135620349-4551f204-d930-4710-be77-77bd1e4a95b7.png">

#### Grand Slams

<img width="673" alt="GrandSlams1" src="https://user-images.githubusercontent.com/61323876/135620548-e28f8f5f-8ac8-491c-b9db-eeea717fb2da.png">

#### Grand Slams - Filter by Tournament

<img width="671" alt="GrandSlamsFilterTournament" src="https://user-images.githubusercontent.com/61323876/135620599-07ed69e6-9d77-446c-99a8-d834e81fe87d.png">

#### Grand Slams - Losing Finalists Tooltip

<img width="671" alt="GrandSlamsTooltipFinalists" src="https://user-images.githubusercontent.com/61323876/135620641-e54d9786-d6ef-4e17-af8a-85140c6404e4.png">

#### Grand Slams - Final Match Table Tooltip

<img width="789" alt="GrandSlamsTooltipFinaltable" src="https://user-images.githubusercontent.com/61323876/135620668-0cc50651-0084-4371-96a3-bc92f4d30af4.png">

#### Grand Slams - Winner Tooltip

<img width="670" alt="GrandSlamsTooltipWinners" src="https://user-images.githubusercontent.com/61323876/135620703-795aca3e-bddf-4448-965f-a78a29c7cb28.png">

#### Player Records - Player Selected

<img width="663" alt="PlayerRecordsPlayerSelected" src="https://user-images.githubusercontent.com/61323876/135621233-53e39920-086e-4123-b080-8a265cdfc3c9.png">

#### Player Records - Tournament Won Tooltip

<img width="820" alt="PlayerRecordsTournamentWon" src="https://user-images.githubusercontent.com/61323876/135621251-e433e109-3d79-43b3-b917-80ecb8c80667.png">


#### Head to Head Stats

<img width="671" alt="HeadtoHeadStats" src="https://user-images.githubusercontent.com/61323876/135620721-58532a49-e8ac-48b8-abcd-3f7baa76d175.png">

#### Head to Head Stats - Career Titles Tooltip

<img width="671" alt="HeadtoHeadStatsCareerTitlesTooltip" src="https://user-images.githubusercontent.com/61323876/135620746-9e8640f9-45ce-4726-a24d-3099ba5e4f51.png">

#### Head to Head Stats - Career Titles YTD Tooltip

<img width="960" alt="HeadtoHeadStatsCareerTitlesYTDTooltip" src="https://user-images.githubusercontent.com/61323876/135620792-85908f66-2f4b-45e4-817a-70deb4fa3827.png">

#### Head to Head Stats - Match Stats Tooltip

<img width="960" alt="HeadtoHeadStatsMatchStatsTooltip" src="https://user-images.githubusercontent.com/61323876/135620807-ab011f0d-2739-4679-b746-88ef7d0a6ed6.png">
