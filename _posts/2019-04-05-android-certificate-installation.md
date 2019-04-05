---
layout: post
title: "AppSec Basics: Your First Pentest"
author:
modified:
tags: [Development,Pentesting,Business]
---

So, another year has come and gone and you still have that feeling. That little voice inside that says, "I wonder how good our cyber security is..." Is that super critical application just sitting out there on the internet scared and alone? Maybe now is finally the time to look into it, but where to start? 

### What kind of test should I use?

Typically, when you hear someone explaining pentesting they divide the testing process into a few groups: black box, white box, gray box, maybe even crystal box, but that is not what I am going to get into here. If you are unfamiliar with those terms, stop now, go read about them a little, and come back to me. I'll wait.

Now, we are going to delve into another breakdown, specifically with regard to application security assessments - static, dynamic, and hybrid. 

Static Code Analysis (SCA) is an analysis of the application source code performed in an offline manner. There are a few different ways to handle SCA, and there are many products that will aid in finding bugs. All of these work in the same way; they look at the text of the source code and the flow of the code paths to determine if any errors, oversights, or misconfigurations will negatively effect the application at runtime. Unfortunately, SCA is prone to false positive findings, as the analysis tool cannot know what data will be in a variable or how other components may alter the code flow. The cleanup process requires human intervention and can be very labor intensive.  Basically, somebody has to make the call whether that log entry really will contain Personally Identifiable Information (PII) or that output string really is vulnerable to Cross-Site Scripting (XSS).

The opposite of static testing, naturally, is dynamic testing. This is where a pentester sits down at the live, running application and tries to break it. Utilizing real servers and live data in the database, the dynamic test will look at every facet of the application as it presents itself to normal users. This is where we find live vulnerabilities like privilege escalation, authorization bypass, and brute-force weakness.

Hybrid is my favorite type of testing, as it really is the best of both worlds for application security. When I work on hybrid tests, I typically have the source code open at the same time as the application, and they feed off of each other.  A hint of a bug in dynamic testing can be verified and refined through examination of the corresponding code. A potential finding from SCA can be tested dynamically to validate whether it really is a vulnerability or not and how difficult it is to exploit. 

Hybrid testing gives the best possible answer to the question that everybody really wants to know -- "What is the risk?" By crafting a proof-of-concept exploit for a given vulnerability, it is easier to determine how much custom code, time, and effort goes into exploitation. The dynamic testing will tell me exactly what an attacker is able to accomplish by performing this attack, and the source code shows me exactly where the vulnerability can be remediated.

### Great, when do we start?

In most modern development cycles there is never really a good time to just stop everything and let the security people poke around doing whatever it is they do. Thankfully, there are ways to perform the assessments out of cycle with little to no interruption to the dev team. With the proper preparation, you will never know we were there (until you get the report).

### How do I get ready?

So here's the thing. You do not want to have to pay me to sit around and wait on administrative processes. I would like to avoid that as well; I want to start immediately providing value to you and making your work life just a little bit better. Check off all of these items ahead of time and we will all be much happier:


**Administrative**

* **Technical Points of Contact**
    If the tester has questions, or something bad happens, we may want to get someone on the phone quickly.  Designate a specific person to be ready to respond to emails and calls.  You don't want to find out on Wednesday that the tester has not been able to log in to the application because the dev password was changed.
* **Application's Purpose**
    This helps to clearly define the risk on any vulnerabilities that may be found. Regulatory compliance requires different protection levels for data such as PII/PHI and financial transaction data. You should be able to talk about what is important to your organization; this will aid the tester in giving a valid report.

**Source Code**

* **Lines of Code**
    Get a rough count of the Lines of Code (LoC) for each language present in your application. It doesn't have to be exact, but it helps in scoping when we know ahead of time if we are dealing with 2 million lines of Java or 20 thousand lines of Python. If this is a mobile app, figure out the number of dynamic screens in use. For an API, the number of method calls. All of this data helps to ensure we cover everything thoroughly.
* **Source Files**
    Package up everything. All of it. Yes, that too. I should be able to unzip and compile it using the same IDE that you use. The SCA process we talked about earlier is so much more thorough if the code compiles without errors. A large project could take hours to scan, so multiply that by the number of times it needs to be redone.
* **Frameworks**
    Make a list of all the frameworks, databases, left-pads, or anything else you use to make the magic happen.
* **Environmental**
    Write up instructions with the compiler configuration, execution flags, environment variables, or any other process that would be helpful. This may not be necessary, but sometimes we stand up our own version of your application. For example, if we find a potential denial-of-service condition it can be safer to not test it in your DEV enironment.

