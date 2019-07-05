Experienced rustaceans say that if you bang your head against the wall for too long, it’s probable that you’re going about it the wrong way. This is true for programming in general, but Rust has the tendency to force a programmer to write applications using the correct architecture.

I’ve been trying all day trying to get a globally-static Actix actor working, and I’m starting to see that I am thinking about it the wrong way. I’ve been trying to use Once::call_once to initialize at most one Table Stats Cache actor that is a static variable, but maybe I don’t have to do that. Rather than having the limit of 1 actor total be enforced in the library, I should just have it be something that the developer-user controls.

On the other hand, it gives them a foot-gun. I think I can still enforce the 1-at-most actor limit by moving the Once::call_once to the main part of the library, instead of within the cache module itself.
