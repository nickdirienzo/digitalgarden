---
{"dg-publish":true,"permalink":"/please-stop-calling-your-auth-startup-open-source/","created":"2025-05-28T19:33:21.444-07:00","updated":"2025-05-30T07:36:40.692-07:00"}
---

I saw yet another Show HN for "open source B2B SaaS auth-in-a-box" today. It feels like the 4th time in the last year. The pitch is always the same: here's everything you need to authenticate in the enterprise, all on GitHub. Except it's not *all* on GitHub.

Since companies like WorkOS have taken off and Auth0 has been acquired by Okta ignoring all the startups, I've been seeing this trend of new companies pop up who lead with open source, build developer mindshare / adoption on GitHub and then you're stuck in their upsell funnel.

My problem is *these projects aren't actually open source*. At best they're open core -- with everything you _actually_ need to deploy these into production locked behind a paywall. At worst, it's a gross bait and switch. 

## Some "open source" examples

I wish I were making this up. Here are some pricing pages for some up and coming open source auth projects.

[Tesseral](https://github.com/tesseral-labs/tesseral): you can't even run on your own domain for < $250/mo.![Pasted image 20250528194401.png](/img/user/Pasted%20image%2020250528194401.png)
[SuperTokens](https://supertokens.com/): multi-tenancy doesn't even come in the free self-hosted version so I suppose you can only support one customer at a time.![Pasted image 20250528194347.png](/img/user/Pasted%20image%2020250528194347.png)
~~[BoxyHQ](https://boxyhq.com): you have to pay to customize branding on something you host yourself.~~ Turns out [Boxy was acquired by Ory](https://www.ory.sh/blog/introducing-ory-polis-for-enterprise-single-sign-on) (congrats team Boxy!) a day after I wrote this, so I'm dropping from this list. 

Ory’s been doing great work here for a long time. But they did raise a $22M Series A in 2022 (congrats team Ory!) so we'll see how things play out over time. Candidly, I haven't been keeping too close an eye on them but I've only ever heard good things.

Companies like WorkOS and Auth0/Okta are clear that you're locked into cloud pricing. At least they're not pretending to be an open source project. You are paying for a company to manage auth for you -- and that's fine. You know what you're buying that lock-in so you can focus on your product.

## How have we not solved auth already?

Getting authentication right is a core part of every business. You can (and probably should) outsource it -- it's not your core competency. Instead of betting your core infrastructure on a startup pretending to be open source, you can bet on something battle-tested that is truly open source.

 [Keycloak](https://www.keycloak.org/) (supported by CNCF) has been doing a fantastic job here for over 10 years. They now support [Organizations](https://github.com/keycloak/keycloak/issues/30180) and there's work to be done still, but the path is there for true open source B2B auth support. Thanks Keycloak maintainers and CNCF for doing the hard work.

There's a whole [ecosystem of plugins](https://www.keycloak.org/extensions) which gives a chance for companies to contribute back openly so everyone can benefit. Originally organizations were supported by one of these extensions and now it's becoming a part of the core functionality. This is a win for Keycloak and a win for the software industry.

Auth feels like it should be solved territory by now. Logging in with Google or Microsoft Entra is a standard OAuth2 flow. A ton of companies have standardized around Okta for SSO. And supporting SSO is achievable through a number of open source projects like Keycloak, [passport-saml](https://www.passportjs.org/packages/passport-saml/), [django-saml2-auth](https://github.com/grafana/django-saml2-auth), and others.

## Where do we go from here?

It's frustrating seeing a core piece of infrastructure be touted as open source when the "open" part is just lead gen.

With auth being easier, the next hurdle is around identity management. That's getting easier too: Microsoft's Graph API is actually really good and Google Workspace has APIs for users. And for everything that's not Microsoft or Google there is SCIM.

There are so many companies out there that rely on Microsoft or Google. I don't think you need some open-core SaaS vendor in the middle anymore. You can just directly integrate with the identity platforms. 

Hopefully we can stop rebuilding auth every year as a new startup and instead invest in solutions that are truly open, sustainable and portable. 