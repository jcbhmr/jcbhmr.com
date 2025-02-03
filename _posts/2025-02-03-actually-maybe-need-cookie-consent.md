---
title: Actually you might need a cookie consent banner
---

This is not legal advice.

I recently read [Do cookie-free analytics need cookie banners?](https://jfagerberg.me/blog/2022-06-09-analytics-cookie-compliance/). It caught my eye because I thought "hey, does GoatCounter need a cookie consent banner?"

I'm not expert so I don't know. Probably?

Here's the highlights from the article that changed my mind and helped me understand why any analytics (probably) require cookie banners.

> you can’t access any data that’s stored on terminal equipment (nor store any new data), unless
>
> - you have consent, or
> - it’s strictly necessary for fulfilling what the user asked for.

> > While cookie-based analytics are often considered as a ‘strictly necessary’ tool for website operators, they are not strictly necessary to provide a functionality explicitly requested by the user […]. As a consequence, these cookies do not fall under the exemption.
>
> So in short: if you have a technical need to access the data to serve the user’s request, then you’re good without explicit consent. If it’s just because you want to, then that’s no good.

And the kicker conclusion from Johan Fagerberg:

> Unfortunately I came to the rather demotivating conclusion that there simply isn’t any way to implement web analytics without running afoul of the ePrivacy Directive.

Sooooo yeah. I won't be recommending GoatCounter anymore as a "you don't need a cookie banner" solution.

Another interesting note is that it seems like the ePrivacy directive is the real thing that protects users from unnecessary data collection with regards to analytics and not the GDPR.

TLDR: If you're operating inside the EU it seems like analytics require a cookie consent banner. At least according to [Johan Fagerberg](https://jfagerberg.me/).
