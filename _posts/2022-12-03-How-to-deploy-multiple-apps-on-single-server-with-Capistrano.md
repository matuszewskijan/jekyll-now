---
layout: post
title: How to deploy multiple Rails apps on a single server with Capistrano
live: true
---

Almost everyone uses cloud hosting solutions like Heroku or AWS nowadays(and I don't expect it to change even after [this DHH post](https://world.hey.com/dhh/why-we-re-leaving-the-cloud-654b47e0)). But if you still self-host your apps then [Capistrano](https://github.com/capistrano/capistrano) would be well known to you.

My client had an uncommon requirement - to deploy two versions of the same application twice on one server. We were already using `Capistrano` there so I started lurking how to quickly fulfil the client's needs. 

Not getting into the details of `why` let me show you `how` to do it:
```ruby
# config/deploy.rb
APPLICATIONS = %w[app1 app2]

raise 'Pass APPLICATION environment variable' unless APPLICATIONS.include?(ENV['APPLICATION'])

set :application, ENV['APPLICATION']

append :linked_files, *%w[config/database.yml config/master.key config/credentials.yml.enc]
```

We're defining a list of application names that are allowed on our server. Users deploying the app will have to provide an `APPLICATION` environment variable, an error will be raised when it's missing.

We also have to define `linked_files` with list of the files that should be different between applications. In this case it's different set of database credentials, `master.key` file and different Rails encrypted credentials. If you're unfamiliar with `linked_files` then see [capistrano-linked-files](https://github.com/runar/capistrano-linked-files).

The command that we'll use to perform the deployment:
```sh
APPLICATION=app2 bundle exec cap production deploy
```

From the Capistrano perspective pretty straightforward, isn't it?

There was a little bit of additional `Nginx`, `Passenger` and `Let's Encrypt` configuration to be done but nothing unusual so I am not going to cover it today.

Thank you for reading!