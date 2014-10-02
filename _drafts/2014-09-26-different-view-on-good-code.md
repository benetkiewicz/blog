---
layout: post
title:  "Beyond SOLID - what else is important while maintaining existing enterprise systems?"
date:   2014-09-26 16:57:59
categories: quality
---

Good old principles
--------------
We all have good coding practices hard-coded into our brains. Most dev conferences have at least few talks about how to write good quality code. I also know all these principles, I try to incorporate them into my projects. These are:

* keep it simple
* have unit tests, automatic system tests
* write documentation
* encapsulate
* SOLID
* remember about security
* world peace
* etc.

But this is my eighth year in the industry, writing enterprise applications for a living, and you know what? There are other important things to get job done! I will even risk the blasphemy and say - more important.

To explain myself, I first need to put things into context. Imagine working for medium-size software company that writes own code but also gets requests for adding features for existing code or maintain old code (just keep it alive and keep up to date with, let's say, new law regulations). Your company exists 10+ years, people are coming and going, business partners (guys who gave you tasks to maintain their code) are changing too. One day you come to work, sit at your desk and find yourself assigned to implement quick change in one of the projects your company just received from a client...

Story
-------------
> It was a week before they clicked into proper TFS checkboxes to give me access. That week will not push the deadline. Moreover that TFS if behind quirky VPN and it took 4 attempts to fully clone, pardon, check-out the code. I'll probably end up working in offline mode and right clicking source code files to remove ___read only___ attribute. I'll get lost between Work Items. I will not understand checkin policy and I'll have trouble pushing, pardon, checking-in my work. This nightmare will be repeating itself for each new member of my team. And finally in the end someone will come and say that my license is invalid and I should just a zip package to a client.

> OK, so I've got the code. I load solution into VS and hit F5. Nuget is downloading components, project starts and takes you to home page, right? Wrong! I've got 596 compilation errors. One of the projects didn't load at all because it is hacky ninja web deployment project very popular seven years ago, whose developer died from the old age in 2010. NHibernate dll is referenced from the `bin` folder which is obviously empty and some old JSON library's strong name has invalid key.

> Once I'm done with fixing the compilation process, I promise myself that I will not be so naive again today. You remember that there's a database for this project, some guy emailed you 340MB backup a week ago, so you start with that. Boomer. Backup didn't go through your company filters, you have to write another email and ask the guy to SFTP it to you. Fortunately that was quick, you proceed. Where's the connection string. `CTRL+SHIFT+F`, whoa, there's no connection string. You dig through the code. Jesus Christ, what a pile of crap. Got it! There's no connection string because author of this app decided to go with his own approach - connection string is saved in the secret database table, for which the connection string is stored custom config section in web.config. Oh yeah baby, now we're getting somewhere. You restore the backup, fix the custom config and have a brief moment of sanity - why the hell I am discovering this? Why didn't I just followed the deployment or quick start document? Right, there isn't one.

> F5. Runtime Exception! Maps component trial library expired about a year ago. You fix that. F5! Runtime exception! Redis cache needs read/write access rights to `c:\temp\hollyMolly` directory. F5! Login page! Finally!

> User: admin. Password: sex. Runtime exception! You go through the code again. Got it! This project uses asp.net auth provider hosted on Oracle... Set up oracle, set up auth database, F5! User: admin. Password: sex. Runtime exception! You forgot to run VPN and the custom provider loads addtional role permissions from a hard-coded user repository.

> Finally, Jesus Freakin' Christ, it is working! You have access, you can add records, run reports, generate payments... nope, correct that, payments are not working because they redirect to a 3rd party gateway which requires SSL certificate to work. Nevermind, your task is simple and does not involve payments. Wait! You have a task to accomplish and now YOU CAN DO IT!

> Requirement says: on the last step of the registration process add a button that will generate a file with the basic record data for offline processing. Wait, what is registration process? There are few steps while adding a record but is that registration? What do they mean by basic record data. What should be the format of the file? Where should I put that freakin' button???

> Calm down. Let's see how this process looks like, let's add a record, proceed to the last step and maybe you'll figure it out or at least you'll find a good spot for the button. Second to last step - identity verification: __please provide your last U.S. address and we'll ask you a few questions about your life to confirm identity__. But I live in Poland!!!

Conclusions
-------------
In the situation I described, is strictly understood good code the most important? I dare to say no.

I mean, come on! How often do you see well written code that already hit the production and works there for more than a quarter? Is it one out of four? Then you're lucky. Business will not be interested that you need to get your hands dirty with the spaghetti code though. You're on your own. It is just you and that pile of crap. Now imagine a sexy fairy popping out of your laptop and asking you what would you like this project to have or didn't have, so you can do your job fairly quickly and stable? You can wish for anything but better code quality. Remember - spaghetti stays!

__Source control__ - let's start at the base. You need to have modern, pleasant to work with source control system like git or mercurial. It is free, super easy to set up, easy to share code with new members of a team by simply adding new (internal) remotes, encourages experimenting... It should also be free (to some extent) from strict company policy rules. Rule should be simple, if you're a dev you have a right to read and write code. Without that you cannot quickly shift or replace devs in your project.

__Quick start doc__ - self explain

__Easy project setup__ - nuget, standard configuration, database scripts, well known conventions and industry standards. Any gotha's explained in quick start.

__Easy account access/setup__ You wont do much if you cannot access certain app accounts and roles. If setting up and using a new user account requires two VPNs and an expensive license, you will not have velocity.

__Common language with business__ - if business says tenant, means state and you see organization in the code, things cannot go well in that project.

__Lower environments mirror PROD__ - I cannot stress that enough. If you use different setup for QA you will miserably fail.

__Mocks and fakes where necessary__ That stands in conflict with previous one but there are expensive services (data providers) and location based services (identity confirmation) and services that generate side effects (payments) and if it is extremely hard to set up in lower environment, use fake ones. Try to make them work as similar to original one as possible and make sure to embed some automation for QA to work with. Ex. timeouts on certain data input to test how application deals with service being unavailable.

__Great feature spec__ - There is one tool that helps even developer that has no experience in the project to accomplish his first task in the project. It is global search (ctrl+shift+F). But one has to search for something! The more detailed the spec, the better chance that implementation will be correct. Spec need to contain before and after, acceptance criteria, some simple acceptance tests, screen shots.
__

Summary
-----------
For some projects you probably can't do a lot about the quality of the code. But fixing things I described above is much easier, quicker and less risky than a total project rewrite. Seriously consider doing this, especially if you're a team leader or you'll be handing the code over to another person. With all these simple rules in place you'll be able to deliver fast and stress-free even if you work with spaghetti, not code.
