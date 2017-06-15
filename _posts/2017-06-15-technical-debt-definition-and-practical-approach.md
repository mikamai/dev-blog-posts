---
ID: 1505
post_title: 'Technical Debt: Definition and Practical Approach'
author: Nicola Racco
post_date: 2017-06-15 15:14:38
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2017/06/15/technical-debt-definition-and-practical-approach/
published: true
---
I know this post doesn't invent anything new and you may have read about technical debt other hundred of times. This post is just another take on the Technical Debt matter, with the description of the practical approach we follow in Mikamai.

The process of software development is traditionally composed by three phases: development, testing, release. These phases can then be further detailed or splitted: the development phase may comprehend frequent deploys on an *alpha* environment which will contain incomplete features but will allow to receive feedback easily; testing might then happen on different environments, sequentially (i.e. the first one is an isolated environment with an empty data set, the second one is an isolated environment with a realistic data set, the third one is accessible from the internet and can be used for stress testing, and so on)

The one thing happening at the end of the traditional development life-cycle is the release, where the term “release” is much more like a “furniture shipment” than a digital good delivery. In this traditional lifecycle the product is indeed considered stable, without any issues, and we can rapidly forget it and move on to the next *product*. There can be a period of controlled operativity where we’re admitting that the app may be affected by some bugs, that will be fixed. But anything weird happening after this period is considered *expected*.

Problem is: a software feature is not a desk, or a seat.
<!--more-->

After the release, even after a reasonable amount of time, we could discover that the developers didn’t fully understand the specs, with the needing now of rewriting parts of code in a hurry, without considering too much themes like code readability.

It may also happen that the delivered feature is good, it’s delivering the business value we had requested, be performant and without bugs. And even in this case, even with this excellent development, a change in the company strategy may require changes to the product some months or years from now.

Even in the most lucky cases, just the passing of time increases day after day the chance that the code need to be changed again, to accommodate a technological upgrade for example.

Whatever way it will happen, suddenly we’re handling this horrific code, written by developers that left the team years ago, written with an ancient technology, and even a small tiny little change, which normally would require just a day, is now requiring weeks.

For example we may found ourselves to have written an e-commerce in a month two years ago, but to need now two months to change a single feature, arguing on whose fault is it and if it’s still reasonable to keep maintaining a so pricey tool.

We’re exaggerating, but not that much.

## What is technical debt?

