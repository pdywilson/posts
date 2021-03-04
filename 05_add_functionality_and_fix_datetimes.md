# Data Engineering Toy Project
## Add functionality and fix datetimes

I found a funny bug in my project today. The website usually updates every hour, printing a timestamp of the last update as proof. I saw it hadn't updated since Feb 28, 11 pm and it was already March. The code that retrieves the current data from the database was the following SQL call:

```
select * from dublinrents order by timestamp desc limit 1;
```

The problem with this is that I had created my `timestamp` column as a text column and the order by was retrieving the lexicographically ordered latest, which of course doesn't make any sense. So I had to change the dates from text to datetimes, which I used the following transformation:

```
select datetime(substr(timestamp,7,4)||'-'||substr(timestamp,4,2)||'-'||substr(timestamp,1,2)||' '||substr(timestamp,12,8)) from dublinrents order by timestamp desc limit 1;
```

I updated my tables to have the timestamp column of type datetime and we're back up-to-date.
