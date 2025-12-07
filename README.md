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

| Index | School Name                                             | Average Math |
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

| Index | School Name                                             | Total SAT |
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

| Borough   | Number of Schools | Average SAT | Std. SAT |
|-----------|--------------------|-------------|----------|
| Manhattan | 89                 | 1340.13     | 230.29   |

# 2. Visualizing the history of Nobel Prize winners

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
