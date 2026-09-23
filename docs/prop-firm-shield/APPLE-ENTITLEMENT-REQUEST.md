# Family Controls entitlement request

Apple grants the distribution entitlement per bundle identifier, and separately for
each Screen Time extension. Four requests in total: the app, the DeviceActivityMonitor
extension, the ShieldConfiguration extension, the ShieldAction extension. Development
builds work without approval; App Store distribution doesn't. Replies take from a few
business days to a few weeks, and Apple reviews by hand, so file this now, before any
iOS code exists. The request form is linked from the Family Controls documentation
under "Requesting the Family Controls entitlement" in the developer portal.

Bundle identifiers below are placeholders until the name clears.

## Text for the form

App name: Wideberth (working title)

Bundle ID: com.stanchion.wideberth
Extensions: com.stanchion.wideberth.monitor (DeviceActivityMonitor),
com.stanchion.wideberth.shieldconfig (ShieldConfiguration),
com.stanchion.wideberth.shieldaction (ShieldAction)

Authorization type: individual. The user manages their own device. No family or child
accounts, no parental features.

Why the app needs Family Controls:

Wideberth is a self-control tool for adults who trade through proprietary trading
firms. Those firms forbid trading in the minutes around scheduled economic releases,
and a single trade inside that window ends the trader's account. The app lets the
user choose their own trading apps in FamilyActivityPicker and shields those apps on
their own device during the restricted windows they've configured. When the window
ends the shield lifts automatically. The user can lift it themselves at any time from
the shield card.

This is a digital wellbeing use in the sense Apple describes for individual
authorization: the user sets limits on their own use of specific apps at specific
times, to protect themselves from an impulsive action with a real financial cost. It
is the same pattern as focus and app-limit tools, with a schedule driven by an
economic calendar instead of a fixed timetable.

What the app does not do:

- It never learns which apps the user selected. Tokens stay opaque and never leave the
  device.
- It never reads, modifies or interacts with any other app's data.
- It has no trading functionality and never connects to any trading account.
- It does not use Family Controls for any monitoring, reporting or third-party
  purpose. DeviceActivity is used only to apply and remove shields on the user's own
  schedule.

How each extension is used:

- DeviceActivityMonitor: applies the shield at the start of a scheduled window and
  removes it at the end, so protection works even when the app isn't running.
- ShieldConfiguration: draws the shield card with the name of the event, the time the
  window ends, and two buttons, "Stay out" and "View for 60 seconds".
- ShieldAction: handles the two buttons. "Stay out" closes the shield. "View for 60
  seconds" lifts it briefly and re-applies it.

Privacy: the app collects no personal data through these APIs. Nothing derived from
Family Controls, ManagedSettings or DeviceActivity is transmitted anywhere.
