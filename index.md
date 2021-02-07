# Data Engineering Toy Project
## A minimal example of deploying a data pipeline

Almost one year into quarantine here in Dublin I was wondering whether the notoriously high rent prices in Dublin might actually be going down at the moment. To investigate this I wanted to set up a data processing pipeline that automatically updates a webpage with the current status of monthly rent prices in Dublin, all while trying out some cool new frameworks: google compute cloud, airflow, spark, firebase.

So the first step was to get data on the monthly rents in Dublin. All I found in terms of datasets was the RTB Average Rent Dataset https://www.rtb.ie/research/average-rent-dataset. This dataset gives the average rents for each of Dublin's districts at a quarterly basis. With the last update being Q3-2020 this wasn't up-to-date enough for me, I wanted a more current status of the housing market. So the next idea I had was to crawl daft.ie (Ireland's most used property website) for a more current status on the rents.

The quickest and easiest way to crawl daft.ie for their property prices was to use the python package `autoscraper`. 
```
from autoscraper import AutoScraper
import re

url = 'https://www.daft.ie/property-for-rent/dublin-city/apartments?numBeds_to=2&sort=publishDateDesc'

wanted_list = [re.compile('€.*per month')]

scraper = AutoScraper()
result = scraper.build(url, wanted_list)

for i in range(20,2000,20): #
    url = 'https://www.daft.ie/property-for-rent/dublin-city/apartments?numBeds_to=2&sort=publishDateDesc&from={}&pageSize=20'.format(i)
    scraper = AutoScraper()
    result = result + scraper.build(url, wanted_list)

print("Scraped {} rents.".format(len(result)))
```

This will give me a list of everything that looks like the regular expression '€.*per month' on the webpage. The search is based on the html code that is pulled from the daft.ie url in which I specify my search on apartments for rent in Dublin-City with maximum number of beds of 2. The maximum amount of apartments shown on one page of the daft search is 20 so I go through the list of apartments in increments of 20 with a for loop. The result looks something like this:

    ['€1,501 per month', '€1,800 per month', '€2,000 per month', 'A better kind of renting',...]

Now we need to process this list to get a list of integers so that we can calculate the average and median.

```
numbers = []
for elt in result:
    try:
        numbers.append(int(elt[1:].split(" ")[0]))
    except:
        try:
            numbers.append(int(elt[1:].split(" ")[0].split(",")[0]+elt[1:].split(" ")[0].split(",")[1]))
        except:
            #print(elt)
            pass
print("Processed {} rents.".format(len(numbers)))

avg = sum(numbers)/len(numbers)
median = sorted(numbers)[round(len(numbers)/2)]
```

This simple string processing takes out all the numbers from the 'per month' strings in the list after removing the ',' from e.g. 1,500 as well as unwanted matches that we don't need, e.g. 'A better kind of renting'. The result is a list of integers. Finally, we can calculate the average monthly rent as well as the median.

I was thinking it might be good practice to store this in a database so I imported sqlite3 and created `rent.db`

```
import sqlite3

conn = sqlite3.connect('/content/drive/My Drive/code/rent project/db/rent.db')

cur = conn.cursor()

cur.execute("SELECT * FROM sqlite_master WHERE type='table'")

create_table_sql = """ CREATE TABLE IF NOT EXISTS dublinrents (
                                        timestamp string PRIMARY KEY,
                                        avg float NOT NULL,
                                        median float NOT NULL
                                    ); """

if conn is not None:
    try:
        c = conn.cursor()
        c.execute(create_table_sql)
    except Error as e:
        print(e)

else:
    print("Error! cannot create the database connection.")
```

Together with a datetime stamp I can now save my current average and median into my SQL database.

```
from datetime import datetime
now = datetime.now()
dt_string = now.strftime("%d/%m/%Y %H:%M:%S")
print("Datestamp now: {}".format(dt_string))

#import average mean median etc to sql
def insert_to_db(dt_string, avg, median):
    sql = ''' INSERT INTO dublinrents
                VALUES(?,?,?) '''
    cur = conn.cursor()
    cur.execute(sql, (dt_string,avg,median))
    conn.commit()

insert_to_db(dt_string,avg,median)
print("updated db")
```

After running all of the above we can query our `rent.db` for the latest average monthly rent in Dublin.

```
import sqlite3

conn = sqlite3.connect('/content/drive/My Drive/code/rent project/db/rent.db')

sql = ''' SELECT * FROM dublinrents ORDER BY timestamp DESC LIMIT 1'''
cur = conn.cursor()
r = cur.execute(sql)
current = r.fetchall()
curr_timestamp = current[0][0]
curr_avg = round(current[0][1])
curr_median = round(current[0][2])

print("The current rent average is €{}, the median is €{} per month. - {}".format(curr_avg,curr_median,curr_timestamp))
```

which yields something like this:

```
The current rent average is €1855, the median is €1800 per month. - 06/02/2021 21:10:28
```

