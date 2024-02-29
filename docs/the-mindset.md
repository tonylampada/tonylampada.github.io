# Software Engineering: The Mindset

Before we think about software development good practices in a technical sense, I believe it's worth thinking about a "higher-order"
good practice, which is approaching it with the right mindset. 

A "contextualizing prompt" for you to embody before you take action 😅.

This article is a collection of tips that, in my experience help create thriving software projects.

## 1. Tech debt - pay it, while it's cheap.

You probably have some, right?
Maybe left over from a not-so-distant past when you were sprinting to quickly build an MVP.

But looking forward, to keep growing your software product and keep it healthy, you'll need [good architecture](/good-architecture/), 
good abstractions and **revert the growing tendency** of tech debt. 

This requires making a decision that, from this point onwards, we'll make sure debt only decreases.

A good way to do that is to work on bug 🐞 fixes! 
Bugs are rarely the result of a single isolated problem. 
Instead, they are usually the result of some deeper related debt around some functionality. 
We should try to work on bugs in way that fixes them quickly first, 
but then do a deeper critical analysis to understand what is the kind of debt that allowed that bug to exist, 
and then work to decrease or maybe even eliminate that debt.

## 2. Code is a place.
I'm sure you've felt it. 
When you're "in the zone", code is not only a thing that you manipulate. 
It becomes a _space_ that surrounds your conscience, a _place_ around you. 

![image](https://github.com/tonylampada/tonylampada.github.io/assets/218821/19d1e66e-acef-4c02-8f66-638a1b543164)


I saw [this movie](https://www.imdb.com/title/tt1219289/) once about a regular guy who found a pill that made him super smart. 
He lives alone in a messy apartment, and the very first thing he does when he's in "super-smart mode" is clean the place: 
do the dishes, put out the dirty clothes lying around, take out the trash… 
It was like he knew he needed to clean the place in order to think straight. It blew my mind 😲😲, I never forgot that. 
"The first thing you do when you're super smart is to make the space around you clean and organized". This feels exactly right.

## 3. Chaos and Order
I read once that our brain has two sides because we have evolutionarily adapted to an interesting aspect of reality: 
the world is made of chaos and order, familiar and unfamiliar, explored and unexplored. 
And our conscience is the process that transforms chaos into order. 

It's something like: When you're in the middle of chaos, at first you don't know what to do. 
So you do nothing, you're paralyzed by chaos. 

But you're not doing nothing. You're paying attention. You're learning about this new unfamiliar environment. 
Until you understand just enough to reorganize it a little bit. This is a seed of order. 

As you keep working in that seed, you change the once unfamiliar environment around you for the better, 
you create an expanding domain of order out of chaos.

As a programmer, you sure have lived through this process many times. 
For example at first you may not have a clear idea of how state flows through a particular part of the code. 
Then after studying, testing, debugging, you start to wrap your head around it. 
And then _your brain starts to poke you with ideas_ on how to improve that code somehow.

Programmers should feel discomfort in chaos, but never feel intimidated by it, since you know how to thrive in it. 

You're an agent of order. A bringer of order through attention, learning, and work.

# 4. Think of code as a representation of knowledge.

Not just a sequence of instructions.

When we're learning how to code, code is just that: a sequence of instructions. 
As we gather experience through the years and work on more complex systems, we realize that code _should_ be more than that. 

Complex systems are made of smaller parts that _know_ something, or _know how_ to do something. It's very useful to think about building software as something analogous to organizing a tree of knowledge. 

We need to write code in a way that reveals our intentions - the what and the why (not only the how). We need to choose carefully in which branch of the tree each piece of knowledge should be hanging, in a way that [knowledge is not duplicated](https://verraes.net/2014/08/dry-is-about-knowledge/) and is available/connected to the other branches that need it. A good approach to code organization mindset is to think about it in terms of ["plumbing code and intelligence code"](/plumbing-and-intelligence), and then **avoid mixing** those two types.

# 5. Business knowledge > Tech knowledge

If we want our code to also be a representation of knowledge, then we must choose what kind of knowledge we want it to represent. 

There is knowledge about the technology - the _how_ - and knowledge about the business - the _what_ and the _why_. 

Our code should represent both, but the business knowledge should be primary and the technology knowledge secondary. 

More concretely, for example, let's say you have a software product for booking travels, the **names** of files, folders, variables, methods etc should "shine" things like {reservation, passenger, schedule, etc} **brighter** than things like {listeners, adapters, middlewares, triggers, endpoints, etc}. 

Technology and Business are both very important. And when representing knowledge as code, the business matters more. The vocabulary of the business should be the vocabulary of the tech.

# 6. Every improvement counts

You are part of a complex ever-evolving sociotechnical (people + technology) system.
You are both an element that participates in that system's processes and a agent of change within that system. 

When you change the system for the better you should feel good about it, and when others do it **we should celebrate and show appreciation** for it. 

By doing that we encourage the kind of behavior that keeps steering the system to a better place.

# 7. Compounding interest, for better or worse

We can all understand why tech debt is called "debt" - it has an interest rate, it grows if you don't pay it. Worse, it's compounding interest - the higher the debt, the quicker it grows. Yikes! 😱

But, on the bright side, what is "true for evil" is also "true for good". For example a single new automated test might:

* prevent an old bug from ever surfacing again, 
* create a piece of "executable documentation" for a module in the code
* allow the team to work on that feature faster next time

The investments we make in quality will also produce ever growing dividends in the future.

# 8. Self documenting

There's this idea that code should be [self-documenting](https://medium.com/codex/towards-self-documenting-code-371364bdccbb) and shouldn't need any comments to explain it. That comments are [like deodorant](https://medium.com/@harshavardhandharmavarapu/code-smell-track-them-efficiently-b20ead6d1363) that "disguise" bad smell without getting rid of what's causing it. _A comment is a failure to express yourself in code. If you fail, then write a comment; but try not to fail._ ([Uncle Bob](https://twitter.com/unclebobmartin/status/870311898545258497)).

I believe this idea is absolutely correct and even that it's generalizable for documentation in general. The best documentation is the one that you don't need to write, because it's already encoded in culture, and/or tools, and/or automation. People "do the right thing" either because it's automated, or because everyone else around them already do it, or because the tools in place makes it the easier path forward.

For example a repository's README should have sections like:

* 1. What is this repo
* 2. How to dev
* 3. How to run tests
* 4. How to deploy
* 5. How to observe
* 6. Internal architecture

But ideally the actual contents under those sections should be as short as possible like simply "start the devcontainer and run `start.sh` or "run `npm run test` from the root folder".

# 9. Code reviews: 3 reasons why

Code reviews are a great idea. It's no wonder everyone does it. And I think it's important to know why we do it. 

If you ask yourself why, the most likely reason that will come to mind is a) **"to proofcheck each other's work"**. Indeed that's very important, we do catch a lot of bugs during CR before they make it to prod.

But that's not the only reason. Here's two more: b) **to share knowledge**. Shared knowledge multiplies impact - you learn something nice and apply it somewhere else. It also de-risks having important company knowledge stored in just one head.

And c) **to standardize and converge**. If we don't look at each other's code, we're a lot more likely to reinvent the wheel, to create different solutions to the same problems. Peeking at each other's work allows everyone to reuse and standardize solutions to problems.

So, it's good to have those in mind when doing code reviews 😉.
