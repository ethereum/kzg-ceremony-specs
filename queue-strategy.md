# Aims

The trusted setup’s queuing strategy should aim for:
- optimal throughput of contributions 
- minimising occurrences of failed or missed contributions
- simplicity of the API required 


The KZG ceremony will be high-profile, and is expected to attract large numbers of willing, eligible contributors. In past ceremonies we have seen interest 
concentrate in the periods following an announcement such as a tweet by a prominent community member. It is desirable to convert that interest into
valid contributions. However, the ceremony requires a strict chain of contributions - one contribution cannot start until it has possession of the prior contribution. 


# Queue Protocol

Participants must qualify by signing in. See [contributor identfication spec](./contributor-identification.md). Once signed in, they may join the queue. From this point,
the client should remain online until the contribution is complete. 

The queue will be maintained by the coordinator. The queue will advance whenever a user completes their contribution or is determined to have left the queue.
The latter will occur as a result of the client failing to check in at the required time. 

Contributors who leave the queue may rejoin at a later stage. Contributors may contribute only once. 
Individuals will not be prevented from rejoining under a different ID, subject to passing the anti-sybil tests.

Transcripts for all sub-ceremonies will be collected and submitted together. A valid contribution requires all 4 contributions to pass validity tests.

The coordinator will ensure that no more than a single contributor is contributing at any given time. `Contributing` here means the full life-cycle:
- Passing the last valid transcript to the contributor
- Waiting for the computation to be performed
- Receiving the updated transcript
- Validation by the Coordinator
- Notifying the next contributor to begin
