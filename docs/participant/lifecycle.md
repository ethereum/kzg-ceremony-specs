# Participant API

## Rationale

The KZG trusted setup will benefit from have a diversity of approaches at all points in the application stack. We wish to encourage a variety of UI modules, covering multiple modes, hardware platforms, OS varieties, etc. This document is intended for those who wish to implement a client to perform the computation.

This API aims to be simple to implement yet sufficient to cope with the real-world variations that will be encountered.

## Reference

The REST end points noted in this document are described in detail here: [API Trusted Setup MPC](../../apiSpec/sequencerApi.yml)

## Overview

The participant client will follow a session life-cycle with these stages:

- [Landing](#landing)
- [Authentication & joining the lobby](#authentication--joining-the-lobby)
- [Periodically trying to contribute](#periodically-trying-to-contribute)
- [Computing contribution when asked to by the sequencer](#computing-contribution-when-asked-to-by-the-sequencer)
- [Post-contribution cleanup](#post-contribution-cleanup)

## Landing

The starting point for the client need only provide a mechanism to initiate login. Additional, optional data could be presented, including overall ceremony data, queue length, etc. Such data can be retrieved using the `/ceremony/status` end point.

Any time-consuming actions required to derive randomness should have been done right away (even before authentication), so that the contribution time is reduced to the minimum.

## Authentication & joining the lobby

Participants are required to identify themselves and pass some anti-sybil qualifying tests (see [Contributor Quaification](./contributorQualification.md)) 

The client must login using the `login` end point. A successful request will return a session token which must be passed in subsequent requests (except where noted).

The authentication token will remain valid for the duration of the session, i.e. until the client logs out, successfully contributes, or closes the web page. Users will not be able to log in again in the future. 

## Periodically trying to contribute

Successful authentication means that the participant joined the lobby and is required to call `/lobby/try_contribute` at specified intervals. Successful authentication starts the deadline timer.

When the participant calls `/lobby/try_contribute`, it can receive one of two responses:
* a contribution file. This starts the contribution timer. By the end of it, the participant needs to send their contribution to `/contribute` endpoint
* indication that they need to continue polling `/lobby/try_contribute` at specified interval

Failing to call `/lobby/try_contribute` in time (or `/contribute` when the participant was selected for contribution) will result in the participant being logged out and blacklisted from future contributions.

### Sequencer

The sequencer will track individual participants, their last check-in time and session id. The queue behavior is described in [Queue Strategy document](./queueStrategy.md).

## Computing contribution when asked to by the sequencer

At some point, the `/lobby/try_contribute` endpoint will return a contribution file. This means the participant has specified time to calculate their contribution (see [Participant document](../participant/participant.md)). Failing to do so in time results
in their contribution being rejected, and their account blacklisted and logged out.

The contributions will be sent using the `/contribute` endpoint where [sequencer will run validations](./sequencer.md#verification)
If the contribution is correct, the participant will receive a contribution receipt. In either case, the participant
will be blacklisted from future contributions.

In the event of error, the client might call `/contribution/abort` to cooperatively release their computation slot and allow
others to participate sooner.

## Post-contribution cleanup

Once the contribution has been returned, the client may perform additional tasks such as:

- Verification
- Capture and display of the contribution signatures.
- Submitting an attestation

These actions are not mandatory, although strongly recommended.

## Rate Limiting
- `login` requests will be limited to 50 per hour from a single IP address.
- Other requests without valid authentication will be limited to 100 per hour from a single IP address
- Other requests with valid authentication will be limited to 50 per minute from a single IP address