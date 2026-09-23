# Prop News Guard, platform annex

Companion to `TOTAL-BRIEF-Prop-News-Guard.md`, which wins on product. This file
carries what each OS actually allows for the two locked behaviours, Soft gate and
push, so CC architecture starts from facts, not hopes. 23 Sep 2026.

## Locked (from the total brief)

- Phone, tablet and desktop all in scope. Shared codebase preferred.
- Soft gate is the default protection mode. Same behaviour on every device.
- Push is required on every surface. Server-scheduled ladder: T-60, T-15, T-5, T-1,
  open, end. Local-only reminders are not accepted as the primary mechanism.
- MVP track: Android phone and tablet Soft gate plus push, then Windows desktop Soft
  gate plus push, then iOS and iPad push with Soft gate as far as allowed, then packs,
  then stores.
- Never touches a trade. Never logs into an account. Never runs on a terminal.

## Devices

| Surface | Soft gate | Push channel | Track |
|---|---|---|---|
| Android phone and tablet | Full, foreground detection plus overlay | FCM high priority, full-screen intent, alarm repeat | MVP 1 |
| Windows desktop | Full, MT5 launch and focus detection | Windows notification centre, fed by app socket or WNS | MVP 2 |
| iPhone and iPad | Screen Time shield, Apple's card | APNs Time Sensitive | MVP 3 |
| macOS | NSWorkspace activation, own panel | APNs for Mac, or menu-bar agent socket | Post-launch |

## Soft gate, what each OS allows

Android. Foreground app via UsageStatsManager with the PACKAGE_USAGE_STATS special
access, granted once in Settings. Poll from a foreground service at 1s during active
windows only, idle otherwise. Gate is a SYSTEM_ALERT_WINDOW overlay. MT5 package is
net.metaquotes.metatrader5, MT4 is net.metaquotes.metatrader4. The total brief says
"overlay / accessibility, like focus apps". Use UsageStats, not AccessibilityService.
Play review rejects accessibility use that isn't for accessibility, and focus apps
that lean on it get pulled. Known risk: Samsung, Xiaomi and OnePlus battery managers
kill the service. Onboarding needs the unrestricted-battery prompt and a persistent
notification during windows.

Windows. MT5 is terminal64.exe, MT4 is terminal.exe. Launch via a WMI process-start
event subscription, focus via SetWinEventHook on EVENT_SYSTEM_FOREGROUND. Gate is a
topmost, full-screen window on the monitor MT5 is on. Ships as its own tray app.
It must not be the MTBridge agent: the total brief says the MVP never runs on a
terminal and must be invisible to the firm, and MTBridge is a terminal-side
connector. Keep the guard a separate, unrelated process.

iOS and iPadOS. No API reports another app launching or gaining focus. The one legal
route is the Screen Time stack: FamilyControls with individual authorisation,
ManagedSettings shields, DeviceActivity schedules. The user picks MT5 in
FamilyActivityPicker (the token is opaque, we never learn the bundle id). A
DeviceActivityMonitor extension applies the shield at window open and lifts it at
window end, and it runs when the app is killed. The shield card is Apple's template:
title, subtitle, icon, one or two buttons, taps handled by a ShieldAction extension.
"Hold 3 seconds to view only" is not possible on the card; the nearest is a
secondary button that lifts the shield for a fixed period. Needs the Family Controls
distribution entitlement from Apple, request it now, it's the long pole. This matches
the brief's "full shield entitlement is Phase 1.5".

macOS. NSWorkspace didLaunchApplicationNotification and
didActivateApplicationNotification give launch and focus for MetaTrader 5.app. Gate
is a floating NSPanel. The app must run as a login item or menu-bar agent. Screen
Time shields are not available to third-party Mac apps.

## Push, what each OS allows

Recommended shape: the ladder is scheduled on the server against the calendar and
the user's basket. Each rung carries an alert id. Every device also mirrors the
ladder as OS-scheduled local notifications, deduped by alert id. Remote is primary
because it reacts to calendar changes and rule edits. Local is the safety net
because APNs coalesces, FCM throttles and Doze delays. A quiet single ping is the
failure mode the brief names; belt and braces is how you avoid it.

iOS. APNs with interruption level time-sensitive. Needs the
com.apple.developer.usernotifications.time-sensitive entitlement. Breaks through
Focus when the user allows it. Critical Alerts (bypass the silent switch) need a
separate Apple approval that trading apps rarely get. Don't plan on it.

Android. FCM high-priority message wakes the app from Doze. Full-screen intent needs
USE_FULL_SCREEN_INTENT, and on Android 14 and later it is not auto-granted for
non-alarm apps, the user turns it on in Settings. Alarm-style repeat until dismissed
is fine once that's granted. Exact alarms for the local mirror need
SCHEDULE_EXACT_ALARM or the setAlarmClock path.

Windows. WNS needs an MSIX-packaged app with a Store or Azure identity. Simpler for
MVP: the tray app holds a socket to the backend, receives the rung, raises a toast
through the notification centre with a registered AUMID. Survives MT5 on another
monitor. Does not survive the tray app being closed, so the app should also register
scheduled toasts for the mirror. Decide MSIX versus signed installer in
architecture; MSIX gets you WNS for free.

macOS. APNs works for signed, notarised Mac apps that have launched once. Fallback is
the menu-bar agent socket, same as Windows.

## Cost lines to price before build

- Calendar API licence (the brief already flags this as the biggest running cost).
- Push infra: APNs is free, FCM is free, the scheduler and its database are yours.
- Apple Developer Program, Google Play, and code-signing certificates for Windows.

## Open

- Name. Grok's candidates are Bench, Still and Holdoff. Trademark and domain check
  before the pick.
- Family Controls entitlement request date.
- MSIX versus signed installer for Windows.
- Push plus local mirror, or push only. Annex recommends both.
