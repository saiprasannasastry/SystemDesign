Assumptions/Requrments

we show every 1 hour data
we show the next 24 hour data hour by hour and next 7 days data day be day
should we show the day's high and low temperatures or not
should we show an enum for corresponding temperature(sunny , cloudy, snowy, windy, rainy)
we dont show a lot of other data like sunrise sunset, visibility etc
consistency is no big deal a delta of 1 degree is allowed
we don't care about concurrent updates because of the above constraint
8 )highly available but even if data is not present we can show old data
data modelling
~1 billion people subscribed
response from weather API ~1MB
so we need 1Tb of data per hour so 24-30 Tb of data per day with erasing the data of 1 hour interval
RDBMS struct would look like
idx
city Next24Hours(jsonb) NExt7days(jsonb) weatherType(enum)
Redis
key :city value is dictionary of interface , with value expiring every hour

DATAbase a centrally located database that each region and zone can listen on and a cache that is present on every single region for quick retrieval is delta is between o-1 degree
https://app.diagrams.net/#Lweather.drawio
since we have made an assumption that we keep updating the redis, concurrency isnt an issue, our replication server will send data to RDBMS and we can keep the zones DB TO main DB in sync every x interval

on the client side/phone we cache the data so that we show the same in every interval, only when a refresh is made we pull the data from CDN (if the delta is ~1 degree dont even update any database) else update to redis and update to client.
else make a call every 60 min to update the DB
Redis to RDMS replication is managed by replication server.
if key is not in redis delete that in database as well(this is for expiry case)
The reason i want a RDBMS as well is , if the entire region goes down then instead of querying the CDN we can check in RDBMS.
I've added those thoughts that came to my mind in 45 min(i actually considered this as an interview and did it).
Let me know if something isn't clear or alternate suggestions are welcome.
Upvote if you liked the design :)
i did miss how many servers we would need . but here it is
approx 30servers serving 10000 API's per minute(New Request's to the cdn most of the time users don't go ahead and refresh that page) gives us 7.5 mil API's that could be answered over all regions per min. That's really good. although i highly doubt so many API calls being made
GCP has around 25 regions, so 302510000(API's)
so per hour the above 60 ~=450 mil per hour.
increase the servers2 and we have 1billion new API calls, scale the server vertically and we have a lot more api's answered
