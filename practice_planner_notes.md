Practice Planner Notes

**Current Setup:** 

- Current videos are pulled from storage “drills/foldername”  
- Google sheet has title, description and link:  
  - Link currently is simply the link to mytpi website for the drill.  
  - If we were to use the mytpi website video, we should export the embedded video link from there and put it into the google spreadsheet.   
  - That way we could use the embedded video link and load it in an iframe or media player.  
  - Difference in this approach is that we are no longer using videos in storage (these were all placeholders), we would need to refactor the storage so that it references the csv file for the video links and not the storage itself.   
- If the drill does not exist in the “practice\_activities.csv” we fetch the data from the “drills.csv” and then will go and label the golf areas as “off the tee”.  
  - Solution would be to only use one of the csv files, preferably the “practice\_activities.csv” and that is because it has more relevant fields.   
- Saved practice plans are stored as a document in a subcollection under the document for the users collection. 

**Questions:** 

- Is the “social media golf drill database” the desired db to pull drills from or is it the “golf drill database”.  
- What is the difference between drills and games between “drills.csv” and “practice\_activities.csv”.   
  - When do we want to use one over the other? How are these in line with the plan generation?  
- What about plus handicaps? 

**Things to add:** 

- Remove equipment field. ✅  
  - We dont want to remove this since it could lead to users getting drills that require tools they dont have.  
- drop down for time available. ✅  
- Can’t click not sure if others are selected. ✅  
- Is question 5 needed? ✅  
- Replace 5 with 9\. ✅  
- Mobile responsiveness. ✅  
- Adjust time barriers so that it is a slider from 30 minutes to 5 hours. ❗  
- Talk to Jerome about question 7 and fix the boxes to match the csv file. ❓  
- Figure out if activity list is still needed❓  
- Play around with the Iframes, and research embedded social media clips for front end ❓

Walkthrough of functions

