# Messages

Back to [index](README.md) · [sitemap](../sitemap.md) · reach rules: [sitemap 1.4](../sitemap.md#14-who-can-see-or-reach-whom)

Direct chats are one-to-one. Group chats belong to a section or subject and follow enrollment (STU-01); people do not create them by hand.

---

## P11 Messages — `/messages`

| | |
|---|---|
| File | `src/routes/_app/messages.tsx` (two-pane layout) + `messages/index.tsx` (nothing selected) |
| Permission | `message.send` |
| Modules | STU-03, PRO-03, CHR-06, DEAN-06, GUD-05, OSAS-04 |
| Stories | US-16 |
| Tests | STU-03-V/I, PRO-03-V/I, CHR-06-V/B, DEAN-06-V/B, GUD-05-V/I, OSAS-04-V/I |
| Template | T5 Conversation |

**Access by role**

| Role | Access | Can start a conversation with |
|---|---|---|
| Student | ✓ | Own professors, permitted peers; own class groups are listed automatically |
| Professor | ✓ | Assigned students, faculty colleagues; own class groups |
| Program Chair | ✓ | Program faculty; students only in approved special circumstances (TBD) |
| Dean | ✓ | Faculty; students only in approved special circumstances (TBD) |
| Guidance, OSAS | ✓ | Authorized faculty |
| Organization, Staff | † | TBD (D-06) |
| Administrator, President | — | |

**Layout**

| Region | Components | Notes |
|---|---|---|
| List heading | `SectionHeader level={1} split title="Messages"`, `actions` = `IconButton variant="solid" icon="plus" label="New message"` | |
| Search | `Search label="Search conversations"` | Searches names and group titles, not message text in other people's chats. |
| Filter | `Tabs` (All, Direct, Groups) with `count` for unread | |
| List | DS-3 `ConversationList`: `Avatar` / `AvatarGroup`, name or group title, `RoleBadge`, last-message preview, time, unread count | Previews never include case content. |
| Right pane | `EmptyState title="Pick a conversation"` | Desktop only; phones show the list alone. |
| New message | `Modal title="New message"`: `ComboBox label="To"` (permitted recipients only, `emptyMessage="No one you can message matches"`), `TextArea label="Message"`, `ModalActions` Cancel / Send | |

**States**

| Case | Result | Test |
|---|---|---|
| No conversations | `EmptyState title="No messages yet"` with action "New message" | |
| Recipient outside scope | Not in the `ComboBox`; a crafted request is denied and nothing is created | STU-03-I, PRO-03-I, GUD-05-I, OSAS-04-I |
| Chair or Dean to a student outside an approved circumstance | Not offered (rule TBD) | CHR-06-B, DEAN-06-B |

---

## P12 Conversation — `/messages/$threadId`

| | |
|---|---|
| File | `src/routes/_app/messages/$threadId.tsx` |
| Permission | `message.send` and membership in the conversation |
| Modules | STU-03, PRO-03, CHR-06, DEAN-06, GUD-05, OSAS-04, ADM-02 (flags) |
| Tests | STU-03-V/I/B, PRO-03-V/I/B, GUD-05-B, OSAS-04-B |
| Template | T5 Conversation |

**Access.** Participants only (S). Everyone else gets the restricted record copy, including moderators until a message is reported (moderation access to private chats is TBD, PRO-03-B).

**Layout**

| Region | Components | Notes |
|---|---|---|
| Header | `Row`: `Avatar` or `AvatarGroup`, title, `RoleBadge`, member count; `MenuButton iconOnly` (Members, Mute) | Members open in a `Drawer side="end"`. Mute uses a `Switch` there. |
| Case warning | `Alert tone="neutral"` "Keep case details in the case record." | Guidance and OSAS conversations (GUD-05-B, OSAS-04-B). |
| Messages | DS-3 `MessageList` with `Divider label` per day and DS-3 `MessageBubble` (sender and `RoleBadge` in groups, time, body, own or other side) | `Button variant="tertiary"` "Load earlier messages" at the top. |
| Bubble actions | `MenuButton iconOnly` per bubble: Copy, Report message | Report opens a `Modal` with a reason and the note "A report starts a human review. It isn't a finding." |
| Composer | DS-3 `MessageComposer`: `TextArea label="Message"`, `IconButton variant="solid" icon="arrow-right" label="Send"`, attachment `IconButton icon="paperclip"` † | Enter sends, Shift+Enter adds a line (`Kbd`). |

**States**

| Case | Result | Test |
|---|---|---|
| Send to a permitted member or class group | Message persists with sender and time for members | STU-03-V, PRO-03-V |
| Not a member, or a guessed URL | Restricted record copy; nothing created | STU-03-I, PRO-03-B |
| Flag harmful text | Goes to human review on P49; the flag alone proves nothing | STU-03-B |
| Send fails | `Alert tone="danger"` beside the composer; text kept | |
| Restricted student | Professor conversations keep working (US-49); other limits TBD | STU-11-B |

**Open.** Organization and Staff messaging (D-06); Chair and Dean messages to students; moderator access to private chats; attachments; retention (D-08).
