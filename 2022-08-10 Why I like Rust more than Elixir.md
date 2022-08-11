Having tried building a Fantasy Basketball app for a Dynasty league that I'm in, I decided to experiment with rebuilding it in Rust after getting about 1/5 of the way done with all the server-side work in Elixir + Phoenix. Before I list the reason for this decision, I'd like to point out that the thing I noticed about working in Elixir + Phoenix that I like very much, is that it takes care of a lot of things for you.

No, seriously. It does. Here's a list of the major ones:

1. CLI commands for creating your database, DB migrations with schema creation.
2. User creation + authentication.
3. CLI command to start a built-in webserver so that you don't have to code one yourself with DB connections.
4. You can start developing your app at the URL routing step, not the code-your-web-server step.
5. Can "live-debug" via running elixir code on your running application via `iex -S mix phx.server`.
6. OTP / Actors / message-passing is a really awesome thing to have built into the standard library.

All in all, going from reading tutorials to building out an app happens very quickly. So why am I deciding to switch? Two reasons:

1. Lack of types, and related to that,
2. IDE integration is poor.

Once you do get going, it's still very hard to tell what objects functions expect as their arguments, and whether you are transforming data correctly, so a lot of development feels like trial and error. While all programming has this to some extend, the trial-and-erroring in Elixir feels very strong. Oftentimes I found that my development flow looked like this:

1. Pass an object into a function.
2. `IO.puts` the output.
3. Refresh the page that calls the function or run the function via live-debug.
4. Look for the console log or error message.
5. Infer from there what went right/wrong.

The thing is, a lot of this goes away with static typing. And along with static typing, your code editor or IDE of choice (VSCode in my case) will be able to MUCH better infer what you should be passing into that function, as well as warning you ahead of time whether your argument is what is expected.

So, while I very much loathe the idea that I'm going to have to implement a DB connection pool and user auth + public/logged-in paths on my own, I'm feeling excited about the prospect of working with types again.
