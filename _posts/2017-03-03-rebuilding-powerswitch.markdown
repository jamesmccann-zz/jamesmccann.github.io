---
layout: post
title: Rebuilding Powerswitch
---

<div class="mv3">
  <figure>
    <a href="/images/2017/03/03/powerswitch-rebuild-header.png">
      <img class="mb2" src="/images/2017/03/03/powerswitch-rebuild-header.png" alt="Powerswitch as it was in August 2016, Powerswitch in February 2017." />
    </a>
    <figcaption class="center tc"><em>Powerswitch as it was in August 2016, Powerswitch in February 2017.</em></figcaption>
  </figure>
</div>

For the past few months I've been working hard on rebuilding [Powerswitch](https://www.powerswitch.org.nz){:.link.dim.blue} for Consumer NZ along with
[Jack Callister](http://www.jackcallister.com/){:.link.dim.blue}.

When we kicked off this project in August of 2016, Powerswitch was a legacy platform in a maintenance only phase of life. The Powerswitch platform consisted of a Rails web application built in 2010 on Rails 2.3 and subsequently upgraded to Rails 4, a Rails 2.3 microservice for calculations, and a Microsoft Access Database for power plans and tariff information. I'll break down the responsibilties of each of these pieces, but a tl;dr on what we did:

- we replaced the Rails web app with a Node web server and React client,
- we kept the Rails 2.3 calculator service (now Rails 5), but stripped it of it's database and refactored it as a stateless calculation engine,
- and our MS Access database has been retired and replaced by a new Rails app for data-entry administration of power plans and reporting.

## August 2016: the state of things

In 2016 Consumer determined that Powerswitch needed to be replaced or retired. The Powerswitch platform we were tasked to replace was starting to show it's age in a few ways.

For some background, the Powerswitch web app is a multi-step questionnaire, asking users to tell us a few details about
their household and power usage so we can run some calculations and tailor an estimate for the appropriate cost of power from a range of retailers. A user can see a list of results of power plans that offer them potential savings, view details about each plan, and if they want to begin a switch to that plan without leaving the Powerswitch site. This last feature is the most important: the aim of Powerswitch is to benefit all electricity and gas users in New Zealand by increasing competition in the retail market and encouraging people to switch their supply to retailers offering better plans.

To make this app run, the Powerswitch Calculator maintained a database of all the electricity and gas retailers in New Zealand, the plans available from each of those retailers, and the tariff prices associated with each plan, e.g. the daily rate of the connection, and the price per kilowatt hour of electricity to each of the meters that the plan supported. The Calculator did not have a CRUD interface for managing these plans and tariffs - a
Microsoft Access database was maintained offsite by Consumer's pricing analyst and periodically the entire database would be dropped and replaced
with an up-to-date dump of the Access db. This was called publishing and happened about once a month but occasionally more frequently.

The Calculator provided an API to the Rails web application for generating results whenever a user submitted a questionnaire. The results API would return
a list of electricity, gas, or dual fuel (combinations of electricity and gas) plans along with the estimated annual cost of the plan based on the
questionnaire responses of the user.

The system above had worked well since it was deployed in 2010, but the cracks were starting to show:

- the front-end web app was not responsive and mobile experience was terrible, very few of our switches came from mobile devices and the bounce rate was high
- the questionnaire design was not encouraging, users were shown a wall of questions with checkboxes with little visual help to guide them through
- price updates were time consuming and even the smallest price change could only be done with MS Access and a full re-publish of the entire database
- there was a demand for new features to encourage switching and help users reduce power consumption, these were difficult to implement in the legacy platform

## August 2016: designing the rebuild

Our aim was to rebuild the entire platform starting mid August with a target deploy date of 31 October. In those early days this seemed pretty achieveable...

We decided to break the delivery into two phases. Firstly, we would redesign the front-end site from the ground up to be responsive and with a more user friendly
design. Our wonderful UX designer Elle redesigned out questionnaire into a multi-step process that asked the user only one question at once, allowing them
to focus on each answer individually without becoming distracted or confused about what step they were on. During the first phase, the calculator service would
stay relatively untouched and the APIs it provided would be used to generate results for the new front-end. The second phase was designed to rebuild the backend
administration: a new web app for adding and updating retailers, plans, and calculations with a consistent API that could be easily plugged into the front-end
when the time came.

This approach had a few really important benefits that paid off well for us. Rebuilding the front-end and launching it with an as-is backend reduced the number of moving parts that were in development at once. We removed a lot of complexity from this initial part of the project until it just became a job of building a responsive web app to display data that already existed. We could ship this rebuild quickly and ensure it was stable before launching the more complicated rebuild of the backend and calculations engine. We also had a consistent baseline at every step of the way. The legacy platform had virtually 0 tests - the only measure of correctness was whatever the old site was doing. So while we developed the new front-end we constantly compared results to the live site in parallel and ensured that there were no unintented changes between them. This became our integration test suite. This approach again paid off when we began testing the results of the new backend - we had a new live site that had the same calculations as the old site, and a new staging site with new calculations that we could compare to.

The two-phase build also had another hidden benefit: no feature creep. Second system syndrome is real and it hurts, but in this case we tried not to add any new features with a policy to not add any non-critical code to the old backend during the build (helped by the fact that nobody wanted to write new code in the 6 year old Rails app). Most front-end features that had been designed but couldn't fit into this setup were cut - even today we still haven't added a lot of these features because they haven't become important and this is a good thing.

We've reduced a lot of complexity by deferring new work for as long as possible, and the new features we're adding now are also much easier today on top of the new platform - this has been a win/win situation for us.

## September 2016 to November 2016: redesigning the front-end

We replaced the old powerswitch.org.nz Rails app with a new NodeJS web app and a React client frontend.

<div class="mv3">
  <figure>
    <a href="/images/2017/03/03/powerswitch-results-trends.png">
      <img class="mb2" src="/images/2017/03/03/powerswitch-results-trends.png" alt="Powerswitch results and trends pages in 2017." />
    </a>
    <figcaption class="center tc"><em>Rebuilt results (L) and price trends (R) in February 2017.</em></figcaption>
  </figure>
</div>

Our goals for rebuilding the user facing site were: speed and ease of completing the questionnaire, and a responsive design that performed well on all devices. We decided to build a client-side app on top of React to tick off our speed goals. Having a client-side app for the multi-step questionnaire provides instant feedback for users as they provide answers and complete each step. There's no page loading between steps and the questionnaire feels more engaging. The small checkboxes from the old site have been replaced with larger icon buttons to support touch interfaces which serve a dual purpose of being easier to use on a mobile, but also easier to understand thanks to the visual prompt.

Using Node on the server side made it easier to work with the modern JS stack. NPM, webpack, and ES6 are first class citizens here and we found server-rendering React to be far easier on Node than in a Rails stack where a separate JS engine needs to run server side. The Node server responsibility is handling client side requests and calling into some internal service-side APIs provided by another Node service (called the Platform API) to return data.

The Platform API is a small Node service layer that decouples the front-end server from the services it needs to fetch data from. When we started building the front-end we were still using a combination of the legacy backend and some new endpoints in our new skeleton backend, but to the front-end server it's all the same. The complexity of whether data was fetched from the old calculator or the new plans database was all contained within the Platform API and switching them over when we retired the legacy stack was as easy as a simple route change in the Platform layer. We also tap into some external services via this Platform layer, e.g. [AddressFinder](https://addressfinder.nz){:.link.dim.blue}
for address lookups - using the Platform API to contain this external service allows us freedom and flexibility to change the underlying service without affecting any other parts of the app.

Thanks to simplifying the development required here, we were able to ship the new front-end in the first week of Novemeber (just slightly over our target of October 31).

## November 2016 to February 2017: redesigning the back-end

As soon as the new front-end was live we got to work rebuilding the backend. Our primary goal here were to completely remove the reliance on the Microsoft Access database for publishing new plans and tariffs. Access was a significant risk: only one person had the local environment and expertise with it's schema to manage price changes, and the "all or nothing" publishing approach required a lot of care to not stuff up. We wanted the plans data to be managed by a CRUD interface that was easy for anyone to pickup with a basic understanding of power pricing.

In saying that though, power pricing turns out to be a pretty complex domain! There's 20 retailers in New Zealand that Powerswitch supports. Each of those retailers has different power plans and prices in a range of electricity and gas networks throughout the country - we've got 129 distinct networks that retailers offer power prices on, and about 750 unique sets of plans that are offered in those networks. Over the entire country we support 20700 different plans, and
34700 individual tariffs, combinations of which are used to make up the plans.

So we came up with a domain model that would suit normalized CRUD data entry, breaking plans up into plan sets, tariff sets, and tariffs so that our data entry required to support this is as minimal as can be. Regions in New Zealand are broken down into electricity and gas network locations and each retailer has a mapping from their available plans into a set of available network locations so that we don't have to duplicate plans that are offered in more than one network.

Now the normalized data model is fantastic for CRUD - there's no data duplication and everything is relatively easy to update once and it's good to go. But this data model is a horror to query. Our front-end was based around the notion of "plans", it doesn't know or need to know about how that plan is composed of a set of fields from a "plan set", and another set of fields from a "tariff set". We can return all of the data together as a plan on the fly as a JSON response, but querying many plans over a number of tables joined like this is inefficient and might hit performance issues when the dataset grows large.

In addition, we needed to support editing the data in the CRUD tables and saving it in a "draft" state for sign-off. For example a retailer gives us new price information, we need to update that and save a draft, send them a report of the
price changes and the plans affected with the draft changes applied, and then if everyone is happy - publish that change to live. To do this in a relational db with a normalized schema often involves a completely duplicated "published" schema to store the published versions of all records.

The complexity of having our entire normalized data model stored twice in draft and published schemas and then on the fly joined to make a JSON response seemed a bit unweildy, so we opted for a different solution: CQRS.

### CQRS

CQRS stands for [Command Query Responsibility Segregation](https://martinfowler.com/bliki/CQRS.html){:.link.dim.blue} (Martin Fowler can describe it far better than I ever will). In short: it's the introduction of a separation of the models used for updating records to the models used for querying those same records, to avoid a shared model which does neither well.
The idea of this approach is to allow for an optimal workflow for updating records (e.g. normalised CRUD) and an optimised denormalized form to read the same data in other interactions when we're interested in viewing it all at once and not interested in updating or deleting certain components.

In our situation CQRS is a natural fit: we have one system that is designed to be used by expert users to update draft data and publish all of the records that combine to make power plans and another system that only needs efficient read-only access to a combined view of the published plans. This is also a natural separation for our draft and published states: we can maintain all our data in draft in the CRUD tables, and only store published data in the denormalised query plans table.
Our API we provide to the front-end only uses the published plans, and it's amazingly easy to query thanks to having no joins to other tables - every query is just a simple `where` on a single field, voila! We're free to make as many updates as we want to our draft schema and nothing else outside of the backend app knows any better.

### Publishing and Managing Revisions

<div class="mv3">
  <figure>
    <a href="/images/2017/03/03/powerswitch-revision-sets.png">
      <img class="mb2" src="/images/2017/03/03/powerswitch-revision-sets.png" alt="Editing revision sets in the Powerswitch Admin app." />
    </a>
    <figcaption class="center tc"><em>Plan changes and price updates are
    published through Revisions.</em></figcaption>
  </figure>
</div>

We designed our own publishing system on top of this separation that we call revisions and it's also been extremely easy to implement. Our schema relies on the draft plans model having dependent fields in other models. The process for publishing a single draft plan is simple (and idempotent): just join all the fields you need together and dump them into a published plan record, done! So anytime we change any (plan, plan set, tariff set, tariff value) from "clean" we just store it's attributes as JSON in a revision. Each revision affects many plans through the related models so when we go to publish that revision we just query for the draft plans, and call the same simple publish process on all of them. If we want to rollback a revision it's easy: just update the model with the copy of the last known "clean" attributes we stored as JSON.

We combine these revisions into "sets" that we name to tie together all of the changes - this is a bit like a staged git commit where we have a number of files/models with changes and we can either commit those changes (republish all the affected plans) or rollback by replacing the draft data with the stored JSON attributes. We keep all the changes in each revision set related to each other so they can be published individually once they're signed off.

As an example: we might make a revision set called __Contact Energy Feb 2017__ and proceed to make some price updates in one of the tariff sets for Contact plans in Auckland. We go and update all of the models and each revision is saved into our set. When we're done we can look at the set, see a diff of all the changes, and generate a report to send off
to Contact for sign-off. When everyone's happy we can publish the revision set and all the Contact plans in Auckland that are affected by the change will be republished with the new prices. Hooray!

This system is working really well for us so far and has been a joy to build. The code is simple. We don't have a lot of publishing concerns (auto-publishing associations anyone?) bleeding all through our codebase, and I could easily explain the entire publishing flow to someone in under an hour.
Only one class has responsibility for publishing plans, and it's easy to read and easy to test.

In February 2017 we launched our new backend. Aside from some small data structure changes we had virtually no changes to our front-end - our Platform API just changed from serving data from one backend service to another, and everything else carried on exactly as you'd expect. Our launch took less than an hour with no issues - an achievement we are really proud of.

### Today: a retrospective

Looking back on our progress, I'm really pleased with the platform we're working with today, but there are still improvements to be made and lessons to take away - here's a few of mine:

- **We underestimated how long data would take to get right**: migrating six years of plans data from an Access DB to a new schema and redesigned platform took longer than expected,
and it's definitely something I'll look out for in future. We spent a lot of time in CSVs, importing in and out of PostgreSQL, transforming and mapping data and verifying the data
integrity. It requires care and it's time consuming, lesson learnt here.
- **Services are great but deployment is complex**: we now have have four services behind Powerswitch, and ensuring these are deployed in parallel and operating correctly definitely
has devops overheads. There's probably too much coupling between our services, as ideally it'd be possible for our data-loading backend to go down without losing the front-end site,
but this isn't the case today. Taking care to design service lines along [Bounded Contexts](https://martinfowler.com/bliki/BoundedContext.html){:.link.dim.blue} would have helped us here: if our published plans and other front-end facing data was in it's own service then the CRUD backend could function completely independently from the user facing site. This is on our list of future refactorings to work towards.
- **Remote working is fantastic**: We started this project with two devs based on opposite sides of the world while Jack was living in London. Our pricing analyst lives
in Fielding, I live in Gisborne, and the Consumer office is in Wellignton. Jack is on his way back to Europe this week and so far we haven't missed a beat. We use Slack for regular comms and Skype whenever it's required. I really have to acknowledge Consumer for their willingness to take a risk and engage with us as remote devs which has allowed us flexibility and work-life balance from wherever we are - thank you!

And finally just to finish - some more technical details on our stack (let me know if any of them are interesting enough for a follow up):

- Front-end: [React](https://github.com/react/react){:.link.dim.blue}, [Router](https://github.com/ReactTraining/react-router){:.link.dim.blue}, [MobX](https://github.com/mobxjs/mobx){:.link.dim.blue} for state management, [JSON Web Tokens](http://jwt.io/){:.link.dim.blue} for authentication, [Skeleton.css](http://getskeleton.com/){:.link.dim.blue} as the only CSS "framework", server-rendering of all React views,
hybrid SPA - we've got server side routing to separate React client-side SPA's.
- Back-end: Rails 5, CRUD views are standard Rails with isolated React components where required, Rails controllers for the admin, [Grape](https://github.com/ruby-grape/grape){:.link.dim.blue} for API endpoints, JWT authentication for the API endpoints,
[Tachyons](http://tachyons.io/){:.link.dim.blue} for CSS (virtually no other custom CSS!), [PostGIS](http://www.postgis.net/){:.link.dim.blue} for mapping addresses into geofenced network boundaries, "use case" architecture for most actions.

I'd love to hear from you if any of the above was interesting and you would like to discuss more about it or similar experiences you've had! Please shout me out on [Twitter](https://twitter.com/jmccnz){:.link.dim.blue} or flick me an [email](mailto:jmccnz@gmail.com){:.link.dim.blue}.
