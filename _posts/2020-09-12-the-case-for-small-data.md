---
date: 2020-09-12
title: The case for small data
categories:
author_staff_member: dave
---

In [Data practices for scrappy startups](https://www.tabbydata.com/2020/09/10/data-practices-for-scrappy-startups/) I outlined an efficient system for small startups wanting to amass their data in one place. However, that article does little to address the question of _why_ small startups would want to do this. In fact, it only devotes a few sentences to that question:

> [...] small amounts of user/customer activity can still result in highly valuable data. How are people finding your new feature? Do free users engage with it as much as paid users? What can you learn from looking at cohort behavior before/after a pricing change?

This article will attempt to make a case for small startups amassing their data in one place, and specifically implementing technology like a "data warehouse" or a "data lake" early on, even if the volume of data is initially small.

The argument can basically be broken into three pieces:

## 1. Getting all your data into one place enables new perspectives.

A friend shared [this Thinkgrowth piece](https://thinkgrowth.org/the-startup-founders-guide-to-analytics-1d2176f20ac1) recently, and I think it's worth reading. That article makes a strong case for "why startups need to collect data" but would encourage startup leaders to wait until the company has 20-50 employees before implementing data infrastructure. Prior to that point, startups should only "use built-in reporting capabilities of [their] SaaS products and track UTMs."

The problem with relying on built-in reporting capabilities of your third-party tools is that it results in a very fragmented view of the world. Your email marketing software might tell you about open rates, but how can you correlate that with how those same readers behave when they open your app? Fortunately, 2017 was a lifetime ago in technology terms, especially when it comes to data infrastructure. The current cost, in both time and money, of implementing a data warehouse for a 1-3 person startup is at least 10x less now than it was then, so there's no need to wait until your company reaches the 20 employee mark.

## 2. Making it easy to instrument your own systems and collect your own measurements early on is a "secret weapon."

Besides agility and tenacity, startups have few advantages in the marketplace. The ability to instrument and optimize every part of your business acts as a multiplier of agility; it allows you to be agile _in the right direction_. This is difficult to understand until you've seen it in practice—anyone with experience at companies like Google, Facebook, or Amazon have seen it firsthand, but until recently the same capability was not available to extremely early stage or bootstrapped startups, whereas now it is.

## 3. If you accept that you will _eventually_ want or need such a system, getting it into place early allows you to build a data-driven culture from the beginning and allows you to build data integrations as you go rather than trying to "catch up" later.

Old, established, brick and mortar companies have spent _billions_ implementing data practices as part of their so-called "digital transformations." The best way to avoid this fate is to be "digital" from the start. For the most part, this is the natural state of scrappy technology startups, but in the past decade there's been a huge gap in the level of data infrastructure available to tiny startups vs. their more established counterparts. Now that the playing field has started to level, tiny startups should embrace these capabilities early rather than finding themselves in a position where they need to "catch up" later.
