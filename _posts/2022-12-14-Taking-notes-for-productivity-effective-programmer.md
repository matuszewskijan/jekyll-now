---
layout: post
title: Taking notes for productivity - Effective Programmer pt. 1
live: false
---

***

I'd like to welcome you to the first article from the **Effective Programming** series. As it's the first episode I'd like to give you a longer introduction to the series:

My goal is to write about things I find useful in my everyday work. I don't claim to be a productivity expert, just doing my best to organize my work in the best possible way.

The topic of being effective is so wide that there are books written about it. I will try to narrow down the scope of each article as much as possible, to focus only on the essential things from my perspective.

**Shortly about me**, to show you what I am doing and what kind of problems I face:

I am a **freelance full-stack developer working mostly with Ruby on Rails and React**. While being a freelancer I usually work for **multiple clients** who care more about the output of my work than how long it took.

The output, in other words: **value** - it's what I am focusing on. My work made me realize that everything could be optimized and while finding *the better way* to do things is sometimes difficult at the beginning, in the end, it makes everyone happier.

And I'd be simply happy if you would find anything useful in my articles.

***
# Notes

Everyone was taking(or making) them in school days, except the arrogant/genius kids who trusted that they will remember everything. If you were one of the geniuses who indeed didn't have to take notes, good for you.

In school, I knew that I should note at least something(still my notebook was usually one of the emptiest).

But then, a few years fast forward to my first job and you know what? **I was entirely sure that I could take almost everything straight into my head. And I regret that.**

I've started taking notes, it wasn't an instant change, I had to grow up, and humble down. There were more and more things to remember and I was simply forced to find a more convenient place for storing the information than in my head.

# Why you should consider taking notes?

## Make the knowledge easily accessible

The knowledge is usually accessible... somewhere. Don't force yourself to Google every time you need to use "this" command, or link to an API documentation page that is difficult to find. Do not spend minutes scrolling for a message that your colleague sent a few months ago about a specific topic.

Having it all saved in a known place means you know where to look for which means you could easily access it.

Imagine a car repair shop, all tools should have their own place there. It helps the mechanic as he always knows where to look for a certain thing. Otherwise, his work will quickly become frustrating and more about searching for tools than repairing cars.

It's a little bit extreme example but my point is that programming work is also way easier when we know where to look for proper things.

## Be professional

Imagine your manager asking you a specific technical question about the thing you worked on a few months ago. If you would send him your notes about that topic clarifying his question it would be definitely impressive, it's a great way to show that you are professional and you treat your work seriously.

It was a cool scenario, but I do see a better one:

## It could be more than a note

What if you could **share** your notes with everyone? The question won't be made in the first place or you'll just answer with a link.

I believe that **good notes could be easily translated into documentation. If you find the information essential then others would likely also benefit from it.**

Also, don't get me wrong, notes are supposed to help you in the first place, but usually, with little to no effort, they could help many others.


# How to take notes

How to structure it, what to put inside, those are all things that you should know the answer. Everyone is having a different style and I could only encourage you to find your own, as in the school: some would write a list with important points, others create graphs, others just write a few sentences.

It's not about the method. I could tell you that it's a good idea to always give yourself a good description of the context. It's always a good idea to maintain good formatting and check against typos(Grammarly could help).

## How to define a good note?

If it helps you and you came back to it then it's a sign of it.

But there is a better method: **share your notes with others**. Don't trust yourself that what you create is good. They might ask clarifying questions or tell you that they don't like something inside. No matter what is the feedback **you should take it**. If there is no feedback - that's also fine, you've done your part.

# How to organize the notes?

Enough of theory, let's get into some examples of how I work with the notes.

It depends on the thing I work on. I am trying to organize notes per-topic to have them easily available. For example: If I work on `project X` there are high chances I do have opened:

**1)** This project's Slack channel or
**2)** Code editor with its project repository.

Knowing that I'd start considering if I could store my notes in any of these places. Just to remove any kind of friction between the work and writing notes. Talking about sharing, both `Slack` and `Github` are publicly available for other team members so they seem to be a good fit.

Let's go through different topics and see how I organize notes for them:

## Project Notes

The main note-taking tool for every client I work with is **Slack**. Why? It's the place where everyone exchanges knowledge, sometimes you just have to distil the essential bits from the discussion.

How exactly do I use it for taking notes?

1. Send a message to me with any useful commands/scripts/domain knowledge:

	Create a new message and add yourself as a recipient. It will create a private channel where you could send anything. But, I recommend you always ask the question *"Is there anyone else who could also benefit from it?"*

	If the answer is yes then...
1. Create a public note-sharing channel

	It's a great place to share important lessons from our work. I am not an Agile expert, but praising the sharing culture seems to align well with the manifesto.

	The plus of doing it on Slack is that it's having low entry barriers. It's way easier to send a short Slack message than to add a new section to the documentation.
1. Add any useful message to `Saved Items`:

   Someone shared a valuable link, piece of information, command or anything you find useful? Just save it and look into the `Saved Items` card. If you'd always save the essential messages I am sure soon you will have a good knowledge database there.
1. Share production command execution on a shared channel:

	It's loosely related to the note-taking topic but it worked really well in one of my previous projects so wanted to share it:

	Create a separate channel for any developer actions on your production application and automatically post them on this channel. Set up the console in the way that it asks for his name and the purpose of the session. Then add this context to all of the things shared on Slack.

	Thanks, [Arkency](https://blog.arkency.com/keep-your-team-up-to-date-on-production-data-changes/) for sharing the idea, it was a huge win on multiple fields as:
	- every developer is instantly notified about executing special code in production, it's easy to ask for details
	- when you know that anything you execute will be seen by others you think twice before doing so
	- you have the history of all executed scripts and commands in one place, it's useful for audits but also when something has to be re-executed then you could easily find the right command on the channel.

Slack is good for taking notes, but at some point, you should move the notes into places like project wiki pages or Notion, but as mentioned if you have good notes then you won't find it difficult to transform them into documentation.

If you are currently working alone and Slack isn't a thing for you then I'd encourage you to read next section:

## Technical notes

We're getting outside the project/company. Anything it is the `Note` application might work, but it likely means you will never share your thoughts with others. **Maybe you should consider starting a blog?**

Start small, just a few phrases and a short code example. It doesn't have to be as long as the thing you are reading now(we're getting there!)

If not a blog maybe just post it on **Twitter**?

### Collecting feedback is an awesome opportunity for learning

Case study: I've recently published a [short article about deploying Rails applications with Capistrano](https://jmatuszewski.com/How-to-deploy-multiple-apps-on-single-server-with-Capistrano/) that gained some attention only because people were saying that my solution is crappy. But they also shared a better one, I'd never learn this better approach without sharing this post.

For the things that really don't fit anywhere, I usually use [Gist](https://gist.github.com/). Gists are usually private so only you see them, but at any moment you could publish them.

## Getting Offline

It might be unpopular but I feel that with all of the benefits, salary packages, remote work and text communication it's easy to become out of touch with reality. I find it soul-healing to take a pen and paper from time to time, and go completely offline just to organize my thoughts and I'd recommend you do the same.