**Test Environment**

* **Make it Work**
    I hope it goes without saying that dynamic testing needs a working environment, but that is also a bit of an oversimplification. The environment needs to be as close to PROD as possible, within reason. For the purpose of this post we are just talking software; you can use any old server you scrounged up to run it. Personally, I think DEV machines should be slow but we can leave that for a happy hour rant. Little things can make a big difference, like logging level, server configuration, and encryption. If you are doing this for compliance reasons, try to avoid having to explain to an auditor, "The test was in DEV but PROD is not like that and I swear we are good...".
* **Integrations**
    Every now and then I run into an application that is missing some key functionality in DEV. This is your choice of course, but testing 75% of your business critical finance application is disingenuous at best. At worst, you miss a critical vulnerability that costs you a loss of all your sensitive data. It is completely understandable that some applications have features that are just not possible to test except in PROD. In this case, build a test harness that can fake it. Other integrations like SSO or OAuth can really ruin my day if nobody thinks of them until after the test starts.
* **Users and Roles**
    If the application requires pre-registered user accounts, create one for every role. One of the biggest sources of weakness we see is a mismatch in authorization expectation versus reality. Both horizontal (e.g.: user-to-user) and vertical (e.g.: user-to-admin) privilege escalation is a potential problem that requires the full suite of user roles to test. If there is a separate administrative section of the application, make sure we at least look at it so that we can try to get there as a regular user.
* **Data**
    Use test data; no exceptions. Will we need any specific data to make the forms work? I am not going to test using my own SSN or PII, so provide a test. If you have sections that do a background info check for user validation, we need to know what they are. Answers to pre-configured security questions? Bank account information? We need all of it.

**Access**

* **Web App**
    Plan ahead of time for how the tester is going to get to your application. If it is already exposed to the internet, no need to worry. Internal networks, VPN configs, and 2FA tokens can take days to get working. Internal name resolution can really make for some fun troubleshooting when the tester is not using the internal DNS server. On-site and after-hours testing are available if necessary but, as you can imagine, travel can complicate the scheduling, and it increases the cost.
* **Conflicts**
    Most of the time our testing is non-destructive, but every now and then bad stuff happens. I myself have changed an application from English to French and could not figure out how to change it back. I've also accidentally removed all the modules from a customizable page. Both times, it would have been really inconvenient if someone else was relying on testing at the same time. Schedule the assessment during a time when nobody else needs to use that environment, when no new deployments are planned, and in an environment that can withstand an uh-oh moment. 
* **Certificates**
    Any application that requires client certificates needs extra time for setup and troubleshooting. Mutual authentication with TLS certificates is a great way to protect your webservice from unauthorized access... and also from being tested. Verify a few weeks prior to the test if the app requires a public client cert, or if self-signed is acceptable. Check the expected format so we do not end up playing "Guess the Encoding" the night before the test.
* **Hosting**
    Third-party hosting providers frown upon unexpected attack floods. Be cognizant of not only the hosting provider but also any integration with partners so that the testing remains in the scope of properties owned. We can typically get permission to test with third-party hosting or cloud providers, but this must be requested well in advance to ensure adherence with any processes on their end.
* **Blockers**
    Do you want me to test your application or your WAF? Sure, it can help stop some attacks, but that is not the point of this assessment. We want to highlight the underlying weaknesses, not camoflauge them. I may not be able to get past your WAF / load-balancer / firewall stack in a week, but what if an attacker has unlimited time? What if a vulnerability is announced a year, a month, or a day after we hand you a report?

**Special Considerations**

* **Previous Tests**
    We see a lot of applications that require annual testing for regulatory compliance. Looking at any previous tests AFTER we do our own work can help to verify that any open findings have been taken care of and that nothing was missed.
* **Delicate Flowers**
    Last, but certainly not least, talk to the admins, developers, and owners to find out of there is anything that needs special care or a sensitive touch. I am here for you; hiding anything only helps those unpaid pentesters when the application goes live. If that one system is too fragile to withstand a full test, talk to me. If I can knock it over, would you rather find out during a maintenance window or during the middle of a business day when someone else does?

### Conclusion

If you do not watch the news, allow me to summarize. Criminals are hacking everything that is connected to the internet, and some things that are not. While there are indeed people that prefer to have an adversarial "Red Team" type of test, that is not always the best path. If you got this far in my post, you probably already know that is not what you are looking for. Working closely with a testing provider and fully preparing beforehand will yield far better results. 

If your organization is new to cyber security, do not worry about perfect scores or who to blame. The past is gone, make a plan for the future that will help to keep your organization off of the list of public security breaches.

Pick something and get started. 