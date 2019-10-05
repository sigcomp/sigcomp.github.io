---
layout: page
title: About Us
menu: main
permalink: /about/
---

Mission Statement
---------------------------------

>ACM Comp places students in a competitive environment to encourage
self-improvement and provide a benchmark by which they can measure
their growth. Members will build up algorithmic problem-solving skills
and learn to communicate their solutions in a supportive environment.
Our focus on practical application of the Computer Science curriculum
gives students the confidence to succeed in research and industry.

**SIG-Comp welcomes students of _all_ majors and skill-levels with open arms.**

### Interested?

- Join our [Discord channel](<https://discord.gg/4t954Ad>).

- Join the [E-mail list](<https://groups.google.com/a/mst.edu/forum/#!forum/acm-comp-grp/join>) (Make sure that you are signed into your student account).

{{ site.semester }} Leadership
------------------------------

Chair: {{ site.data.contacts.chair.name }}

Advisor: {{ site.data.contacts.advisor.name }}

Our Committees
--------------

#### Event Committee: 

- **Lead** - {{ site.data.contacts.eventmaster.name }}
{% for person in site.data.contacts.eventcommittee %}
- {{ person }}
{% endfor %}

#### Web Committee: 

- **Lead** - {{ site.data.contacts.webmaster.name }}
{% for person in site.data.contacts.webcommittee %}
- {{ person }}
{% endfor %}
