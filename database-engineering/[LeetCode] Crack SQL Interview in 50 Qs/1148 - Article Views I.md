## Problem: Authors Who Viewed Their Own Articles

# Table 

- Views(article_id INT, author_id INT, viewer_id INT, view_date DATE)

## Goal

- Return distinct author IDs (id) who have viewed their own articles (i.e., author_id = viewer_id)

##  Solution (SQL)

```sql
SELECT DISTINCT author_id AS id
FROM Views
WHERE author_id = viewer_id
ORDER BY id;
```

## Notes:
- DISTINCT to avoid duplicates (as table may have repeated views).
- WHERE author_id = viewer_id finds self-views.
- ORDER BY id ensures ascending order as required.