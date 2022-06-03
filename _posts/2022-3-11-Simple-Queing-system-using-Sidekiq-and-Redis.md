---
layout: post
title: RoR - Client Queueing system using Sidekiq and Redis
live: true
---

Given the requirements that client submit himself to a queue that should return results of some heavy background task. The server should be able to skip clients that already left the queue before their input were processed.

It doesn't sound very complicated at the first glance, but Googling for any solution doesn't return any resources except implementing queuing libraries like RabbitMQ. Our goal were to make the solution relatively straightforward and doesn't increase complexity of the app by additional library.

The application were already using `Sidekiq` as background job processor and `Redis` as queue store. Given that I've tried to see what we could do using only these.

While lurking through Redis documentation I've found it's [TTL(time-to-live)](https://redis.io/commands/TTL) functionality:
```sh
redis> SET mykey "Hello"
"OK"
redis> EXPIRE mykey 10
(integer) 1
redis> TTL mykey
(integer) 10
redis>
```

It gave me the idea "if Sidekiq job is pushed to Redis queue under specific key maybe we could set TTL on this key?"

How do we know the key that Sidekiq used? It's simple, when you use `perform_async` on any Sidekiq worker you will get the key as the result(it's named JID in sidekiq(add link)):
```ruby
001:0> job_id = SomeSidekiqWorker.perform_async
=> "fc3f44f792492883d843fac4"
```

Next step will be to set the expiration time for our JID:
```ruby
# Rails applications likely do have Redis instance defined
redis = Redis.new(...)
redis.setex(job_id, 2)
```

Now if the scheduled job won't be picked by Sidekiq worker in next 2 seconds the job will be completely removed from the queue. I've mentioned that we do want the job to be performed only as long as the is present on the page. The last step we need to add is some kind of polling that will reset TTL.

For this [ActionCable](https://guides.rubyonrails.org/action_cable_overview.html#example-1-user-appearances) might be ideal.

[Check provider job ID](https://github.com/GalacticPlastic/ironhack/wiki/Active-Jobs%2C-Sidekiq%2C-Redis#job-id) maybe it's possible to do it with ActiveJob?

```
job_id = Sidekiq::Client.push(queue: '', class: '', args: [])
redis.setex(job_id, 2)
redis.llen('queue:default')
```


[https://picsum.photos/]
