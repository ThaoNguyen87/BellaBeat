# BellaBeat
This is a data analysis project to practice SQL, spreadsheet, Tableau and R. It is a capstone project in Google Data Analytics courses.   

## 1.	Summary of the business task
Bellabeat has a wide range of wellness products targeting woman who are conscious about their health. It includes an app that collects data on activity, sleep, stress, reproductive health, and mindfulness habits. The data collected through the app reflects how the users use the company’s product, their daily wellness habits which the company needs to understand. Bellabeat’s management team require this data to be analyzed to provide the insights that help serve their customers better. These insights will guide marketing strategies and help Bellabeat management team explore opportunities to improve their revenue.
The key stakeholders here are: Chief Creative officer, key member of executive team and marketing analytics team.

## 2.	Description of data sources used  
•	**Storage**:  the data is stored in .csv file in local drive. The master files are: dailyActivities_merged.csv, heartrate_seconds_merged.csv, sleepDay_merged.csv  
•	**Format**: Some files are in wide format for e.g minuteCaloriesWide_merged.csv, minuteIntensityWide_merged.csv, while others are in long format for e.g dailyActivities_merged.csv, dailyIntensities_merged.csv, dailySteps_merged.csv, heartrate_seconds_merged.csv etc.   
•	**Data Integrity**: There are some missing data about sleep: there are no records on some days for some users, no records about weight for some users.
There are 33 records for 33 users although it was stated there are only 30 participants.  
•	**Data organizations**: there are 3 categories about this data set:  
| Activity Data | Sleep Data | Weight Info
|---------------|------------|------------
|• Number of Steps| •Total sleep time (in minutes) per day | • Weight in Kg
|•	Distance|•	Total time in bed (in minutes) per day| •	BMI
|  1.	Very Active|    
|  2.	Moderately Active|  
|   3.	Light Active | 
 |  4.	Sedentary  |
 |•	Time (in mins) |  
 |  1.	Very Active  |
 |  2.	Fairly Active | 
 |  3.	Lightly Active | 
 |  4.	Sedentary  |
 |•	Calories|
  
There are 3 main data files that I will focus on:  
1)	dailyActivity_merged  
2)	sleepDay_merged  
3)	weightLogInfo_merged  


## 3.	Documentation of data cleaning and manipulation  
**DailyActivity_merged**
-	First I check if the data have any duplicates. Using the “Remove duplicates” function in excel to see if any rows are identical, then remove them. The result is 0 duplicates.  
-	Check for missing values. The result: no missing values  
-	Check for inconsistent data type: The result: all consistent data type, no string mixed up where numerical values are expected.  
-	Check for leading and trailing spaces. The result is no extra spaces.  
-	Using BigQuery to check on the number of distinct users that upload their health data, there are total of 33 distinct users  

 **sleepDay_merged**
 -	Cleaning the SleepDate column by removing the time and AM. Only the d/mm/yyyy string remains. Using LEFT function to take in the first characters until the first empty space encountered.  
-	Check for duplicate rows. Using “Remove duplicates” function in excel or using SQL to check for duplicate rows. Total 3 duplicate rows removed.    
  
 ```{r}  
SELECT Id,  
        SleepDay,  
        TotalSleepRecords,  
        TotalTimeInBed,  
        TotalMinutesAsleep,  
        count(*)  
FROM `deductive-nexus-390616.FitBit_Fitness.SleepDay`  
GROUP BY Id,SleepDay,TotalSleepRecords,TotalTimeInBed,TotalMinutesAsleep  
HAVING COUNT(*) > 1 
```

**weightLogInfo_merged**  
-	Check for number of distinct logged in users: only 8 users logged in their weight info
  
```{r}
Select count(distinct Id),  
From `deductive-nexus-390616.FitBit_Fitness.WeightLog`
```
  
-	Check for duplicates: 0 duplicates  
```{r}
Select Id,  
        Date,  
        WeightKg,  
        WeightPounds,  
        Fat,  
        BMI,  
        IsManualReport,  
        LogId,  
        count(*),  
From `deductive-nexus-390616.FitBit_Fitness.WeightLog`  
Group By Id,Date,WeightKg,WeightPounds,Fat,BMI,IsManualReport,LogId  
Having count(*)>1  
```  
-	Extract date and time from Date. For simplicity purposes we keep the date portion from the timestamp by using Excel function LEFT and SEARCH for the first space delimiting the date and time part.
  
