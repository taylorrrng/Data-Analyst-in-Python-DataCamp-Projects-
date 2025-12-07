# 1. Exploring NYC Public School Test Results Score

**a. NYC schools with the best math results**

```python
condition = 0.8 * 800
best_math_schools = (schools.loc
    [
        schools["average_math"] >= condition,
        ["school_name", "average_math"]     
    ]
    .sort_values("average_math", ascending=False)
)
print(best_math_schools);
```

Code outcome:

| Index | school_name                                              | average_math |
|-------|----------------------------------------------------------|--------------|
| 88    | Stuyvesant High School                                   | 754          |
| 170   | Bronx High School of Science                             | 714          |
| 93    | Staten Island Technical High School                      | 711          |
| 365   | Queens High School for the Sciences at York College      | 701          |
| 68    | High School for Mathematics, Science, and Engineering    | 683          |
| 280   | Brooklyn Technical High School                           | 682          |
| 333   | Townsend Harris High School                              | 680          |
| 174   | High School of American Studies at Lehman College        | 669          |
| 0     | New Explorations into Science, Technology and Math (NEST)| 657          |
| 45    | Eleanor Roosevelt High School                            | 641          |

**b. Top 10 performing schools based on combined SAT scores**

```python
schools["total_SAT"] = schools["average_math"] + schools["average_reading"] + schools["average_writing"]
top_10_schools = (schools[["school_name", "total_SAT"]]
                  .sort_values("total_SAT", ascending=False)
                  .head(10)
)
print(top_10_schools);
```

Code outcome:

| Index | school_name                                              | total_SAT |
|-------|----------------------------------------------------------|-----------|
| 88    | Stuyvesant High School                                   | 2144      |
| 170   | Bronx High School of Science                             | 2041      |
| 93    | Staten Island Technical High School                      | 2041      |
| 174   | High School of American Studies at Lehman College        | 2013      |
| 333   | Townsend Harris High School                              | 1981      |
| 365   | Queens High School for the Sciences at York College      | 1947      |
| 5     | Bard High School Early College                           | 1914      |
| 280   | Brooklyn Technical High School                           | 1896      |
| 45    | Eleanor Roosevelt High School                            | 1889      |
| 68    | High School for Mathematics, Science, and Engineering    | 1889      |

**c. Single borough with largest std in combined SAT scores**

```python
stats = (
    schools.groupby("borough")["total_SAT"]
    .agg(num_schools = "count", average_SAT = "mean", std_SAT = "std")
    .round(2)
    .reset_index()
)
largest_std_dev = (stats.loc[
                   stats["std_SAT"].idxmax(),
                   ["borough", "num_schools", "average_SAT", "std_SAT"]]
    .to_frame().T
    .reset_index(drop=True)
)
print(largest_std_dev);
```

Code outcome:

| Borough   | num_schools        | average_SAT | std_SAT  |
|-----------|--------------------|-------------|----------|
| Manhattan | 89                 | 1340.13     | 230.29   |

# 2. Visualizing the History of Nobel Prize Winners

**a. Most commonly awarded gender and birth country**

```python
top_gender = nobel["sex"].value_counts().idxmax()
print("Top gender is", top_gender)

top_country = nobel["birth_country"].value_counts().idxmax()
print("Top country is", top_country)
```
Code outcome:

```
Top gender is Male
Top country is United States of America
```

**b. Decade with the highest ratio**

```python
nobel["decade"] = (nobel["year"] // 10) * 10
decade_counts = nobel["decade"].value_counts().sort_index()

nobel["usborn"] = nobel["birth_country"] == "United States of America"
usborn_counts = nobel[nobel["usborn"]]["decade"].value_counts().sort_index()

ratio = usborn_counts / decade_counts
max_decade_usa = ratio.idxmax()

print("Decade with highest ratio is", max_decade_usa)
```

Code outcome:

```
Decade with highest ratio is 2000
```

