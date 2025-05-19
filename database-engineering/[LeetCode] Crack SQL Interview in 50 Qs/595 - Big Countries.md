# Problem: Find Big Countries

## Table

- World(name VARCHAR, continent VARCHAR, area INT, population INT, gdp BIGINT)

## Big country criteria

- area >= 3000000 OR
- population >= 25000000

## Goal
- Return name, population, and area of big countries.

## Solution (SQL)

```sql
SELECT name, population, area
FROM World
WHERE area >= 3000000 OR population >= 25000000;
```

## Notes

- Logical OR used to satisfy either condition.
- GDP and continent are not needed in result.
- No sorting required — return in any order.