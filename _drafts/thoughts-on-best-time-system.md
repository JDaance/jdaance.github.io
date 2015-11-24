---
layout: post
title: Thoughts on best time system and sync
category: technical
---

So yeah one of my goals is to eventually have a best times system in the game where we can compete against each other. This is not that difficult to do, its just a bunch of UI and server and database stuff. I need to host it somewhere. 

To get what I'm talking about, check out this UI sketch:

![Best times UI draft]({{ site.baseurl }}/assets/best-times-draft.png)

Beautiful huh?

The best times will be available for each level, but also as a total time per chapter, and the overall game TT. This is a nice feature because you want to have a better TT then your friends, no doubt :)

For replays I want the user to be able to decide if they want to share these or not. The default will be not to share. When you set a new record time, the game should ask "Would you like to share this replay or not?". Maybe you dont want other people to see how you got that time. Keeping it secret for a while, letting people figure stuff out. Then you can turn on replay share later if you so decide.

As the developer and owner of the database I will get access to all replays of course, which I will have to handle with responsibility.

One pain point is account creation and stuff like that, so my plan is to use Facebook login. I dont like being tied to FB but it comes with a few advantages:

- No need for our own login system
- Most people already have a Facebook account
- We can access friends who have the game and show friends times
- There is a Facebook SDK for login on all platforms

Disadvantages may be something like:

- Some people hate Facebook
- Some people are scared the game will do something to their FB stuff (which it cant)
- Some people dont have a FB account
- Losing your FB account might block you from the game
- Some people don't want to compete with their real name

The last disadvantage could be solved by allowing some kind of alias. But the pain or fear of that might still be felt by the user. People missing Facebook account might just register some temporary account. Losing FB account I should be able to handle by support mail.

Anyway those pros/cons really make the decision land in the Facebook camp, albeit it doesnt feel 100%. Alternative login methods could be added later.

## Technical breakdown

Technically its rather simple. When a user logs in to the game (via facebook sdk) the game will sync all data with the server. It will send all times and replays, and the server will merge that with whatever we already got on the user, taking the best times and replays for each level. Then the best stuff is sent back to the phone. This way we also have syncing between devices, "cloud save", etc.

### Server

Server needs to hold all players accounts, and all their level times and replays. This is needed for syncing at least. And the server should respond to the games request about best time tables, replay views etc. The server is the master. The data will be stored in a standard SQL database. I'm used to postgres so I will use that.

I have worked with websites a lot, with .NET, PHP, Java, node mainly. I was never happy with any of these so I have tried a few different languages and frameworks since I have a free choice this time (with my own product). After trying a few different things I settled for Python with Django. I tried building it on Flask, but I missed my batteries included Django. Things just go so quickly with migrations and everything. I built a report system for a client with Django this past month and it delivered all the way.

### Client/Game

Technically the game part is pretty simple. It already supports HTTP requests, which will be used to communicate with the server. The big thing is to build and flesh out the UI. As mentioned it will just integrate with Facebook SDK for the login stuff. Then we need screens for showing best times and setting up the login for the first time. I also want to show some kind of sync status, I don't like it when its too quiet and I cant confirm whats going on.

![Best times settings draft]({{ site.baseurl }}/assets/best-times-settings-logged-in.png)

Not completely happy with the status panel yet..

When the user first logs in, the game and server will sync, with the server picking all the best times. Then every five minutes or so, or when a new record is set, the game will check if a sync is needed. All new records will be flagged with `synced: false` and then the game will know if there is anything to sync.

### API

To support all operations between client and server this is the API I'm thinking about. It will be JSON based.

    POST /login { times_and_replays }
    returns { updated_times, replays_that_were_better_on_server }

    POST /sync { times_and_replays_not_yet_synced }
    returns { updated_times, replays_that_were_better_on_server }

    POST /get-replay { level_time_id }
    return { level_time_id, replay_data }

    POST /set-replay-share-on-odd { level_id, on_off }
    return { success: true }

    POST /get-times-for-level { level_id [, league] }
    return { top100_times_for_level }

    POST /get-total-times-for-levels { level_ids [, league] }
    return { top_100_total_times_for_levels }

The league argument may not be used, but I want to add the ability to only see your FB friends times, or maybe national leagues. The possibilities are endless here.

I think this very basic set of commands will cover the functionality of the entire system. I'm thinking about using the facebook_access_token as the identifier for the particular install, since that can verify the actual facebook user and secure unauthorized access.

All API calls will then be wrapped in something like this

    POST /* { facebook_access_token, <additional args from above> }
    returns {
        success: bool
        data: <the stuff returned above>
        error: <error message on fail>
    }

Anyway I have a basic Django structure set up, with models and unit tests for get-replay and set-replay-share as a first test. The UI for best times is also under way.
