### DESIGN URL SHORTNER

# SCOPE

FUNCTIONAL:
1. Long URL --> short URL KEY ---> PERSITED (DB)
2. Short URL --> redirects --> LONG URL
3. SHORT URL config: (could have expiry time/ custom domain key/ user_id)
4. User Analytics (count how many times the link was clicked)

# SCALE
1. 1M per day
2. Based on 1M requests 6 characters are enough or we can have 7
3. How long to persist URLs? (Forever/ do we want recycle it?)
4. Whats the read to write ratio? 100:1

# Considerations

1. Do we want to also ratelimit the user/ IP?
2. If redirect is not applicable or expired we want to return 410 Gone/ 404 Not Found
3. To avoid collisions WE ARE USING 6 CHARCHSTERS BASE62 WHICH HAS 54B AND CAN EXTEND TO 7 CHARCH WHICH HAS 3B NEW COMBINATIONS

## ENDPOINTS
WHEN WE WANT TO SHARE A PROFILE/ LINK TO A PRODUCT/ TRACKING DETAILS THIS CAN BE TRIGGERED THEN
POST /api/createUrl  
    REQUEST   { original_url : "<<long url>>", OPTIONAL[custom_alias: <<company domain key>>] expiry_date: <<data>>, user_id : "<<user id>>" }
    RESPONSE  {short_url: "<<key>>", create_timestamp: <<timestamp>>, expiry_timestamp: <<timestamp>>} --> (register the mapping in DB)

HANDLED BACKED PART OF MAINTAINENCE
DELETE /api/deleteUrl
    REQUEST { short_url: "<<key>>" } ---> (delete from DB)

WHEN SOME ONE SHARED A URL LINK, NOW I CLICK IT TO VIEW THE ACTUAL PAGE
GET /api/shortUrl
      RESPONSE 302 FOUND REDIRECTS Location: <<original link>>

## DATABASE

I'd use POSTGRESQL to maintain strong consistency between key value pair and this is simple key value pair data. With postgresql we cana also store analytics like user clicked

SCHEMA:
record_id
short_url
user_id
long_url
create_at
expire_at
custom_alias
user_analytics <<user clicks>>

## HASHING LOGIC
CREATE URL

1. CHECK IF CUSTOM ALIAS IS AVAILABLE, IF AVAILABLE USE THAT
2. ELSE, GENERATE THE RECORD IN DB, and get the RECORD ID
3. NOW GET THE BASE62 (a-zA-Z0-9) KEY OF THE RECORD ID, IF SHORTER ADDJUST WITH CHARACHS THIS IS WHERE LENGTH IS IMPORTANT
4. NOW UPDATE THE RECORD WITH THIS KEY AND RETURN THIS KEY WITH EXPIRY DATE DEFAULT (5 DAYS SOMETHING LIKE THAT)

DELETE URL
1. TAKE THE SHORT URL, LOOKUP DELETE THAT ROW

GET/ REDIRECT
1. TAKE SHORT URL, IF SHORT_URL EXPIRED RETURN 404/410
2. TAKE SHORT URL, LOOK UP AND REDIRECT TO THAT LONGURL (302 LOCATION FOUND)

## SINGLE POINT DESIGN

[USER] --> /api/axfgyaan11 --------->    [URL SHORTNER]  ------> [DB]
                           <---------                    
                           HTTP 302 (https://anmma.ambbba/oqjm ap11/home/user_id)

## EXPAND FOR SCALING
                                                                        {REDIRECT TO SERVER (PICK WHICH HAS LESS REQUEST, PICK WHICH                                                                   REPSONDS QUICKLY, PICK WHICH IS GEOGRAPHICALLY NEAR, PICK BASE ON ROUND ROBIN)}
[USER]    -->        [PROXY SERVER/ API GATEWAY]  -->   [APPLICATION SERVER]    ----> [URL SHORTNER SERVER 1]   [URL SHORTNER SERVER 3]           
                                                                                 [URL SHORTNER SERVER 2]    [URL SHORTNER SERVER 4]


                                                                                ----> [GLOBAL DB (SHARED BY ALL)] --> (MASTER/SLAVE BACKUPS)

CHOICES: DB CAN EITHER BE GLOBAL AND ACCESSABLE TO ALL (RECOMMENDED) - BECAUSE IF SERVER FAILED DATA WON'T BE LOST AND OTHER SERVER CAN PICK                                                                        THE JOB AND ACCESS THE SAME VALUES
                                                                       TRADEOFF - SINGLE POINT FAILURE (WORKAROUND IS TO HAVE MASTER SLAVE/                                                                                                          STANDBY MASTER)
         
                                                                                
SINCE READ RATIO IS 100:1 WE WANT TO MAINTAIN CACHING TO REDUCE LATENCY

        CHOICE: CACHE PER SERVER/ GLOBAL CACHE --- LEAN TOWARDS GLOBAL CACHE TO AVOID SINGLE POINT OF FAILURE.
                                                                        
                USER ----> [LOAD BALANCER] ---> [APPLICATION SERVER] ----> [CACHE SERVER] -----> [CACHE SERVER] -----> [CACHE DB] [CACHE DB] [CACHE DB] (USE CONSISTENT HASHING)
                       <---- {CACHE HIT} <------
                            {CACHE MISS}
                                                 ---> [URL SHORTNER SERVERS]  ---> DB ----> LOG                                                                                                                                          INTO                                                                                                                       COMPLETE DB (NO SQL DB) 
                  <----- {REGISTER TO CACHE} <------
                                      
 ADD RATE LIMIT
                                            {ALLOWED}
                 [API GATEWAY] ---> DB/CACHE ------->
                                                         <----- {NOT ALLOWED} 429 HTTP 
            User → API Gateway with rate limiting → Load Balancer → App Servers → Cache → DB


                                                         
USER ANALYTICS

                                            LOG USER CLICKS FROM CACHE SERVER/ URL SHORTNERY SERVER TO A GLOBAL DB <--- ADD LOGS/ DASHBOARD                                             TO MONITOR FAILURES AND TROUBLESHOOT ERRORS SET ALERTS ON FAILURES AND ADD LOGS
                                            GATHER DETAILS FRO USER_ANALYTICS FROM DB 
                                                    
