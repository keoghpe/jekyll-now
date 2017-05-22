---
layout: post
title: Running Rails in Production Environment Locally
---

There are some Rails issues that may require me to run a server locally with the production environment to debug. An example of such an issue might be a missing asset. Running the server in the production environment isn't difficult, although there are a couple of non obvious steps.

First, you need to precompile your assets:

```
rake assets:precompile
```

Next, you'll want to serve these files from Rails (you probably serve these files from Nginx or Apache in production). Update `production.rb`:

```
  config.serve_static_files = true
```

You'll also want to set `force_ssl` to `false`:

```
  config.force_ssl = false
```

If you don't do this the browser will keep trying to access localhost over https because you've sent the HSTS header. You'll need to clear your browser cache to access localhost over http again.

Finally, run the rails server in the production environment:

```
rails s -e production
```

Bask in the glory of your production environment, obliterate that tricky bug and high 5 everyone in the office.

Don't forget to revert these changes once your done. If you have accidentally deployed these config changes please return the high 5s collected in the previous step to their original owners.
