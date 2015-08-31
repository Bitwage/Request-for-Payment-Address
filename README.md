# Requesting payment details from a wallet

## The BIP 70 approach

This is for the use case where a user wishes to receive payment by supplying a third-party with a `PaymentRequest` as defined in BIP 70.

Proposed sequence

1. Payer (e.g. BitWage) creates a link on a website following the BIP 72 URI scheme and informs user: bitcoin:?r=https://example.com/users/abc123
2. User clicks on the link which triggers a wallet launch through standard protocol handlers in the browser
3. Wallet issues a GET request against the `r` link
4. Payer responds with a `PaymentRequest` marked as Version 2 and containing an optional BIP 70 extension field which is a `PaymentRequestSpecification` 
5. Wallet decodes the `PaymentRequestSpecification` to extract salient details (amount to be paid, maximum number of outputs, expiry time, signed with particular Bitcoin address, callback URL etc)
6. Wallet constructs a `PaymentRequest` in accordance with the `PaymentRequestSpecification` and POSTs it to the callback URL
7. Payer decodes the `PaymentRequest` and verifies it matches the `PaymentRequestSpecification`
8. Payer optionally creates a Bitcoin transaction to meet payment if occurring immediately
9. Payer issues `Payment` containing the transaction (or zero bytes if not present)
10. Wallet POSTs `PaymentACK` to indicate end of conversation

The above is just a rough guide to start a conversation. 

## The URI approach

This is for the use case where one app wishes to communicate directly with another and there is no direct channel available.
