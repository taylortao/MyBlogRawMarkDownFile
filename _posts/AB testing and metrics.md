---
title: AB testing and metrics
date: 2017-01-04 13:49:25
categories:
 - IT Technology
tags:
 - AB Testing
 - Measurement
---

### Procedure to conduct an A/B test
> ** Step 1: ** UX Code Instrumentation - Server / Client : Usually in UX code base, we need to log some logs, and in client, we can ping a url to log to server side.
> ** Step 2: ** A/B Flight
> ** Step 3: ** Generate A/B test scorecard
> ** Step 4: ** Analysis, and decide next step (ship or iterate)

<!-- more -->

### Some success metrics
> Sessions/UU

Average number of visits per user. It is a long term metrics.

> Session Success Rate, Time to Success

It is important to define a click that a user is satisfied with, saying a **successful click**, it is a dwell time of at least 30 seconds or is the last click in a user's session, and without query reformulation, **a query reformulation** occurs when one query is followed by another query in the same session, whose query text is very similar to the preceding one. 
Based on this successful click, there is **Session Success Rate**, means the average probability that a Session is successful.

**How to define a session is successful?**
The user actions determine the probability for success of a given session, with everything else contributing zero success, such as: 
**Successful Click (1.0 success):** A successful click not immediately followed by a reformulation.
**Answer Hover (0.5 success):** Hovering over an answer for at least 2 seconds.
**Carousel Arrow Click (0.1 success):** A click on a carousel "next" or "previous" arrow.
**Tab Click (0.75 success):** A click on a tab that opens it for at least 5 secs.
**Inline Expansion Click (0.75 success):** A click on a chevron inline to unhide a certain block of information.


**Time to Success** is an unconditional version for time to first SAT click of a successful query.

> Utility Rate of a Web Page

The utility metric takes a holistic view of the session. It takes into account of all the user interactions with the web page, the main purpose is to understand the satisfaction, dissatisfaction of the user as they progress in their timeline. All user actions can be classified to successful, ambiguous or unsuccessful depending on our knowledge, all the actions can be given a value between -1 (extreme failure) and 1(extreme satisfaction).

When these actions occur within a session, they have an effect that is specific to that session. If the user performed an action whose outcome is not desirable, this dissatisfaction will color the time after that action until a new action is performed. On the other hand, if the action led the user to a useful destination, their time on the destination is enjoyable. Hence, the time after a user action can be viewed as a payout: a successful action will cause the user to have a good time after that action while a failed action will cause the user to struggle after that action and before next action.

So, payout can be defined as like:
![payout definition](payoutDefinition.png)

and utility rate can be defined as like:
![utility rate definition](utilityRateDefinition.png)
