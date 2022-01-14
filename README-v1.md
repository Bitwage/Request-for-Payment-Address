(Notice: Both the URI and the BIP70 methods have a security vulnerability. For more information check issue [#1](https://github.com/Bitwage/address_request/issues/1))


# Request for Payment Address (v2)

Inspired by the 2015 "Request for Payment Address" by Tim Horton (tim@airbitz.co), Paul Puey (paul@airbitz.co), John Lindsay, & Aaron Voisine

## Motivation

Request for Payment Address allows for a fluid interaction between payer and payee
applications. The payer application requests a payment URI from a payee wallet. The
wallet generates a new payment URI and returns control back to the payer application.

## Specification

A Request for Payment Address is a URI which includes a return URI. The request URI is
launched via the payer application and is handled by the payees preferred wallet application.
The wallet then generates a payment URI and builds a bitcoin URI scheme (BIP 21). The
wallet then appends the bitcoin URI to the return URI via the addrparameter and then opens
the new response URI.

Due to the need for QR code scanning ability, this specification does NOT use the x-callback-url specification.


### URI Structure to Request Payment Address

`reqaddr://?[params…]`

To target specific wallets, `reqaddr` can be replaced by wallet specific URI prefixes such as `edge:`, etc. or an https deep link URI.

Example: `edge://?[params…]` or `https://deep.edge.app/reqaddr`

### Payer

request uri = `reqaddr://?` params

`params = param [ "&" params ]`

`param = [ racodes / rapost / raredir / rapayer / ]`

`racodes = "racodes=" *qchar` List of chain code and token codes separated by `-`. Use `_` to combine a chain code and token code pair. ie `ETH_USDC`. Token codes are optional if specifying the primary chain's coin. Example to specify a request for a bitcoin address, ethereum address, and USDC on the ethereum chain: `BTC-ETH-ETH_USDC`

`rapost = "rapost="` URL to POST the results of the address request

`raredir = "raredir="` URL to redirect user to after address request. This URL will also have address request results as URI params

`rapayer = "rapayer="` URI encoded payer name (optional)

Either `rapost` or `raredir` must be specified

Example

`reqaddr://?racodes=BTC-ETH-ETH_USDC&rapayer=Bitwage`

### Payee

If the `rapost` param is present, payee should send a POST API call to the `rapost` param URL with a JSON payload of the following format

```
{
  "[currency code]_[token code]": "[payment URI]",
  "[currency code]_[token code]": "[payment URI]",
  "[currency code]_[token code]": "[payment URI]"
}
```
where `payment URI` is a properly formed cryptocurrency public address or payment URI

Example:

```
{
  "BTC": "bitcoin:3MdvZezmNMspo5WmLLiBekwkHvKeiMKhgb?amount=0.0123&label=Satoshi%20Nakamoto&message=Thanks%20For%20Freedom",
  "ETH": "0x756ce3a5921b7a673b670f6de8572644ffc76152",
  "ETH_USDC": "0x18af1a7713dc3524519a7d1d3eab77828c1202c6"
}
```

If the `raredir` param is present, payee should redirect the user to the `raredir` param URL with the following format:

`[raredir]?[chaincode]_[tokencode]=[uri encoded payment URI]&[chaincode]_[tokencode]=[uri encoded payment URI]`

Example

`raredir=https://bitwage.com/getaddr`

Redirect to:

`https://bitwage.com/getaddr?BTC=bitcoin%3A3MdvZezmNMspo5WmLLiBekwkHvKeiMKhgb%3Flabel%3DSatoshi%2520Nakamoto%26message%3DThanks%2520For%2520Freedom&ETH_USDC=0x18af1a7713dc3524519a7d1d3eab77828c1202c6`

Wallets are strongly encouraged to require user interaction before sending a response to the requesting server. This will typically be a popup notifying the user that a website has requested a payment address, with `[OK]` and `[CANCEL]` buttons.

### XXX TODO: Full Examples 
