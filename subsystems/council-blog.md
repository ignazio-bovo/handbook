---
description: >-
  A blog where different posts can be published through the proposal system and
  replyied to by anyone.
---

# Council Blog

## Introduction

Using the proposal system blogpost can be made and edited. Members can reply to posts and also to replies making it a nested reply system. Posts can be locked and unlocked through the proposal system, when a post is locked no new replies can be created. Also, replies can be either editable or non-editable, editable replies can be edited throughout their lifetime, they require a deposit which is returned when you make it no longer editable through the `delete_reply` extrinsic.

## Roles

The relevant actors in the forum are

* **Member:** members reply to posts or replies by other users.

## Concepts

### Locking

A blogpost can either be locked or unlocked, when a post is locked it can no longer be modified. Lock/unlock is done through the proposal system.

### Post

A post is defined by the following

* **Locked:** Whether the post is locked or not.
* **Title Hash:** Hash of the title.
* **Body Hash:** Hash of the body.
* **Replies count:** The number of total replies in the post.

### Reply

A reply is defined by the following

* **Text Hash:** Hash of the text of the reply
* **Owner:\*** Creator of the Reply
* **Parent Id:** Id of the parent reply or post
* **Clean Up Payoff:** Reward for deleting the reply
* **Last Edited**: Last time the reply was edited

A reply can be deleted by anyone after `ReplyLifetime` passed since the reply was last edited.

## Constants

The following constants are hard coded into the system, they can only be updated with a runtime upgrade.

| Name             | Description                                                        | Value     |
| ---------------- | ------------------------------------------------------------------ | --------- |
| `REPLY_DEPOSIT`  | Required deposit to create a reply.                                | `fill-in` |
| `REPLY_LIFETIME` | <p>Number of blocks until a reply<br> can be deleted by anyone</p> | `fill-in` |

## Extrinsics

### Create Reply

**Parameters**

| Name             | Description                           |
| ---------------- | ------------------------------------- |
| `participant_id` | Member id of the reply creator.       |
| `post_id`        | Parent post id.                       |
| `reply_id`       | Parent reply id if any.               |
| `text`           | Text of the reply.                    |
| `editable`       | Whether the Reply is editable or not. |

#### Conditions

* Signer's account corresponds to participant.
* Post with `post_id` exists and is unlocked.
* If any `reply_id` has been created at some point.
* If `editable` signer hast at least `REPLY_DEPOSIT` balance.

#### Effect

* Creates a post with hashed text `text`.
* Stores it in storage if `editable`.
* Discount `REPLY_DEPOSIT` from signer's account.

### Edit Reply

**Parameters**

| Name             | Description                   |
| ---------------- | ----------------------------- |
| `participant_id` | Member id of the caller.      |
| `post_id`        | Parent post id.               |
| `reply_id`       | Reply id of the edited reply. |
| `new_text`       | New text of the reply.        |

#### Conditions

* Signer's account corresponds to participant.
* Post with `post_id` exists and is unlocked.
* `reply_id` exists in storage(has been created as editable and has not been erased).
* `participant` is Reply's owner.

#### Effect

* Reply will be updated with the new text hash.
* `last_edited` will move to current block.

### Delete reply

**Parameters**

| Name             | Description                        |
| ---------------- | ---------------------------------- |
| `participant_id` | Member id of the caller.           |
| `post_id`        | Parent post id.                    |
| `reply_id`       | Reply id of the edited reply.      |
| `hide`           | Whether to hide the deleted reply. |

#### Conditions

* Signer's account corresponds to participant.
* Post with `post_id` exists and is unlocked.
* `reply_id` exists in storage(has been created as editable and has not been erased).
* `participant` is Reply's owner or `REPLY_LIFETME` has gone by since Reply `last_edited`.

#### Effect

* Reply will be removed from the storage.
* Reply's `clenaup_pay_off` is transferred to signer's account.

## Examples

**WIP**