The concept of *technical debt* is a metaphor created by [Ward Cunningham](https://it.wikipedia.org/wiki/Ward_Cunningham) to describe the complexity in software projects. His idea is that releasing a first-time code is like going into debt, which can be paid back with a refactoring. Every minute spent on this not-quite-right code increases the interest on the debt.

Cunningham didn’t just find a good metaphor to describe complexity, but this same metaphor also works quite well to describe the worst case scenario, **if development doesn’t follow the correct methodology, the interest on the debt may go up so much to paralyze the project**.

Obviously there are developers and Developers, consultants and Consultants. Experience helps a lot in writing *better* code, in understanding the specifications and write the best implementation on the first try. But this is not enough: **sometimes even the best code is anyway difficult to understand and change, thus increasing the technical debt.**

Usually we tend to consider technical debt like something that can affect only the “old code”. But it’s not so obvious that a code written in the ‘70s will be more complicated of another one written in the ‘90s.

Another definition of Technical Debt, more commonly accepted, is the one provided by [Michael Feathers](https://michaelfeathers.silvrback.com/) in [Working Effectively with Legacy Code](https://www.amazon.it/Working-Effectively-Legacy-Michael-Feathers/dp/0131177052): **it’s code without test coverage**. It’s a good definition, which can helps us to avoid the mistake of associating the term technical debt with the term *old code*.

The reason why writing tests is such a good precaution against technical debt is that **tests provide an implicit documentation on the application functioning.**

So instead than talking of just “untested code”, we’re talking of:
- Code not easily readable;
- Code that doesn't contain explanations of the logic;
- Not having explanations of the ideas and the decision process that lead to writing the code.

Which offers a completely different point of view on this matter. Because at this point **technical debt is not only a technical problem, but it’s also a communication problem**.

## How is technical debt generated?

Let’s use the e-commerce example described at the beginning.

After the first 100 lines of code this technical debt could be paid back showing the code to a colleague. This simple operation would mean sharing the knowledge, having a feedback on the logic correctness, and a confirmation that the code is readable by others.

After a month, when the e-commerce was released, we had the minimum required to have a functioning platform. In this moment the technical debt could be paid back showing the buying process to our colleagues, checking again the code so we can be sure there are no dark spots too complex to read, and writing some automatic tests that can grant the purchase process is functioning. This technical debt is much higher than before, but still acceptable.

Two months after the release we’re asked of adding an asiatic country: everything need to change, from taxes to customer fields. Given that any change can break the purchase process developers have started testing their changes on the staging environment after each push. This is slowing down the development, and the technical debt is now so high that we’re starting to feel the first problems. To pay back this debt we should now completely change our strategy, starting to dedicate some time to write tests and clean the more problematic points of the code.

Six months have passed. On each new release we’re spotting new bugs, and for this reason it has been hired a QA specialist, whose duty is to visiting the staging environment before each production release to help spot and fix bugs. In this moment the interest on the technical debt is so high that we’re effectively paying a new employee, which would not be needed otherwise.


After an year: Company grew up, and now we need to add the brand concept to the e-commerce. This requires a complete refactoring to the app. The team is doing their best, but development is going slowly. I like to think that at this point the technical debt has gained a physical form: it’s a snake swallowing its own tail. Technical debt is not so high that any development is slower, that the team is tired to work on the project but at the same time is so busy to not have time for refactoring.


We’re not so close from the worst case scenario I mentioned before: rewriting the app costs too much (and rewriting from scratch is never a good choice), changing the app is instead too risky.


Summarizing, every time we’re talking of:
- Code difficult to change;
- Code difficult to read;
- Unstable app;
- Slow development cycle;
- Team hard to expand;
- Application hard to upgrade;
- Application hard to maintain;
- Low performances.

We’re talking of technical debt.

## How can we reduce the technical debt?

Technical debt is a communication problem, and this inevitably brings to mind the [Conway Law](https://en.wikipedia.org/wiki/Conway%27s_law): **organizations which design systems ... are constrained to produce designs which are copies of the communication structures of these organizations**.

Even before solving the technical debt into detail, changing the code, we need to solve the communication problem. We need to determine which communication protocols we’re going to use to document the project itself, the functional specifications, the ideas behind the code, and so on. Only after having decided which protocols to use we can delve into the code.

What are this communication protocols, which can help so much into reducing the technical deb?
- Behavioural tests: automatic tests which test features at a high level (i.e. Using the browser in a web product). More than their value as tests, they’re also usually easy to understand and can reveal a lot about the intentions of the developers;
- The code itself:
  - Comments and code formatting. Even a simple blank line between to code blocks can communicate a lot (for example two blocks that deal with two distinct phases of the process);
 	- Naming of variables, objects, methods, functions. Naming a variable “value” doesn’t say that much on what it may contain, and naming a method “calc” doesn’t say that much on what it’s calculating;
 	- Errors: raising errors with a detailed description of what is happening may save your day in a year from now, when that error will pop up from nowhere during a process that is considered ultra-stable;
 	- Again on the errors: there is nothing worse than not raising errors. The worst case is when you’re not receiving any error, still you don’t get any result (e.g. You see the “purchase successful” page but no order has been generated). Errors are useful, always.
- Version control system: A well written commit on Git may be the most precious tool to solve an incomprehensible bug, or when you’re reading an obscure line of code.
- Typical “project documentation”: setup procedures, deploy procedures, how to create a staging/production environment, how to execute a system task, and so on;
- Company chatroom, slack, forums, mailing lists. These tools have a persistent history and we can use them to keep track of discussions on the decision process;
- And there are a lot more than these, some of them more obvious than others.

Using the right communication protocols can help us to build something new, something that can allow knowledge sharing between developers (present and future ones), something that acts as a communication bridge between the owner of the platform and the developers of the platform.

Once we’ve set the communication protocols, we can start analyzing the project as it’s an ancient tomb, documenting each finding and trying to give an explanation to each object.

But the real cleaning should be done gradually, while we’re also delivering new features maybe. But we should avoid changing everything at once. **Rewriting a product is almost never a good idea**, because rewriting causes the loss of “implicit” specifications, of those hidden features that were contributing to give the product the experience we knew.

There are some other side effects when doing a refactoring on large scale:
- It requires a substantial budget;
- It requires to stop developing new features while cleaning the code;
- It increases the chance to generate bugs because we could clean features whose effects are not fully understood, or because the testing process cannot test all their edge cases.

Let’s imagine that our house is messy and some guests are coming (buying a new house and sell the old one is not an option). If we try to clean all the rooms together we’re risking to go out of time (and guests would see our house in a messy state). We could try to do just a rough cleaning of all the rooms, but our house would be still messy, only less messier than before. We could instead ignore the rooms when guests will not see (our restroom and the kitchen), and concentrate on the ones they’ll visit for sure (the living room and the bathroom).

We could proceed with the same approach when refactoring. We create new features using the new communication protocols, and every time our feature deals or need to change an old feature, we also clean that feature upgrading it to the new standard.

## A practical approach

Experience has taught us that we need to follow a method when a client contact us for refactoring, upgrading or maintaining legacy applications.

In these cases, the first step is a **Code Inspection**, to evaluate the situation. We do a code analysis (with the help of some tools), so we can get an idea of what points of the code are more harder to read or to change, of what security vulnerabilities are present, of the status of the documentation. As a result **we deliver to the customer a documentation which describes what we have found and what steps the customer could take to improve the situation**.

At this point it’s up to the customer to decide if this code inspection should result in starting a process of improvement. If the process starts, it’s time to **define a team**:
- If there’s already a senior technical team our developers usually joins the existing team;
- If there’s a technical team but it’s composed mainly by junior developers, we usually try to leave the new developments to the existing team (followed by a mentor). At the same time another team composed by specialists works on the maintenance;
- If there’s no technical team, we can provide one.

Then there’s the **definition of a release life-cycle**:
- In the worst cases we cannot do this without also creating an operations team;
- If there are no counter issues we usually try to put in place a continuous release process;
- In special cases (for example when each release requires to produce specific documentation for users of the platform) we advise to work with milestones and scheduled releases;

And finally we can **define the ceremonies and the communication protocols**: this step changes a lot from customer to customer, because some of them have enough time to participate to each communication protocol, some others may only join us for a review/planning meeting each week, some others don’t like video conferencing tools, and so on. The following is a typical setup:
- 15 minutes standup every morning at 10:00 (except Wed);
- 30 minutes review meeting every Wednesday at 15:00;
- 30 minutes planning meeting every Wednesday at 15:30;
- Production release every Thursday at 02:00;
- Retrospective and progress report every first friday of the month at 17:00;
- Definitions of *Done*. Example:
  - For a feature to be released in staging is needed:
 	  - Automatic unit tests;
 	  - *Code Review* from a colleague;
 	  - Being documented in the *Changelog*;
 	  - To have at least one integration test.
  - For a feature to be released in production is needed:
 	  - To be approved at the Review meeting by the *Product Owner*;
 	  - Be translated in each supported language.
 	- For a bugfix to be released is needed:
    - To have at least one test that is testing the absence of the bug.

Now the team can start working on the project. **The objective is to create stability, to reach that point of equilibrium between growth and stability in which developments at par while keeping the technical debt low.**

Usually during time we also find ourselves reducing the team size because:
- The amount of automatic tests reduce the regressions caused by any new feature;
- The code becomes more readable and more manageable;
- Junior developers learn how to write code with a low (or absent) level of technical debt.

The wealth we’re creating at the end of this process allow us to deliver new features easily. At the same time this creates a new class of developers, more *virtuous* and careful about certain themes. **In the end we can say that this process is becoming part of the company culture, changing the result of the Conway Law**.