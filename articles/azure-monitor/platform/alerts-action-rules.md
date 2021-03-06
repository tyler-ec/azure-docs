---
title: Action rules for Azure Monitor alerts
description: Understanding what action rules are, and how to configure and manage them.
author: anantr
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 04/25/2019
ms.author: anantr
ms.component: alerts
---

# Action rules (preview)

This article describes what action rules are, and how to configure and manage them.

## What are action rules?

Action rules allow you to define actions (or suppression of actions) at any Resource Manager scope (Subscription, Resource Group, or Resource). They have a variety of filters that allow you to narrow down to the specific subset of alert instances that you want to act on. 

With action rules you can:

* Suppress actions and notifications if you have planned maintenance windows or for the weekend/holidays, instead of having to disable each alert rule individually.
* Define actions and notifications at scale: Instead of having to define an action group individually for each alert rule, you can now define an action group to trigger for alerts generated at any scope. For example, I can choose to have action group 'ContosoActionGroup' trigger for every alert generated within my subscription.

## Configuring an action rule

You can access the feature by selecting **Manage actions** from the Alerts landing page in Azure Monitor. Then select **Action Rules (Preview)**. You can access them by selecting **Action Rules (preview)**  from the dashboard of the landing page for Alerts.

![Action rules from the Azure Monitor landing page](media/alerts-action-rules/action-rules-landing-page.png)

Select **+ New Action Rule**. 

![Add new action rule](media/alerts-action-rules/action-rules-new-rule.png)

Alternatively, you can also choose to create an action rule while configuring an alert rule.

![Add new action rule](media/alerts-action-rules/action-rules-alert-rule.png)

You should now see the action rule creation flow open. Configure the following elements: 

![New action rule creation flow](media/alerts-action-rules/action-rules-new-rule-creation-flow.png)

### Scope

First choose the scope, that is, target resource, resource group, or subscription. You also have the ability to multi-select a combination of any of the above (within a single subscription). 

![Action rule scope](media/alerts-action-rules/action-rules-new-rule-creation-flow-scope.png)

### Filter criteria

You can additionally define filter(s) to further narrow down to a specific subset of the alerts on the defined scope. 

The available filters are: 

