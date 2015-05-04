---
layout: post
title: Telekom bug bounty part 1 (Remote Code Execution)
comments: true
tags: bugbounty CVE-2013-2251
---

During easter 2015, we decided to try our luck on the [bug bounty program of the German Telekom](https://www.telekom.com/bug-bounty-program). This program is special as
Telekom only pays bounties for bugs with a high security impact. In fact, the program details limits this to SQL Injection
and Remote Code Execution (RCE) vulnerabilities in web based portals. To bad, however it also limits the stuff that we have
to look for.

In contrast to most other companies that offer bug bounties, the German Telekom is quite big (228 000 Employees) and exists since 1996. If you ever worked in a big company you know that most of them establish new web based portals/applications all the
time. So with some luck it might be possible to find a old application with some exploitable vulnerabilities.

Long store short, we discovered some interesting bugs and reported them to the Telekom Cert. While we did not receive money
for all vulnerabilities it was still a interesting experience.


Exploiting CVE-2013-2251
------------------------

The first vulnerability that we exploited was CVE-2013-2251, a well known RCE vulnerability in the JEE framework "Struts 2". Struts 2 based applications use the file type ".action", so we just googled for that:

![Searching for Struts 2 Applications](/assets/images/telekom_bug_bounty_google.png)

As you can see, this resulted in only one system/application. In a next step we used various command names to get more information about the system and the JEE application. This led to following error message which revealed the used Tomcat version:

![Tomcat Version](/assets/images/telekom_bug_bounty_tomcat.png)



The used Tomcat version is quite old but does not have a easy exploitable vulnerability. The version information is still important as it allows you to make some guesses about the system. Tomcat 6.0.20 was released on 3 Jun 2009. If the administrators didn't update the application server since then, they probably also didn't update the framework used by the application, even if it contains a critical issue.


In the last years, the Struts2 framework had several serious remote code execution vulnerabilities. We decided
to try CVE-2013-2251 first, for following reasons:

* It is well documented, various PoC examples are available
* Exploitation can be done via GET requests, this simplifies the creation of PoC URLs (we are lazy)

We used the following PoC to verify that the application is indeed vulnerable:

https://hosting-guestlogin.telekom.de/sessionmanagement/u4/loginExists.action?redirect:%24{new%20%20%20java.lang.String(%20%27rce-poc%27%20%20)

The application tried to redirect us to "rce-poc", which means that the issue can be exploited.

We created a additional check that called the OS command ping to ping an external host, which we were able to verify via tpcdump:

![Pinging our own host](/assets/images/telekom_bug_bounty_icmp.png)

Telekom response
----------------
The issue was reported to the Telekom Cert team which confirmed it immediately. Unfortunately we didn't receive any money for that bug as
it is based on a third party component. We expected that when we read the disclosure policy.