**c. Decade and Nobel Prize category had the highest proportion of female winners**

```python
nobel["female"] = nobel["sex"] == "Female"
female_ratio = nobel.groupby(["decade", "category"])["female"].mean()
max_ratio = female_ratio.idxmax()
max_female_dict = {max_ratio[0]: max_ratio[1]}
print("Decade and Nobel Prize are", max_female_dict)
```

Code outcome:

```
Decade and Nobel Prize are {2020: 'Literature'}
```

**d. The first woman to ever win a Nobel Prize**

```python
earliest_year = nobel[nobel["female"]]["year"].min()
first_woman = nobel[(nobel["female"]) & (nobel["year"] == earliest_year)]
first_woman_name = first_woman["full_name"].iloc[0]
first_woman_category = first_woman["category"].iloc[0]
print("First woman's name is", first_woman_name)
print("Her category is", first_woman_category)
```
Code outcome:

```
First woman's name is Marie Curie, née Sklodowska
Her category is Physics
```

**e. Individuals with more than 1 prize**

```python
name_counts = nobel["full_name"].value_counts()
repeat_list = name_counts[name_counts > 1].index.tolist()
print("Individuals with more than 1 Nobel Prize are", repeat_list)
```

Code outcome:

```
Individuals with more than 1 Nobel Prize are ['Comité international de la Croix Rouge (International Committee of the Red Cross)', 'Linus Carl Pauling', 'John Bardeen', 'Frederick Sanger', 'Marie Curie, née Sklodowska', 'Office of the United Nations High Commissioner for Refugees (UNHCR)']
```

# 3. Analyzing Crimes in Los Angeles

