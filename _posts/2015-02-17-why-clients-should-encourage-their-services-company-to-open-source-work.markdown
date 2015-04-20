---
layout: post
title: Why Clients Should Encourage Service Providers to Contribute to Open Source
---

This is a post I have wanted to write for some time after experiencing
first-hand the difficulty of suggesting open-sourcing code within a
client project. From my experience, the question of making code free
and available to anyone does not resonate with clients paying for the code.

The status quo mentatility that we shouldn't open-source code a client has
paid for not only leads to a lack of contributions from service providers that otherwise
use a lot of open-source libraries; it is also likely to produce a lower
quality, more expensive product.

Here are a few reasons why you as a client should encourage a freelancer
or service provider to open source your code.

## Products of the Provider are Publically Visible

If you do not not employ your own technical staff (i.e. software developers),
there is a risk involved in blindly trusting the skill of contracted developers.
Just because a system appears to meet requirements in the user-facing interface does not imply that the code
itself is well designed and tolerant to future changes. A codebase could
be poorly designed, poorly documented and not easily extendable or maintainable.
This leads to significantly higher maintenance and future development
costs.

This effect also works in the positive sense: a services provider or
freelancer being required to open-source work they are producing for a
job has an interest in not producing poor quality code as well. An
organisation with an open source profile of poorly implemented client
projects will pay with their reputation. By encouraging service
providers to open source as much code as is reasonably possible, there
is a greater incentive for developers to take more care with what they
write.

## Developers are Encouraged to Write More Maintainable Code

Contempary wisdom in software development advises developers to split code
into small, reusable, testable, general purpose "modules" that implement a
single piece of functionality. Complex systems built around this
concept tend to be easier to maintain, less prone to bugs, and cheaper
to change when adding new functionality.

Small, general purpose modules are also great candidates of code within
your project that can be easily open sourced.  By encouraging your services provider
to open source as much code as possible, developers are able to design
code in a modular style from the outset of your project. You receive the
benefit of cheaper development and long-term maintenance, and the
open-source community receives the benefit of the open-source modules
you are able to spin-off in the process.

Depending on the project and nature of the work, the amount of code that
can be identified as general purpose enough to be worth open-sourcing
will differ. It is not in your best interests as a client to open source
any core business logic that gives your product or service a competitive
advantage. However, depending on your domain, there is likely to be
opportunities for general purpose solutions to the specific problems of
your project.

## Community Support

One of the most significant benefits of open-source code is the
community that collectively improves it. Popular open-source software
benefits from communities of thousands of developers using the code,
finding bugs, and contributing patches. Even without community patches,
additional users of your open-source code increases the number of use
cases and makes it far more likely for bugs to be encountered sooner and
made visible to developers. Bugs that could have been encountered in
your own software and fixed at a cost can now be found earlier and fixed
free of charge.

Community support is bi-directional. It's impossible today to
build and deploy a complex system without relying on open-source code.
From small code libraries to operating systems or web server software,
your company already makes use of open-source software everday.
Contributing some of your own products back to the community a great
way to support the open-source movement and the work of thousands of
developers worldwide who put their time and effort into developing
open-source.

"OK, OK - these benefits are all well and good", you say,
"but how should I decide when and what code you can open-source?"

In my opinion, trust your developers! Most developers that I have met have a keen interest in
increasing their own open-source profile. Developers also tend to know
when something that they have been tasked with building would be helpful
as an open-source library - likely one of the first things they do when
building a new feature is to check if it exists already as an
open-source library that can be reused.

Encourage your developers to make suggestions when they feel that they could publish areas of their
work and they will jump at the chance.






