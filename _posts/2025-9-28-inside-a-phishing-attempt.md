---
title: "Inside a Phishing Attempt: Technical Breakdown of a Spoofed Email"
description: "Someone tried to phish me using a spoofed  email!"
layout: post
author: "0xRar"
tags:
- CTI
- DFIR
- Phishing
---

I opened my public email a few days ago, and i found an interesting mail supposedly 
"coming from" `support@nexcess.net`, but protonmail actually flagged it for
being spoofed which was good.

A little introduction: nexcess is a hosting service provider based in the USA,
they provide services such as VPS Hosting, Cloud, Dedicated server hosting and more.

<b>What is email spoofing?</b> , email spoofing is a common tactic used by threat actors
where they send an email with a forged sender to make it look like it came from a trusted
sender, this is possible because SMTP(`Simple Mail Transfer Protocol`) does not have a 
feature that authenticates email addresses.

<img src="https://cdn.jsdelivr.net/gh/0xRar/blog@main/assets/blog-posts-content/inside-a-phishing-attempt/email-img.png" />

## Technical Details

Lets get more into the technical details, the threat actors didn't really do enough research
because they didn't seem to realize that i don't actually use nexcess as a hosting provider 😁
which was a bit funny rather almost my entire website runs on github infra via their github 
pages.

<b>How to really know if the email is spoofed?</b>, the answer is view the headers of the email
instead of only looking at the `from:` or sender.

### Email Headers

```eml
Return-Path: <postmaster@86e835319c.nxcli.io>
X-Original-To: rarsec@proton.me
Delivered-To: rarsec@proton.me
Authentication-Results: mail.protonmail.ch; dkim=fail (Bad 0 bit
    rsa-sha256 signature.) header.d=86e835319c.nxcli.io header.a=rsa-sha256
Authentication-Results: mail.protonmail.ch; dmarc=fail (p=none dis=none)
 header.from=nexcess.net
Authentication-Results: mail.protonmail.ch; spf=none smtp.mailfrom=86e835319c.nxcli.io
Authentication-Results: mail.protonmail.ch; arc=none smtp.remote-ip=209.87.149.245
Authentication-Results: mail.protonmail.ch; dkim=fail reason="key not found in DNS"
 (0-bit key) header.d=86e835319c.nxcli.io header.i=@86e835319c.nxcli.io
 header.b="XFVjBFda"
Received: from cloudhost-3927890.us-midwest-1.nxcli.net
 (cloudhost-3927890.us-midwest-1.nxcli.net [209.87.149.245])
```

Lets focus on this: `dmarc=fail (p=none dis=none) header.from=nexcess.net`

DMARC(Domain-based Message Authentication, Reporting, and Conformance) fail means the email
did not pass SPF (Sender Policy Framework) or DKIM (DomainKeys Identified Mail), and since
the DMARC policy was set to `p=none` protonmail will not block or quarantine which is the
reason it made it through to my inbox and not junk/spam folder.

<img src="https://cdn.jsdelivr.net/gh/0xRar/blog@main/assets/blog-posts-content/inside-a-phishing-attempt/DMARC-Infographic.png" />

As you can see here the `smtp.mailfrom=86e835319c.nxcli.io` is pointed to a subdomain under
nxcli.io which is not controlled by nexcess its actually controlled by the threat actor
who is a nexcess user trying to phish other nexcess users, every user has a Temporary 
CNAME-Subdomain as explained on nexcess's official help page: [Nexcess: New Temporary Domain on All Plans].

Unfortunatly i was too late the cname was already gone so i couldn't really check what ports
were open on `86e835319c.nxcli.io` but its safe to assume that port 25/tcp, 587/tcp were open
which is what smtp works on.

[Nexcess: New Temporary Domain on All Plans]: https://www.nexcess.net/help/nexcess-new-temporary-domain-on-cloud-plans/

### Email Content

The email content is as basic as it may sound, just some text to scare you enough to click
the button and open the url.

<img src="https://cdn.jsdelivr.net/gh/0xRar/blog@main/assets/blog-posts-content/inside-a-phishing-attempt/email-content.png" />

Would've really loved analyzing it further but its redirecting me to a page on an iranian website 
thats returning 404 Not Found Status Code.

<img src="https://cdn.jsdelivr.net/gh/0xRar/blog@main/assets/blog-posts-content/inside-a-phishing-attempt/wheregoes.png" />

## Prevention
- To prevent email spoofing you would need to publish SPF, enable DKIM signing,
and enforce DMARC with quarantine/reject.

Or Simply enough lets take google's dmarc record as an example:
```eml
v=DMARC1; p=reject; rua=mailto:mailauth-reports@google.com;
```

## Thank you for reading !
Please don't hesitate to provide feedback, See you space cowboy.

