---
title: "How to fix silenced push notifications on Android"
draft: false
date: 2023-03-05T06:00:00+01:00
tags: 
- Push Notifications
- Notification channel
- Android
- Flutter
- Dart
- Mobile
categories:
- Flutter
description: "When working on an application that notifies users in the case of an evacuation, my team and I encountered an interesting bug. Other developers are probably going to run into the same issue, so here's how to fix it."
images: []
resources:
- name: "featured-image"
  src: "featured-image.png"
---

<!--more-->

## The issue

When working on an application that notifies users in the case of an evacuation, my team and I encountered an interesting bug. Our notifications on Android were *silent* by default, even though they were sent as critical alerts from the backend.

In our code, we hooked into the notification handler and showed the notifications locally using `flutter_local_notifications` to be able to adjust some of the content.

So I started debugging and found the issue after a while...

### Androids importance level

In Android we have to create a notification channel to be able to show local notifications. This notification channel two properties that are of interest: `Importance` and `Priority`.

`Priority` is only used for Android 7.1 and lower according to [the documentation](https://developer.android.com/develop/ui/views/notifications/channels#importance), so that was not of interest for us here. The `importance` setting however, *was* interesting. But wait - we already set this to max in our code? What could possibly be wrong?

```dart
static final _channel = AndroidNotificationChannel(
  'alarm_notification_channel',
  'Alarm Notifications',
  description: 'This channel is used for alarm notifications.',
  importance: Importance.max, // The importance was set to max...
  playSound: true,
  sound: const RawResourceAndroidNotificationSound('alarm'),
  enableVibration: true,
  vibrationPattern: Int64List.fromList([0, 30000]),
);
```

If you read [the documentation](https://developer.android.com/develop/ui/views/notifications/channels#importance) carefully, you'll notice that the highest value for the `Priority` setting is `max`. That is fine.

But the highest `Importance` option is `high`, not `max`. The `flutter_local_notifications` plugin also supports `max` for the `Importance` setting, so this makes it very easy to accidently choose the wrong option. To fix this problem, we just have to change the `Importance` setting:

```dart
static final _channel = AndroidNotificationChannel(
...
  importance: Importance.high, // Set this to 'high' instead of 'max'
...
);
```

I hope this will save someone a few hours of valuable time debugging this!

## Lessons learned

When setting these kind of options, don't blindly assume what the value means. Always check out the official documentation to make sure the setting will do what you expect it to do.
