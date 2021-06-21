https://www.youtube.com/watch?v=JQDHz72OA3c

# State assumptions
1) Users are anonymous
2) User enter a long url, we return the tiny URL
3) User enters tiny url and our services needs to redirect him to a website
4) requests are spread across regions and are not consistent
5) Service has high availability
  - optionally we can store the expiry time and delete it

## Out of scope
1) User registration
2) User setting the shortLink

# Data constraints
 - size of long URL ~1.8kb
 - short url ~ 0.2 kb 7characters
 - expiry/created at ~0.1 kb
 - 30 million users per month
   - write = 3*users
   - read = 6*users 

- 60GB of DATA per month
- ~720GB per year or 0.7TB

# Internal mechanism
- md5Hash. not a good one as if the hash is longer than the storage length then we can't store in db
- base 62  why 62 instead of 10 is that we have 62^^7 combinations corresponding to 10^7

# DATABASE TRADE OFFS

| RDBMS.        |  NOSQL(REDIS/MEMCACHE) |
| ------------- | ------------- |
| ACID PROPERTy  | HIGHLY available |
|can't scale easily even if we shard(complexity is very high) | easily scalable  |
|long DB queries   | write/Read inefficiany (since we don't have locks like RDBMS)| 

# DESIGN
API POST /config/v1/tinyURL
  my server redirectly to my acual server

  POST /config/v1/longURL -d{url :}
  response {
  url :'www.myservice.com/q1e43q6'
  }
  
RDBMS DESIGN increases DB reads inturn increases RESPONSE time for every new READ also possibilty of code randomly generating the same thing in one instance
![RDBMS](https://user-images.githubusercontent.com/36291746/122772994-8e234b00-d2c5-11eb-9b37-a770fd2e0f1f.png)

Nosql same as above, we could have some sort of counter we can add and mitigate the random generaation of samething but still what if the counters go down/ more importantly since its vertically scaled we have concurrancy issues and it's hard to mitigate with nosql

solution use zookeeper which stores a range of counters so that we can directly push to DB without any checks
![with zookeeper](https://user-images.githubusercontent.com/36291746/122774928-40a7dd80-d2c7-11eb-836b-40b72f4f7ad0.png)

