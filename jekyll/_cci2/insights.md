---
layout: classic-docs
title: "Using Insights"
short-title: "Using Insights"
description: "Viewing the status of repos and test performance"
order: 41
version:
- Server v2.x
- Cloud
---

## Overview


The CircleCI Insights feature provides a dashboard for viewing the health and
usage of your repositories build processes. _Insights_ provides time-series data
overviews of credit usage, success rates, pipeline duration, and other pertinent
information.

This document describes how to access the Insights feature on CircleCI Cloud and Server.

## Usage (CircleCI Cloud)

To access a project's insights, view a pipeline's workflow and click the
 **Insights** button. Alternatively, you may access the Insights page by
 clicking on the **actions** button while viewing the _pipelines dashboard_.

{:.tab.insight-access.Access_by_pipeline}
![]({{ site.baseurl }}/assets/img/docs/screen_insights_access-1.png)

{:.tab.insight-access.Access_by_workflow}
![]({{ site.baseurl }}/assets/img/docs/screen_insights_access-2.png)


### Workflow Overview

The insights page provides workflow details plotted over the last 90 days (with
custom date ranges coming soon). You may also filter by different workflows at
the top of the page. The following data is charted under the workflow overview:

- All workflow runs
- Workflow success rate
- Workflow duration
- Workflow credit usage

### Job Overview

Switch to the **Job** tab to view cumulative time-series data on a per-job basis:

- Total credits used
- Duration (the 95th percentile)
- Total runs
- Success rate

---

## CircleCI Server Insights

<div class="alert alert-warning" role="alert">
  <p><span style="font-size: 115%; font-weight: bold;">⚠️ Heads up!</span></p>
  <span> The following section refers to using the Insights page on the CircleCI <i>Server</i> product. </span>
</div>

### Overview

Click the Insights menu item in the CircleCI app to view a dashboard showing the health of all repositories you are following. Median build time, median queue time, last build time, success rate, and parallelism appear for your default branch. **Note:** If you have configured Workflows, graphs display all of the jobs that are being executed for your default branch.

![header]({{ site.baseurl }}/assets/img/docs/insights-1.0.gif)

The image illustrates the following data about your builds:

- Status of all your repos building on CircleCI in real time
- Median queue time
- Median build time
- Number of branches
- Last build

### Project Insights

Click the Insights icon on the main navigation, then click your repo name to access per-project insights.

The per-project insights page gives you access to the build status and build performance graphs for a selected branch.

![header]({{ site.baseurl }}/assets/img/docs/insights-current-build.png)

- **Build Status:** The Insights dashboard shows the last 50 builds for your default branch. Click a branch in the top right corner to access over 100 build/job statuses for the selected branch.

- **Build Performance:** The Build Performance graph aggregates your build/job data for a particular day and plots the median for that day going back as far as 90 days. Monitor the performance of your repo by clicking a particular branch.

### See Also

Refer to the [Collecting Test Metadata]({{ site.baseurl }}/2.0/collect-test-data/) document for instructions to configure insights into your most failed tests.

