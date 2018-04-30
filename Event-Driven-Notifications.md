# Event Driven Notifications

Evergreen has the ability to issue notifications based on events that happen in the system.

## Patches
Evergreen can notify you based on the outcome of your patches. To enable these, go to your [user profile](https://evergreen.mongodb.com/settings) and:
1. Enter your Github Username (so we can identify which commits you make)
2. Optionally, enter your Slack username (if you want to be notified on Slack)
3. Under Notification Settings, select the Slack or Email next to "Patch Finish".

For all new patches you create in the future, you'll receive an email.

## Project Admin Notifications
Project administrators may enable certain notifications at the project level:

### Build Break Notifications
When a task that was previously green (or is a new), becomes red or purple, a build-break notification will be issued. If the original committer can be identified, Evergreen will lookup their build-break notification settings, and follow that. 

If we can't identify the original committer, Evergreen will notify project admins.