**a. Import required libraries**

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
crimes = pd.read_csv("crimes.csv", dtype={"TIME OCC": str})
crimes.head()
```

Code outcome:

| Index | DR_NO     | Date Rptd  | Date Occ   | Time Occ | Area Name   | Crime Description   | Vict Age | Vict Sex | Vict Descent | Weapon Desc | Status Desc | Location                         |
|-------|-----------|------------|------------|----------|-------------|----------------------|----------|----------|--------------|-------------|-------------|----------------------------------|
| 0     | 220314085 | 2022-07-22 | 2020-05-12 | 1110     | Southwest   | THEFT OF IDENTITY    | 27       | F        | B            | NaN         | Invest Cont | 2500 S SYCAMORE AV               |
| 1     | 222013040 | 2022-08-06 | 2020-06-04 | 1620     | Olympic     | THEFT OF IDENTITY    | 60       | M        | H            | NaN         | Invest Cont | 3300 SAN MARINO ST               |
| 2     | 220614831 | 2022-08-18 | 2020-08-17 | 1200     | Hollywood   | THEFT OF IDENTITY    | 28       | M        | H            | NaN         | Invest Cont | 1900 TRANSIENT                   |
| 3     | 231207725 | 2023-02-27 | 2020-01-27 | 635      | 77th Street | THEFT OF IDENTITY    | 37       | M        | H            | NaN         | Invest Cont | 6200 4TH AV                       |
| 4     | 220213256 | 2022-07-14 | 2020-07-14 | 900      | Rampart     | THEFT OF IDENTITY    | 79       | M        | B            | NaN         | Invest Cont | 1200 W 7TH ST                     |

**b. Hour with highest frequency of crimes**

```python
crimes["TIME_STR"] = crimes["TIME OCC"].str[:2] + ":" + crimes["TIME OCC"].str[2:]
crimes["TIME"] = pd.to_datetime(crimes["TIME_STR"], format="%H:%M").dt.time
crimes["Hour"] = pd.to_datetime(crimes["TIME_STR"], format="%H:%M").dt.hour
peak_crime_hour = crimes["Hour"].value_counts().idxmax()
print("Peak crime hour:", peak_crime_hour)
```

Code outcome:

```
Peak crime hour: 12
```

**c. Area with largest frequency of night crimes**

```python
crimes["NIGHTTIME"] = crimes["Hour"].apply(lambda x: True if (x >= 22 or x <= 3) else False)
night_crimes = crimes[crimes["NIGHTTIME"] == True]
peak_night_crime_location = night_crimes["AREA NAME"].value_counts().idxmax()
print("Area with most night crimes:", peak_night_crime_location)
```

Code outcome:

```
Area with most night crimes: Central
```

**d. Number of crimes committed against each age group**

```python
crimes["Vict Age"] = crimes["Vict Age"].astype(int)
crimes = crimes[crimes["Vict Age"] >= 0]
group_0_17 = crimes[(crimes["Vict Age"] >= 0) & (crimes["Vict Age"] <= 17)]
group_18_25 = crimes[(crimes["Vict Age"] >= 18) & (crimes["Vict Age"] <= 25)]
group_26_34 = crimes[(crimes["Vict Age"] >= 26) & (crimes["Vict Age"] <= 34)]
group_35_44 = crimes[(crimes["Vict Age"] >= 35) & (crimes["Vict Age"] <= 44)]
group_45_54 = crimes[(crimes["Vict Age"] >= 45) & (crimes["Vict Age"] <= 54)]
group_55_64 = crimes[(crimes["Vict Age"] >= 55) & (crimes["Vict Age"] <= 64)]
group_65_plus = crimes[crimes["Vict Age"] >= 65]
count_0_17 = len(group_0_17)
count_18_25 = len(group_18_25)
count_26_34 = len(group_26_34)
count_35_44 = len(group_35_44)
count_45_54 = len(group_45_54)
count_55_64 = len(group_55_64)
count_65_plus = len(group_65_plus)
victim_ages = pd.Series({
    "0-17": count_0_17,
    "18-25": count_18_25,
    "26-34": count_26_34,
    "35-44": count_35_44,
    "45-54": count_45_54,
    "55-64": count_55_64,
    "65+": count_65_plus
})
print("Number of crimes per victime age group:", victim_ages)
```

Code outcome:

```
Number of crimes per victime age group: 0-17      4528
18-25    28291
26-34    47470
35-44    42157
45-54    28353
55-64    20169
65+      14747
dtype: int64
```

# 4. Hypothesis Testing with Men's and Women's Soccer Matches

**a. Data preparation**

```python
import pandas as pd
women = pd.read_csv('women_results.csv')
men = pd.read_csv('men_results.csv')
print(women.info())
women.head(5)
```

Code outcome:

```
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 4884 entries, 0 to 4883
Data columns (total 7 columns):
 #   Column      Non-Null Count  Dtype 
---  ------      --------------  ----- 
 0   Unnamed: 0  4884 non-null   int64 
 1   date        4884 non-null   object
 2   home_team   4884 non-null   object
 3   away_team   4884 non-null   object
 4   home_score  4884 non-null   int64 
 5   away_score  4884 non-null   int64 
 6   tournament  4884 non-null   object
dtypes: int64(3), object(4)
memory usage: 267.2+ KB
None
```

| index | Unnamed: 0 | date       | home_team | away_team | home_score | away_score | tournament        |
|-------|-------------|------------|-----------|-----------|------------|------------|-------------------|
| 0     | 0           | 1969-11-01 | Italy     | France    | 1          | 0          | Euro              |
| 1     | 1           | 1969-11-01 | Denmark   | England   | 4          | 3          | Euro              |
| 2     | 2           | 1969-11-02 | England   | France    | 2          | 0          | Euro              |
| 3     | 3           | 1969-11-02 | Italy     | Denmark   | 3          | 1          | Euro              |
| 4     | 4           | 1975-08-25 | Thailand  | Australia | 3          | 2          | AFC Championship  |

**b. Filtering the data**

```python
women_fifa_2002 = women[
    (women['date'] > '2002-01-01') &
    (women['tournament'] == 'FIFA World Cup')
]

