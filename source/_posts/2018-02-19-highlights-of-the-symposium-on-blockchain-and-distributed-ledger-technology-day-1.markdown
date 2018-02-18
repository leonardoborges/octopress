---
layout: post
title: "Highlights of the Symposium on Blockchain and Distributed Ledger Technology - Day 1"
date: 2018-02-19 10:00
comments: true
categories: [conferences, blockchain]
---

Earlier this week I had the chance to attend the [Symposium on Blockchain and Distributed Ledger Technology](http://blockchain.unsw.edu.au/symposium18/HTML/index.html) organised by UNSW. I wasn't aware that UNSW was so heavily involved in the community until this event and have since learned about their [interest group](http://blockchain.unsw.edu.au/) which has been going on for a while.

The talks had a healthy mix of research and industry case studies with subjects ranging from the technical challenges of current blockchain implementations to the legal implications of distributed ledger technology.

I'm not sure when and if slides will be published so I'll do my best to summarise the talks and share its highlights. Most likely this summary will reflect my own biases on the subjects which interest me the most.

# Day 1

## An Introduction to Distributed Ledger Technology - Ron van der Meyden (UNSW)

Ron is the organiser of the interest group and this particular event. In this introduction he gave an overview of what a blockchian is and how it works. A great start for the day!

## Ripple - Dilip Rao

Dilip is Ripple's Global Head for Infrastructure Innovation. Currently with about $94M in funding, Ripple is backed by some fairly big enterprises and banks as well as high profile customers such as Westpac, Santander, BBVA and dozens more.

Ripple has made some interesting trade-offs:

*   It accepts and embraces the fact that banks **will not** run on a public ledger. It simply won't happen
    *   One of the reasons discussed is the fact that through a combination of various sources and pattern analysis one could figure out trading volumes between banks and enterprises and they are not onboard with that.
*   As such Ripple uses its own private ledger based on the [Interledger protocol](https://interledger.org/). Only on-boarded participants - banks - may interact with it. 
    *   The product is called [xCurrent](https://ripple.com/solutions/process-payments/). It allows banks to settle cross-border in real time.
*   This highlights an interesting fact: **banks don't need the Ripple cryptocurrency (XRP) to settle transactions**

The Interledger protocol allows multiple ledgers to communicate. Currently it can communicate with the XRP(Ripple) ledger to provide fast payment channels. Due to this specialised nature, XRP now has the cheapest cost per transaction as well as being the fastest clocking in at 1500 transactions per second. [This article](https://ripple.com/insights/ripple-continues-to-bring-internet-of-value-to-life-new-features-increase-transaction-throughput-to-same-level-as-visa/) has some more information about this.

These are some impressive numbers - effectively they can match VISA's throughput - and has definitely brought up Ripple into my radar once again.

Ripple also offers another product, [xRapid](https://ripple.com/solutions/source-liquidity/), aimed at enabling realtime payments and on-demand liquidity by using XRP. Dilip offered Cuallix as a case study for this product. [Cuallix is using xRapid to reduce the cost of sending cross-border payments from the US to Mexico](https://ripple.com/insights/ripple-continues-to-bring-internet-of-value-to-life-new-features-increase-transaction-throughput-to-same-level-as-visa/). 

P.S.: Just as I was writing this section I came across [another article](https://www.businessinsider.com.au/ripple-xrp-western-union-payments-testing-2018-2) indicating Western Union is doing experiments with Ripple as well.

## Blockchain deconstructed : contracts vs smart contracts - Fritz Henglein (University of Copenhagen)

This talk focused on the technical details of smart contracts as currently implemented in platforms such as Ethereum. In particular the fact that the contract rules and its execution are intertwined.

Fritz began his talk with a summary slide on smart contracts which allowed him to expand on the above:

{% img center /assets/images/posts/crash-smart-contracts.jpg 250 333 Crash course on Smart Contracts terminology %}

Due to this mixing of rules (contract checking) and execution (contract actions) in the source code - as in Ethereum smart contracts - there is no possibility of having multiple strategies which may be private and can be reused in different smart contracts.

Fritz argues that contract actions - which he called strategies - should be stored separately to the contract rules. In doing so such strategies can be private and re-used in different contracts. He calls this model Managed Contracts which combine the contracts with join execution strategies:

{% img center /assets/images/posts/managed-contracts.png 250 333 Managed Contracts %}

He then moves on to discuss the trade-offs and vulnerabilities of Ethereum smart contracts. I particular he raises these issues:

*   Transaction order dependency
    *   Messages may have different effect depending on the order in which they are received
*   Smart contracts may behave differently depending on the timestamp of a block
    *   The timestamp of a block is controlled by miners. He raises that this exposes the contracts to clock manipulation attacks. Frankly I'm not sure if this attack can succeed if a single miner is the bad actor. I'd need to think through this but if someone has an example handy, that'd be great.
*   Exception handling and programming language subtleties:
    *   Fragile gas management and limited stack 
    *   Differences in language constructs such as send vs call
*   Reentrancy bugs

Fritz's proposal to deal with this are Managed Contracts and he refers to a paper - which he co-authored - called Compositional Formal Contracts. The paper proposes the following properties:

*   Separation of concerns
*   Domain-oriented code
*   Analysable
*   Composable

I couldn't find a paper with that exact title but did find two papers co-authored by him with a similar title. [Here](http://www.diku.dk/~simonsen/papers/j6.pdf) and [here](http://www.diku.dk/~simonsen/papers/c2.pdf). If you know which one he was referring to, please let me know.  

## Rightsfusion Pty Ltd - Solara.io - [Leon Gerard Vandenberg](http://bit.ly/fuzo-weal) | [slides](https://www.dropbox.com/s/2mgqobppc3n4etl/UNSW%20Blockchain%20Symposium%20Briefing%202018-02-12.pdf)

[Solara](http://www.solara.io/) has an interesting premise.  In the energy space, consumers and traders have difficulty determining & validating green, renewable energy from fossil fuel energy. Solara allows communities and prosumers (producer / consumers) to factionalize or leverage new ownership models and redistribute and/or monetise their energy & their energy data.

In addition, SOLARA Platform aims to provide a proveably green & clean digital solar asset to a variety or energy data exchanges & markets. The concept of a Tokenised 'Safe Haven Asset Class' based on a hybrid portfolio of Solara PAT tokens that  could be synthesised by quants to eventually provide a stable coin was contemplated by Leon. (refer to Solara White Paper for details on Project Asset Tokens - PAT tokens)

Another interesting aspect is that Solara is looking to provide "fit for purpose hardware" Solara Hardware Modules to bridge the needs of both the metering and the blockchain worlds - think IoT eSIM cards which are also a secure blockchain keystore (wallet).

Leon & team is currently working towards an ICO (private pre-sale for their SOL Token is now on-going) and looking to build out more team members (including University Labs) and RightsFusion will engage in projects for Solar Communities and solar industry participants.


Lastly, Leon mentioned [Polymath](https://www.polymath.network/) in the context of ICO investments: Polymath (Canada) is working on a securities token or their ST standard. Because Solara's PAT Tokens are a type of Financial Product - PAT Tokens could embrace the Polymath ST standard to extend their eventual reach and a broader participation model.

> Note: I also got some [extra resources](https://showcase.dropbox.com/s/RightsFusion-Pty-Ltd-Public-Documents-Public-Presentations-Events-5jLk5XfBukYVeUsZm0CAB) about SOLARA straight from Leon.

## The power of possibilities - Niki Ariyasinghe - R3/Corda

Niki gives an overview of [Corda](https://www.corda.net/), an open-source blockchain project designed for building financial services infrastructure.

The project seems to have come about from the need for banks to collaborate and understand what blockchain means for financial services.

Corda displays a couple of intresting properties:

*   It allows for transaction privacy
*   It has a pluggable consensus mechanism allowing consensus algorithms to be chosen at transaction time and not at the blockchain level
    *   I don't know enough about corda but this particular point makes me nervous :)

The highlight of this talk for me is [Project Ubin](http://www.mas.gov.sg/Singapore-Financial-Centre/Smart-Financial-Centre/Project-Ubin.aspx): a project by the Monetary Authority of Singapore to provide Central Bank backed Digital cash with the goal to use Distributed Ledger Technology (DLT) to clear and settle payments and securities.

## Platform cooperatives - Browen Morgan from UNSW

As per Wikipedia: _"A platform cooperative, or platform co-op, is a cooperatively-owned, democratically-governed business that uses a protocol, website or mobile app to facilitate the sale of goods and services"_

I didn't take many notes during this talk but captured a few interesting links:

*   [CULedger](http://culedger.com/): A permissioned, private blockchain
*   [Loomio](https://www.loomio.org/): Collaborative decision-making software
*   [Provenance](https://www.provenance.org/): Building trust in goods and supply chain
    *   It's worth noting that the issue of provenance has been mentioned quite a few times during the Symposium

I particularly found ingenious the idea behind the following projects:

*   [Arcade.city](https://arcade.city/): Think peer-to-peer Uber
*   [Swarm.city](https://swarm.city/): A decentralised commerce platform 

## Issues for law and the legal profession - Lyria Bennet Moses - UNSW 

The last one for the day, this talk is the one I was looking forward to the most due to my work at [MODRON](http://www.modron.com/). It explores concerns and legal challenges with using blockchain technology.

According to Lyria - and it's certainly true in non-blockchain tech circles as well such as AI and Machine Learning - people ask if blockchain and smart contracts will replace lawyers. 

Her short answer is no. However certain roles might be replaced. It is already happening in domains such as exchange of documents. Basically she thinks that _"the things that junior lawyers do"_ will be replaced by (blockchain)technology.

In addition she offers certain applications where blockchain might be particularly well suited to disrupt such as Registries and Intellectual Property. In contrast, the problem of Information Sharing in law enforcement was mentioned as one needing a cultural solution, not a technological one.

Additional challenges with smart contracts:

*   Liability for delays and/or errors
    *   e.g.: wrong transaction recorded, delays affecting price points...
*   Form of order in, say, land law
*   Jurisdiction - in which jurisdiction are you?
*   Data Protection:
    *   The right to change data
    *   The right to be forgotten
    *   EU law seems to require these properties
*   Regulation
    *   Similar issues to when digital signatures started being used for contracts

All the above will also have implications to how lawyers are trained. I love that Lyria mentioned this as I've been casting what we do at [MODRON](http://www.modron.com/) under a similar light: our human-centric approach looks to enhance what lawyers can do and over time this will invariably lead to new types of legal professionals.

Lyria's closing remarks raised the question of wether blockchain even is the thing which will solve these problems. It's too early to tell but it's encouraging to see the legal profession as a whole taking the opportunity to re-evaluate how things are done.

What a day! So much to think about and so many great people to connect with. Look out for my [summary of day 2](/2018/02/16/highlights-of-the-symposium-on-blockchain-and-distributed-ledger-technology-day-2/)!
