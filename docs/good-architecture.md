Sometimes we know something but we don't know that we know them. 
Or we just forget to apply what we already know in our daily lives. 
This doc is more of a **reminder** (instead of a lecture) of one such thing.

## Good Architecture

### What is good architecture?
Software has good architecture when it's inexpensive and painless to change or increment its behavior. 
As a bonus, well architected software tends to be much more reliable and bug-free.

### Why does it matter?
The world is constantly moving beneath our feet. 
Therefore we need to keep forever changing our software to adapt to it. 
Doing that painlessly and inexpensively is good for business. 
Having reliable software that doesn't break also boosts the reputation of whoever made it, and that is also good for business. 
Business is a lot about trust after all.

### How do we achieve good architecture?
That is the "million dollar question" that I try to answer here so keep on reading. 
But there's a few concepts and ideas we need to establish first, like "cognitive load", "tech debt" and "management style traps".

## Cognitive load

When we build software, we need to **think** about a lot of things. 
How is this going to behave? 
What stuff is available for me to reuse? 
How is this gonna impact other parts of the whole?
All that thinking takes up **space in your brain** - which is limited. 
That is cognitive load. 
I guess we can think of cognitive load as "the amount of brain RAM required to do something".

High cognitive load is a strong indicator of bad architecture (and usually a precursor to bugs). 
Well architected software is made of "pieces" that have a well defined responsibility and are not too "tangled up" with other pieces, 
so you can change each piece without having to think too much about possible unintended side effects.

## Tech debt

This is part of the "laws of physics" of building software. 
I guess it's more or less like the second law of thermodynamics: 
something like "entropy **will** grow unless you spend some energy to prevent that".

We developers are able to take "unhealthy shortcuts" to build stuff.
The situation is like: you have something to build and it's going to take a couple of days to finish it properly, 
but for some reason you have to (or you think you have to) deliver it today. 
Many times, **we do know how** to pull that off. 
Developers have the amazing ability to **BORROW EFFICIENCY FROM THE FUTURE**! 
This means we can be more efficient today (and maybe meet our deadline), 
but that **will** necessarily come at the price of being less efficient in the future (hence, "debt"). 
This usually results in a piece of badly architected software (cheap to create today, expensive to change tomorrow).
We can **pay that debt later** by going back to that code and rewriting it in a better way. 
And if we do, it's all good. 
But if we don't, the size of the debt **tends to grow with time**. 
So the tech debt is the kind of debt that comes with some unknown interest rate.

An example, just to clarify. Classic DRY* violation.

* [DRY](https://thevaluable.dev/dry-principle-cost-benefit-example/) - Don't Repeat Yourself

You have this…

![CleanShot 2024-02-11 at 18 50 07@2x](https://github.com/tonylampada/tonylampada.github.io/assets/218821/eb7f5a2a-d687-4818-93da-94e7a859cfb8)

And then comes in a new requirement to support paypal as a new payment type. 
Needs to be live yesterday. So we quickly do this.

![CleanShot 2024-02-11 at 18 51 20@2x](https://github.com/tonylampada/tonylampada.github.io/assets/218821/a91ebfeb-8208-4cc3-976c-eb42aae7d1d3)

Quick and dirty. 

Not needing to touch (and test) existing processPayment code is "quick" - also, we are lazy bastards who hate having to **test** the same things over and over again.

Duplicating "apply discount" logic (and possibly payment error-handling logic) is "dirty".
 
This is a tech debt. We need to go back later and fix this. 
If we don't, then later someone else will follow that same bad, 
code-duplicating pattern when we need a third payment method 
(and that will increase the debt amount - this is the interest rate we talked about).

Here's a metaphor to remember:
Codebase = planet. Tech debt = baobabs. You = little prince.

![CleanShot 2024-02-11 at 18 55 08@2x](https://github.com/tonylampada/tonylampada.github.io/assets/218821/49a5326d-66cf-4a40-84c9-6303e5d2854a)
_The little prince baobabs metaphor. There are many baobabs in life. Tech debt is one of them._

## Broken windows
"Broken Windows" is a theory about crime developed by sociologists in the 1980s. 
A gross oversimplification of it is: "vandalism is contagious". 
And there is a direct link between the broken windows findings and the **social dynamics** of tech debt when building software **as a team**. 
The harsh truth is that **you won't care about tech debt if everyone else doesn't either**. It's harsh, but it's true.

## Two Traps

![CleanShot 2024-02-11 at 18 58 24@2x](https://github.com/tonylampada/tonylampada.github.io/assets/218821/6dc06623-b2dc-40e7-bf29-c755b8a1ab95)

Regardless of religion, this is also true for software development 🙂. There are many "bad ways" and a few "good ways". 
One relevant example is the balance between prioritizing "value work" and "quality work" - 
I hate to put it like this because it kind of creates a false dichotomy of "quality vs speed" 
and I really believe that quality **leads to** speed - but the spectrum exists nevertheless, and **both ends of that spectrum are traps**.

### 1. The rewrite trap

"New tech [insert shiny technology here] just came out and it's awesome! 
Let's migrate a bunch of our stuff to use that!

Or

"We have a bad codebase, let's throw it out and do a fresh start"

Those are bad choices. 

* a) technology choice is meant to fulfill business needs, not developers' high-tech fantasies. And 
* b) if that "bad codebase" supports your business (as in there are clients paying to use it), it's not "bad" at all.
Grow up and have some love and respect for it! 🙂. That "bad" codebase keeps the business healthy and supports us so we can continue to learn and do new cool things.

Having fallen into those traps myself, all the times I did, I later realized that it wasn't the code that was bad. 
It was **I that was bad** at refactoring! 😬

## Good architecture - how to create it

So, back at the million dollar question, here's an attempt to answer it:

You do two things.

### 1. Set the Culture

The first, most important thing (like, say, 80% importance), is to: **cultivate a good engineering culture**. As team members we should agree that **we all** care deeply about ~our planet~ the quality of our codebase and will do our best to get rid of ~baobabs~ high cognitive load and tech debt. Leaders should send strong messages about the importance of good architecture and how it is aligned with business and professional success.

### 2. Set the Standards

If we have #1, everyone kinda wants the same things, all that's left is "just work. 
We gotta **define what we mean** by good/bad architecture. 
Think about generic guidelines that will have a tendency to reduce cognitive load and keep tech debt at bay.  
You can see further on giant's shoulders: 
[DRY](https://thevaluable.dev/dry-principle-cost-benefit-example/), 
[KISS](https://blog.codinghorror.com/kiss-and-yagni/), 
[YAGNI](https://blog.codinghorror.com/kiss-and-yagni/), 
[The 12 factors](https://12factor.net/), 
[Low coupling / high cohesion](https://stackoverflow.com/a/39988/8854363), 
are all ideas and frameworks you can and should pick and adapt to your context.

Then write it down, give some examples. 
Cultivate a set of "living documents" that is continuously consulted and updated with new standards we want to implement, antipatterns we want to avoid, etc. 
That will serve as a guide to orient team members to gradually improve code quality whenever they touch it. 
Team members can reference it in code review conversations, etc.

Those two things reinforce each other. 
It's like #1 we want the same things and #2 we speak the same language, like a tribe! 
You need #1 to start the spark and then then you need #2 to fuel the fire and keep it burning. 
