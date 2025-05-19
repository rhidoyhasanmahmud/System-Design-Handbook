# Problem: Find Invalid Tweets

## Table
- Tweets(tweet_id INT, content VARCHAR)

## Criteria
- Tweet is invalid if LENGTH(content) > 15

## Goal
- Return tweet_ids of invalid tweets.

## Solution (SQL)

```sql
SELECT tweet_id
FROM Tweets
WHERE LENGTH(content) > 15;
```

## Notes:

- LENGTH() counts characters in content.
- Only need tweet_id of invalid entries.
- Result can be in any order - no ORDER BY required.