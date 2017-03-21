---
ID: 20
post_title: 'The Q in your QA, or: how to ensure code quality, part I'
author: Cecilia Nardini
post_date: 2016-07-08 09:21:22
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/07/08/the-q-in-your-qa-or-how-to-ensure-code-quality/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/147086687489/the-q-in-your-qa-or-how-to-ensure-code-quality
tumblr_mikamayhem_id:
  - "147086687489"
---
Code quality is an elusive objective. In principle, everyone agrees to it, but the means to achieve it are varied and opinions on them even more so. In what will hopefully be a series of posts, I’d like to share some thoughts on the matter.

Today we’re going to focus on the QA practice of code reviews or CR. Code review is a process of peer evaluation in which code written by a developer is inspected by another person to look for defects and improvements.
Here in Mikamai we introduced CRs some months ago as a step to improve our QA process; we have found this practice to be mostly beneficial, and we feel ready to share our take on the matter in the form of giving our answer to some Frequently Asked Questions about this topic.

<!--more-->
<h3>What is the true purpose of CRs?</h3>
Code reviews shouldn’t be primarily aimed at correctness. They are meant to improve code readability and maintainability and code-base consistency.

In addition to that, CRs have an educational value.
In that respect there are differences between CRs and pair programming. While the educational aim of pair programming is encouraging the transfer of knowledge and problem-solving skills between members of the team, CRs are primarily a means for educating the team on what the team’s standards are.

In order to fulfill these aim, CRs should be fair and open and the team’s standards should be agreed upon and accepted by every team member to begin with. You can consult <a href="http://codeahoy.com/2016/04/03/effective-code-reviews/">this helpful post</a> on
how to make fruitful code reviews.
<h3>Which is more important, testing or doing CRs?</h3>
<strong>Short answer: both.</strong>

Yes, a CR can spot basic mistakes or points where the code will not work as expected. However, the code should arrive at the CR stage already tested and production-ready.

Furthermore, since automated testing is code, it too is subject to review, and this is where CRs can reveal a meta role with regards to testing. A review can provide insight on the test coverage, by pointing out whether success and failure cases get adequate coverage or whether non-trivial cases are taken into account.
<h3>What about legacy code?</h3>
Given that one of the problems of legacy code is the lack of tests, it is legitimate to wonder what is a safe and QA-aware way to make changes to existing legacy code. Definitely an age-old question.

The very first piece of advice is that of introducing tests whenever possible: refactorings and adding of new features should come with their own test suite, while a change big or small in an existing chunk of code can provide the opportunity to give some coverage to previously untested behaviors.

All this is very obvious, yet not always feasible; additionally, even in the best case scenario regressions are always lurking in those untested parts of the code.
Manual testing is often the last resort in this kind of situation, but can CRs be of help?
Bearing in mind that, as we have said, CR is not a substitute for testing, CRs can provide an additional advantage if the reviewer knows the legacy code base well and thus can point out inconsistencies with the existing code or things that will not work as expected due to some quirks of the legacy code.

This is all for today, if you found this post interesting stay tuned for more to come!