---
description: >-
  Staking is the primary mechanism by which impact on the trajectory of the
  system is allocated across actors.
---

# Staking

## Introduction

Staking, or bonding, is the act of locking up funds under some terms so that they are not transferable and otherwise not entirely usable as they otherwise would be. The terms, referred to as _unstaking terms_ describe the circumstances under which the funds may begin to cease being staked. This may involve who is able to initiate this, at what time and whether there is a possible delay from initiation to completion. This time lag is referred to as the _unstaking period._ Another critical term will also be whether and some part, possibly all of, the funds may be burned. This is referred to as _slashing._

## **Purpose**

Staking is used for two purposes to serve the system as a whole by providing more robust incentives for socially optimal conduct in some role that impacts the overall success of the system.

1. **Exposure:** By requiring that someone who occupies a role that impacts the value of the system has exposure to that value in their portfolio. For this requirement to be effective, this exposure should not be hedgeable, and it is generally assumed that markets for this are missing. It is also assumed that any harm or benefit that results from the actions of the actor will capitalize in the value of the platform, and thus be partially reflected in the value of the stake. This should in total discourage harmful conduct and encourage beneficial conduct.
2. **Punishment:** In cases where it is possible to, if only imperfectly, have the system adjudicate whether an actor has acted harmfully, the ability to slash funds as a result of such detection can generate very strong incentives for pro-social behavior. The adjudication may be purely cryptographic, or it may require some level of social consensus. In either case, to the extent that it reliably can detect failure - that is avoiding false positives and negatives, it is a very cost-effective means of generating incentives compared to the first approach. It's cheaper because it allows for less capital to be locked for a given level of deterrence effect.

## Account Locks

The way staking is implemented is with the use of account [locks](staking.md). Each purpose above has one or more fixed number of locks associated with it, each with its own fixed ID. This means it is very easy to simply look at an account and understand in what staking activity it is involved. Some purposes allow more than one account to hold stake for the given purpose, others do not. Some purposes allow for staking any account, while others require that you are staking with an account that has been bound to a specific membership. The this binding constraint comes from purposes where staking itself is associated with membership, and this binding allows initiation of staking with a single extrinsic signed with membership credentials, rather than having an additional extrinsic for each arbitrary account used for staking on each occasion.

## Account Binding

When initiating staking of some kind, it is very often - although not always, in the context of inhabiting some other kind of role, like being a member. This means that the initiating the staking effectively requires proving two things at the same time

1. You occupy the role you claim, by virtue of controlling the role account.
2. You control the funds living on the staking account in question.

Both are achieved by signing with for some account and in general they will not be the same. As a result, it is required that a user connects - or _binds_, a given account which holds funds for the purposes of staking, to their membership, in advance of being able to use that account for staking as that member. This binding is a two step process, per account, where the first step is to turn the account into a _staking candidate_ by issuing a request to bind to a given member by signing via this account, and the second step is for the member to accept this candidate, using the controller account for that membership.

## Lock Overview

In what follows we attempt to briefly summarizes the what locks exist for what purposes, and on what accounts they are applied. A critical concept we also cover is that of _account reuse_ for staking across different purposes, captured in the column `Compatible Lock IDs`. Briefly stated, given staking purposes A and B, we can say that stake is reusable across A and B if a token staked towards one can simultaneously count as staked towards the other. Since staking is implemented with locks, and locks do not stack, this means that a single account cannot be used for staking across two non-reusable purposes.

`<figure?>`

Reusability does imply that if there is a slashing event in the context of one of the two, stake in both has been reduced. This must be accounted for by platform actors, and for this reason it is currently not possible to reuse stake across two activities where both are subject to slashing. In fact, the main reuse allowed currently is when exactly one of A or B is voting or invitation funds. This is primarily to counteract the fact that voting is not incentivized, hence lowering the cost to vote is critical, and that invited users need to be able to use the system, respectively.

| Purpose                       | Binding |       ID      | Compatible Lock IDs  |
| ----------------------------- | :-----: | :-----------: | -------------------- |
| Voting                        |    No   |       0       | All                  |
| Vesting                       |  No\*\* | \*b"vesting " | All                  |
| Council Candidate             |   Yes   |       1       | 0, 10, \*b"vesting " |
| Councilor                     |   Yes   |       2       | 0, 10, \*b"vesting " |
| Validation                    |    No   | \*b"staking " | 0, 10, \*b"vesting " |
| Nomination                    |    No   | \*b"staking " | 0, 10, \*b"vesting " |
| Proposals                     |   Yes   |       ?       | 0, 10, \*b"vesting " |
| Storage Worker/Lead           |   Yes   |       ?       | 0, 10, \*b"vesting " |
| Distributor Worker/Lead       |   Yes   |       ?       | 0, 10, \*b"vesting " |
| Content Directory Worker/Lead |   Yes   |       ?       | 0, 10, \*b"vesting " |
| Forum Worker/Lead             |   Yes   |       ?       | 0, 10, \*b"vesting " |
| Membership Worker/Lead        |   Yes   |       ?       | 0, 10, \*b"vesting " |
| Builders Worker/Lead          |   Yes   |       ?       | 0, 10, \*b"vesting " |
| HR Worker/Lead                |   Yes   |       ?       | 0, 10, \*b"vesting " |
| Marketing Worker/Lead         |   Yes   |       ?       | 0, 10, \*b"vesting " |
| Invitation                    |   No\*  |       10      | 0...?                |
| Staking Candidate             |   Yes   |       ?       | 0, 10, \*b"vesting " |
| Bounty Entry                  |   Yes   |       ?       | 0, 10, \*b"vesting " |

\* It is not possible to initiation the invitation lock, it is automatically applied when a new member is invited on, hence the question of whether binding is required for applying the lock does not even apply.\
\*\* Vesting is only going to be setup for accounts originating from mainnet genesis block, and so by definition no binding would be needed for that.

## Reservation

WIP.

## Slashing

Slashing is the act of reducing the balance of an account by some amount, and also reducing the size of the lock which represents the use case under which the slashing occurs.

## State bloat

Some modules such as forum and working group allows user to call extrinsics that allows them to occupy storage. This can be a problem since some malicious users could use this to fill up the storage, either taking all the allotted space for that given storage map or claim storage space until no single node has enough storage. Furthermore, there is no incentive even for regular users to cleanup their storage use.

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
