# Prop Firm Shield, working brief

Working name still open: Bench / Still / Holdoff.

Source of truth is TOTAL-BRIEF on the desktop (Grok's draft). This file carries the
items that are locked as of 22 Sep 2026 plus the platform reality behind each one, so
the cloud sessions have something to build against. Merge TOTAL-BRIEF into this file
when it lands in the repo.

## What it is

A rule-protection companion for prop-firm traders. It knows the trader's restricted
windows (news blackouts, session cutoffs, cooldowns after a loss, daily-loss lockouts)
and gets between the trader and MT5 when one of those windows is active.

Two behaviours are locked:

1. Soft gate. When MT5 opens or comes to the front during a restricted window, the app
   puts a gate in the way: what the window is, when it ends, and a deliberate way
   through. Same behaviour on every device.
2. Push. A remote-scheduled alert ladder on every surface. Local-only timers are not
   accepted as the primary mechanism.

## Devices and tracks

| Surface | Soft gate | Push channel | Track |
|---|---|---|---|
| Windows desktop | Full, detect MT5 launch and focus | Desktop notification centre via WNS, or MTBridge agent socket | MVP |
| Android phone and tablet | Full, detect MT5 foreground | FCM high priority, full-screen intent | MVP |
| iPhone and iPad | As far as iOS allows (Screen Time shield) | APNs Time Sensitive | Post-MVP |
| macOS | As far as macOS allows (NSWorkspace activation) | APNs for macOS, or menu-bar agent socket | Post-MVP |

## Soft gate, per platform

Windows. MT5 is terminal64.exe. Process start via WMI event subscription, focus via
SetWinEventHook on EVENT_SYSTEM_FOREGROUND. Gate is a topmost overlay window. The
MTBridge agent already runs beside MT4/5 on these machines, so the gate can live in the
same process tree. No OS limits.

Android. Foreground app via UsageStatsManager with PACKAGE_USAGE_STATS (special app
access, user grants it once in Settings). Poll from a foreground service at 1s. Gate is
a SYSTEM_ALERT_WINDOW overlay. MT5 package is net.metaquotes.metatrader5. Risks:
OEM battery killers (Xiaomi, Samsung, OnePlus) stop the service; the app needs the
"unrestricted battery" prompt and a persistent notification. Do not use
AccessibilityService for this, Play review will reject it.

iOS and iPadOS. There is no API that reports another app launching or gaining focus.
The one legal route is the Screen Time stack: FamilyControls (individual
authorisation), ManagedSettings shields, DeviceActivity schedules. The user picks MT5
in FamilyActivityPicker (the token is opaque, the app never learns the bundle id). A
DeviceActivityMonitor extension applies the shield at window start and lifts it at
window end, and it runs when the app is killed. The shield card is Apple's template:
title, subtitle, icon, one or two buttons, with a ShieldAction extension handling taps.
This needs the Family Controls distribution entitlement from Apple, which is a form and
a wait. It is a real gate, just not our UI.

macOS. NSWorkspace didLaunchApplicationNotification and
didActivateApplicationNotification give launch and focus for MetaTrader 5.app
(CrossOver wrapper). Gate is a floating NSPanel. The app must be running as a login
item or menu-bar agent. Screen Time shields are not available on macOS.

## Push, per platform

Ladder is scheduled on the server against the trader's rule set and account state.
Each rung has an alert id. Every device also mirrors the ladder as OS-scheduled local
notifications, deduped by alert id, because push delivery is not guaranteed on any
platform. Remote is the primary; local is the safety net, not the other way round.

iOS. APNs with interruption-level time-sensitive. Needs the
com.apple.developer.usernotifications.time-sensitive entitlement. Breaks through
Focus if the user allows it. Critical Alerts (bypass silent switch) need a separate
Apple approval that trading apps rarely get; do not plan on it.

Android. FCM high-priority message. Full-screen intent needs USE_FULL_SCREEN_INTENT;
on Android 14 and later it is not auto-granted for non-alarm apps, the user turns it
on in Settings. Exact alarms for the local mirror need SCHEDULE_EXACT_ALARM or the
setAlarmClock path.

Windows. Two options. WNS needs an MSIX-packaged app with a Store or Azure identity.
The MTBridge agent already keeps a live socket to the Karnek backend, so the simpler
path is: agent receives the rung, raises a toast via the Windows notification centre
with a registered AUMID. Survives MT5 being on another monitor, does not survive the
agent being stopped. Recommend the agent path for MVP and WNS only if packaging
happens anyway.

macOS. APNs works for signed, notarised Mac apps that have launched once. Fallback is
the menu-bar agent socket, same as Windows.

## Design

Same OTTO metal system as the rest of Karnek. Next design pass can add a Windows Soft
gate frame and a tablet home before the Claude Code architecture passes.

## Open

- Name: Bench / Still / Holdoff. Needs trademark and domain check before the pick.
- Whether the Windows gate ships inside the MTBridge agent or as its own tray app.
- Family Controls entitlement request timing, since Apple's review gates the iOS track.
- Full TOTAL-BRIEF text to be merged into this file.
