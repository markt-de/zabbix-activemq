#### Table of Contents

1. [Description](#description)
1. [Concept](#concept)
1. [Requirements](#requirements)
1. [Macro reference](#macro-reference)

## Description

A Zabbix template to monitor Apache ActiveMQ Artemis. Legacy ActiveMQ "Classic" is not supported by this template.

The template uses JMX and Javascript preprocessing, so no additional scripts are required.

The template includes the following:

* Discovery rules with 60+ item prototypes (which easily translates to 500+ items per node)
* Item prototypes cover all layers: brokers, clusters, addresses and queues

## Concept

This template was designed around a unique monitoring concept. In order
to understand the design decisions made, a quick summary is outlined below.

Design goals:

* Avoid false-alerts
* Avoid trigger flapping
* Reduce overall monitoring "noise"

To achieve these goals the following principles were applied:

* Trigger severity has a **distinct meaning**
  * `Warning` – Alert is only displayed in Zabbix GUI. No notifications will be send.
  * `High` – All previous, plus the alert is send via e-mail. (Typical 5x8 alert.)
  * `Disaster` – All previous, plus the alert is send via instant messaging or SMS. (Typical 7x24 alert.)
  * Note: Zabbix "Actions" need to be configured accordingly.

* Enforce 3-step escalation and delay alarms
  * After at least two failures a `Warning` trigger may fire.
  * If the problem persists for the time period specified by `{$ESCALATION_1_DELAY}`, then the `High` trigger fires.
  * If the problem persists for the time period specified by `{$ESCALATION_2_DELAY}`, then the `Disaster` trigger fires.
  * These triggers have dependencies defined, i.e. a `Warning` trigger is closed when the `High` trigger fires.

As a result, *every* alarm starts with a `Warning` trigger, which should
not send any notifications. Only if enough time has passed, then a `High`
or `Disaster` trigger will be activated and notifications will be send.

This simple approach ensures that load spikes or small hiccups do not
cause any alarms/notifications. Additionally triggers are not resolved
too soon (which would result in trigger flapping), but instead a certain
amount of time has to pass.

Of course, this delay can be customized by defining `{$ESCALATION_1_DELAY}`
and `{$ESCALATION_2_DELAY}` on a per-host level.

## Requirements

Basic requirements for this template:

* Zabbix Server version 4.4 or later
* ActiveMQ Artemis version 2.14 or later
* Legacy ActiveMQ "Classic" is NOT SUPPORTED
* JMX credentials must be provided by using the `{$JMX_USERNAME}` and `{$JMX_PASSWORD}` (host) macros
* JMX interface must be configured with host/port information

Besides that the 3-step escalation requires that you define the following
global macros:

* `{$ESCALATION_1_DELAY}`, i.e. with a value of `600`
* `{$ESCALATION_2_DELAY}`, i.e. with a value of `1200`

If you want to completely disable the escalation delay, then you may
set these values to `0` (NOT RECOMMENDED).

## Macro reference

The following macros are being used in this template:

* `{$ARTEMIS_ADDRESS_MEMORY_USAGE_1}` - The threshold for `AddressMemoryUsagePercentage` before the `Warning` trigger is fired.
* `{$ARTEMIS_ADDRESS_MEMORY_USAGE_2}` - The threshold for `AddressMemoryUsagePercentage` before the `High` trigger is fired.
* `{$ARTEMIS_ADDRESS_MEMORY_USAGE_3}` - The threshold for `AddressMemoryUsagePercentage` before the `Disaster` trigger is fired.
* `{$ARTEMIS_DISK_USAGE_1}` - The threshold for `DiskStoreUsage` before the `Warning` trigger is fired.
* `{$ARTEMIS_DISK_USAGE_2}` - The threshold for `DiskStoreUsage` before the `High` trigger is fired.
* `{$ARTEMIS_DISK_USAGE_3}` - The threshold for `DiskStoreUsage` before the `Disaster` trigger is fired.
* `{$JMX_USERNAME}` - The username for JMX access.
* `{$JMX_PASSWORD}` - The password for JMX access.

