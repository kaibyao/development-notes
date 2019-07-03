# Table Stats Cache Actor design

Let's think about how the actor for the Table Stats Cache would work.

Ideally there would just be 1 actor total that takes in messages across all threads and responds with table stats. If there were one actor per *thread*, then there could be some thrashing on the DB whenever the cache resets, as it would make the same DB call once per thread per cache reset, which just sounds bad.

Which means that when Actix first initializes the HTTP server on multiple threads, it should only initialize the stats cache once total.

Which means that even though the cache will be accessed by multiple threads, only one of those threads will initialize the actor.

Which means there needs to be a function on the cache struct that uses `Once` to initialize itself only 1 time total.

So what happens to the other threads when they first request for the Actor's address? They should somehow wait until initialization is done, and then share the initialized Addr.

There should probably be a publically available function that returns a future with the actor's address. if the actor has not yet initialized, initialize it and then return the address. Otherwise return the initialized actor's address. Which means we need to store the Addr on the actor.