## 4. Analysis and visualization  
1.	We want to know how many steps users do on each day of the week. This is to check on their workout routine on a daily basis.
```{r}
SELECT   
  CASE   
      WHEN extract(DAYOFWEEK from da.ActivityDate)=1 THEN 'Sunday'  
      WHEN extract(DAYOFWEEK from da.ActivityDate)=2 THEN 'Monday'  
      WHEN extract(DAYOFWEEK from da.ActivityDate)=3 THEN 'Tuesday'  
      WHEN extract(DAYOFWEEK from da.ActivityDate)=4 THEN 'Wednesday'  
      WHEN extract(DAYOFWEEK from da.ActivityDate)=5 THEN 'Thursday'  
      WHEN extract(DAYOFWEEK from da.ActivityDate)=6 THEN 'Friday'  
      WHEN extract(DAYOFWEEK from da.ActivityDate)=7 THEN 'Saturday'  
  END AS DayOfWeek,  
  Round(Avg(TotalSteps))  
FROM FitBit_Fitness.DailyActivity da  
GROUP BY DayOfWeek  
```  
Below is the result of the query: 
|DayOfWeek	|AverageSteps
|-----------|------------
|Sunday	|6933	  
|Thursday |	7406	  
|Tuesday	|8125	  
|Saturday	|8153	  
|Wednesday	|7559	  
|Monday	|7781	  
|Friday	|7448	  
  
