---
description: >-
  Staking is the primary mechanism by which impact on the trajectory of the
  system is allocated across actors.
---

# Staking

## Introduction

Staking, or bonding, is the act of locking up funds under some terms so that they are not transferable and otherwise not entirely usable as they otherwise would be. The terms, referred to as _unstaking terms_ describe the circumstances under which the funds may begin to cease being staked. This may involve who is able to initiate this, at what time and whether there is a possible delay from initiation to completion. This time lag is referred to as the _unstaking period._ Another critical term will also be whether and some part, possibly all of, the funds may be burned. This is referred to as _slashing._

## **Modes**

Staking is used in two modes to serve the system as a whole by attempting to providing more robust incentives for socially optimal conduct in some role that impacts the overall success of the system.

1. **Exposure:** By requiring that someone who occupies a role that impacts the value of the system has exposure to that value in their portfolio. For this requirement to be effective, this exposure should not be hedgeable, and it is generally assumed that markets for this are missing. It is also assumed that any harm or benefit that results from the actions of the actor will capitalize in the value of the platform, and thus be partially reflected in the value of the stake. This should in total discourage harmful conduct and encourage beneficial conduct.
2. **Punishment:** In cases where it is possible to, if only imperfectly, have the system adjudicate whether an actor has acted harmfully, the ability to slash funds as a result of such detection can generate very strong incentives for pro-social behavior. The adjudication may be purely cryptographic, or it may require some level of social consensus. In either case, to the extent that it reliably can detect failure - that is avoiding false positives and negatives, it is a very cost-effective means of generating incentives compared to the first approach. It's cheaper because it allows for less capital to be locked for a given level of deterrence effect.

## Activities

One can currently stake funds for a range of activities, and the table below lists them, along with

| Activity | Exposure | Punishment |
| :--- | :---: | :---: |
| Voting | Yes | No |
| Staking Candidate | Yes | No |
| Council | Yes | No |
| Validation | Yes | Yes |
| Nomination | Yes | Yes |
| Proposals | Yes | Yes\* |
| Worker\*\* | Yes | Yes |

_\* It varies across_ [_proposal types_](../governance/proposals.md#proposal-type) _whether punishment is actually used, but in the interest of keeping the staking model simple, it is assumed it always is. \*\* Can be both lead an non lead workers in a working group._

## Implementation

The way staking is implemented is with the use of account [locks](staking.md). Each purpose above has one or more fixed number of locks associated with it, each with its own fixed ID. This means it is very easy to simply look at an account and understand in what staking activity it is involved. Some purposes allow more than one account to hold stake for the given purpose, others do not. Some purposes allow for staking any account, while others require that you are staking with an account that has been bound to a specific membership. The this binding constraint comes from purposes where staking itself is associated with membership, and this binding allows initiation of staking with a single extrinsic signed with membership credentials, rather than having an additional extrinsic for each arbitrary account used for staking on each occasion.

## Reuse

Given staking purposes A and B, we can say that stake is reusable across A and B if a token staked towards one can simultaneously count as staked towards the other. Since staking is implemented with locks, and locks do not stack, this means that a single account cannot be used for staking across two non-reusable purposes.

&lt;figure?&gt;

Reusability does imply that if there is a slashing event in the context of one of the two, stake in both has been reduced. This must be accounted for by platform actors, and for this reason it is currently not possible to reuse stake across two activities where both are subject to slashing. In fact, the only reuse allowed currently is when exactly one of A or B is voting. This is primarily to counteract the fact that voting is not incentivized, hence lowering the cost to vote is critical.

The table below summarizes the current reusability relationships, changing them currently requires a runtime upgrade.

|  | Staking Candidate | Invitation | Voting | Council Candidate | Councilor | Validation | Nomination | Proposals | Worker\* |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| Staking Candidate | No | Yes | Yes | Yes | Yes | Yes | Yes | Yes | Yes |
| Invitation | - | No | Yes | Yes | Yes | Yes | Yes | Yes | Yes |
| Voting | - | - | No | Yes | Yes | Yes | Yes | Yes | Yes |
| Council Candidate | - | - | - | No | Yes | No | No | No | No |
| Councilor | - | - | - | - | No | No | No | No | No |
| Validation | - | - | - | - | - | No | No | No | No |
| Nomination | - | - | - | - | - | - | No | No | No |
| Proposals | - | - | - | - | - | - | - | No | No |
| Worker\* | - | - | - | - | - | - | - | - | No |

The table is symmetric, as all reuse relationships are symmetric, hence cells are omitted beyond the diagonal to ignore duplicate specification.

_\*Any working group, and whether lead or normal worker._

## Locks and Binding

In what follows we attempt to briefly summarizes the what locks exist for what purposes, and on what accounts they are applied.

| Purpose | Binding | ID |
| :--- | :---: | :---: |
| Voting | No | 0 |
| Council Candidate | Yes | 1 |
| Councilor | Yes | 2 |
| Validation | No | 3 |
| Nomination | No | 4 |
| Proposals | Yes | 5 |
| Storage Worker | Yes | 6 |
| Content Directory Worker | Yes | 7 |
| Forum Worker | Yes | 8 |
| Membership Worker | Yes | 9 |
| Invitation | Yes | 10 |
| Staking Candidate | Yes | 11 |

## Slashing

Slashing is the act of reducing the balance of an account by some amount, and also reducing the size of the lock which represents the use case under which the slashing occurs.

## State bloat

Some modules such as forum and working group allows user to call extrinsics that allows them to occupy storage. This can be a problem since some malicious users could use this to fill up the storage, either taking all the alloted space for that given storage map or unboundedly claim storage space until no single node has enough storage. Furthermore, there is no incentive even for regular users to cleanup their storage use.

That's why when a user can use up storage on top of the transaction fee we require either a deposit or additional stake.

When no longer using up the storage the stake is released or the deposit is paid off.

While staking is easier to implement when we are already requiring an stake account, a deposit allows to incentivize other users cleaning up by paying the deposit to them instead of the original person making the deposit.

As it stands now the modules require a deposit to prevent the state bloat are:

* Forum pallet
* Pallet Discussion
* Blog

The pallets requiring a stake to prevent state bloat are:

* Working Group
* Membership

