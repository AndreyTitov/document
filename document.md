# Integration process

1. [Overview](#overview)
2. [Integration credentials](#integration-credentials)
3. [Integration Process](#integration-process)
    - [Sequence of steps](#sequence-of-steps)
    - [Integration diagram](#integration-diagram)
    - [Check before starting to work in Prod Environment](#check-before-starting-to-work-in-prod)
    - [Sequence diagram for deposit](#sequence-diagram-for-deposit)
    - [Sequence diagram for payout](#sequence-diagram-for-payout)      
4. [Getting all available payment methods](#get-info-by-payment-methods)
5. [Initiate Transaction](#initiate-transaction)
    - [Deposit Transaction](#deposit-transaction)
    - [Payout Transaction](#payout-transaction)
6. [Get Transaction Status](#get-transaction-status)       
7. [Transaction Status Callback](#transaction-status-callback)
    - [Transaction Status Callback for deposit](#transaction-status-callback-for-deposit)
    - [Transaction Status Callback for remit](#transaction-status-callback-for-remit)
8. [How to make a Refund](#how-to-make-a-refund)
9. [Getting merchant account balance via API](#how-to-check-balance-merchant-cabinet)
10. [Get rates](#get-rates)




<u>Your merchant personal credentials will be sent to you in a separate file.</u>

## Overview {#overview}

This document describes the interaction between parties to make deposits and remits.
There are several parties involved in the Host-to-Host flow process:

1. **Customer** -  who wants to pay or get a payout, your client
2. **Merchant** - your company
3. **H2H-Backend** - APS backend

**There are two modes possible:**

1. Deposit flow
2. Remit flow

## Integration credentials {#integration-credentials}

During the onboarding process you will receive credentials to access APS API for staging and later for production environments.

- Merchant GUID - unique merchant identifier, used in API endpoint path as `merchantGUID`

- App Token - authentication token used to access API, passed in `X-App-Token` header of API requests

- App Secret - authentication secret used to access API, passed in `X-App-Secret` header of API requests

- Callback secret key - key to validate callback requests from APS, status callback contains request signature

- API URL - url used to access API, mentioned as `H2H_BACKEND` in the examples
    - Staging environment: https://fpf-api.armenotech.net

    - Production environment: https://fpf-api.proc-gw.com

**The values will differ for Staging and Production environments.**

## Integration Process {#integration-process}

### Sequence of Steps

1. Merchant (you) and APS (we) make agreements about payment methods, countries, and the limits of transfers. You can ask your account manager about limits and GEO.

2. We send you credentials for Stage Environment through a one-time link via email.

Before issuing a request, ensure that the set of tokens for your merchant guid matches the provided credentials. Please check these credentials before making a request to the APS team.

3. You are studying this document and make tests:

  - go to [Getting all available payment methods](#get-info-by-payment-methods). Make this request and in the response you will get an array of payment methods available to you. In each method, the payment currency, fees, and limits are described in details.

  - Make 1 or 2 test requests and test refund. You can learn [How to make a Refund](#how-to-make-a-refund).

4. If all tests are done and all documents are signed, you make the necessary settings and we send you credentials for Prod Environment.

The credentials for Prod Environment differ from the credentials for Stage Environment. 

The requests will work the same way. We will send you the document for the Prod Environment and all the necessary links and guides will be with you.

5. Now you have access to Merchant Portal, where you can login using the email (on that email you gave received the credentials).

How to work with Merchant Portal you can learn from the Merchant Portal Guide.

### Integration Diagram

![integration steps](steps.png)

### Check-list before Go Live {#check-before-starting-to-work-in-prod}

1. Make sure that your operations team has access to the `external_message` field in the callback - this is where you will find the reason for the failure. This will significantly reduce the number of requests to our support portal.

2. Before launching traffic, notify us at least 1 day in advance; do not launch traffic without our approval.

3. After integration is done you can go to Support Portal, where the operational team helps you to solve the problems in Production. You will be given the access to our support portal: to get this access, you need to send to your account manager emails of your employees. For all questions about live traffic, you will need to contact this portal. Before writing there, use this link to find out reasons of errors: 

  - [Portal](https://armenotech.atlassian.net/servicedesk/customer/kb/view/959840305) for deposit transactions

  - [Portal](https://armenotech.atlassian.net/servicedesk/customer/article/1311867029) for remit transactions.

### Sequence Diagram for Deposit {#sequence-diagram-for-deposit}

![Sequence Diagram for Deposit](Sequence_Diagram_For_Deposit.png)

The happy-path flow is as follows:

1. A customer initiates a deposit request via a specific payment method, which could be accessed through a designated button or link on the merchant's website or application.

2. The merchant provides the transaction data to the payment system.

3. The PSP sends the how link to the Merchant.

4. Merchant redirects the customer to the payment form.

5. Upon successful deposit registration, the H2H redirects the customer back to the merchant's website.

6. The PSP processes the payment and sends an HTTP callback to the merchant with the transaction result. See [Get Transaction Status](#step3) chapter for details.

### Sequence Diagram for Payout {#sequence-diagram-for-payout}

![Sequence Diagram for Payout](Sequence_Diagram_For_Payout.png)

The happy-path flow is as follows:

1. The customer initiates a payout request using a specific payment method, accessible through a designated button or link on the merchant's website or application.

2. The merchant provides the transaction data to the payment system.

3. Upon successful payout registration, the PSP redirects the customer back to the merchant's website.

4. The PSP processes the payout and sends an HTTP callback to the merchant with the transaction status. See [Get Transaction Status](#step3) chapter for details.

## Getting all available payment methods {#get-info-by-payment-methods}

The `GET /info` request retrieves information about the available payment methods and payment details.
By making  this request, you can get all the necessary information about the connected payment methods.


Example request:

export function GetInfoByPaymentMethod() {
	const {siteConfig} = useDocusaurusContext();
    const { search } = useLocation();
    const urlParams = new URLSearchParams(search);
    const merchantGUID = urlParams.get('merchant_guid');

  return (
      <CodeBlock
        language="bash"
        title="Get payment methods">
        {
        `curl ${siteConfig.customFields.FPF_URL}/api/v3/merchantGUID/info \\
        --header 'X-App-Token: APP_KEY'
        --header 'X-App-Secret: APP_SECRET'
        `}
      </CodeBlock>
  );
}

 <GetInfoByPaymentMethod/>


Example response:
```
  {
    "methods": [
      {
        "guid": "c8e18874-ce90-4a6a-86cd-bcd17f815476:d93717d0-86d0-435a-8191-ea1b43d04a96",
        "fee_fixed": 0,
        "fee_percent": 0,
        "customer_fee_percent": 0,
        "customer_fee_fixed": 0,
        "min_amount": 1.43,
        "max_amount": 570.72,
        "label": "EUR",
        "logo_url": "https://fpf-assets.armenotech.com/mastercard.svg",
        "mobile_logo_url": "https://fpf-assets.armenotech.com/mastercard.svg",
        "payment_group": "BANKCARD",
        "payment_group_name": "Bank card",
        "digits_delivery": 2,
        "digits_asset": 2,
        "rate": 1.4,
        "delivery_currency": "iso4217:EUR",
        "method_currency": "BRL",
        "method_symbol": "R$",
        "country_code": "WWC",
        "direction": "in"
        "deposit": {
          "guid": "c8e18874-ce90-4a6a-86cd-bcd17f815476:d93717d0-86d0-435a-8191-ea1b43d04a96",
          "fields": {
            "external_id": {
              "optional": true,
              "description": "Merchant’s external ID",
              "hidden": true
            },
            "redirect_url": {
              "optional": true,
              "description": "url where a customer will be redirected with parameters status=[successful|failed|canceled], transaction_id, method, amount, currency",
              "hidden": true,
            },
            "status_callback_url": {
              "optional": true,
              "description": "Callback URL",
              "hidden": true
            }
          },
        }
      }
     ]
 }
 ```

The response is an array of payment methods available to you. In each method, the payment currency, fees, and limits are described in detail.

| Name                 | Description                                                  |
|----------------------|--------------------------------------------------------------|
| `guid`                 | Payment method guid                                         |
| `fee_fixed`            | Fixed fee according to the contract with the merchant        |
| `fee_percent`          | Percent fee according to the contract with the merchant      |
| `customer_fee_percent` | Percent fee to be paid by an end user according to the contract with the merchant |
| `customer_fee_fixed`   | Fixed fee to be paid by an end user according to the contract with the merchant |
| `min_amount`           | Minimum possible amount per transaction                      |
| `max_amount`           | Maximum possible amount per transaction                      |
| `label`                | Method Label                                                 |
| `logo_url`             | Method Logo                                                  |
| `mobile_logo_url`      | Mobile method logo                                           |
| `payment_group`        | Additional information on the method                          |
| `payment_group_name`   | Additional information on the method                          |
| `digits_delivery`      | Number of significant digits for delivery currency            |
| `digits_asset`         | Number of significant digits for asset                        |
| `rate`                 | Conversion rate of asset to delivery currency                 |
| `delivery_currency`    | Currency code                                                |
| `method_currency`      | The currency in which all amounts are calculated using this method |
| `method_symbol`        | Currency symbol (may be missing) to display to the client     |
| `country_code`         | Country code                                                 |
| `direction`            | Two input/output values are possible. They say in which direction the payment takes place in = deposit, out = remit |
| `deposit/remit`        | The object containing the fields of the payment method is called deposit/remit |
| `fields`               | An object that lists the fields required for transfer in each payment method. Within the same payment method, the fields may also change, we recommend that you make a request /info before paying and collect the fields every time |
| `field.optional`             | If true, the parameter is not required to be filled in        |
| `field.description`          | Description of the field                                     |
| `field.hidden`               | Recommendation whether to hide this field from customer      |


## Initiate transaction {#initiate-transaction}
### Deposit transaction {#deposit-transaction}

In the example request you can see how to make deposit transactions:

Example request:

export function InitiateTransaction() {
	const {siteConfig} = useDocusaurusContext();
    const { search } = useLocation();
    const urlParams = new URLSearchParams(search);
    const merchantGUID = urlParams.get('merchant_guid');

  return (
      <CodeBlock
        language="bash"
        title=" Initiate transaction">
        {
        `curl POST ${siteConfig.customFields.FPF_URL}/api/v3/merchantGUID/transactions \\
--header 'Content-Type: application/json'
--header 'X-App-Token: {APP_KEY}'
--header 'X-App-Secret: {APP_SECRET}'
--data-raw '{
      "amount": 100.2,
      "fields": {
          "transaction": {
              "deposit_method": "06d34870-a9d3-4baa-afce-995c1f385d1d:81180240-65f1-4ed0-9f1f-be1e0b4fa985",
              "deposit": {
			"redirect_url": "https://www.google.com",
			"status_callback_url": "https://fast-payment-flow-merchant.armenotech.dev/94b9f3ed-ee84-4789-8c4b-41ee8c61ad41/transactions",
			"external_id": "external transaction identifier, string",
			"payer_id": "external payer identifier, string",
			"customer_ip_address": "93.109.244.86",
                        "from_country": "GB",
			"billing_street": "Test street",
			"billing_town": "London",
			"billing_post_code": "0839",
			"from_email": "customer@gmail.com"
              }
          }
      }
  }'
        `}
      </CodeBlock>
  );
}

 <InitiateTransaction/>


Request to create a transaction, it is important to observe the correct nesting in the request body.

| Name                                      | Description                                                                                             |
|-------------------------------------------|---------------------------------------------------------------------------------------------------------|
| `amount`                                    | Transaction amount body                                                                                 |
| `deposit_method` | Payment method guid                                                                                 |
| `deposit`      | An object containing fields by transaction                                                              |
| `status_callback_url`                       | An URL where callback requests will be sent on a transaction changes status.                              |
| `redirect_url`                              | A URL where a customer will be redirected upon deposit completion. This fields is only applicable to deposit transactions. Some payment methods do not support redirection.                                                          |
| `external_id`                               | A Merchant’s ID of a transactions. It is a string up to 96 character long. There is not requirement external_id to be a unique value across all merchant’s transaction.               |
| `payer_id`                                  | A Payer's identifier in Merchant's internal system.It is a string up to 96 character long.It is not required payer_id to be unique accross merchant's transactions.                                 |
| |   Optional parameters, but we strongly recommend to fill in them also. They are very important for conversion.| 
| `customer_ip_address`                               | ip-address of customer               |
| `from_country`                               | country from what you are making the transaction               |
| `billing_street`                               | client’s street that he provided to the bank while opening his bank account               |
| `billing_town`                               | client’s town that he provided to the bank while opening his bank account               |
| `billing_post_code`                               | client’s postcode that he provided to the bank while opening his bank account               |
| `from_email `                              | 	client’s email that he provided to the bank while opening his bank account               |


Example deposit response:

```
  {
      "id": "1a2d87dc-3a6e-48dd-be69-8ad6492cc8b4",
      "amount": 1000,
      "amount_in": 1000,
      "customer_fee": 0,
      "contract_fee": 0,
      "merchant_fee": 0,
      "amount_out": 1000,
      "amount_body": 1000,
      "how": "https://cert.monnetpayments.com/api-payin/payment/voucher/27685?verification=4bdf290c8145af72f75c7fcefb99f33f5ac29eac7071d5dacf03e518e895a4f3d8e91dc803af10195f00ee0c58d7e4c6091ed2c8cfaa631d953bcae250a132d0",
      "extra_info": {
          "displayed_fields": [
              "is_redirect"
          ],
          "external_transaction_id": "1a2d87dc-3a6e-48dd-be69-8ad6492cc8b4",
          "is_redirect": true,
          "transaction_id": "1a2d87dc-3a6e-48dd-be69-8ad6492cc8b4"
      }
  }
   ```
In response you will get the following:

| Name                                      | Description                                                                                             |
|-------------------------------------------|---------------------------------------------------------------------------------------------------------|
| `id`                                    | Transaction identifier                                                                                |
| `amount` | Transaction amount                                                                                 |
| `amount_in`      | Amount that a customer pays                                                              |
| `customer_fee`                                    | Fee amount charged from customer according to merchant contract                                                                                |
| `contract_fee` | Fee amount charged by APS according to merchant contract                                                                                |
| `merchant_fee`      | Calculated fee according to merchant contract. Fee held by merchant, can be negative depending on merchant's wish to compensate customer fee.                                                              |
| `amount_out`                                    | Amount the APS settles to the merchant’s account                                                                                |
| `amount_body` | Amount of the payment without commission                                                                                 |
| `amount_in`      | Amount that a customer pays                                                              |
| `how`                                    | External URL to your checkout form. Merchant application redirects customer to external system to confirm and execute payement.                                                                                |
| `external_transaction_id` | Transaction identifier in external system. For Binance method element contains APS transaction identifier.                                                                                |
| `is_redirect`      |                                                               |
| `merchant_external_id`      |          Merchant’s ID of a transactions received in /transactions request as `external_id`                                                     |

**Note**: If you get a response that is not 200 OK, you can ask the integration team what is the problem.

   
#### Handling the "how" field

##### Redirect to checkout form

Redirect to checkout form is used for methods required by the customer to complete payment on a web form. In this case `how` filed contains a link to the web form where customer provides payment details.

Merchant application should redirect customer to checkout form using URL recieved in `how` element of `/transactions` response.

For example:  `https://app.payment.com/payment/secpay?linkToken=793c9fb78be54d898a041d370303c8c7&_dp=Ym5jOi8vYXBwLmJpbmFuY2UuY29tL3BheW1lbnQvc2VjcGF5P3RlbXBUb2tlbj1CYVpqdmZoWDI0NUcyaWF5UGV0TVFReURMeWF6SnM2SSZyZXR1cm5MaW5rPWh0dHBzOi8vc3RhZ2UtNWNjODdlZjgtY2NkNy00Nzc0LTg2ZTAtNDViNTE1MzQ5MjVjLXBheW1lbnRkcml2ZXIuYXJtZW5vdGVjaC5uZXQvcmVkaXJlY3Q_dHJhbnNhY3Rpb25fZ3VpZD05NDk0NmM2Ny0zODk1LTRiYjEtODg4Yi1lNzNlNzM0YWNmNDYmY2FuY2VsTGluaz1odHRwczovL3N0YWdlLTVjYzg3ZWY4LWNjZDctNDc3NC04NmUwLTQ1YjUxNTM0OTI1Yy1wYXltZW50ZHJpdmVyLmFybWVub3RlY2gubmV0L3JlZGlyZWN0P3RyYW5zYWN0aW9uX2d1aWQ9OTQ5NDZjNjctMzg5NS00YmIxLTg4OGItZTczZTczNGFjZjQ2`

You will need to take the “how” link from the response and redirect customer to checkout form:

![hosted page](hosted_page.png)

 

The customer will fill in the card details.

##### Direct post of bank card data

APS bank card payment method, that supports direct post of payment details: card number, cardholder name, expiry date and cvv/cvc. For APS bank card deposit method make POST request to how with customer’s payment details.

When payment method is bank card, you can transfer payment details to us.

```json
Content-Type: application/json
```

Request example:

export function AutoVars() {
const {siteConfig} = useDocusaurusContext();
const PAYMENT_FORM_URL = siteConfig.customFields?.['PAYMENT_FORM_URL'];


return (
    <CodeBlock>
{` curl '${PAYMENT_FORM_URL}/payment/TRANSACTION_ID/merchantGUID' \  
  -H 'Content-Type: application/json' \  
  --data-raw '{
    "number": "4111111111111111",
    "holder": "John Doe",
    "exp_month": 10,
    "exp_year": 25,
    "csc": "010"
  }
    

`}
    </CodeBlock>
);
}

 <AutoVars/>



Request fields:

| Name       | Type           | Description                    |
|------------|----------------|--------------------------------|
| number     | string         | Bank card number               |
| holder     | string         | Card holder                    |
| exp_month  | int            | Month of expiration date       |
| exp_year   | int (last 2 digits) | Year of expiration date    |
| csc        | string (3 digits) | CVC/CVV code                 |
| from_api   | boolean        | If `false` then a customer will be redirected to the 3ds page,If `true` then the link to the 3ds page will be returned as a `auth_3d_url` attribute of the response. |

Success response example:
```
200
{
  "status": "success",
  "auth_3d_url": "https://some3ds.com"
}
 ``` 
| Name        | Type               | Description                                                           |
|-------------|--------------------|-----------------------------------------------------------------------|
| `auth_3d_url` | string (url, optional) | 3DS authorisation url where a customer should be redirected to.        |

Fail response example:

```
400
{
  "status": "fail",
  "retryable": false,
  "external_status": "something_went_wrong",
  "external_message": "We weren't able to process your transaction."
}
  ```

**Possible cases**


| Case                                         | Status code | Retryable | Example response                                                                                                                 |
|----------------------------------------------|-------------|-----------|----------------------------------------------------------------------------------------------------------------------------------|
| Request success and transaction went with 3Ds flow. | 200         | false     | ```{ "auth_3d_url": "https://example.com/auth_3d_url", "redirect_url": "https://example.com/redirect_url", "status": "success" }```    |
| Request success and 2d flow                  | 200         | false     |``` { "auth_3d_url": "https://example.com/auth_3d_url", "redirect_url": "https://example.com/redirect_url", "status": "success" }   ``` |
| Invalid order state                          | 400         | false     |``` { "auth_3d_url": null, "redirect_url": "", "status": "fail", "retryable": false, "external_status": "something_went_wrong", "external_message": "We weren't able to process your transaction." } ```   |
| Bad request (Invalid request format)         | 400         | false     | ```{ "auth_3d_url": null, "redirect_url": "", "status": "fail", "retryable": false, "external_status": "something_went_wrong", "external_message": "We weren't able to process your transaction." }   ``` |
| Card holder is empty                         | 400         | true      | ```{ "auth_3d_url": null, "redirect_url": "", "status": "fail", "retryable": true, "external_status": "card_holder_is_empty", "external_message": "Card holder field is empty." } ```   |
| Card expired                                 | 400         | true      | ```{ "auth_3d_url": null, "redirect_url": "", "status": "fail", "retryable": true, "external_status": "card_is_expired", "external_message": "Your card has expired." } ```   |
| Invalid expire month                         | 400         | true      |``` { "auth_3d_url": null, "redirect_url": "", "status": "fail", "retryable": true, "external_status": "invalid_card", "external_message": "Invalid card details." }  ```  |
| Invalid CSC                                  | 400         | true      |``` { "auth_3d_url": null, "redirect_url": "", "status": "fail", "retryable": true, "external_status": "invalid_card", "external_message": "Invalid card details." } ```   |
| Invalid card number length                   | 400         | true      |``` { "auth_3d_url": null, "redirect_url": "", "status": "fail", "retryable": true, "external_status": "invalid_card_number_length", "external_message": "Invalid card number length." }  ```  |
| Invalid card number check digit value        | 400         | true      | ```{ "auth_3d_url": null, "redirect_url": "", "status": "fail", "retryable": true, "external_status": "invalid_card", "external_message": "Invalid card details." }  ```  |
| Service Error                                | 400         | false     |``` { "auth_3d_url": null, "redirect_url": "", "status": "fail", "retryable": false, "external_status": "something_went_wrong", "external_message": "We weren't able to process your transaction." }   ``` |
| Provider Error (Error while processing request) | 400         | false     |``` { "auth_3d_url": null, "redirect_url": "", "status": "fail", "retryable": false, "external_status": "external_error", "external_message": "Invalid response from bank." }  ```  |
| Visa not allowed                             | 400         | false     | ```{ "auth_3d_url": null, "redirect_url": "", "status": "fail", "retryable": false, "external_status": "visa_not_allowed", "external_message": "Bank card type Visa is not allowed." }  ```  |
| Card user not authorized for payment         | 400         | false     |``` { "auth_3d_url": null, "redirect_url": "", "status": "fail", "retryable": false, "external_status": "verification_error", "external_message": "Payment verification has failed." }  ```  |
| Insufficient funds                           | 400         | false     |``` { "auth_3d_url": null, "redirect_url": "", "status": "fail", "retryable": false, "external_status": "insufficient_funds", "external_message": "Insufficient funds on the card balance." }  ```  |

**Note**: you don't need "how" field for remit payment, because you have given us the card number on initiate step.

If you have made a request successfully you can find out the transaction status in two ways:

- Waiting for the callback (you can learn information about the callbacks [here](#step4)).

- Making a request about status transaction (you can learn information about the requests [here](#step3)).


### Payout transaction {#payout-transaction}

Payout is called “remit” in our requests.

**Note**: to make payout transaction you should have funds on your balance. Remember to top up your balance by the financial chat. We will give you an access to this chat.

**Note**: to make  payout transaction on Staging Environment we will top up the your balance.

To be sure about all required fields use the [Getting all available payment methods](#get-info-by-payment-methods)

Example of bank card payout transaction request:

export function InitiateTransactionPayout() {
	const {siteConfig} = useDocusaurusContext();
    const { search } = useLocation();
    const urlParams = new URLSearchParams(search);
    const merchantGUID = urlParams.get('merchant_guid');

  return (
      <CodeBlock
        language="bash"
        title=" Initiate transaction">
        {
        `curl POST ${siteConfig.customFields.FPF_URL}/api/v3/merchantGUID/transactions \\
--header 'Content-Type: application/json'
--header 'X-App-Token: {APP_KEY}'
--header 'X-App-Secret: {APP_SECRET}'
--data-raw '{
      "amount": 100.2,
      "fields": {
          "transaction": {
              "remit_method": "06d34870-a9d3-4baa-afce-995c1f385d1d:81180240-65f1-4ed0-9f1f-be1e0b4fa985",
              "remit: {
			"external_id": "merchant_external_id_string",
			"payer_id": "merchant_payer_id_string",
			"customer_ip_address": "93.109.244.86",
			"referer_domain": "www.externalwebsite.com",
			"redirect_url": "https://www.yourswebsite.com/redirect",
			"status_callback_url": "https://www.yourswebsite.com/callback",
			"from_country": "USA",
			"to_bank_card": "2222444422224444",
			"to_bank_card_exp": "12/31"
              }
          }
      }
  }'
        `}
      </CodeBlock>
  );
}

 <InitiateTransactionPayout/>


Request to create a transaction, it is important to observe the correct nesting in the request body.

| Name                                      | Description                                                                                             |
|-------------------------------------------|---------------------------------------------------------------------------------------------------------|
| `amount`                                    | Transaction amount body                                                                                 |
| `remit_method` | Payment method guid                                                                                 |
| `remit`      | An object containing fields by transaction                                                              |
| `status_callback_url`                       | An URL where callback requests will be sent on a transaction changes status.                              |
| `redirect_url`                              | A URL where a customer will be redirected upon deposit completion. This fields is only applicable to deposit transactions. Some payment methods do not support redirection.                                                          |
| `external_id`                               | A Merchant’s ID of a transactions. It is a string up to 96 character long. There is not requirement external_id to be a unique value across all merchant’s transaction.               |
| `payer_id`                                  | A Payer's identifier in Merchant's internal system.It is a string up to 96 character long.It is not required payer_id to be unique accross merchant's transactions.                                 |

Example of payout transaction response:

```
{
      "id": "dfba9a85-464f-4f88-9afc-865296bca4ef",
      "amount": 103.7,
      "amount_in": 103.7,
      "customer_fee": 0,
      "contract_fee": 3.7,
      "merchant_fee": 3.7,
      "amount_out": 100,
      "amount_body": 100,
      "extra_info": null
}
   ```


## Get Transaction Status {#get-transaction-status}

You can request the transaction status via API by transaction id.

Example request:

export function GetStatusTransaction() {
	const {siteConfig} = useDocusaurusContext();
    const { search } = useLocation();
    const urlParams = new URLSearchParams(search);
    const merchantGUID = urlParams.get('merchant_guid');

  return (
      <CodeBlock
        language="bash"
        title="Get Status Transaction">
        {
        `curl ${siteConfig.customFields.FPF_URL}/api/v3/merchantGUID/TRANSACTION_ID \\
        --header 'X-App-Token: APP_KEY'
        --header 'X-App-Secret: APP_SECRET'
        --header 'Content-Type: application/json'
        `}
      </CodeBlock>
  );
}

<GetStatusTransaction />


Example response:

```
{
    "id": "f2a86886-675c-4f55-a247-757a4ef55e53",
    "external_id": "test 2434",
    "status": "completed",
    "refunded": true,
    "amount": 3,
    "amount_in": 3,
    "amount_out": 3,
    "amount_fee": 0,
    "amount_body": 3,
    "customer_fee": 0,
    "merchant_fee": 0,
    "external_message": "APPROVED",
    "refunds": {
        "amount_refunded": "3",
        "amount_fee": "0",
        "payments": [
            {
                "id": "ee9de236-6f6a-48b7-b1ad-bb496e6ba5dc",
                "status": "completed",
                "amount": 3,
                "rrn": "688447"
            }
        ]
    }
}
```

A JSON object containing the following fields:

| Name                 | Type          | Optional | Description                                       |
|----------------------|---------------|----------|---------------------------------------------------|
| `id`                   | `UUID`          | `true`     | Transaction ID                                    |
| `merchant_external_id` | `string`        | `true`     | External transaction ID                          |
| `status`               | `string enum`   | `true`     | status of transaction |
| | `pending_sender`: waiting for a customer to make a payment. | |
| | `pending_external`: 1. a customer has made a payment and the transaction is on the AML-check. Most likely, it will pass. 2. a merchant has initiated a refund and a transaction is pending for a refund to be completed. | |
| | `completed`:  transaction has been completed. | |
| | `error`:  transaction error. | |
| `amount`               | `float`         | `false`    | Amount from the request                          |
| `refunded`               | `bool`         | `true`    | Indicates whether a transaction was refunded.                          |
| `amount_in`            | `float`         | `true`     | Sender amount. For deposit transactions, this value indicates the amount the customer has paid in the contract currency unit. For payout transactions, this value indicates the amount the merchant has charged to execute the transaction.                                   |
| `amount_out`           | `float`         | `true`     | Receiver amount. For deposit transactions, this value indicates the amount credited to the merchant. For payout transactions, this value indicates the amount credited to the customer.                                   |
| `amount_fee`           | `float`         | `true`     | Fee according to contract                        |
| `customer_fee`         | `float`         | `true`     | Calculated fee to be paid by a customer          |
| `merchant_fee`         | float         | true     | Calculated fee paid or earned by a merchant. If value is positive then the merchant should pay; if the value is negative then the merchant benefits. |
| `refunds.amount_refunded`           | `float`         | `true`     | Amount to refund                        |
| `refunds.amount_fee`           | `float`         | `true`     | Fee according to contract                        |
| `refunds.payments[?].id`           | `UUID`         | `true`     | Refund id                        |
| `refunds.payments[?].status`           | `float`         | `true`     | status                        |
| | `pending`: we have sent a request to the acquiring bank to initiate a refund. After receiving feedback on the request, the status of the refund will be updated. You can monitor the update of the refund status in your merchant back office. | |
| | `completed`: transaction has been fully refunded. | |
| | `rejected`:  refund has been rejected. | |
| `refunds.payments[?].amount`           | `float`         | `true`     | Amount to refund + fee according to contract                        |
| `refunds.payments[?].rrn`           | `string`         | `true`     | Retrieval Reference Number - identifier of refund from payment provider                         |



## Transaction Status Callback {#transaction-status-callback}

**Before going live, make sure** your operations team gets access to the external reason that we send you in the callback in the `external_message` field. This is necessary to understand the reasons for rejected transactions.

The transaction status callback is used to notify merchants about the transaction status after a payment operation.

PSP requests merchant’s system by sending a request to endpoint provided in /transactions request instatus_callback_url field. The merchant's system should respond with a 200 OK status.

### Transaction Status Callback for deposit {#transaction-status-callback-for-deposit}

Example response:

```
  "payload": {
    "amount_body": 500,
    "amount_in": 500,
    "amount_out": 475.5,
    "asset": "stellar:APSUSDM:GB7OUO5NY5WQKXJJ7PFFZEJOKN4BA7IOEN3Z6SWAY26LGTREJJYZH2ZT",
    "delivery_currency": "iso4217:USD",
    "deposit_details": {
      "bank_card_mask": "471227******0407",
      "from_first_name": "Saurabh",
      "from_last_name": "Ghodmare",
      "payment_group": "bank_card",
      "payment_method": "r:fab9dcd0-bb93-4f09-b65f-2edb69a3a213"
    },
    "md5_body_sig": "30e93a687812eb759f867d55d77a064d",
    "md5_sig": "a99ec94e699e361fdb33aa50140e474b",
    "merchant_external_id": "202678449",
    "refunded": true,
    "refunds": {
      "amount_fee": "2",
      "amount_refunded": "502",
      "payments": [
        {
          "amount": 502,
          "fee": 2,
          "id": "2a8bf37c-3d3d-4280-9e42-680c8ce6150a",
          "rrn": "Not Provided",
          "status": "completed"
        }
      ]
    },
    "sep31_status": "completed",
    "seq": 1724999384570,
    "status": "refunded",
    "transaction_id": "b829f009-afe0-45c2-9996-8941f80bcb0e"
  },

```

Request body contains the following elements:

| Field                | Description                                                                                      | Required |
|----------------------|--------------------------------------------------------------------------------------------------|----------|
| `transaction_id`       | PSP transaction id                                                                               | yes      |
| `status`               | the status of transaction                                                                    | yes       |
| | `canceled`:     transaction is canceled by a payment provider, this means that payment is incompleted. | |
| | `expired`: the payment was not done in time and the transaction is expired. | |
| | `payed`:  the payment was completed by a customer but the money hasn’t been settled to the PSP account. In rare cases. | |
| | `done`: transaction is successfully completed. | |
| | `refund_pending`: we have sent a request to the acquiring bank to initiate a refund. After receiving feedback on the request, the status of the refund will be updated. You can monitor the update of the refund status in your merchant back office. | |
| | `refunded`: transaction has been fully refunded. | |
| | `refund_rejected`:  refund has been rejected. | |
| `md5_sig`              | MD5 signature to validate callback request. `md5(transaction_id+status+secret_key)` **Deprecated!** Please use `md5_body_sig` instead                                                    | yes      |
| `seq`         | time in UnixMilli format at the moment the callback was created                                                          | yes      |
| `sep31_status`         | Sep31 transaction status. One of:   - completed - error                                                           | yes      |
| `refunded`             | `true`/`false`                                                                                       | yes      |
| `refunds.amount_fee`             |Fee according to contract                                                    | yes      |
| `amount_refunded`             |Amount to refund                             | yes      |
| `refunds.payments[?].amount`             |Amount to refund + fee according to contract                                          | yes      |
| `refunds.payments[?].fee`             | Fee according to contract                                                                                      | yes      |
| `refunds.payments[?].id`             | Refund id                                              | yes      |
| `refunds.payments[?].rrn`             | Retrieval Reference Number - identifier of refund from payment provider                           | yes      |
| `refunds.payments[?].status`             | status of refund                                     | yes      |
| | `pending`: we have sent a request to the acquiring bank to initiate a refund. After receiving feedback on the request, the status of the refund will be updated. You can monitor the update of the refund status in your merchant back office. | |
| | `completed`: transaction has been fully refunded. | |
| | `rejected`:  refund has been rejected. | |
| `md5_body_sig`         | MD5 signature to validate callback request. `md5(transaction_id+sep31_status+refunded+secret_key)`                                                     | yes      |
| `deposit_details`      | Details of a deposit transaction. Specific payload depends on a payment method. Examples are:             | no       |
| | ```  {    "payment_group": "bank_card",  "bank_card_mask": "1000****0001"} ```| |
| | ```  {   "payment_group": "bank_account"} ```| |
| | ```  {          "payment_group": "ewallet","ewallet_account_id": "475846215"} ```| |
| | ```  {    "binance_open_user_id": "8f825dd63a8dee291766c81cc4df5c62",  "ewallet_account_id": "578144430", "payment_group": "ewallet" "payment_method": "a44ba598-bec9-439e-baef-4dc8b0ae2e4b:80210375-51a1-4ac7-8d83-d3b31857a818",```| |
| | `payment_group` field may be one of the following values: `bank_card`,`cash_collection`, `crypto`, `ewallet`,`bank_account` ||
| `merchant_external_id` | A value of external_id field sent in initial request.                                             | no       |
| `amount_in`            | Amount a customer has payed.                                                                     | yes      |
| `amount_out`           | Amount the PSP settled to the merchant’s account.                                                 | yes      |
| `external_status`      | Status from the local provider                                                                   | no       |
| `external_message`     | An error details                                                                                 | no       |

### Transaction Status Callback for remit {#transaction-status-callback-for-remit}

Request example:

```
"payload": {
    "amount_body": 100.2,
    "amount_in": 100.2,
    "amount_out": 94.68,
    "asset": "stellar:PURPLE:GBT4VVTDPCNA45MNWX5G6LUTLIEENSTUHDVXO2AQHAZ24KUZUPLPGJZH",
    "delivery_currency": "iso4217:EUR",
    "payout_details": {
      "bank_card_mask": "401200******3010",
      "payment_group": "bank_card",
      "payment_method": "a7b5ac9a-dac2-46c7-919c-9e478e3daf62:1c032242-bc6f-4b0e-b8e9-7d3fca7a668f"
    },
    "external_message": "The transaction was completed successfully",
    "md5_body_sig": "cee95760668eb0963ab1dc5304fb6046",
    "md5_sig": "6fc67e043cb47b3cd7066450959117fe",
    "merchant_external_id": "123321",
    "refunded": false,
    "sep31_status": "completed",
    "seq": 1724822157474,
    "status": "done",
    "transaction_id": "07f62464-fcf1-47b3-a804-3af29b884134"
  },
  ```
Request body contains the following elements:

| Field                | Description                                                                                      | Required |
|----------------------|--------------------------------------------------------------------------------------------------|----------|
| `transaction_id`       | PSP transaction id                                                                               | yes      |
| `status`               | the status of transaction                                                                    | yes       |
| | `expired`: the payment was not done in time and the transaction is expired. | |
| | `done`: transaction is successfully completed. | |
| | `refunded`: Payout failed. Funds were returned to the merchant. | |
| `md5_sig`              | MD5 signature to validate callback request. `md5(transaction_id+status+secret_key)` **Deprecated!** Please use `md5_body_sig` instead                                                    | yes      |
| `seq`         | time in UnixMilli format at the moment the callback was created                                                          | yes      |
| `sep31_status`         | Sep31 transaction status. One of:   - completed - error                                                           | yes      |
| `refunded`             | `true`/`false`                                                                                       | yes      |
| `md5_body_sig`         | MD5 signature to validate callback request. `md5(transaction_id+sep31_status+refunded+secret_key)`                                                     | yes      |
|`payout_details`      | Details of a payout transaction. Specific payload depends on a payment method.   | no|
| |  ```  {   "payout_details": {      "payment_group": "cash_collection",      "ready_to_receive": true,      "collection_pin": "123987"}} ```| |
| | `payment_group` field may be one of the following values: `bank_card`,`cash_collection`, `crypto`, `ewallet`,`bank_account` ||
| `merchant_external_id` | A value of external_id field sent in initial request.                                             | no       |
| `amount_in`            | Amount a customer has payed.                                                                     | yes      |
| `amount_out`           | Amount the PSP settled to the merchant’s account.                                                 | yes      |
| `external_status`      | Status from the local provider                                                                   | no       |
| `external_message`     | An error details                                                                                 | no       |



## How to make a Refund {#how-to-make-a-refund}

If your deposit transaction is in the `completed` status, then you can make a refund on it.

 Depending on the payment method and issuer a refund can take

- 1 or 2 hours

- from 1 to 15 bank days

- 8-10 weeks in rare cases.

Example request:


export function HowToMakeRefund() {
	const {siteConfig} = useDocusaurusContext();
    const { search } = useLocation();
    const urlParams = new URLSearchParams(search);
    const merchantGUID = urlParams.get('merchant_guid');

  return (
      <CodeBlock
        language="bash"
        title="How to make a refund">
        {
        `curl POST ${siteConfig.customFields.FPF_URL}/api/v3/merchantGUID/TRANSACTION_ID/refund \\
        --header 'X-App-Token: APP_KEY'
        --header 'X-App-Secret: APP_SECRET'
        --header 'Content-Type: application/json'
        --data-raw '{
		        "reason":"{DESCRIPTION}"
                    }   
        `}
      </CodeBlock>
  );
}

 <HowToMakeRefund/>


Example response:

1. Status code 200: the body is empty
```
{"id":"refund_id"}
```
2. Status code 400/500 and example body: 
```
"bad_request, refund is not supported for this transaction"
```
**Note**: the refund transaction can have the following status:

- `refunded` - refund is done on acquirer side 

- `refund_rejected` - refund can’t be proceeded


## Getting merchant account balance via API {#how-to-check-balance-merchant-cabinet}

We will give you `APP_TOKEN` and `APP_SECRET` separately.

Example request:

export function HowToCheckAccountBalance() {
	const {siteConfig} = useDocusaurusContext();
    const { search } = useLocation();
    const urlParams = new URLSearchParams(search);
    const merchantGUID = urlParams.get('merchant_guid');
    const merchant_cabinet_API = urlParams.get('merchant_cabinet_API');

  return (
      <CodeBlock
        language="bash"
        title="How to check account’s balance using merchant_cabinet API">
        {
        `curl ${siteConfig.customFields.merchant_cabinet_API}/api/v2/merchantGUID/balance\\
        --header 'X-App-Token: APP_KEY'
        --header 'X-App-Secret: APP_SECRET'
        `}
      </CodeBlock>
  );
}

 <HowToCheckAccountBalance/>

 Example response:

 ```
{
    "data": [
        {
            "balance": 306.07,
            "asset": "USD",
            "asset_key": "ATUSD"
        },
        {
            "balance": 25,
            "asset": "EUR",
            "asset_key": "PURPLE"
        }
    ],
    "total": 332.57,
    "total_currency": "USD"
}
```

## Get Rates{#get-rates}

You can calculate the amount by taking into account the rates and commissions through our endpoint.

When making a deposit, specify `payin`, when remit, specify `payout`.

Also when making a remit you can calculate the *delivery_amount* by taking account the rates and commission through `inverse-rates` endpoint if you don’t have information about amount. Use *delivery_amount* as *amount* field in the body for `inverse-rates` endpoint.

Example request:

export function GetRates() {
	const {siteConfig} = useDocusaurusContext();
    const { search } = useLocation();
    const urlParams = new URLSearchParams(search);
    const merchantGUID = urlParams.get('merchant_guid');

  return (
      <CodeBlock
        language="bash"
        title="Get Rates">
        {
        `curl POST ${siteConfig.customFields.FPF_URL}/api/v3/merchantGUID/[payin/payout/inverse]-rates \\
              {
               "amount": 2,
               "fee_percent": 12,
               "fee_fixed": 10,
               "digits_asset": 2,
               "digits_delivery": 2,
               "rate": 30
              } 
        `}
      </CodeBlock>
  );
}

 <GetRates/>


Example response:

```
  {
    "amount": 2,
    "delivery_amount": 367.2,
    "fee": 10.24
  }
``` 




