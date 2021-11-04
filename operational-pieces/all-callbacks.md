# All Callbacks

## What is it?

As operations extend into weeks and months, the number of callbacks and tasks will start to become unmanageable normally. The goal is to have a cleaner view of all the callbacks that are currently active and inactive across an operation. When you remove a callback from the [Active Callbacks](active-callbacks.md) page, they're still available for viewing here.

## Where is it?

To access this view, go to "Operational Views" -> "All Callbacks" from the top navigational bar.

![](<../.gitbook/assets/Screen Shot 2020-02-17 at 10.35.08 PM.png>)

## How to use it?

The interface will by default show the latest 15 callbacks created in an operation. This includes both callbacks that are currently active in the "Operational Views" -> "Active Callbacks" view as well as those that have been removed from there. This is indicated by the coloring and presence of an additional button here. To move a callback back to the active callbacks page, simply click the "Make Callback Active" button.

By default, no tasks are loaded for the callbacks, but the `Load Tasks` button will load all of the current tasks. From there, clicking on an individual task will load its responses and run it through any potential browser scripts as well. Once you've loaded tasks, you can click `Hide Tasks` to hide them again:

Currently, the only filtering capability exists via the hostname for the callback. To apply this, simply type the hostname (or part of the hostname) as regex into the top right search bar and click the filter icon. From there, a new request for the first 15 matches will be returned. Clicking new pages or changing the page size will still apply with the value in the search bar until it's removed.

![Filter for just callbacks where the hostname includes "spoo"](<../.gitbook/assets/Screen Shot 2019-07-27 at 7.34.17 PM.png>)