Warning: This next part may sound a bit technical and confusing, but it was really surprisingly easy to set it all up.
The next step was to deploy our findings onto a webpage, so that everyone can have a look at the average price for which an apartment is currently posted on daft.ie in Dublin. First, I needed a server that is always online so that I can run my crawl script in regular time intervals in an automated fashion. I decided to use my free cloud computing credits on the Google Cloud Computing platform to set up a virtual machine and configure this to run my script every 24 hours. I also wanted to try something new for the task of serving the website, i.e. making it accessible in the internet and went for Google's Firebase web app deployment framework. In the past I used the Apache web server for similar tasks, but I must say it went way easier with Firebase. Let's go.

First we configure a VM instance (Virtual Machine) on Google Compute Cloud. I went with e2-medium (2 vCPUs, 4 GB memory), the standard debian image and a zone in europe which is where I live. I also activated HTTP and HTTPS traffic. Clicking on the SSH button opens up a terminal in the browser and we can go ahead and copy our Python code onto it. To run the python script I first had to install `pip` and `autoscraper` on the machine. After testing that the crawl Python script works, I went ahead and set up a cron job to run it every 24 hours. This is done by editing the crontab with `crontab -e`

```
PATH="/usr/local/bin:/usr/bin:/bin"
20 4 * * * python3 /home/pdywilson/rentcrawler/crawl.py >> ~/cron.log 2>&1
```

Note that I set the PATH here to make python3 callable for the cron service. I also rerouted the output as well as the error output to `~/cron.log` so that I can check if it worked or what the error message says if it doesn't with `cat ~/cron.log`.

To make it work I also had to make the python script executable with `chmod u+x crawl.py`

Now that I update my SQL database with the newest read from daft.ie every day, I'd like to see the news on the internet, i.e. on a webpage. As mentioned I am using Firebase for this and here I just followed the Get-Started-Guide at https://firebase.google.com/docs/hosting/quickstart. This quick start guide deploys a website at <projectname>.web.app showing the contents of an index.html file on my VM instance. All that's left to do is populate this index.html file. To do this I used this quick-and-dirty python script:

```
#check most recent
import sqlite3
conn = sqlite3.connect('/home/pdywilson/rentcrawler/db/rent.db')
sql = ''' SELECT * FROM dublinrents ORDER BY timestamp DESC LIMIT 1'''
cur = conn.cursor()
r = cur.execute(sql)
current = r.fetchall()
curr_timestamp = current[0][0]
curr_avg = round(current[0][1])
curr_median = round(current[0][2])
print("The current rent average is €{}, the median is €{} per month. - {}".format(curr_avg,curr_median,curr_timestamp))
website = """<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Dublin Rents</title>
</head>
<body>
    <div id="message">
    <h2>Average Monthly Rent in Dublin</h2>
    <h1>Rents are on 🔥</h1>
    <p>The average rent in Dublin is: €{}</p>
    <p>The median rent in Dublin is: €{}</p>
    <p>Last updated: {}</p>
    </div>
</body>
</html>""".format(curr_avg,curr_median,curr_timestamp)
f = open("/home/pdywilson/rentmanhost/public/index.html", "w")
f.write(website)
f.close()
```

This script fetches the latest results from the SQL database and then populates the HTML string with the newest numbers on monthly rents in Dublin. The index.html file is then updated with the HTML string.

Finally, I can automate updating the website by adding the firebase deploy statement to the crontab.

```
PATH="/usr/local/bin:/usr/bin:/bin"
20 4 * * * python3 /home/pdywilson/rentcrawler/crawl.py && python3 /home/pdywilson/rentcrawler/create_website.py && cd /home/pdywilson/rentmanhost && firebase deploy --only hosting >> ~/cron.log 2>&1
```

That's it! We can now open https://rentmanhost.web.app/ in the browser and check out the newest numbers on monthly rent in Dublin.

## Summary

Today we had a look at a small example of deploying a data pipeline that automatically crawls the web for data, updates a SQL database, calculates simple stats and publishes the results onto a website. 

1. We used Python and the package `autoscraper` to scrape the web for data.
2. We used SQL to store our data into a persistent database.
3. We used Google Compute Engine and cron to schedule an update of the webscraping Python script.
4. We used Firebase to host a website.
5. We used cron to automatically run our python scripts to update our database and website.

## Next steps

1. Using cron is a bit fiddly, so I'd like to replace it with the scheduling framework `airflow`. 
2. Our data crawler is a bit slow, I want to see if I can optimize it using `PySpark`.
3. I'm also thinking about containerizing this application, to make it easily deployable and scalable.
4. One of my goals for the near future is to deploy a real-time streaming data pipeline. I will be able to use the learnings from this toy example to get started on writing a proper streaming pipeline with real-time analytics soon!


All in all I'm happy with how this toy project went, I'm especially happy about trying out Firebase for the first time, it did exactly what I wanted and was super easy to use. Thanks for reading! :)