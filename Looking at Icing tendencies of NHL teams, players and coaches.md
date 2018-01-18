# Looking at Icing tendencies of NHL teams, players and coaches

This research is the result of a conversation with @RobVollmanNHL and looks at the last couple of NHL season for icing committed by teams, players and coaches.

## Goal

I'm no expert by any means on hockey, but generally speaking an icing is a negative event; icing tires out the team, it allows the opposing team to change their lineup and pick their preferred match-ups, and the face-off following icing is in the defensive zone of the icing team, with a roughly 50% change of regaining possesion of the puck (and thus a 50% chance of giving the puck to your opponent in their attacking zone).

A first step in this research showed me what players iced the puck most, and I then aggregated that data per team. It made me wonder what the influence of the coach was on these icing tendiencies; do some prefer the temporary relief that icing can provide over the potentially negative consequences described above? Does it just come with a certain playing style coaches prefer?

## Data
To do this anlysis I needed data for icing events, which team committed the icing, what players were on the ice, and who was coaching that team. For the icing events I use the R package NHLscrapr from A.C. Thomas and Samuel L. Ventura ([link](https://cran.r-project.org/web/packages/nhlscrapr/index.html)) to pull play-by-play data from the NHL's HTML reports per game ([example]( http://www.nhl.com/scores/htmlreports/20172018/PL020043.HTM)). This gives me the events per game(-number), the players on ice for each team, the teams, and to a certain degree the team that iced the puck (more to follow below).

Still missing were the coaches of the teams for which I used different R-code that reads the NHL API Life Feed JSON files ([example](http://statsapi.web.nhl.com/api/v1/game/2017020019/feed/live)) and pulls both coaches and the game number, which can be linked to the data described above.

This data is combined and manipulated in Google Big Query afterwards to generate the datasets used in Tableau Public to analyse the data and create the eventual visualization. The "icing team" was derived from the value in the "homezone" field. And depending on if the icing team was the home or away team, the "icing lineup" was created by concatenating the home or away lineup.

I should note that to my knowledge it is impossible to determine for every icing which team iced the puck; the icing team can not be determined when the "homezone" field contained other values then "Def" or "Off" (like "", "Neu" and "Unk"). Checking on a subset of all data I found about 1% of the data contained these values, and were excluded from the research as they could not be assigned to a team or player.

## Process details
