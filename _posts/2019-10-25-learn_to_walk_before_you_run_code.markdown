---
layout: post
title:      "Learn to Walk Before you Run... Code"
date:       2019-10-25 16:47:19 +0000
permalink:  learn_to_walk_before_you_run_code
---


The most important lesson I learned in my Rails final project? Learn to walk before you run.

I decided to create an issue tracking system akin to Atlassian's JIRA. I use JIRA at work every day and I love it. It makes organizing my work much easier because I can go to one place to see everything I have to do in a given week.

Users can create and assign tickets to other users and those tickets can be assigned to different statuses based on where the assignee is in the process of completing that task. At my company, we have statuses like "Inputs" -- the creator of the ticket needs to provide more information in order for the assignee to get started on it -- "Execution" -- the assignee is in the process of completing the task -- "In QA" -- the assignee has executed the task and is getting their work reviewed by a QA engineer and "Complete" -- the task is done. There are many others, but those are some of the basics that a lot of SaaS companies use. Tickets have due dates so the assignee knows when the task needs to be done, and users also have the ability to comment on tickets and ask for updates on what is being done and if the assignee is going to be able to complete the task by the due date.

In short, JIRA has many features and is very easy to use once you get used to it, but there is one feature I wish it had--the ability to reply to a specific comment. If the task described in the ticket has many stakeholders, you might have a slew of users commenting on the ticket, asking different questions or making different requests according to their own interests and needs. You might get responses to multiple comments and it may not always be clear which responses are relevant to which comments. For example, Allison, the QA engineer on the project, might comment with "When do you think we'll be able to start testing this?" Then Alex, the product manager, asks "When do we expect this change to go into production?" Jimbo, the assignee responds with "End of the quarter."

Who is Jimbo responding to? Allison or Alex? Testing and pushing to production likely have different timeframes depending on the scope of the project. If you're a good little responder, you'll make it very clear which comment you're responding to, potentially copy-pasting the specific comment in your own response and providing your response in-line, in bold, in red with the original comment. Any programmer would see this and think, "Hmm, this is inefficient. Let's fix it!" That's what I set out to do with this project.

So, I built out the models and set up all of the necessary relationships between users, tickets, comments and replies. A comment belongs to a user and a ticket, and a comment has many replies. A reply belongs to a user and a comment. Pretty easy, right? I then built out the controllers and views and ran into an issue. The way I had it set up, a reply form would appear for _every_ comment that existed on a ticket. Meaning, if someone were to comment with "Inputs look good!" and someone else commented with "Where are we on this ticket?", the user would see two reply forms. One could imagine a ticket having many, many comments, resulting in a bad user experience when the UI is cluttered up with a reply form for each of those comments. So, I figured I would try my hand at JavaScript, a language which, at the time, I had no experience with whatsoever.

Bad idea. At least, the way I went about it was a bad idea. Googling a specific technical problem is a good idea if you have some notion of how the language you're trying to code the solution in works. That was not the case for me, but I went on googling away anyway.

Obviously, syntax is key, and I knew none of it for JQuery or JS. Every time I found something on Stack Overflow that looked like it would solve my problem, it didn't, and I would spend hours trying to figure out why. I would actually get mad at the users on Stack Overflow who had answered the OP's question After about a week of banging my head against my keyboard, I finally figured it out. I don't even remember what the "aHA" moment was because I think I had blocked the whole miserable experience from my memory, but this is what ended up working for me:

```
<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.4.1/jquery.min.js"></script>

<script>
$(document).on('turbolinks:load', function(){
  $("div.comment").click(function(){
    $(this).parent().find(".new-reply").toggle("fast");
  });
});
</script>
```

I added this to the `head` of my `application.html.erb` and voila, it worked. Basically what this does is toggle some CSS between `style="display: block;"` and `style="display: none;"` for any HTML element with an ID of "new-reply" whenever the associated comment is clicked by the user. So the reply form is still there for each individual comment, it's just not displayed until the user clicks on the comment.

Was getting this bit of functionality working worth the time and the headache? Definitely not. Would it have been way easier if I had a shred of knowledge of how JS or JQuery worked before trying to implement this? Absolutely. Lesson learned--learn to walk before you run.
