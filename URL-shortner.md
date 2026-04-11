### DESIGN URL SHORTNER

# SCOPE

FUNCTIONAL:
1. Long URL --> short URL KEY ---> PERSITED (DB)
2. Short URL --> redirects --> LONG URL
3. SHORT URL config: (could have expiry time/ custom domain key)
4. User Analytics

# SCALE
1. 1M per day
2. Based on 1M requests 6 characters are enough or we can have 7
3. How long to persist URLs?
4. Whats the read to write ratio? 100:1

# Considerations

1. Do we want to also ratelimit the user/ IP?
2. If redirect is not applicable or expired we want to return 410 Gone/ 404 Not Found
3. To avoid collisions can use username

## ENDPOINTS

POST /api/createUrl  
    REQUEST   { original_url : "<<long url>>", custom_alias : "<<company domain>>", expiry_date: <<data>>, user_nam : "<<user name>>" }
    RESPONSE  {short_url: "<<key>>"} --> (register the mapping in DB)

DELETE /api/deleteUrl
    REQUEST { short_url: "<<key>>" } ---> (delete from DB)

GET /api/shortUrl
      RESPONSE 302 FOUND Location: <<original link>>

## DATABASE

I'd use POSTGRESQL to maintain strong consistency between key value pair and this is simple key value pair data. With postgresql we cana also store analytics like user clicked

SCHEMA:

user_id
short_url
long_url
create_at
expire_at
custom_alias
user_analytics

## HASHING LOGIC

1. Basically use base64(7 charch combinations A-Za-Z0-9) this will give unique combinations

## DESIGN

[USER] --> /api/axfgyaan11 --------->    [URL SHORTNER]  ------> [DB]
                           <---------                    
                           HTTP 302 (https://anmma.ambbba/oqjm ap11/home/user_id)
