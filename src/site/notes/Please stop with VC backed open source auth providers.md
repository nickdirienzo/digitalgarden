---
{"dg-publish":true,"permalink":"/please-stop-with-vc-backed-open-source-auth-providers/"}
---

I saw yet another Show HN for "open source B2B SaaS auth-in-a-box" today. Here's all the login types you could want, SSO, SCIM, and self-service configuration -- all on GitHub! But you have to pay for *something* for this to be production ready.

I get it. There is enterprise value to be had and building for the enterprise is hard. 

This feels like the 4th time this year I've come across an "open source" project promising to solve auth in an open source way because companies like WorkOS have taken off and Auth0 has been acquired into Okta ignoring all the startups. 

These projects aren't even open source in nature; at best they're open core. Everything you _actually_ need to deploy these into production are behind a paywall. At worst, it's a gross bait and switch. I wish I were making this up. Here are some pricing pages for some up and coming projects in the space.

[Tesseral](https://github.com/tesseral-labs/tesseral): you can't even run on your own domain for < $250/mo ![Pasted image 20250528194401.png](/img/user/Pasted%20image%2020250528194401.png)
[SuperTokens](https://supertokens.com/): multi-tenancy doesn't even come in the free self-hosted version ![Pasted image 20250528194347.png](/img/user/Pasted%20image%2020250528194347.png)
[BoxyHQ](https://boxyhq.com): why are SSO and Directory Sync in Free Forever and Self Hosted Premium? Guess you have to pay for custom branding. ![Pasted image 20250528194528.png](/img/user/Pasted%20image%2020250528194528.png)

Ory’s been doing great work here for a long time. But they did raise a $22M Series A in 2022 (congrats team Ory!) so we'll see how things play out over time. Candidly, I haven't been paying too close an eye but I've only ever heard good things.

Companies like WorkOS and Auth0/Okta are clear that you're locked into cloud pricing. At least they're not pretending to be an open source project. You are paying for a company to manage this problem for you -- you know what you're buying that lock-in.

Getting authentication right is a core part of every business. Yes, you can (and probably should) outsource it. But there are better, real open source alternatives that do exist and are solving the "CIAM" problem (omg when did a Gartner category appear for this space). We should be spending our resources supporting these real open source projects instead so they can build for the larger software community:

 [Keycloak](https://www.keycloak.org/) (supported by CNCF) has been doing a fantastic job here for over 10 years. They now support [Organizations](https://github.com/keycloak/keycloak/issues/30180) and there's work to be done still, but the path is there for true open source auth support. (Thanks Keycloak maintainers and CNCF!)

Are there others here? I have only come across Keycloak which truly is an incredible effort.

It's frustrating seeing a core piece of infrastructure for every software stack be touted as open source but actually not be. 

Fortunately some of this is getting easier because Microsoft's Graph API is finally good and Google Workspace has APIs for getting users. Nearly all companies fall into one of these two buckets. I think this should give more companies the options to support the enterprise through direct API integrations instead of some kinda-open-source-VC-backed-auth-provider.

/rant