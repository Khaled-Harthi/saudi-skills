# Payment Errors

## Gateway Response Codes

When `status === 'failed'`, access error info via:
- `source.message` - Human-readable message
- `source.response_code` - Gateway response code (string)

- Check this web page for full list of response codes [Gateway Response Codes](https://docs.moyasar.com/guides/references/gateway-response-codes)
- Check this web page for full list of error message translations [Payment Errors](https://docs.moyasar.com/guides/references/payment-errors)