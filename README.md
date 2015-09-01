# Requesting payment details from a wallet

## The BIP 70 (v2) approach 

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


### Security implications

There is no way for the user wallet to authenticate itself to the service i.e. the service cannot know if the wallet that contacted it belongs to the user.

A scenario to illustrate this problem:

- Alice wants to add a withdraw address to her online exchage
- The exchage website presents her a QR code that uses this BIP70 extension
- Mallory, that stands behind Alice, scans the QR code and contacts the exchange before Alice
- Alice sees a success message in the exchage interface and (probably) an error in the wallet
- Not giving it a second thought she clicks withdraw and the coins go to Mallory


Some solutions are:

- The wallet could generate a PIN number that the user should provice to the service. The service then contacts the wallet with the PIN and the wallet replies with the address. The downsides of this approach are that the service has to have a way of accepting the user input and that it requires the service to contact the wallet (not possible in stateless HTTP)
- The user manually checks their address. This has a bad usability and questionable security.
- The user pre-authenticates the wallet to the service, like providing the first address and the subsequent BIP70 calls are signed with it. This method does not allow one-off payments and is complicated to setup.



## The URI approach

This is for the use case where one app wishes to communicate directly with another and there is no direct channel available.

``bitcoin:?req-addressrequest=myscheme://myaction[&message=<message>][&req-network=<main|test>]``

e.g. bitcoin:?req-addressrequest=https://bitwage.com/user/alice&secret=ocVGmg5GQlCxkYKpppNSJLp0zVAvzM
> secret is to prevent frontrunning attack (double use)
> uri is used for server to know which user to set it for.

### EDITS
- bitcoin:  ->  bitcoin-ret: for user experience on unsupported wallets
- x-cancel- dont trust anything at all so dont want to go to that url - seems fine
- aaron- useability?  -leave x-error
- so: &x-success=
- &x-error=
- &x-source=Foldapp
- &category=Expense%3ACoffee

- ``bitcoin-ret://x-callback-url/request-address?``

##### for multiple currencies:
- bitcoin-ret changed to litecoin-ret
- request-address to request-litecoin-address

##### additional params:
- &default = 1
- &max-number =  (as many as it can but cannot exceed)
- and 0 is as many and no limit
- show me used?

#### addressrequest GET from wallet to address requester: 
``myscheme://myaction&address=<bip21_returnURI>``

e.g. https://bitwage.com/user/alice&secret=ocVGmg5GQlCxkYKpppNSJLp0zVAvzM&address=litecoin:3LgUNDLAnw2hHocP7ApPLfGmqXnjwaVDgN
> bip21 return uri used instead of raw address to tell server the coin type

address list can be implemented with a & separation in the bip21_returnURI:
e.g. https://bitwage.com/user/alice&secret=ocVGmg5GQlCxkYKpppNSJLp0zVAvzM&address=litecoin:3LgUNDLAnw2hHocP7ApPLfGmqXnjwaVDgN&3LgUNDLAnw2hHocP7ApPLfGmqXnjwaVDgS&3LgUNDLAnw2hHocP7ApPLfGmqXnjwaVDgW

### EDITS
multiple key value pairs- same key name (address) and different values
- e.g.: &address=litecoin:3LgUNDLAnw2hHocP7ApPLfGmqXnjwaVDgN&address=litecoin:3LgUNDLAnw2hHocP7ApPLfGmqXnjwaVDgS
- &x-source=wallet

### Security implications

Similar to the BIP 70 (v2) approach above.
