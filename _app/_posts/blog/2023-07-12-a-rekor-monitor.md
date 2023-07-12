---
layout: post
title: Monitoring and analytics for Rekor
desc: I built a small Rekor monitoring system on top of Grafana
tags: sscs
---

Over the past weeks and months the idea of keeping tabs on the contents of the transparency log kept spooking around in my mind.
After diving into Go and a few attempts that failed due to over-engineering,
I decided to built a monitoring system on top of Grafana.
The data is fed into a PostgreSQL database after having been crawled and pre-processed by Rekor.

This link gives you a first look at a public analytics dashboard of Rekor:
[https://rekor-monitor.flxw.de/public-dashboards/5a433a2a9b0a42859b884f1ec4e8862c]()

I will expand this post in the future with architecture diagrams, monitoring mechanisms and design decisions.

