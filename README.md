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


# Request for Payment Address via URI Approach

Copied from "Request for Payment Address" by Tim Horton (tim@airbitz.co)
Modified by Paul Puey (paul@airbitz.co) per 2015-08-31 meeting with John Lindsay & Aaron Voisine

## Motivation

Request for Payment Address allows for a fluid interaction between payer and payee
applications. The payer application requests a payment address from a payee wallet. The
wallet generates a new payment address and returns control back to the payer application.

## Specification

A Request for Payment Address is a URI which includes a return URI. The request URI is
launched via the payer application and is handled by the payees preferred wallet application.
The wallet then generates a payment address and builds a bitcoin URI scheme (BIP 21). The
wallet then appends the bitcoin URI to the return URI via the addrparameter and then opens
the new response URI.

This specification is based off the x-callback-url specification. The URI must follow the format
described by RFC 3986. Bitcoin URIs must follow the format described by BIP 21.

### X-Callback-URI Structure

`[scheme]://x-callback-url/[action]?[params…]`

### URI Structure to Request Payment Address

`bitcoin-ret://x-callback-url/request-address?[params…]`

To target specific wallets, `bitcoin-ret` can be replaced by wallet specific URI prefixes such as `airbitz:`, `bread:`, `multibit:`, etc.

To make altcoin specific address requests, the `bitcoin-ret` prefix can be changed to various altcoin names such as `litecoin-ret:`. Should a specific wallet need to be targetted, the [action] parameter can be changed instead.

Example: `airbitz://x-callback-url/request-litecoin-address?[params…]`

### Grammar

#### Payer:

request-uri = `bitcoin-ret://x-callback-url/request-address?` params

`params = param [ "&" params ]`

`param = [ x-source / x-success / x-error / max-number / otherparam ]`

`x-source = "x-source=" *qchar`

`x-success = "x-success=" x-success-uri`

`x-error = "x-error=" x-error-uri`

`x-cancel = "x-cancel=" x-cancel-uri (DO NOT USE. Some security implications)`

`network = "network=" [main|test]` `main` is assumed if `network=` is omitted and therefore `network=test` should be the only option ever used.

`max-number = "max-number=" maximum number of addresses to return to Payer`

`max-number` is never guaranteed to be returned to Payer but Payee will not exceed `max-number` of addresses to be returned. If omitted, `max-number=1` is assumed. If `max-number=0`, wallet should return as many addresses as it can comfortably handle. Wallet should always return at minimum one address regardless of `max-number` value.

#### Payee:

`response-uri-success = x-success-uri "?address=" bitcoin-uri [ "&" bitcoinparams ]`

`response-uri-error = x-error-uri [ "?errorMessage=" *qchar ]`

Multiple addresses may be given by specifying multiple `address=` parameters on the x-success-uri

Please see BIP 21 for a definition of qchar​, bitcoinparams, and otherparams.

This spec also supports the optional `category=` param which specifies to the wallet a financial category which all incoming funds may be tagged with. ie. "Income:Wages". The `category=` param should have the format `[Category]:[Subcategory]`.

`[Category]` can only be one of `Income`, `Expense`, `Transfer`, or `Exchange`.

`[Subcategory]` can be a near unlimited set of various categories such as `Salary`, `Food & Dining`, `Clothing`, etc.. Category and Subcategory must be separated by a colon `:` and URI encoded.

Wallets are strongly encouraged to require user interaction before sending a response to the requesting server. This will typically be a popup notifying the user that a website has requested a payment address, with `[OK]` and `[CANCEL]` buttons.

### Example 1

#### 1. Payer creates return URLs.

Success: `https://bitwage.co/user/123`

Error: `https://bitwage.co/user/123/error`

#### 2. Payer issues a request for a payment address via a `bitcoin-ret:` URI encoding the return URIs. Additional params are included with the URI, in this case `x-source=` and `category=`.

  `bitcoin-ret://x-callback-url/request-address`
    `?x-success=https%3A%2F%2Fbitwage.co%2Fuser%2F123`
    `&x-error=https%3A%2F%2Fbitwage.co%2Fuser%2F123%2ferror`
    `&x-source=Bit%20Wage`
    `&max-number=2`
    `&category=Income%3ASalary`

#### 3. Payee receives the `bitcoin-ret:` with action of `request-address` and generates two payment address: 
  `1k5hxxtkjrzQV3Q8oAVdKzVcKEnE4EeSf`
  `1z4lkhgYD5Gjd24gHHR677flkj49gDdRe`

#### 4. Payee constructs two bitcoin URIs
  `bitcoin:1k5hxxtkjrzQV3Q8oAVdKzVcKEnE4EeSf`
  `bitcoin:1z4lkhgYD5Gjd24gHHR677flkj49gDdRe`

#### 5. Payee launches the success return URI including the `address=` parameters to the encoded bitcoin URIs. Payee also
includes a `x­source=` parameter back to the payer.
`http://bitwage.co/?user=123&x-source=Airbitz&address=bitcoin%3A1k5hxxtkjrzQV3Q8oAVdKzVcKEnE4EeSf&address=bitcoin%3A1z4lkhgYD5Gjd24gHHR677flkj49gDdRe`

#### 6. Payer Now can handle the return URI and extract the payment address via the `address=` parameters

### Example 2

#### 1. Payer creates a return URI
  Success: `foldapp:request?id=42`
  Error: `foldapp:request/error`

#### 2. Payer issues a request for a payment address via a `bitcoin-ret:` URI, encoding the return URIs.
  `bitcoin-ret://x-callback-url/request-address`
  
  `?x-success=foldapp%3Arequest%3Fid%3D42`

  `&x-error=foldapp%3Arequest%2Ferror`

  `&x-cancel=foldapp%3Arequest%2Fcancel`

  `&x-source=Foldapp`

  `&category=Expense%3ACoffee`

  `&notes=Refund`

3. Payee handles the `bitcoin-ret:` URI and generates a payment address
  `1V8h9kUDXtBEtWwcWRqd5s9A5pxjjdTFy`

4. Payee constructs a bitcoin URI
  `bitcoin:1V8h9kUDXtBEtWwcWRqd5s9A5pxjjdTFy&label=Nakamoto`

5. Payee issues content of `bitcoin-ret` appending a parameter of `address=`
  `foldapp:request?id=42`

  `&x-source=Airbitz`

  `&address=bitcoin%3A1V8h9kUDXtBEtWwcWRqd5s9A5pxjjdTFy%26label%3DNakamoto`

6. Payee Now can now handle the return URI and extract the payment address via the `address=` parameter

