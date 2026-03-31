---
{"dg-publish":true,"permalink":"/notes/the-software-supply-chain-is-an-active-combat-zone/","tags":["essay"],"created":"2026-03-31T12:43:35.793-07:00","updated":"2026-03-31T15:40:36.927-07:00"}
---

Software supply chain attacks are increasing in frequency and severity. This month alone we've seen multiple unrelated campaigns hitting the open-source ecosystem simultaneously. This isn't one bad week. It's the new normal.

Two weeks ago, an automated bot exploited a misconfigured `pull_request_target` workflow in [Trivy's GitHub Actions](https://thehackernews.com/2026/03/trivy-security-scanner-github-actions.html) to steal a Personal Access Token. That gave the attackers (a group tracked as [TeamPCP](https://www.sans.org/blog/when-security-scanner-became-weapon-inside-teampcp-supply-chain-campaign)) enough to compromise Trivy's release pipeline, poison 75 of 76 version tags, and push credential-stealing malware through official releases. From there, the stolen credentials cascaded:

- **March 19:** Trivy compromised. Credential stealer deployed via GitHub Actions, Docker Hub, and binary releases.
- **March 23:** Checkmarx KICS VS Code extensions and GitHub Actions compromised using credentials from the Trivy breach.
- **March 24:** [LiteLLM compromised on PyPI.](https://futuresearch.ai/blog/litellm-attack-transcript/) Malicious versions harvested SSH keys, cloud credentials, and Kubernetes secrets.
- **March 27:** [Telnyx compromised on PyPI.](https://telnyx.com/resources/telnyx-python-sdk-supply-chain-security-notice-march-2026) Same playbook.

Just today, [Cisco's source code was stolen](https://www.bleepingcomputer.com/news/security/cisco-source-code-stolen-in-trivy-linked-dev-environment-breach/) leveraging the Trivy vector. SANS analysts believe TeamPCP may have a stockpile of compromised publishing credentials and could be operating as an Initial Access Broker. This is likely not over.

Separately, and this appears to be a different actor entirely, [axios was compromised](https://socket.dev/blog/axios-npm-package-compromised) on npm yesterday. The attacker used a stolen long-lived npm access token to publish malicious versions of one of the most widely used packages in the JavaScript ecosystem (~100M weekly downloads).

Last September, the [Shai-Hulud campaign](https://www.cisa.gov/news-events/alerts/2025/09/23/widespread-supply-chain-compromise-impacting-npm-ecosystem) hijacked 18 popular npm packages by phishing maintainers with fake 2FA reset emails. As an engineer, we tend to think we're too smart to fall for social engineering. We run these simulations for a living at Mirage, and I can tell you: we're not. The attacks are better than most people expect.

Different actors, different methods, all hitting at the same time. We can't control upstream maintainers, but we can harden how we consume their work:

1. **Save exact versions.** Pin direct dependencies and lock the entire dependency tree.
2. **Set a maximum package age.** Something sensible like 7 days so that brand new malicious versions aren't automatically installed. npm, pnpm, Yarn, and Bun all support this natively now. Renovate and Dependabot respect it too for automated PRs.
3. **Pin GitHub Actions to commit SHAs, not version tags.** Tags can be force-pushed. This is exactly how the Trivy compromise propagated.
4. **In Node, don't allow install scripts.** This was what allowed the axios C2 to run. In Python it's worse: you can't prevent `.pth` files from executing.

Software supply chain security used to be something you audited quarterly. That cadence doesn't match the threat anymore. Treat your dependency pipeline like infrastructure that's actively under attack. Because it is.