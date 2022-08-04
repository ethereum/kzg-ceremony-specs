# Aims

Our most important aim with the trusted setup is to avoid all practical possibility of a single actor controlling the contributions. A secondary aim is to avoid censoring anyone who wishes to contribute. 

Our chosen approach is to encourage a large number of participants, with the expectation 
that collusion, coercion, or interference is increasingly improbable with larger numbers of contributors. 
Having a low barrier to entry with regard to the technical requirements and time-and-attention requirements means we draw from a large pool of potential candidates.

A conflict arises between the above aims when we decide to defend against sybil operators using exclusionary criteria. 
We will inevitably exclude some interested and genuine participants who don’t meet the criteria. The overriding aim is 
to avoid attempts by malicious operators controlling large numbers of sybil identities. A balanced approach will allow 
a large number of unique contributors and allow the possibility of a small number of sybil identities. 

# Contribution Flow

* Contributors will log in using one of the accepted authentication methods
* The chosen identifier will be validated. Only valid identifiers will be able to progress to the following steps
* A JWT will be issued in response to the login request. 
* The JWT will be required in request headers for all subsequent requests. The token will be valid for a period of several hours. The contributor should not need to go through login again before
completing their contribution, provided they do not log off or close the page.

# Authentication Methods

* Ethereum accounts 
* GitHub 

## Common Criteria

Identifiers must not have previously submitted a contribution to this ceremony, nor attempted a submission. 

## Authentication by ETH Account.
### Identifiers: Eth accounts and ENS names

Participants will be identified using Ethereum accounts. We have no need to prohibit pseudonymous contributions, given that they have already passed anti-sybil checks.
However, well-known names may add to the credibility of the ceremony. For any address with an ENS name reverse-lookup configured, the ENS name will be used in place of the address. 

Identifiers will be added to the transcript and will become part of the permanent record of the ceremony. 

### Acceptance Criteria

The ceremony will entail a sign-in step for all contributors. The browser client will request that a message be signed, and will pass the address and signature to the server as a sign-in request.

The first attempt to sign in will trigger an assessment of the account’s eligibility.  
Eligible accounts will proceed to an ENS name lookup, and will then be registered by the server.
To pass the assessment, the account will require a **nonce ≥ 3** on mainnet at the time the assessment is done.

## Authentication via GitHub

### Acceptance Criteria

All GitHub accounts that pass GitHub OAuth will be accepted.

### Identifier
The GitHub handle will be used as the identifier in the ceremony contribution record. 


# Implementation Notes

The coordinator must remain responsible for evaluation accounts.

The planned handover of the coordinator role will entail continued access to the list of all accounts used thus far, along with their status. 

