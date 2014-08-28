---
layout: post
published: true
title: Simple laptop backup system
description: A quick post on part of my backup system.
---

Just a quick post to help me get back in the habit of writing. This is on my laptop backup process; I frequently need to travel, and I feel better knowing that all my half-comitted git repositories are safe.

I am using the [horcrux](http://chrispoole.com/project/horcrux/) tool to run my backups. This makes the backup as simple as

    sudo mount /mnt/usb_hdd
    horcrux auto usb_hdd

In addition to this, I have a daily cron job that checks the date of the horcrux log and emails me if it is older than 5 days. In this way I get reminded to update my backup every 5 days.