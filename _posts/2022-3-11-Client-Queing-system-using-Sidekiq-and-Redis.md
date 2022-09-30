---
layout: post
title: Ruby on Rails - Client Queueing system using Sidekiq and Redis
live: true
---


## The problem

Some time ago I've been working on a portal issuing COVID-19 passports, our goal were to let users input their personal details into the form on a web page and then based on the input validation pass the certificate to the user. Requirements sound quite straightforward, however we had two main concerns beforehand:
1. **Security** - patient data can't be stolen, we have to make sure no one will obtain data from our database with any kind of attack.
2. **Performance** - it was peak of the pandemy so we were aware that houndreds of thousands people will go into application in a very short period of time.

## Architectural solutions

We assumed that we have to put our users in some kind of queue as the process of passport generation were taking some time relying on 3rd party API.

After many hours of discussion we've decided that the best solution will be a 3(and a half) layered architecture:
1. **Static front-end server** - Next.js - blazing fast server because all of it's files were static
2. **Public Rails API** - all of the front-end queries were landing there, it's only responsibility were sanitizing the inputs and scheduling background workers, without access to the database.
    - **Redis** - the only touch point between the public API and separated backend. Being our main point of the client queue we relied on.
3. **Rails Backend** - unaccessible from internet, stored in a DMZ. Having access to all of the sensitive data.

![Vaccination Portal Architecture](/images/vaccination-portal-architecture.png)


We were sure that the passport generation must be processed as a background job. `Sidekiq` were our first choice as the number of concurrent jobs processing the data were limited only by the 3rd party API rate limits.

### Implementation

You already know what was the final project structure, but it was a result of multiple experiments. We expected a really heavy load onto the application.

Sidekiq by default works as FIFO([First-In-First-Out](https://www.geeksforgeeks.org/fifo-first-in-first-out-approach-in-programming/?ref=lbp)) queue. But we wanted to optimize it more - what if an user enters the queue but doesn't wait on the page until his request is processed?

It will be processed anyway in the default setup, however there is a relatively simple solution that could allow us optimize it.

### Meet [Redis TTL (time-to-live)](https://redis.io/commands/TTL)
```sh
redis> SET mykey "Hello"
"OK"
redis> EXPIRE mykey 10
(integer) 1
redis> TTL mykey # after 4 seconds
(integer) 6
redis> GET mykey # after more than 10 seconds
(nil)
```

Each background job is scheduled under a separate key in `Redis`. Given the knowledge of this key you could utilize `TTL` function to wipe the job from queue after X number of seconds.

When you schedule Sidekiq worker with `#perform_async` the Redis key is the result of this method call(it's named JID in [sidekiq](add link)):
```ruby
001:0> job_id = SomeSidekiqWorker.perform_async
=> "fc3f44f792492883d843fac4"
# place for ActiveJob example

```

Next step will be to set the expiration time for our JID:
```ruby
# Rails applications likely do have Redis instance defined
redis = Redis.new(...)
redis.setex(job_id, 2)
```

Now if the scheduled job won't be picked by Sidekiq worker in next 2 seconds the job will be completely removed from the queue. To prevent this behaviour the front-end have to ping back-end as long the user is on page.

Simple Javascript `setInterval` could do the job, or if you want something more sophisitcated then you can use [ActionCable](https://guides.rubyonrails.org/action_cable_overview.html#example-1-user-appearances).

When our back-end receives the ping it should recall `redis.setex(job_id, 2)` to reset the time counter.

* * *
#### Did you know?
It's possible to schedule a Sidekiq worker from outside of your application:
```ruby
Sidekiq::Client.push(queue: 'external', class: 'ExternalWorker', args: [])
```
all you need is access to the Redis instance of source app.
* * *

### Inform the user about queue length

```ruby
# https://www.rubydoc.info/github/mperham/sidekiq/Sidekiq/Queue#initialize-instance_method
queue = Sidekiq::Queue.new('default')
aprox_wait_time = queue.length / queue.latency
```

Having the queue length and the latency(time between last processed job) we could try to calculate how long the user will wait.
Let's assume that client number in queue is 600 and the queue latency is 0.5s => `(600 * 0.5 = 300s => 30min)`.

This calculation will give you only the estimation as latency could vary. ATM I don't know the method to find out the position of key in Redis list. If we'd be able to find out it then we could recalculate the position for every front-end ping, currently we could only calculate it at the moment of scheduling the worker.


### Result

Using this strategy you can scale your application easily, simple spawning of additional Sidekiq workers is very simple and cost-efficent, the only limit might be database capacity but it's a completely different topic. External API limits might also cause problems there, but if you can't increase the limits then even the best solutions won't help.
