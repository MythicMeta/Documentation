# Reports

## What is it?

This is your standard PDF / JSON report generation. There will soon be options to generate other document types and customize the report.

## Where is it?

Report generation is located in "Reporting" -> "Generate Report" from the top navigation bar.

## How to use it?

Right now there are only a few different methods for generating a PDF output. By default, it's just the new callbacks and tasks instances. You can toggle to include command output as well and then you get the additional choice of how to organize the output: strictly adhering to time, or grouped based on the task that issued it (similar to how you see it in the Mythic UI).

{% hint style="warning" %}
When opting to include all command output in your PDF, you could end up with VERY large PDFs that become much harder to read.
{% endhint %}

![Accessing report](<../.gitbook/assets/Screen Shot 2019-07-28 at 8.25.34 PM.png>)

The report can be downloaded by clicking the blue link at the top when you create it. After the fact, the report can always be downloaded from the Services -> Host Files page.

### Reporting Types

There are currently two ways you can get all of the information. You can either generate a PDF (which, if you include all of the command output, can be an insanely massive file that might peg your CPU), or you can generate the output as a JSON file.

For the PDF version, the output is color coded to help with things -

* pink - New callback
* white - New tasking
* blue - New output
