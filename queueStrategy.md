# Aims

The trusted setup’s queuing strategy should aim for:
- optimal throughput of contributions 
- minimising occurrences of failed or missed contributions
- simplicity of the API required 
- avoiding behaviour that could be interpreted as censorship

The KZG ceremony will be high-profile, and is expected to attract large numbers of willing, eligible contributors. In past ceremonies we have seen interest 
concentrate in the periods following an announcement such as a tweet by a prominent community member. It is desirable to convert that interest into
valid contributions. However, the ceremony requires a strict chain of contributions - one contribution cannot start until it has possession of the prior contribution. 


# Queue Protocol

Participants must qualify by signing in. See [contributor qualification spec](./contributorQualification.md). Once signed in, they may join the queue. From this point,
the client should remain online until the contribution is complete. 

The queue will be maintained by the coordinator. The queue will advance whenever:
- a particpant completes their contribution, or
- is determined to have left the queue This will occur if the client fails to check in by the deadline (see [client API spec](./clientApiSpec.md)), or 
- has taken too long to submit their contribution.  

Contributors who leave the queue (and are detected as such by the coordinator) may not rejoin at a later stage. Contributors may contribute only once. 
Individuals will not be prevented from rejoining under a different ID, subject to passing the anti-sybil tests.

Submissions for all sub-ceremonies will be collected and submitted together. A valid contribution requires all 4 contributions to pass validity tests. (See [ transcript schema spec](./transcriptSchema.json)). 

The coordinator will ensure that no more than a single contributor is contributing at any given time. `Contributing` here refers to the full life-cycle:
- Passing the last valid transcript to the contributor
- Waiting for the computation to be performed
- Receiving the updated transcript
- Validation by the Coordinator
- Notifying the next contributor to begin

# Coordinator Handover

Handover from one coordinator to another will occur at infrequent, planned intervals over the time span of the ceremony. To facilitate handover, the coordinator is able to stop accepting new entries to the queue, and to halt processing once the queue is exhausted and the last contribution completely processed. 
