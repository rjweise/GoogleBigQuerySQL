# Looking at Icing tendencies of NHL teams, players and coaches

This research is the result of a conversation with @RobVollmanNHL and looks at the last couple of NHL season for icing committed by teams, players and coaches.

## Purpose
I'm no expert by any means on hockey, but generally speaking an icing is a negative event; icing tires out the team, it allows the opposing team to change their lineup and pick their preferred match-ups, and the face-off following icing is in the defensive zone of the icing team, with a roughly 50% change of regaining possesion of the puck (and thus a 50% chance of giving the puck to your opponent in their attacking zone).

![alt text](https://github.com/rjweise/RESEARCH---NHL-Icing-and-Coaching-tendencies/blob/master/Screen%20Shot%202018-01-19%20at%208.02.21%20PM.png)
[A first step in this research](https://public.tableau.com/profile/rj7974#!/vizhome/IcingPrototype2/Icingperteamandplayer) showed me what players iced the puck most, and I then aggregated that data per team. It made me wonder what the influence of the coach was on these icing tendiencies; do some prefer the temporary relief that icing can provide over the potentially negative consequences described above? Does it just come with a certain playing style coaches prefer?

## Data
To do this anlysis I needed data for icing events, which team committed the icing, what players were on the ice, and who was coaching that team. For the icing events I use the R package NHLscrapr from A.C. Thomas and Samuel L. Ventura ([link](https://cran.r-project.org/web/packages/nhlscrapr/index.html)) to pull play-by-play data from the NHL's HTML reports per game ([example]( http://www.nhl.com/scores/htmlreports/20172018/PL020043.HTM)). This gives me the events per game(-number), the players on ice for each team, the teams, and to a certain degree the team that iced the puck (more to follow below).

Still missing were the coaches of the teams for which I used different R-code that reads the NHL API Life Feed JSON files ([example](http://statsapi.web.nhl.com/api/v1/game/2017020019/feed/live)) and pulls both coaches and the game number, which can be linked to the data described above.

### NHL Play by Play reports
I should note that to my knowledge it is impossible to determine for every icing which team iced the puck; the icing team can not be determined when the "homezone" field contained other values then "Def" or "Off" (like "", "Neu" and "Unk"). Checking on a subset of all data I found about 1% of the data contained these values, and were excluded from the research as they could not be assigned to a team or player.

Looking at the icing events in the HTML reports from NHL ([example](http://www.nhl.com/scores/htmlreports/20162017/PL020001.HTM)) the extracted file gives me individual on-ice events with information about period, time, event type, a description, and who were on the ice: ![alt text](https://github.com/rjweise/RESEARCH---NHL-Icing-and-Coaching-tendencies/blob/master/2018-01-19_11-28-13.png "NHL Play by Play report example")

After extracting this file through R and saving it as a csv to open in Excel, we're looking at the following for the same events: ![alt text](https://github.com/rjweise/RESEARCH---NHL-Icing-and-Coaching-tendencies/blob/master/2018-01-19_11-35-50.png "NHL Play by Play extracted example")
ZOOM IN...
![alt text](https://github.com/rjweise/RESEARCH---NHL-Icing-and-Coaching-tendencies/blob/master/2018-01-19_11-41-15.png)

### The actions that need to happen now are:
* filter out the icing events
* identify and assign the icing team and non-icing team
* identify and assign the icing players on the ice ("committing" the icing) and the ones... uhm... "forcing" the icing?

The icing events can be filtered easily through the etype column: etype = "ICING". It also shows which players were on the ice, and in 99% of the cases the homezone column will specify "Off" or "Def". Assuming this means the zone for the hometeam where the puck was iced from (as in "passed from before it crossed the red goal line"), we can determine what team (but not which player) iced the puck.

To confirm the assumption let's look at the image of the report and the underlying data. After 5 seconds in the game an icing happens (report) in homezone "Off" (data). Since this game is in Ottowa (report) the icing is "passed from" Ottowa's offensive zone (data). The following faceoff is won by Toronto in their defensive zone (report), recorded as "Off" in the homezone column (data). This makes sense, as Toronto iced the puck from their own zone, Ottawa's offensive zone, leading to a faceoff in Toronto's defenisive, Ottoawa's offensive zone.

Still with me?

I checked the lineups for the following faceoffs including ones in the home team's defensive zone, and the logic is confirmed:
* if the homezone is "Off" for an icing event, the away team iced the puck
* if the homezone is "Def" for an icing event, the home team iced the puck

Leaves us with those icing events where the homezone column has a different value then "Off" or "Def"; looking at the data for the 2016-2017 season I found 10,264 icing events (etpe = "ICING"), with the Homezone values divided as follows:
![alt text](https://github.com/rjweise/RESEARCH---NHL-Icing-and-Coaching-tendencies/blob/master/2018-01-19_12-52-22.png)

Not a significant number of records, but still. Looking at some individual occurrences the only helpful data in the report and dataset is the next line, which specifies which team won the ensuing faceoff and **in what zone that faceoff took place**. Unfortunately it seems common that it's actually not the next record. For example:
![Alt text](https://github.com/rjweise/RESEARCH---NHL-Icing-and-Coaching-tendencies/blob/master/2018-01-19_13-01-36.png)

So, if we have an icing event, and the homezone is not "Off" nor "Def" we need to look at the first record after the icing event that is a faceoff event. Isn't this fun!?! But now look at the image again: we have a stopage for icing at 12:12 in the 1st period of this game between Montreal and Buffalo, which - at the same time - is a penalty for Foligno. NHL needs to make more money, so let's have a TV timeout. Ok, we're back, with a Faceoff **at centre ice** ("FAC	MTL won Neu. Zone"). Qu'est-ce que Fuck??? Stop messing with my head, Gary B.! Sure, sure, it's an exception within the 1% of data that is an exception to normal Icing data. No need to start threatening me with moving me to Seatle! :scream: But how a re good-willing amateur analysts supposed to make sense of this?!? Namaste. :pray: I digress...

## Data Process details
I downloaded the Play by Play data for seasons 2010-2011 and after, with the current season up to game 707. Now that I have all the data I need, I load it into a Google Cloud Platform Storage bucket. From there I can load the individual files into Google Big Query, where the data is filtered, combined and manipulated to generate the datasets used in Tableau Public to analyse the data and create the eventual visualization:

1. I first combined all Play by Play files so get a 2010-2018 file ([SQL](https://github.com/rjweise/RESEARCH---NHL-Icing-and-Coaching-tendencies/blob/master/1_CombiningPbPfilesGBQ))
2. Adding columns for IcingTeam, IcedTeam, IcingLineup and IcedLineup, and calculate the values ([SQL](https://github.com/rjweise/RESEARCH---NHL-Icing-and-Coaching-tendencies/blob/master/2_Adding_columns_and_calculate_values))
3. Add SourceGameNR, HomeCaoch and AwayCoach columns, and then coaches to Icing data. Save resulting Table as 3_NHL_PbP_HtmlReports_2010_2018g707_ICINGevents_JOIN_coaches ([SQL](https://github.com/rjweise/RESEARCH---NHL-Icing-and-Coaching-tendencies/blob/master/AddingCoachingDataToIcingDataGBQ))
4. Add columns IcingCoach and IcedCoach to 3_NHL_PbP_HtmlReports_2010_2018g707_ICINGevents_JOIN_coaches and calculate them ([SQL](https://github.com/rjweise/RESEARCH---NHL-Icing-and-Coaching-tendencies/blob/master/AddingCoachingDataToIcingDataGBQ))
5. Create a list of all players in the Icing datasets, combined with season and team. ([SQL](https://github.com/rjweise/RESEARCH---NHL-Icing-and-Coaching-tendencies/blob/master/5_PlayersSeasonsTeamSQL))
6. Join Player list with icing data for only players, and players-season-team and save to 6_Players_2010_2018g707_IcingIcedRatio and 6_Players_Season_Team_2010_2018g707_IcingIcedRatio ([SQL](https://github.com/rjweise/RESEARCH---NHL-Icing-and-Coaching-tendencies/blob/master/6_PlayersNrOfIcingIcedSQL))
7. We'll do the same for Coaches, creating a lost of Coaches and one combined with season and team: 
7_Coaches_Season_Team and 7_Coaches ([SQL](https://github.com/rjweise/RESEARCH---NHL-Icing-and-Coaching-tendencies/blob/master/7_CreateCoachesList))
8. Now create the join between the coaches list and the Icing data, resulting in 8_Coaches_2010_2018g707_IcingIcedRatio and 8_Coaches_Season_Team_2010_2018g707_IcingIcedRatio ([SQL](https://github.com/rjweise/RESEARCH---NHL-Icing-and-Coaching-tendencies/blob/master/8_CoachesNrOfIcingIcedSQL))

Now I have a ll the data I need to work on visualization in Tableau.

## Data Visualization

