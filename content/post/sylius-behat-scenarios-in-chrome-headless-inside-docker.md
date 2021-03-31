---
title: "Sylius Behat scenarios in Chrome Headless inside Docker"
date: 2021-03-29T22:02:11+01:00
draft: false
categories: 
  - Sylius
tags:
  - sylius
  - devops
  - docker
  - behat
---

Since this [PR](https://github.com/Sylius/Sylius/pull/11505), Sylius has started using Chrome Headless to run theirs Behat scenario tagged with `@javascript` (scenarios that require Javascript engine to run correctly - in short, Javascript suits). After this change, it turned out that Javascript suits speed up from 13 minutes to 8.5 minutes. So, I decided also to run my Javascript suit using Chrome Headless. But there was one problem. All my environment (development, test on the local machine, and test in CI) base on Docker. Until now, I've used Selenium image to run Chrome. This short post shows you how to configure your Docker environment to work with Chrome Headless.

{{< callout emoji="❓" text="You might see this [blog post](https://dev.to/kayneth/how-to-test-your-sylius-plugins-with-selenium-38ci) when you still want to use Selenium with Docker for Javascript suits." >}}

First of all, I suppose that you work with Sylius Standard Edition. If you have a custom Docker configuration, there is no problem with adopting this solution, but it will require more effort from you.
Sylius Standard comes with two docker-compose files: one for development purposes and the second for production. For running Behat scenarios, I suggest creating the third one. We might call them `docker-compose.behat.yaml`. In this file, we might configure Chrome Headless as a Docker service and (optionally) override or create new services.

For brevity, in this blog post, we only create a Chrome service and override the environment setting for the application service. So, our `docker-compose.behat.yaml` file looks like this:

```yaml
version: "3.4"

services:
  php:
    environment:
      - APP_ENV=test_cached
  chrome:
    image: zenika/alpine-chrome
    shm_size: "2gb"
    command: "--enable-automation --disable-background-networking --no-default-browser-check --no-first-run --disable-popup-blocking --disable-default-apps --disable-translate --disable-extensions --no-sandbox --enable-features=Metal --headless --remote-debugging-port=9222 --remote-debugging-address=0.0.0.0 --window-size=2880,1800 --proxy-server='direct://' --proxy-bypass-list='*'"
    volumes:
      - .:/srv/sylius:rw,cached
```

Few lines that require explanation:

1. First of all, you have to override the application service (in our case, called `php` to run in `test_cashed` environment).
2. For Chrome, we use `[zenika/alpine-chrome](https://github.com/Zenika/alpine-chrome)` image. You might use another one (but not all working correctly, this one is also well documented) or create your image (but it might be a waste of time).
3. It is necessary to change the shm size. Without this change, Chrome crashes during tests. So I suggest setting the `shm_size` option to `2gb`.
4. The next thing worth noting is the command for the Chrome container. I use the same command as is used in the [Sylius CI configuration](https://github.com/Sylius/Sylius/blob/master/.github/workflows/application.yml#L387).
5. Last but not least is setting the correct volume. It is very important, and remembering about it might save your time for debugging a problem. It is necessary to upload files in the scenario's steps. Without this line, your test execution returns timeout for all scenarios placed after that one that tries to upload a file. In my example, I share all codebases, but it is possible to share the only directory with fixtures.

The second file we should change is `behat.yaml`. We have to configure the MinkExtension (so we focus only on the `Behat\\MinkExtension` section in this file) base_url option.  In our case, the correct value is [`http://nginx/`](http://nginx/) (it points to the entry point of our application).
Because we run Chrome Headless as a service in a container (instead of the same machine or container as an application), there is also necessary to change the option of `chrome_headless` sessions. This option refers to url to Chrome API, and for us, it is `http://chrome:9222`.

That's all. It's time for testing. For this purpose, run the command below:

```bash
docker-compose -f docker-compose.yml -f docker-compose.behat.yml exec php vendor/bin/behat --colors --strict --no-interaction -vvv -f progress -f pretty --tags="@javascript && ~@todo && ~@cli"
```

I hope that this configuration allows you to speed up your test, especially in CI (I wish to show you a comparison of Selenium and Chrome Headless in the next blog post).