Using Tableau to plot the steps per day of week:   

    
![daily_steps](https://github.com/ThaoNguyen87/BellaBeat/assets/161806127/6d82fc28-9a20-42d9-8b65-c362eac43176)


-	What we are seeing here is Tuesday and Saturday are the 2 days that users do the most work out, on average 8125 steps on Tuesday and 8153 steps on Saturday. High concentration of work out is therefore observed on these 2 days.   
-	The lowest concentration of work out is observed on Sunday with only 6933 steps on average. It is suggested that a marketing campaign to run on Sunday to encourage users to work out more.


2.	We want to do a summary of min, max, average on the Total Steps, Very Active Minutes, Fairly Active Minutes, Lightly Active Minutes, Sedentary Minutes, Calories, Total Minutes Asleep, Total Time In Bed and BMI

```
install.packages("tidyverse")  
install.packages("dplyr")  
library(tidyverse)  
library(dplyr)  
  
dailyActivity <- read.csv("dailyActivity_merged.csv")  
sleep <- read.csv("sleepDay_merged.csv")  
weight <- read.csv("weightLogInfo_merged.csv")  
  
dailyActivity %>% select(TotalSteps,VeryActiveMinutes,FairlyActiveMinutes,  
                         LightlyActiveMinutes,SedentaryMinutes, Calories) %>% summary()  
                
sleep %>% select(TotalMinutesAsleep, TotalTimeInBed) %>% summary()              
             weight %>% select(BMI) %>% summary()  
```


The result of the above summary:
```
 TotalSteps      VeryActiveMinutes    FairlyActiveMinutes    LightlyActiveMinutes  
 Min.   :    0    Min.   :  0.00      Min.   :  0.00         Min.   :  0.0         
 1st Qu.: 3790    1st Qu.:  0.00      1st Qu.:  0.00         1st Qu.:127.0         
 Median : 7406    Median :  4.00      Median :  6.00         Median :199.0         
 Mean   : 7638    Mean   : 21.16      Mean   : 13.56         Mean   :192.8         
 3rd Qu.:10727    3rd Qu.: 32.00      3rd Qu.: 19.00         3rd Qu.:264.0         
 Max.   :36019    Max.   :210.00      Max.   :143.00         Max.   :518.0    
  
       
 SedentaryMinutes    Calories      TotalMinutesAsleep	TotalTimeInBed  
 Min.   :   0.0   Min.   :   0     Min.   : 58.0 	Min.   : 61.0  
 1st Qu.: 729.8   1st Qu.:1828     1st Qu.:361.0 	1st Qu.:403.8  
 Median :1057.5   Median :2134     Median :432.5 	Median :463.0  
 Mean   : 991.2   Mean   :2304     Mean   :419.2 	Mean   :458.5  
 3rd Qu.:1229.5   3rd Qu.:2793     3rd Qu.:490.0	3rd Qu.:526.0  
 Max.   :1440.0   Max.   :4900 	   Max.   :796.0 	Max.   :961.0  
  
 BMI         
 Min.   :21.45    
 1st Qu.:23.96    
 Median :24.39    
 Mean   :25.19    
 3rd Qu.:25.56    
 Max.   :47.54   
```
  3.	The following pie charts represents the percentage of VerActiveMinutes, FairlyActiveMinutes, LightlyActiveMinutes and SedentaryMinutes for easy visualization.  
We also illustrate the amount of time users spend awake on bed and asleep.
```
# Create data for the graph.  
data <- c(21.16, 13.56, 192.8, 991.2)  
labels <- c("Very Active", "Fairly Active", "Lightly Active", "Sedentary")  
Sleep_Data <- c(7.23,0.41)  
Sleep_Labels <- c("Total Hour Asleep","Total Hour Awake on Bed")  
  
piepercent<- round(100*data/sum(data), 1)  
  
# Give the chart file a name.  
png(file = "Activity Chart.jpg")
png(file = "Sleep Chart.jpg")

# Plot the chart.
pie(data, labels = piepercent, main = "Activity chart",col = rainbow(length(data)))
legend("topright", c("Very Active Minutes", "Fairly Active Minutes", "Lightly Active Minutes", "Sedentary Minutes"), cex = 0.8,
       fill = rainbow(length(data)))
pie(Sleep_Data, labels = c("7.23 hour","25 minutes"), main = "Sleep chart",col = rainbow(length(Sleep_Data)))
legend("topright", c("Total Time Asleep","Total Time Awake on Bed"), cex = 0.8,
       fill = rainbow(length(Sleep_Data)))

# Save the file.
dev.off()
```
![Sleep Chart](https://github.com/ThaoNguyen87/BellaBeat/assets/161806127/9856f7d1-20bf-4924-b3e8-6c4595ac7947)
![Activity Chart](https://github.com/ThaoNguyen87/BellaBeat/assets/161806127/efa3a579-f51b-4bdc-a721-3957fd794fa3)

-	As per CDC recommendation, adults should spend 10,000 steps a day however we observe that the mean steps taken is 7638 which is less than the recommendation. More encouraging marketing campaigns can be aimed to stimulate and reward individuals to take more steps daily to reach the goal.
  
-	CDC also recommends 150 minutes of moderately active per week, which works out to be 21.4 minutes per day. Under FairlyActiveMinutes and VeryActiveMinutes, the median stands at 6 minutes and 4 minutes respectively, which adds up to 10 minutes for moderately active time per day. Users only achieve half of the amount recommended by CDC. Again, we can think of rewards program to encourage users achieve more active exercise time.
  
-	Users spend 16.5 hours on average a day being sedentary. If we account for 7 hours sleeping, that is almost all the awake time being sedentary.   
  
-	On average, users spend more than 7 hours asleep, and it takes them on average 39 minutes to fall to sleep. It shows users have adequate sleep. However, 39 minutes to fall asleep is a long duration, longer than most (15-20 minutes). Perhaps some users spend time playing on their device before sleeping. We can implement an alarm feature to remind them to sleep after 15 minutes on the bed.  
  
-	The Mean BMI is 24.39 and Median is 25.19 show a healthy BMI recording for our users.  
  
    
  
   
  4.	Let us explore the relationship between the level of activity and sleep. Will more activity lead to more sleep and vice versa? From the chart below, we do not see a relationship between the two.

  
```
  ggplot(data=merged_Activity_Sleep_Weight) + 
         geom_point(aes(x=TotalMinutesAsleep, y=TotalSteps, color=TotalMinutesAsleep))+
         labs(title="Total MinutesAsleep vs. TotalSteps")
```

![Sleep_steps Chart](https://github.com/ThaoNguyen87/BellaBeat/assets/161806127/abb69110-536f-43a1-9263-778a688a44a4)

  


5.	Let us explore the relationship between the level of activity and calories. Will more activity leads to more calories burnt?


       
```
ggplot(data=merged_Activity_Sleep_Weight) + 
  geom_point(aes(x=TotalSteps, y=Calories, color=TotalSteps)) +
  geom_smooth(aes(x=TotalSteps, y=Calories)) +
               labs(title="Total Steps vs. Calories")
```
  
![Steps_Calories](https://github.com/ThaoNguyen87/BellaBeat/assets/161806127/789f9943-5a83-49fd-a058-d70d2de49355)


The chart above shows a clear trend line and correlation between the number of steps users take and their burn calories. The more steps they take, the more calories burnt and vice versa. There are some outliners: people who take high number of steps but burn less calories than people who take lesser number of steps but burn more calories.  



    
6.	Now we explore the relationship between the number of steps taken and the level of activities.
```
ggplot(data=merged_Activity_Sleep_Weight, aes(x=TotalSteps)) + 
  geom_point(aes(y=VeryActiveMinutes, color = "VeryActiveMinutes")) +
  geom_smooth(aes(y=VeryActiveMinutes, color = "VeryActiveMinutes")) +
  geom_point(aes(y=FairlyActiveMinutes, color = "FairlyActiveMinutes")) +
  geom_smooth(aes(y=FairlyActiveMinutes, color = "FairlyActiveMinutes")) +
  geom_point(aes(y=LightlyActiveMinutes, color ="LightlyActiveMinutes")) +
  geom_smooth(aes(y=LightlyActiveMinutes, color ="LightlyActiveMinutes")) +
  geom_point(aes(y=SedentaryMinutes, color = "SedentaryMinutes")) +
  geom_smooth(aes(y=SedentaryMinutes, color = "SedentaryMinutes")) +
  
  labs(title="Total Steps vs. Active Minutes", x ="Total Steps", y = "Active Minutes") +
  annotate("text",x=30000, y = 1000, label="sedentary", color="black") +
  annotate("text",x=30000, y=400, label="Lightly Active", color="black") +
  annotate("text",x=30000, y=100, label="Very Active", color="black") +
  annotate("text",x=37000, y=50, label="Fairly Active", color="black") 
```


![Steps_ActiveMinutes](https://github.com/ThaoNguyen87/BellaBeat/assets/161806127/36a487af-2074-4221-952e-ab4083e8a713)


The chart shows the majority are taking less than 10000 steps a day and 12-18 hours in sedentary, 5 hours in lightly active activity and ~ 1 hour in very active/fairly active activity.  



7.	We will now explore the relationship between the number of steps taken and its effect on weight. Will people who take more steps a day will have a better weight?


```
ggplot(data=merged_Activity_Sleep_Weight,aes(x=TotalSteps, y=BMI, color=BMI)) + 
  geom_point()
```


![Steps_BMI](https://github.com/ThaoNguyen87/BellaBeat/assets/161806127/c048ff8c-66f9-41c6-8d27-9b8dfcd0de8d)  


We observe the majority of users have a healthy BMI of 23-25, many of whom take daily average around 10,000 steps. There are a few outliners that have unhealthy BMI (the high of 50s) and these users take less than 5000 steps a day. It indicates the more steps one takes daily, the more likely you can have keep BMI in a healthy range.  


## 5.	Key takeaways for shareholders 

  
-	To ensure level of activity are distributed evenly throughout the week, the company can add a feature on the device to remind users to exercise more on Sunday, Thursday and Friday.  
  
-	More encouraging marketing campaigns can be aimed to stimulate and reward individuals to take more steps daily to reach the goal, as data suggests users on average spend only 7768 steps per day.  
  
-	A reward program can be targeted to users who achieve less than the recommended moderately active minutes per day (20-30 minutes/day). Data suggests that half of the users achieved only 10 minutes of moderately active time.  
  
-	Since user spend almost 40 minutes to fall asleep, this amount of time could be attributed to device surfing time. We can implement an alarm feature to remind them to sleep after 15 minutes on the bed.  
  



    
  
  

  


