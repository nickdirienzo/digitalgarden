---
{"dg-publish":true,"permalink":"/please-stop-it-with-the-fake-open-source/"}
---

I saw yet another Show HN for "open source B2B SaaS auth-in-a-box" today. It feels like the 4th time in the last year. The pitch is always the same: here's everything you need to authenticate in the enterprise, all on GitHub. Except it's not *all* on GitHub.

Since companies like WorkOS have taken off and Auth0 has been acquired by Okta ignoring all the startups, I've been seeing this trend of new companies pop up who lead with open source, build developer mindshare / adoption on GitHub and then you're stuck in their upsell funnel.

My problem is *these projects aren't actually open source*. At best they're open core -- with everything you _actually_ need to deploy these into production locked behind a paywall. At worst, it's a gross bait and switch. 

## Some "open source" examples

I wish I were making this up. Here are some pricing pages for some up and coming open source auth projects.

[Tesseral](https://github.com/tesseral-labs/tesseral): you can't even run on your own domain for < $250/mo.![Pasted image 20250528194401.png](/img/user/Pasted%20image%2020250528194401.png)
[SuperTokens](https://supertokens.com/): multi-tenancy doesn't even come in the free self-hosted version so I suppose you can only support one customer at a time.![Pasted image 20250528194347.png](/img/user/Pasted%20image%2020250528194347.png)
[BoxyHQ](https://boxyhq.com): you have to pay to customize branding on something you host yourself.![Pasted image 20250528194528.png](/img/user/Pasted%20image%2020250528194528.png)

Ory’s been doing great work here for a long time. But they did raise a $22M Series A in 2022 (congrats team Ory!) so we'll see how things play out over time. Candidly, I haven't been paying too close an eye on them but I've only ever heard good things.

Companies like WorkOS and Auth0/Okta are clear that you're locked into cloud pricing. At least they're not pretending to be an open source project. You are paying for a company to manage auth for you -- and that's fine. You know what you're buying that lock-in so you can focus on your product.

## How have we not solved auth already?

Getting authentication right is a core part of every business. You can (and probably should) outsource it -- it's not your core competency. Instead of betting your core infrastructure on a startup pretending to be open source, you can bet on something battle-tested for the last decade that truly is open source.

 [Keycloak](https://www.keycloak.org/) (supported by CNCF) has been doing a fantastic job here for over 10 years. They now support [Organizations](https://github.com/keycloak/keycloak/issues/30180) and there's work to be done still, but the path is there for true open source B2B auth support. Thanks Keycloak maintainers and CNCF for doing the hard work.

There's a whole [ecosystem of plugins](https://www.keycloak.org/extensions) which gives a chance for companies to contribute back openly so everyone can benefit. Originally organizations were supported by one of these extensions and now it's becoming a part of the core functionality. This is a win for Keycloak and a win for the software industry.

## Where do we go from here?

It's frustrating seeing a core piece of infrastructure be touted as open source but actually not be. I really wish we would stop this practice for lead gen.

For identity management, some of this is getting easier because major identity platforms are providing APIs for directory syncing: Microsoft's Graph API is finally good and Google Workspace has APIs for getting users. Many, many, many companies use one of these two products for managing their users. For auth, login with Google and Entra are effectively OAuth2 at this point and SSO has become easier to implement.

I think this should give more companies the options to support the enterprise through direct API integrations instead of some kinda-open-source-VC-backed-auth-provider. Hopefully we can stop rebuilding auth every year as a new startup and instead invest in solutions that are truly open, sustainable and portable. 