---
title: "Wordpress - Move cron jobs to CLI"
date: 2021-06-20T20:04:11+02:00
draft: false
categories: 
  - Wordpress
tags:
  - wordpress
  - devops
---

Some time ago we started to observe performance issues in our company’s site built with WordPress. One of the issues we have observed was calling crons in random user requests. It is realized by calling URL `https://yourdomain.com/wp-cron.php`. Although in this file WordPress calls PHP method `fastcgi_finish_request`, which sends a response to the user as soon as possible, the cost of calling sub-requests during the main request is worth acknowledging and improving. There are other drawbacks of this:
* Calling crons in request context means that it is limited, e.g. by php-fpm setting. We often have limited resources (such as memory or time limit) for HTTP requests. 
* What’s worse, calling cron as http request utilizes resource (e.g. php-fpm pools that might be necessary to handle our traffic). 

On the Internet, there are a lot of tips on how to disable making subrequest to call crons and move them to CLI. But there is one problem. A lot of these tips suggest calling these scheduled tasks using `crontab` by calling curl (or other http callings). This addressed first problem and makes user request free of sub-requests, but does not resolve two abovementioned problems. It is surprising because WordPress comes with `wp cron` command, which supports running scheduled tasks in CLI without making HTTP calls. 

In order to do this, you have to add one line to your crontab: 

```shell
*/5 * * * * wp cron event run --due-now --path=path_to_your_wp
```

In this case, we call the scheduled task every 5 minutes. WordPress is responsible for choosing a concrete task to run and runs it. 

Of course, you also have to disable calling sub-request to `/wp-cron.php` file. In order to do that, add the following to your wp-config.php file:

```php
define('DISABLE_WP_CRON', true);
```

In my opinion, you should also deny access to `wp-cron.php` file (because some bots might DDoS our site requesting this endpoint). I suggest making it on the application server level. For example, if you use Nginx,  add the following to your host configuration:

```nginx
location = /wp/wp-cron.php { deny all; }
```

