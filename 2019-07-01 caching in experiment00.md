I'm thinking of having an Actor that is responsible for 1) refreshing the cache, and 2) serving the cached data when given a table name. The problem is when/where to fill the initial cache.

- I could put it into generate_rest_api_scope, but then I would need a separate struct that
  contains the existing db_url string as well as the cache.
- What happens in the situation where the developer manually assigns endpoints instead of
  using generate_rest_api_scope?
- I could just put it into a global state that the select_table_stats endpoint initializes
  once, and manually call that endpoint in my app code when the server first starts up. Feels cumbersome though.
- I think the best option is to have a separate initialization function that can be called from both generate_rest_api_scope as well as manually. That function will have to be called by the developer's code if is_cache_table_stats == true. If they forget, api endpoints will crash (this will quickly be caught in development).