* **Severity**: Select one or more alert severities. Severity = Sev1 means that the action rule is applicable for all alerts with severity as Sev1.
* **Monitor Service**: Filter based on the originating monitoring service. This is also multi-select. For example, Monitor Service = “Application Insights” means that the action rule is applicable for all “Application Insights” based alerts.
* **Resource Type**: Filter based on a specific resource type. This is also multi-select. For example, Resource Type = “Virtual Machines” means that the action rule is applicable for all Virtual Machines.
* **Alert Rule ID**: Allows you to filter for specific alert rules using the Resource Manager ID of the alert rule.
* **Monitor Condition**: Filter for alert instances with either "Fired" or "Resolved" as the monitor condition.
* **Description**: Regex matching within the description defined as part of the alert rule.
* **Alert context (payload)**: RegEx matching within the [alert context](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-common-schema-definitions#alert-context-fields) fields of an alert instance.

These filters are applied in conjunction to one another. For example, if I set 'Resource type' = 'Virtual Machines' and 'Severity' = 'Sev0', then I have filtered for all 'Sev0' alerts on only my VMs. 

![Action rule filters](media/alerts-action-rules/action-rules-new-rule-creation-flow-filters.png)

### Suppression or action group configuration

Next configure the action rule for either alert suppression or action group support. You cannot choose both. The configuration acts on all alert instances matching the previously defined scope and filters.

#### Suppression

If you select **suppression**, configure the duration for the suppression of actions and notifications. Choose one of the following:
* **From now (always)**: Suppresses all notifications indefinitely.
* **At a scheduled time**: Suppress notifications within a bounded duration.
* **With a recurrence**: Suppress on a recurrence schedule, which can be daily, weekly, or monthly.

![Action rule suppression](media/alerts-action-rules/action-rules-new-rule-creation-flow-suppression.png)

#### Action group

If you select **Action group** in the toggle, either add an existing action group or create a new one. 

> [!NOTE]
> You can associate only one action group with an action rule.

![Action rule action group](media/alerts-action-rules/action-rules-new-rule-creation-flow-action-group.png)

### Action rule details

Lastly, configure the following details for the action rule
* Name
* Resource Group in which it will be saved
* Description 

## Example scenarios

### Scenario 1: Suppression of alerts based on severity

Contoso wants to suppress notifications for all Sev4 alerts on all VMs within their subscription 'ContosoSub' every weekend.

**Solution:** Create an action rule with
* Scope = 'ContosoSub'
* Filters
    * Severity = 'Sev4'
    * Resource Type = 'Virtual Machines'
* Suppression with recurrence set to weekly, and 'Saturday' and 'Sunday' checked

### Scenario 2: Suppression of alerts based on alert context (payload)

Contoso wants to suppress notifications for all log alerts generated for 'Computer-01' in 'ContosoSub' indefinitely as it's going through maintenance.

**Solution:** Create an action rule with
* Scope = 'ContosoSub'
* Filters
    * Monitor Service = 'Log Analytics'
    * Alert Context (payload) contains 'Computer-01'
* Suppression set to 'From now (Always)'

### Scenario 3: Action group defined at a resource group

Contoso has defined [a metric alert at a subscription level](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-metric-overview#monitoring-at-scale-using-metric-alerts-in-azure-monitor), but wants to define the actions that trigger for alerts separately for their resource group 'ContosoRG'.

**Solution:** Create an action rule with
* Scope = 'ContosoRG'
* No filters
* Action Group set to 'ContosoActionGroup'

## Managing your action rules

You can view and manage your action rules from the list view as shown below.

![Action rules list view](media/alerts-action-rules/action-rules-list-view.png)

From here, you can enable/disable/delete action rules at scale by selecting the checkbox next to them. Clicking on any action rule opens up its configuration page, allowing you to update its definition and enable/disable it.

## Best practices

Log alerts created with the ['number of results'](https://docs.microsoft.com/azure-monitor/platform/alerts-unified-log) option generate **a single alert instance** using the whole search result (which could be across multiple computers for example). In this scenario, if an action rule uses the 'Alert Context (payload)' filter, it will act on the alert instance as long as there is a match. In scenario 2 as described previously, if the search results for the log alert generated contain both 'Computer-01' and 'Computer-02', the entire notification is suppressed (that is, there is no notification generated for 'Computer-02' at all).

![Action rules and log alerts (number of results)](media/alerts-action-rules/action-rules-log-alert-number-of-results.png)

To best leverage log alerts with action rules, we advise you to create log alerts with the ['metric measurement'](https://docs.microsoft.com/azure-monitor/platform/alerts-unified-log) option. Using this option, separate alert instances are generated based on the Group Field defined. Then in scenario 2, separate alert instances are generated for 'Computer-01' and 'Computer-02'. With the action rule described in the scenario, only the notification for 'Computer-01' would be suppressed while the notification for 'Computer-02' would continue to fire as normal.

![Action rules and log alerts (number of results)](media/alerts-action-rules/action-rules-log-alert-metric-measurement.png)

## FAQ

* Q. While configuring an action rule, I would like to see all the possible overlapping action rules so that I avoid duplicate notifications. Is it possible to do so?

    A. Once you define a scope while configuring an action rule, you can see a list of action rules which overlap on the same scope (if any). This overlap can be one of the following options:
    * An exact match: For example, the action rule you are defining and the overlapping action rule are on the same subscription.
    * A subset: For example, the action rule you are defining is on a subscription, and the overlapping action rule is on a resource group within the subscription.
    * A superset: For example, the action rule you are defining is on a resource group, and the overlapping action rule is on the subscription that contains the resource group.
    * An intersection: For example, the action rule you are defining is on 'VM1' and 'VM2', and the overlapping action rule is on 'VM2' and 'VM3'.

    ![Overlapping action rules](media/alerts-action-rules/action-rules-overlapping.png)

* Q. While configuring an alert rule, is it possible to know if there are already action rules defined that might act on the alert rule I am defining?

    A. Once you define the target resource for your alert rule, you can see the list of action rules which act on the same scope (if any) by clicking on 'View configured actions' under the 'Actions' section. This list is populated based on following scenarios for the scope:
    * An exact match: For example, the alert rule you are defining and the action rule are on the same subscription.
    * A subset: For example, the alert rule you are defining is on a subscription, and the action rule is on a resource group within the subscription.
    * A superset: For example, the alert rule you are defining is on a resource group, and the action rule is on the subscription that contains the resource group.
    * An intersection: For example, the alert rule you are defining is on 'VM1' and 'VM2', and the action rule is on 'VM2' and 'VM3'.
    
    ![Overlapping action rules](media/alerts-action-rules/action-rules-alert-rule-overlapping.png)

* Q. Can I see the alerts that have been suppressed by an action rule?

    A. In the [alerts list page](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-managing-alert-instances), there is an additional column that can be chosen called 'Suppression Status'. If the notification for an alert instance was suppressed, it would show that status in the list.

    ![Suppressed alert instances](media/alerts-action-rules/action-rules-suppressed-alerts.png)

* Q. If there's an action rule with an action group and another with suppression active on the same scope, what happens?

    A. **Suppression always takes precedence on the same scope**.

* Q. What happens if I have a resource monitored in two separate action rules? Do I get one or two notifications? For example 'VM2' in this scenario:

      action rule 'AR1' defined for 'VM1' and 'VM2' with action group 'AG1' 
      action rule 'AR2' defined for 'VM2' and 'VM3' with action group 'AG1' 

    A. For every alert on 'VM1' and 'VM3', action group 'AG1' would be triggered once. For every alert on 'VM2', action group 'AG1' would be triggered twice (**action rules do not de-duplicate actions**). 

* Q. What happens if I have a resource monitored in two separate action rules and one calls for action while another for suppression? For example, 'VM2' in this scenario:

      action rule 'AR1' defined for 'VM1' and 'VM2' with action group 'AG1' 
      action rule 'AR2' defined for 'VM2' and 'VM3' with suppression

    A. For every alert on 'VM1', action group 'AG1' would be triggered once. Actions and notifications for every alert on 'VM2' and 'VM3' will be suppressed. 

* Q. What happens if I have an alert rule and an action rule defined for the same resource calling different action groups? For example, 'VM1' in this scenario:

     alert rule  'rule1' on          'VM1' with action group 'AG2'
     action rule 'AR1'   defined for 'VM1' with action group 'AG1',  
 
    A. For every alert on 'VM1', action group 'AG1' would be triggered once. Whenever alert rule 'rule1' is triggered, it will also trigger 'AG2' additionally. (**action groups defined within action rules and alert rules operate independently, with no de-duplication**) 

## Next steps

- [Learn more about alerts in Azure](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-overview)
