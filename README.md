# Straight Forward: send one message to several private chats

Write a message, long-press **Send** and choose **Forward**: the message goes straight to several people at once, each in
their own private chat, without a "Forwarded from" label. Works for text and for a batch of photos. In a usability test,
sharing was 33% faster than with the existing Forward.

<p>
  <img src="screenshots/01-send-menu-after.jpg" width="24%" alt="Long-press on Send with Forward">
  <img src="screenshots/02-forward-preselected.jpg" width="24%" alt="Forward sheet with the current chat preselected">
  <img src="screenshots/03-sender-none.jpg" width="24%" alt="The message arrives without a sender name">
  <img src="screenshots/04-attach-send-forward.jpg" width="24%" alt="Attached photos and long-press on Send with Forward">
</p>

## Why

People often want to share a personal update, holiday greetings or a set of vacation photos with several friends or family
members individually, without creating a group chat. Today this takes too many steps:

- **Too many steps.** The message has to be sent to one person first, then selected and forwarded to the others.
- **Unwanted attribution.** A forwarded message shows "Forwarded from" with the sender's own name, which makes it feel less
  personal. Hiding it is a separate, manual step, and most people don't know it exists.
- **Fragmented photo sharing.** A message holds up to 10 photos, so a large collection ends up split across several messages
  that all have to be selected and forwarded.

In interviews with 8 Telegram users, sharing vacation and family photos and sending holiday greetings were the most common
cases. Most participants felt uncomfortable receiving a message "forwarded from" the sender themselves.

## What it does

### 1. Forward from the Send button

Long-pressing a sent message opens a menu with **Forward**. The same gesture on the **Send** button opens a menu with enough
room for that same option, so **Forward** is added there too.

### 2. The current chat is preselected

**Forward** opens the familiar sheet, but without the **Select** step: the person of the chat where the message was written is
already selected, so you only add the other recipients.

### 3. No sender name

The message is sent as an original message in every chat. The sender's name is hidden automatically, with no extra steps.

### 4. Photos too

The **Send** button that appears after attaching photos gets the same **Forward** option, so a photo batch can be shared with
several people as easily as text.

## Results

A moderated usability test of the working build, with the same 8 participants. With no hints, 7 of 8 found Straight Forward
on their own. Each sharing task was then timed with and without it:

| Share with several people | Forward | Straight Forward | Time saved |
| ------------------------- | ------: | ---------------: | ---------: |
| Text message              |    18 s |             14 s |        22% |
| One photo                 |    23 s |             17 s |        26% |
| Large photo batch         |    43 s |             25 s |        42% |
| **Average**               | **28 s** |      **18.7 s** |    **33%** |

## How it works

- **The entry points.** The Send button's long-press menu gets a **Forward** item, for a text message and inside the
  full-screen gallery. The attachment panel gets a **Forward** button once photos are selected.
- **The recipients.** Forward opens the standard peer selection sheet with multiple selection already on, the current chat
  preselected and the typed text carried over, so it can still be edited there.
- **Sending.** Nothing is forwarded under the hood: the text (and a recorded voice message, if any) is enqueued as a new
  message in each selected chat, which is why no "Forwarded from" label appears. No server changes are needed.
- **Photos.** The media picker sends its selection to each chat separately, and gives every chat its own album grouping keys,
  so each recipient gets the album as one group. Captions typed on the photos are kept, and the chat input is cleared after
  sending, as with a normal media send.
- **The existing Forward.** It hides "Forwarded from" when every message was written by you and is not itself a forward,
  unless you chose to show the sender in the forward options. The patch also fixes Saved Messages appearing twice in the
  forward sheet when a chat is preselected.

## Applying the change

`straight-forward.patch` is a series of 5 commits made against the official Telegram-iOS master from early October 2026.
From a checkout of that master:

```
git am straight-forward.patch
```

or, to apply it without commits:

```
git apply --check straight-forward.patch
git apply straight-forward.patch
```

Then build as described in the Telegram-iOS README. Most of the feature is in a new file,
`submodules/TelegramUI/Sources/ChatControllerBroadcastMessage.swift`. The rest of the patch is small hooks in the Send
button's menu (`ChatSendMessageActionUI`), the attachment panel and media picker (`AttachmentUI`, `MediaPickerUI`), the peer
selection sheet (`PeerSelectionController`) and the forward flow (`ChatControllerForwardMessages.swift`).

## More

The design was built directly in the source code of Telegram for iOS, for the Telegram Designers contest. The full case study,
with the research, before-and-after screens and a demo video, is at
[marinakuzminchuk.github.io/telegram-straight-forward](https://marinakuzminchuk.github.io/telegram-straight-forward/).
