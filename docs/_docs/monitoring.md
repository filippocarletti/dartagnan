---
title: "Monitoring"
permalink: /docs/monitoring/
---

Machines are monitored using [Collectd](https://collectd.org/) thresholds and notifications.

Collectd gathers metrics and checks collected values agains a configured list of thresholds.
If a value is outside of the acceptable range, a new alert is raised and sent to Dartagnan instance.

Alerts can be generated by Collectd thresholds, cron jobs or even external daemons like NMS which
constantly monitor the status of registered services.

More techincal stuff at [nethserver-alerts](https://github.com/NethServer/nethserver-alerts).

## Custom alerts

**Not yet implemented**

Dartagnan allows the configuration of custom alerts which are automatically deployed to the machines
once a day.