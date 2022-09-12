# Aims

The trusted setup’s queuing strategy should aim for:
- optimal throughput of contributions 
- minimising occurrences of failed or missed contributions
- simplicity of the API required 
- avoiding behaviour that could be interpreted as censorship

The KZG ceremony will be high-profile, and is expected to attract large numbers of willing, eligible contributors. In past ceremonies we have seen interest 
concentrate in the periods following an announcement such as a tweet by a prominent community member. It is desirable to convert that interest into
valid contributions. However, the ceremony requires a strict chain of contributions - one contribution cannot start until it has possession of the prior contribution. 


# Sequencing Protocol

## Authentication & contributor qualification

Participants must qualify by signing in. See [contributor qualification spec](./contributorQualification.md).

Once signed in, participants enter the lobby by calling `/lobby/try_contribute` for the first time. From this point,
the client (participant) should remain online and periodically (at frequency fixed by the sequencer)
call `/lobby/try_contribute` endpoint until the contribution is complete.

Since the sequencer needs to keep track of all participants in the lobby, the lobby size is capped. This prevents high memory usage and long processing times. Participants will not be able to enter when the lobby is full. Each participant can join the lobby at most once.

## Check-in requirements

Participants need to check in by calling `/lobby/try_contribute` at a specified interval (+/- some tolerance to accommodate for network or load issues). Failing to call the `/lobby/try_contribute` on time will remove participant from the lobby and they will be unable to contribute in the future.

If participant calls `/lobby/try_contribute` too frequently, additional calls are ignored and/or rate limit error is returned.

Sequencer maintains a list off all participants and their last check-in times. Every second inactive participants
(participants who failed to call in time) are purged (and blacklisted from joining in the future). When a new check-in arrives,
the sequencer checks if previous_timestamp + lower_bound >= current_timestamp. If not, that call is ignored and the timestamp is not updated. Together, those mechanisms ensure that malicious clients don't try to prevent others from contributing by saturating the sequencer.

## Obtaining a contribution slot

The sequencer listens to `/lobby/try_contribute` calls. When another contribution is already in progress, a `not_ready` response will be returned. Otherwise sequencer returns a contribution file to the client, marks the participant as the current contributor and starts the contribution timer.

A new contributor (participant) will be selected when:
- there is no contribution in progress (i.e. the previous participant finished contributing), or
- the currently selected participant failed to produce a contribution by the deadline (see [Participant Lifecycle](../participant/lifecycle.md)).

Contributors who stop calling `/lobby/try_contribute` (as detected by the sequencer) may not rejoin at a later stage. Contributors may contribute only once. 
Individuals will not be prevented from rejoining under a different ID, subject to passing the anti-sybil tests.

Submissions for all sub-ceremonies will be collected and submitted together. A valid contribution requires all 4 contributions to pass validity tests. (See [transcript JSON Schema spec](../../apiSpec/transcriptSchema.json)). 

The sequencer will ensure that no more than a single contributor is contributing at any given time. `Contributing` here refers to the full life-cycle:
- Passing the last valid transcript to the contributor
- Waiting for the computation to be performed
- Receiving the updated transcript
- Validation by the Sequencer
- Opening up a slot for the next participant who calls `/lobby/try_contribute`

## Contributing

Once the participant finishes computing their contribution, they call `POST /contribute` with the resulting file.
If sequencer determines the contribution was valid (see: [sequencer spec](./sequencer.md)), it returns a contribution receipt.
Otherwise an error is returned. In both cases the participant is blacklisted from joining the lobby and contributing again.

# Sequencer Handover

Handover from one sequencer to another will occur at infrequent, planned intervals over the time span of the ceremony. To facilitate handover, the sequencer is able to stop accepting new participants to the lobby, and to halt processing once the lobby is emptied (either by participants becoming offline or finishing their contributions).