1. Calls the generatePracticePlan(userInput)  
   1. UserInput \= {        handicap: parseInt(formData.handicap, 10),  
        daysAvailable:    parseInt(formData.daysAvailable, 10),  
        timePerDay: parseInt(formData.timePerDay, 10),  
         biggestMisses: formData.biggestMisses || \[\],  
         contactIssues: formData.contactIssues || \[\],  
         desiredBallFlight: formData.desiredBallFlight || '',  
         clubsToImprove: formData.clubsToImprove || \[\],  
      }  
   2. Get drills from drills.csv (this will change to be the new spreadsheet once ready  
   3. Practice activities drills are deprecated?  
2. Call generatePracticePlanLogic(userInput,drills, practiceactivities?)  
   1. Call assignDifficultyLevel(handicap)  
      1. This will determine high, medium or low filters for the drills  
         1. Low  \= handicap\<=9  
         2. Medium= 10 \<= handicap \<20   
         3. High \= handicap\>20  
   2. Call filterDrills(drills, difficultylevel)  
      1. TODO: Remove logic for equipment   
      2. TODO: instead of L1-L4, rework so that it matches high, medium, or low (columns: o,p,q for now)  
      3. Once we have filtered by difficulty we should return all drills that fall under the same category of only difficulty (needs to then be filtered more based on other inputs).  
   3. Call filterActicivities(activities,equipment)  
      1. QUESTION: Could potentially be deprecated, based on whether or not we are using practice activities as well.  
      2. Doesn’t actually have anything  
   4. Call scoreDrills(drills, biggestMisses, contactIssues, desiredBallFlight)  
      PREVIOUS APPROACH:  
      1. Takes all of biggestMisses, contactIssues, and desiredBallfight and adds it to a keyword array.   
         1. Search descriptions of each drill, anytime that it has a keyword appear in its description we add a point to its score.   
         2. Sort by score for all of the drills and return list in order. 

		POTENTIAL NEW APPROACH

* We can keep the keyword and description scoring, but also add column searches based off of new format of the spreadsheet  
  * Searches columns with ball flight issues as well as searches columns with contact issues and sees whether or not there is an x in the column for the respected row. 

  5. Call allocateDrillsToDays( scoredDrills,  filteredActivities,daysAvailable,	timePerDay,difficultClubs,clubsToImprove);	  
     1. Parses timePerDay  
        1. POTENTIALLY DEPRECATED:Splits timePerDay in half and allocates 50% to drills and other 50% to practice activities  
        2. POTENTIAL NEW APPROACH: Allocates all time to drills depending on whether or not we are still using practice activities  
     2. Parses clubsToImprove into focusClubs  
        1. POTENTIALLY DEPRECATED: Adds both clubsToImprove as well as difficultClubs to an array  
        2. POTENTIAL NEW APPROACH: Simply use clubsToImprove since difficultClubs is removed.  
     3. Takes all of the other clubs that were not selected and adds them to the end of the new array clubsByPriority, which has them in order based on priority.   
        1. POTENTIAL NEW APPROACH: In questionnaire have the user rank the clubs by priority, that way clubsByPriority is already ranked and passed through in userInput.  
     4. Initialize array allAvailableDrills   
        1. This array is made by doing a forEach on clubsByPriority, and calling getDrillsForClub(scoredDrills, club)  
           1. POTENTIALLY DEPRECATED: Get’s description of drill, and simply adds the drill based on if the club appears in the description.  
           2. POTENTIAL NEW APPROACH: If there is an x in column for section  “Applies to which club or clubs”, then we add the club to it.   
              1. If all clubs are selected then we may need a new heuristic…   
           3. Then remove duplicates and normalize titles in allAvailableDrills  
     5. Get totalDrillTime, averageDrillDuration, and maxAssignmentsPerDrill  
        1. totalDrillTime \=  timeForDrills \* daysAvailable;  
        2. averageDrillDuration \= totalDrillTime / (daysAvailable \* Math.min(numberOfUniqueDrills, daysAvailable));  
        3. maxAssignmentsPerDrill \= Math.max(1, Math.floor(totalDrillTime / (numberOfUniqueDrills \* averageDrillDuration)));  
     6. Once time boundaries are computed we are going to keep track of assignments per activity  
        1. POTENTIALLY DEPRECATED: Depends on whether or not we are going to keep using activities  
     7. Once time boundaries are computed we are going to keep track of assignments per drill  
        1. drillAssignmentCounts, practicePlan, and lastAssignedDay are initialized as empty objects.  
        2. These are going to be used in for loop to keep track of the drill assignments to hold logic and avoid duplicate assignments.   
     8. Create a for loop for the daysAvailable, this is going to iterate each day of the practice plan and assign drills based on the day.   
        1. Array dayPlan is initialized and has objects pushed to it in the following syntax: {type: "drill", drill: drill,time: timeAllocation}  
           1. POTENTIALLY DEPRECATED: Type may no longer be needed depending on if activities are still in use.   
        2. Array clubsToAssign is initialized as …clubsByPriority  
           1. We are going to loop through clubsToAssign, and call getDrillsForClub(scoredDrills, club) and assign it to clubDrills.  
           2. This way we are going by priority this time.   
           3. clubDrills is then used to be passed through to create   
              drill \= selectDrill(  
              clubDrills,  
              assignedDrillsToday,  
              drillAssignmentCounts,  
              maxAssignmentsPerDrill,  
              lastAssignedDay,  
              day,  
              allAvailableDrills  
              )  
           4. Function selectDrill filters drills that haven't exceeded max assignments and not assigned today  
              1. If no drills fit criteria, try to find drills not assigned today.  
              2. If still no drills, check all available drills to prevent duplicates.  
              3. Sort drills to spread out repeats (least recently assigned first)   
                 1. Might need to rework logic in the third case.   
           5. If we have a drill from selectDrill, then we can move on to see the recommended time from the spreadsheet. Then check if the drill fits into the remaining time, if so we call updateDrillAssignment(drill,  
                 assignedDrillsToday,  
                 drillAssignmentCounts,  
                 lastAssignedDay,  
                 day)  
              1. This essentially updates variables such as tracks lastAssignedDay of the drill, increments the drillAssignmentCounts, and adds it to the assignedDrillsToday.  
           6. Loop terminates once we add the dayplan to be stored under practicePlan\[\`Day ${day}\`\]  
     9. Return practicePlan

What needs to be done:

- Filling out new spreadsheet, and add it to the bucket (Drake, in progress)✅  
- Adjust time barriers so that it is a slider from 30 minutes to 5 hours. (Royce, todo)  
- Change all of the questions to Drake’s new questions  (Royce, todo)  
- Figure out if activity list is still needed❓  
- Play around with the Iframes, and research embedded social media clips for front end ❓  
- 

New Questions:

- What is your current handicap? ✅ (check bug with default value)  
- Which clubs do you have available? (Select all that apply) (rework existing question)  
- Which clubs do you want to improve the most? (Numbered 1-5, 1 being highest priority.  Doesn't require all to be selected.) (make it so they can rank them)  
- Which equipment items do you have available? (Select all that apply) ✅  
- Where are you able to practice? (Indoor, Outdoor, course, range, putting green, chipping green)  
- What swing problem areas do you need help with? (mark all that apply, maybe a not sure box)  
- What ball flight issues are you experiencing? (mark all that apply) (Slicing	,Hooking,Pull, Push) ✅  
- Help with Contact issues(mark all that apply)? (Topping the Ball, Fat Shots,Thin Shots, Heel, Toe) (rework options on this one)  
- How many days per week can you practice? \* ✅  
- How much time can you dedicate to practice each day?(must be at least 30 minutes) ✅

Example response.body from questionnaire

{  
	handicap : integer,  
	Clubs\_available: \[string\],  
	Clubs\_improved: \[string\] (ranking is indexed),  
	equipment : \[string\],  
	locations : \[string\],  
	problem\_areas : \[string\],  
	ball\_flight : \[string\],  
	Contact\_issues: \[ string\],  
	Days: integer,  
	time : integer

}