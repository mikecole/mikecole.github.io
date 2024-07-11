---
layout: post
title: Warming Up An Azure App Service Slot Before Swapping
date: 2024-07-10
tags: azure dev-ops github-action
meta-description: When doing an Azure App Service slot swap deployment, it's important to warm up the staging slot before swapping for more reliable behavior.
---

Azure App Service deployments using slot swapping is an easy way to do zero-downtime deployments. The basic workflow is to deploy your app to the staging slot in Azure, then swap the slot with production. Azure magic ensure that the application will
cut over to the new version without a disruption as long as you've architected your application in a way that supports this. When reading about the `az webapp deployment slot swap` command, one of the first steps is -supposed- to warm up the slot
automatically. In reality what we were finding was a lengthy operation (20+ minutes) followed by a failure with no helpful details. As part of the troubleshooting process I added a step to ping the staging slot after deployment, and I saw that it
resulted in random errors, mostly `503 Service Unavailable`. This was happening on several different systems. I ended up adding a few retries and saw that the site would eventually successfully return `200 OK` after a minute or so and then the slot swap
step would work reliably within 2 minutes. It seems that when your site does not immediately return `200 OK` when starting a slot swap, the command would freak out.

In our deployment GitHub Actions jobs, I added the following step to manually warm up the site. The `warmup_url` should point to a page on your *staging* slot. We have it pointing to a health checks endpoint so we can ensure stability before swapping. `--connect-timeout=30` will attempt to connect to the site for 30 seconds, then retry 4 times with a 30 second delay in between attempts. `-f` argument will tell your retry loop to not fail the step if it receives an http error. `--retry-all-errors` is an
aggressive form of retry which ensures a retry on pretty much everything that isn't a 200 level success code. The second `curl` statement instructs the step to fail, which will fail the entire job altogether.

```yaml
- name: Warmup
  if: inputs.warmup_url != ''
  run: |
    curl \
      --connect-timeout 30 \
      --retry 4 \
      --retry-delay 30 \
      -f \
      --retry-all-errors \
      ${{ inputs.warmup_url }}
    curl \
        --fail-with-body \
        ${{ inputs.warmup_url }}
```

After adding this to all of our slot swap deployments, we consistently saw quite a bit more reliable behavior, and no more frustrating 20 minute black hole failures.