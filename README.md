# Docs for Prime Archive Backend

## Create New Account : 

To create the new account you first have to make the ```Post``` request to the given url 

    POST: /createac

### Input Json : 

```
{
    username : Username of the user
    password : Password of the user
    fname : First name of the user
    sname : last name of the user
    captcha : Pass the captcha code here
    userid : Email here or telegramid
    description : Pass the about or any description here
}
```

### OUTPUT : 

If email is passed then you will get verification url or link in the mail else for telegram you will get the link which will redirect you to telegram where you can verify your account by opening that link (Telegram is not supported yet to always use email, And to differentiate between them it will detect @gmail.com and its mail else anything which dont have this is telegram)

```
{
    "status" : True,
    "msg" : "Account created success",
    "verify" : False, # Verification if its don or not
    "url" : url # Url to see temp only for testing
}
```

### ERROR : 

There are different error which you can get here are the list of all of them, For refrence try to check for **status** if its false then there is some error else task is done

- #### Missing Parameter

If any Parameter is Missing then this error will come

    STATUS : 400
    REASON : Missing Parameter
    JSON : {
                "status" : False,
                "msg" : "Missing Parameter"
            }

- #### Invalid Captcha

If captcha is not valid then this error will come

    STATUS : 422
    REASON : Invalid Captcha
    JSON : {
                "status" : False,
                "msg" : "Invalid Captcha"
            }

- #### Username is already taken

If username is already taken

    STATUS : 409
    REASON : Username is already taken
    JSON : {
                "status" : False,
                "msg" : "Username is already taken"
            }

## Login

To login to the user account first its important to get the session and user can get it from the following url and make sure to make ```Post``` request with following json

    POST: /primtlogin

### Input Json : 

```
{
    userid : mail or tg id here to login
    password : password here which user have set
    captcha : captcha code here
}
```

### OUTPUT : 

If login is success just as other you will get the ```status``` **True** and also get some detail of the user like firstname, lastname or etc and also get the **session** and **reconnect** where if reconnect is **True** then you can ask user if he / she wants to reconnect or not if yes then reconnect and you also get the room id too which user have connected before

```
{
    "status" : True,
    "fname" : fname,
    "sname" : sname,
    "userid" : userid,
    "username" : username,
    "description" : description,
    "reconnect" : reconnect,
    "room" : roomid,
    "session" : session
}
```

### ERROR : 

Just as before there are also different kind of error's and also check for status in the json of the error if its false then there is an error

- #### Missing Parameter

If any parameter is missing which needs to be provided before login then this error will occur

    STATUS: 400
    REASON: Missing Parameter
    JSON: {
            "status" : False,
            "msg" : "Missing Parameter"
          }

- #### Invalid Captcha

If captcha is not valid then this error will come

    STATUS: 422
    REASON: Invalid Captcha
    JSON : {
                "status" : False,
                "msg" : "Invalid Captcha"
            }

- #### User ID not verified

If user id is not verified which means user have created the account but have not verifyed it yet

    STATUS : 403
    REASON : User ID not verified
    JSON : {
                "status" : False,
                "msg" : "User ID not verified"
            }

- #### User Account Not Found

If there is no such user with the given data then this error will occur

    STATUS : 404
    REASON : User Account Not Found
    JSON : {
                "status" : False,
                "msg" : "User Account Not Found"
            }

- #### Invalid Credentials

If password is incorrect then this error will occur

    STATUS : 401
    REASON : Invalid Credentials
    JSON : {
                "status" : False,
                "msg" : "Invalid Credentials"
            }

## Public Room List

To get the public room list to join you can use this and make sure to use ```Get``` request method here not post

    GET: /roomList

### INPUT JSON : 

```
{
    session : Session id here
    username : Username here
}
```

### OUTPUT : 

You will get the **status** True as always but you also get the room data where json are given below how you will see that json, Total online could have some issue but other is fine like room id 

```
{
    "status" : True,
    "data" : {
                id : room_id,
                name : roomname,
                online : total Online,
                files : Total files links or ID
            }
}
```

### ERROR : 

Just like above paths here are the list of errors which could came while getting the public rooms

- #### Missing Parameter

If any parameter is missing which needs to be provided before login then this error will occur

    STATUS: 400
    REASON: Missing Parameter
    JSON: {
            "status" : False,
            "msg" : "Missing Parameter"
          }

- #### Invalid Auth

If auth is invalid then this error will came

    STATUS : 401
    REASON : Invalid Auth
    JSON : {
                "status" : False,
                "msg" : "Invalid Auth"
            }

## Room's Socket

From here we will discuss about how to connect to the room or create room or reconnect to the room what is system message how to read it and all kind of msg which could came from the server side but first to enter or do anything with the room we must have to connect with the room with the ```wss``` sockets here is the url

    wss://api.zerotwo.in/websocket

### Login with session

