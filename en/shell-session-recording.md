# Shell Session Recording

Session recording saves what appears on screen in terminal sessions opened in your organization. Admins can watch the recordings later. Recording is off by default and available on all plans.

---

## Turning recording on

1. On **Account → Organizations**, find the **Shell access** section.
2. Turn on **Record shell sessions**.
3. Read the dialog, choose a retention period (7, 30 or 90 days; 30 by default) and confirm.

Only people who can manage the organization can do this. Turning it on or off and changing the period are written to the audit log.

Recording applies to sessions started after the setting is on. Shells open at that moment are not recorded. Turning it off also affects only new sessions: a session that started with recording is recorded until it closes.

## What users see

While recording is on, the **Terminal** page shows a red band before connecting and throughout the session. It says the session is being recorded and cannot be dismissed.

## What is recorded

- Everything that appears on screen: echoed commands, command output, file contents, environment variables.
- Terminal window size changes and the timing of events.

**Passwords, keys, tokens and personal data that get printed on screen are recorded too.** Review what your teams can see in a shell before you turn recording on.

## What is not recorded

Input that is not echoed is not recorded, such as a password typed at a prompt. A recording is not a list of commands: a command that is not echoed does not appear, and a recording does not prove what a program started from the shell did.

## Watching and deleting recordings

Recordings appear in the **Recordings** list in the same section, with the service, the person, the start time and the date they will be deleted. **Watch** plays a recording; you can pause, speed up and move back and forth. Long quiet stretches are shortened during playback.

- **Delete** removes the recording for good. You are asked to type the service name to confirm.
- Watching and deleting are written to the audit log. The same person watching the same recording is written once an hour.
- The recording of a session still running can be watched but not deleted until the session ends.
- The **Incomplete** label means the end of the session was not recorded. The connection service closing unexpectedly can lose the last few seconds.

## Retention

Recordings are deleted automatically when the period you chose ends. If you change the period, the new one applies to existing recordings too: shortening it means recordings older than the new period are deleted in the next daily cleanup.

## When a session cannot be recorded

While recording is on, if a session cannot be recorded, **the shell does not open, and an open shell closes.** Sessions larger than 64 MiB after compression also close. If recording is unavailable, you need to turn the setting off to open a shell.

---

## Data processing terms

*Version: 2026-11-04. These apply when the feature is turned on and are an addendum to your service agreement.*

**Roles.** Your organization decides to record. For recordings, your organization is the data controller and Komuta is the data processor.

**Your responsibilities.** You are responsible for informing the people who open shells and the people whose data may appear in recordings, for having a lawful basis, and for managing access to recordings within your organization. The band in the terminal does not replace this.

**Data processed.** Content shown on screen during the session, which service and user opened the session, and timing. This content can include personal data and secrets that get printed on screen.

**Purpose.** Recordings are stored only to provide this feature to you. Komuta does not use the content of recordings for any other purpose.

**Location and sub-processor.** Recordings are stored in a separate storage area on Backblaze B2 in an EU region.

**Security.** Each recording is encrypted with its own key using AES-256-GCM. That key is stored in the database, encrypted with Komuta's key. Komuta is therefore technically able to decrypt recordings; encryption stops the storage provider from reading the content, not Komuta.

**Deletion.** Deleting a recording or reaching the end of its period destroys its key and deletes its parts from storage. Deleted parts are fully gone from storage within a few days. Audit logs do not contain the recording's content and follow their own retention.

**Changes.** When a new version of these terms is published, the version date is updated.
