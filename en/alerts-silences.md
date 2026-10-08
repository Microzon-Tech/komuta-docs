# Silences: Planned Muting

**Alerts → Silences** mutes notifications for selected rules during a time window. It does not disable monitoring, fix a problem or resolve a historical event. You can also create a silence from a service’s Alerts page in that service’s context.

## Create a maintenance window

1. Open **Create Silence** and give it a name such as **Planned maintenance**.
2. Select only the **alert rules** affected by the maintenance. Check the service name and rule scope.
3. Set **Starts at** and **Ends at**. The form uses your browser’s local time; communicate the timezone when coordinating with other teams.
4. Add a short comment explaining the maintenance.
5. Create it and check the silence’s publication/synchronization state.

Select at least one rule. All selected rules must belong to a single cluster scope; create separate windows for different clusters. The end must be later than the start. Adjust the prefilled two-hour window to fit the actual work.

## Interpret the states

The page separates windows in progress, upcoming windows and expired windows. A window being active by time alone does not prove it reached the target. Do not rely on muting until publication is confirmed. **Alertmanager** here is the notification component applying the silence; customers are not expected to enter its connection address.

If synchronization to the target fails during creation, the operation returns an error; do not proceed as if a successful silence was saved. Correct the selection if a rule was deleted, the target cannot be resolved or rules from different clusters were chosen.

Use **Expire** to end a running window early, or **Delete** to remove an eligible record. Check the outcome afterward. The current interface does not offer a window-editing form; if a different window is needed, review the old window’s state and create a new one.

## Keep the scope narrow

Selecting an account bandwidth rule affects its account scope, not just one service. Select the few rules affected by maintenance instead of starting with every rule. Choosing a service and selecting a rule are different actions.

Review selected chips after clearing rule search; changing the search does not remove earlier selections. Read-only users can view windows, while creating, expiring and deleting require alert-edit permission.

## Reduce notification volume

- Search for existing rules covering the same service and condition. Compare a disabled duplicate before enabling it.
- Increase the hold duration for brief spikes. Investigate persistent threshold breaches before tuning them away.
- Select channels explicitly; an empty selection may route to matching active channels.
- Choose a notification interval suited to how your team responds. A shorter interval is not a guarantee of faster detection.
- Use narrow, time-limited silences for maintenance and check the result when the work ends.

The Channels view can collapse identical records for readability. This is not a control for merging rules or deduplicating future notifications. This section does not provide an automatic escalation workflow or user-configurable incident grouping.

Next: [Tune rules](alerts-rules.md) · [Investigate silence issues](alerts-troubleshooting.md)