men_fifa_2002 = men[
    (men['date'] > '2002-01-01') &
    (men['tournament'] == 'FIFA World Cup')
]
women_fifa_2002.head(5)
```

Code outcome:

| index | Unnamed: 0 | date       | home_team      | away_team   | home_score | away_score | tournament      |
|-------|-------------|------------|----------------|-------------|------------|------------|-----------------|
| 1600  | 1600        | 2003-09-20 | Nigeria        | North Korea | 0          | 3          | FIFA World Cup  |
| 1601  | 1601        | 2003-09-20 | Norway         | France      | 2          | 0          | FIFA World Cup  |
| 1602  | 1602        | 2003-09-20 | Germany        | Canada      | 4          | 1          | FIFA World Cup  |
| 1603  | 1603        | 2003-09-20 | Japan          | Argentina   | 6          | 0          | FIFA World Cup  |
| 1604  | 1604        | 2003-09-21 | United States  | Sweden      | 3          | 1          | FIFA World Cup  |

**c. Choose correct hypothesis test**

```python
women_fifa_2002['goals'] = women_fifa_2002['home_score'] + women_fifa_2002['away_score']
men_fifa_2002['goals'] = men_fifa_2002['home_score'] + men_fifa_2002['away_score']
women_fifa_2002.head(5)
```

Code outcome:

| index | Unnamed: 0 | date       | home_team     | away_team   | home_score | away_score | tournament      | goals |
|-------|-------------|------------|---------------|-------------|------------|------------|------------------|-------|
| 1600  | 1600        | 2003-09-20 | Nigeria       | North Korea | 0          | 3          | FIFA World Cup   | 3     |
| 1601  | 1601        | 2003-09-20 | Norway        | France      | 2          | 0          | FIFA World Cup   | 2     |
| 1602  | 1602        | 2003-09-20 | Germany       | Canada      | 4          | 1          | FIFA World Cup   | 5     |
| 1603  | 1603        | 2003-09-20 | Japan         | Argentina   | 6          | 0          | FIFA World Cup   | 6     |
| 1604  | 1604        | 2003-09-21 | United States | Sweden      | 3          | 1          | FIFA World Cup   | 4     |

**d. Goal distribution comparison for men vs. women (FIFA 2002)**

```python
plt.hist(women_fifa_2002['goals'], bins=20, alpha=0.6, label='Women')
plt.hist(men_fifa_2002['goals'], bins=20, alpha=0.6, label='Men')
plt.legend()
plt.xlabel('Goals per match')
plt.ylabel('Frequency')
plt.title('Women & Men Goals Distribution')
plt.show()
```

Code outcoome:

<img width="615" height="465" alt="Screen Shot 2025-12-07 at 4 01 43 PM" src="https://github.com/user-attachments/assets/61676348-b97e-478d-a4f3-3fdb378987db" />

**e. Normality test + Mann-Whitney U comparison of men vs. women goals**

```python
from scipy.stats import shapiro
shapiro_women = shapiro(women_fifa_2002['goals'])
shapiro_men = shapiro(men_fifa_2002['goals'])
print(shapiro_women, shapiro_men)
```

Code outcome:

```
ShapiroResult(statistic=0.8491013050079346, pvalue=3.8905201759850683e-13) ShapiroResult(statistic=0.9266489744186401, pvalue=8.894154401688226e-13)
```

```python
import pingouin
mwu_test = pingouin.mwu(x=women_fifa_2002['goals'],
            y=men_fifa_2002['goals'],
            alternative='greater')
print(mwu_test)
```

Code outcome:

```
       U-val alternative     p-val       RBC      CLES
MWU  43273.0     greater  0.005107 -0.126901  0.563451
```

```python
p_val = mwu_test['p-val'][0]
alpha = 0.1
result = 'reject' if p_val < alpha else 'fail to reject'
result_dict = {"p_val": p_val,
              "result": result}
result_dict
```

Code outcome:

```
{'p_val': 0.005106609825443641, 'result': 'reject'}
```