To login its important to get the ```session``` which you will get from the login and after that here is a small conversation of how to auth after login

```
< [CRED]
> USERNAME|:|SESSION
< [ACTION]
```

If this happen then auth is success but if following happens then its an error 

- #### If username or session is incorrect

Username or session is incorrect then this will came try to login again and get the session again or if it happen again then ask user to login again

```
< [CRED]
> USERNAME|:|SESSION
< [Invalid Credentials]
```

- #### If multiple user try to access single account

This error will occur when multiple user try to login or enter the room or do anything again

```
< [CRED]
> USERNAME|:|SESSION
< [MULTIPLE-USER-ERROR]
```

### Config Action, create, join or reconnect

Lets suppose that your have already auth and then its time for action then you there are 3 types of action that you can perform create, join or reconnect which are define below and also remember to always pass them in small latter else you will get the error

- #### To create the new room

This is if you want to create the new room

```
< [ACTION]
> create
< [ROOM LINK] # This is asking for link or id of the media
> https://zerotwo.in/
< [ROOM NAME DETAIL] # Its asking for name and password (Password is optional and if its not passed then its public room else private room)
> My Room|:|my room password
> [SYSTEM] Room **My Room** created success, id `7se4f`
```

- #### To Join the room

This is used if you want to join the room

```
< [ACTION]
> join
< [ROOM] [PASSWORD] # This means its asking for room id and password if it have one
> 7se4f|:|my room password
< [SYSTEM] [https://zerotwo.in/] Welcome in : **My Room**
> [SYSTEM] MyName has joined the room # This message is only visible to the other members of the room except current person
```

- #### Error which could came while joining the room

Here are some errors which could come while joining the room

**If room not found**
```
< [ACTION]
> join
< [ROOM] [PASSWORD] # This means its asking for room id and password if it have one
> 7se4f|:|my room password
< [NOT-FOUND]
```

**If user is banned in the room**

```
< [ACTION]
> join
< [ROOM] [PASSWORD] # This means its asking for room id and password if it have one
> 7se4f|:|my room password
< [SYSTEM] You are banned from the room you cant join it again
```

- #### To Reconnect in the room if network error or other occur

This is process of joining the room again if network error or any other kind of error occur

```
< [ACTION]
> reconnect
< [SYSTEM-RECONNECT]
```

- #### Error which could came while reconnecting to the room

Here are some errors which could came while you are trying to reconnect to the room

**If Session is expired**

```
< [ACTION]
> reconnect
< [SYSTEM] Your session is expired you have to join the room again
```

**IF room is distroyed or is deleted by the owner**

```
< [ACTION]
> reconnect
< [SYSTEM] Room not found or have been closed by the owner
```

## IMPORTANT NOTE

- Make sure that only admin or owner can use the command
- If someone wants to left the room then send .leave message
- Dont let user send the message which starts from . replace it with something like ". or anything and then replace " when gets receved
- Try to send username in capital letter
- Try not to save the session in the cookies because it could change according to the login.
- User should login everytime he open the page or site to do some stuff
- If user is something like admin or owner then you will get there msg like : ```[ADMIN](name)[username]: message here``` or ```[OWNER](name)[username]: message here``` but if user is normal user then it should be like : ```(name)[username]: message here``` this normal message.
- If someone gets disconnected some how like network error or something then you will get message like : ```[SYSTEM-DISCONNECT]:(name)[username]```
- Always remember this system message will always be like : ```[SYSTEM]``` it always starts like this.

### Here are some list of different kind of system messages

- ```[SYSTEM-PRIVATE]``` Its the private message which should be only visible to the current user who is receving it.
- ```[SYSTEM-RECONNECT]``` When reconnect is success.
- ```[SYSTEM-LEAVE]``` When someone leave the chat, You will get this message like this : ```(name)[username] : User have leave the room```

### Command Explaination

These are the list of commands that admin and owner can use, Make sure that command which have **[OWNER]** in the end those are only for owner and other are for all even its admin or owner both can use them.

- ```.admin``` : Create new admin, Pass the username like **.admin username** this. **[OWNER]**
- ```.kick``` : Kick the user from the room, Pass the username same as **.admin** command.
- ```.change``` : Change the media, Pass the link which you want to play right now.
- ```.seek``` : Seek to the specific position, Pass the sec or anything where you wants to seek to.
- ```.pause``` : Pause the media globaly.
- ```.next``` : Play next media.
- ```.previous``` : Play previous media.
- ```.close``` : Close the room (May not work properly).
- ```.radmin``` : Remove the user from admin, pass username as as **.admin**. **[OWNER]**
- ```.add``` : Add new link, pass link same as **.admin** command.
- ```.rearrange``` : Rearrange the links, pass the links same as **.admin** command.
- ```.ban``` : Ban the user from the room, pass the username same as **.admin** command.

#
# **THANKS** FROM - 5M4CK3R
