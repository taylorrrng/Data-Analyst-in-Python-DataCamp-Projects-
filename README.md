# Exploring NYC Public School Test Results Score

**1. NYC schools with the best math results:**

```python
condition = 0.8 * 800
best_math_schools = (schools.loc
    [
        schools["average_math"] >= condition,
        ["school_name", "average_math"]     
    ]
    .sort_values("average_math", ascending=False)
)
print(best_math_schools); ```

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

