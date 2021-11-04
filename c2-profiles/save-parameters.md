# Save Parameters

## What is it?

An operator can provide non-default, but specific values for all of fields of a C2 profile and save it off as an instance. These instances can then be used to auto-populate all of the C2 profile's values when creating a payload so that you don't have to manually type them each time.

## Why have it?

This is a nice time saver when creating multiple payloads throughout an operation. It's likely that in an operation you will have multiple different domains, domain fronts, and other external infrastructure. It's more convenient and less error prone to provide the specifics for that information once and save it off than requiring operators to type in that information each time when creating payloads.

## Where is it?

The `Save Parameters` button is located next to each C2 profile by going to "Global Configurations" -> "C2 Profiles " from the top navigation bar.&#x20;

### How to create an instance

To create a new named instance, select the `Save Instance` button to the right of a c2 profile and fill out any parameters you want to change. The name must be unique at the top though.
