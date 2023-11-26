+++
title = "Custom TLDs"
date = 2023-04-30
updated = 2023-04-30
type = "post"
description = "What I learned from my curiosity about creating custom TLDs"
in_search_index = true
[taxonomies]
TIL-Tags = ["domains", "dns"]
+++

**Context:** I was just looking at some domain names for my name on Google domains. I saw that `vinayvarma.com` was already taken. The other domain names were not that exciting and I was not willing to splurge. I got curious: why can't something like `com.vinayvarma` exist?

Here are some questions I had and answers I found to be satisfactory.

### Who decides what domain endings (TLDs) are valid?

The Internet Corporation for Assigned Names and Numbers (ICANN) is the organization responsible for managing the Domain Name System (DNS) and the assignment of Top Level Domain (TLD) names. it is ICANN's responsibility to determine which domain endings are valid and can be used for websites and other online services.

### Why do I have to pay for custom domains (not even custom TLDs)? Who is this money going to?
If I want to buy vinayvarma.xyz now, I need to pay 860 rupees per year to Google domains. In this case, "google domains" is a company authorized by ICANN to manage the registration of domain names.

The domain name registrar (google domains for example) that you register your domain name with collects the fee and retains a portion of it as a registration fee. The rest of the amount goes toward the various entities responsible for managing the global DNS.

### What is the process to make your own TLD?
TLDR: If I want to have It is super tedious and expensive. [Here](https://serverfault.com/a/243335) is (a subset of) a list of checks that ICANN looks at before approving a new TLD. And it can cost more than USD$200,000.

Here are the general steps involved in creating a new TLD:

1. The first step is to develop a comprehensive proposal that outlines the purpose and goals of your proposed TLD, the community it will serve, and the technical and operational aspects of running the TLD.
2. Once your proposal is complete, you can apply for the TLD through ICANN's New gTLD Program. The application process is lengthy and requires a detailed examination of your proposal, including technical and financial evaluations.
3. Your proposal will be evaluated by ICANN to ensure that it meets a variety of technical, financial, and operational criteria. If your proposal passes the evaluation, you will be granted the right to operate the new TLD.
4. After your TLD is approved, you will need to launch and operate the TLD, including setting up the technical infrastructure, developing policies and procedures, and marketing the TLD to potential registrants.
5. Important to note that the application fee will be $185,000 and once your proposed TLD is approved, you will be charged $25,000 annually.

References: [1](https://dev.to/kailyons/tutorial-make-your-own-top-level-domain-name-like-com-org-and-net-jhd), [2](https://adrianroselli.com/2011/06/make-your-own-tld-i-want-bacon.html), [3](https://archive.icann.org/en/tlds/tld-application-process.htm)

### Why are the same domains available for different prices by different domain registrars?
Context: google domains lists `vinayvarma.xyz` at 860 rupees per year whereas `godaddy` lists `vinayvarma.xyz` at 169 rupees (for the first year).

The prices of domain names can vary across different domain registrars for a variety of reasons. Some factors that may affect the price of a domain name include:
1. Each registrar sets its pricing policies for domain names (popularity of the TLD, length of the registration period, level of customer support included). End of the day, it is a pricing competition.
2. Some domain registrations would come with additional features like email hosting or SSL certificates.
3. The price of a domain registration may also vary based on currency exchange rates, especially if you are purchasing a domain from a registrar in a different country.