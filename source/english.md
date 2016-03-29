---
title: eCommerce Good Practices 

language_tabs:
  - php: PHP

search: true
---

# eCommerce Good Practices

For curiosity or necessity, you must have wondered how you define a credit card number. Some people even find they are random numbers, 
or sequential, awarded by card issuers or by issuing banks, but, in fact, the card number follows a specified standard and it is possible
to know the card issuer, card type and account carrier, just watching the card number. In some cases, in fact, you can even know original 
country of the card, just watching their number.

In order to contribute to reducing the transaction denied by code 14 - Invalid card, Cielo recommends using the Luhn algorithm to verify
the numbers sequence of credit and debit cards used in your store. From this measure, the retailer will be able to prevent a transaction
with the mistyped card is sent for processing.

Therefore, Dell recommends that, at the time the bearer to enter the card number and the Luhn algorithm detects that typing is 
incorrect, the retailer should display clear information to the carrier, requesting that the card is entered again or try another 
card"

## Card Numbers

![Visa Card](//developercielo.github.io/Cartoes-e-validacao/images/cartao.png)

Basically the card number is composed of three parts:

1. **Bin or Inn** - Bank identification number, or Issuer identification number is the number that identifies the issuing bank of Visa,
Mastercard, Amex, among others, through the first digits of the card. In case of the example above, the bin is 4, which is the Visa identifier.
2. **Customer account** - - After the bin, the next digits identify the bearer of the account number with the bank. 
Soon after the bin, the next 14 digits are the customer's account identifier: **012** 0010 3714 111.
3. **Check digit** - This last digit is used to verify that the credit card number is valid. To get the check digit we use the Luhn algorithm.
In the example above, the check digit is **2**.

## Card Information

After validating the card number through the check digit obtained through the Luhn algorithm, we can verify if the card number is correct
according to the chosen card issuer. Cielo does not recommend to card issuers make a validation by BIN - first digits of the card; this recommendation is
important because there may be collision same number of BINs for different card issuers. Some card issuers have 13, 15 or 16 digits and the CVV has 3 or 4 digits. 
The table below shows the number of digits of each respective card issuer and CVV. Use this information in conjunction with the Luhn algorithm to complete 
validation of the customer's card number.

|Card issuer|Number of digits|CVV digits|
|--------|---|-----------------|--------------|
|Visa|13 or 16 digits|3 digits|
|Mastercard|16 digits|3 digits|
|Amex|15 digits|4 digits|
|Diners Club International|14 digits|3 digits|
|JCB|16 digits|3 digits|
|ELO|16 digits|3 digits|

# Validation of card numbers

<aside class="warning">Cielo does not perform support or responsible for the care of such implementation. This tutorial is purely 
informational, with the purpose of helping the retailer to reduce the number of denied transactions with code 14.</ Aside>

## Validation of card numbers

The validation of the most card issuers in the world is done through the Luhn algorithm, also known as module 10. 
Let's assume that the customer has informed the following number of credit card: 4,012,001,037,141,112.

The first step is remove temporarily the last digit if the digit 2. The new number will look like this: 401,200,103,714,111.

|Position| 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 |
|----------------|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|Card Number|4|0|1|2|0|0|1|0|3|7|1|4|1|1|1|*x*|

The second step is to multiply, starting from the first digit, all digits are in even position by 2 and all the digits 
in odd position for 1:

|Position| 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 |
|----------------|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|Card Number| 4 | 0 | 1 | 2 | 0 | 0 | 1 | 0 | 3 | 7 | 1 | 4 | 1 | 1 | 1 | - |
|Multiplications|x2|x1|x2|x1|x2|x1|x2|x1|x2|x1|x2|x1|x2|x1|x2||-|
|Results|8|0|2|2|0|0|2|0|6|7|2|4|2|1|2||-|

The third step is to take the results of multiplications and add up all the digits:

|Digits|Calculation|Result|
|-------|-------|---------|
|802200206724212|8+0+2+2+0+0+2+0+6+7+2+4+2+1+2|38|

The fourth step is to obtain the remainder of the Euclidean division of the result by the third step 10: 38/10 = 3, 
with remainder 8. The fifth step is to subtract the rest 10 10 - 8 = 2

|IV Calculation|Quotient|Rest|Calculation V|Result|
|----------|---------|-----|---------|---------|
|38/10|3|8|10-8|2|

The number 2 is the check digit. To verify if the client informed by the card number is valid, just check the number of the 
first step + the check digit is equal to the number of card informed by the customer.

Another example of the calculation, this time with a Martercard card:

|Card| 5 | 4 | 5 | 3 | 0 | 1 | 0 | 0 | 0 | 0 | 0 | 6 | 6 | 1 | 6 | 7 |Result|
|------|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---------|
|Step 1|5|4|5|3|0|1|0|0|0|0|0|6|6|1|6|||
|Step  2|10|4|10|3|0|1|0|0|0|0|0|6|12|1|12|||
|Step  3|1+0|4|1+0|3|0|1|0|0|0|0|0|6|1+2|1|1+2||23|
|Step  4||||||||||||||||23%10|3|
|Step  5||||||||||||||||10-3|**7**|

## Backend Validation

But even with the tester plugin in the frontend, it is essential, even for security considerations, a fact validation occurs on the 
backend. For this, the tool below will help you to do this validation:

```php
<?php
function cardIsValid($cardNumber)
{
    $number = substr($cardNumber, 0, -1);
    $doubles = [];

    for ($i = 0, $t = strlen($number); $i < $t; ++$i) {
        $doubles[] = substr($number, $i, 1) * ($i % 2 == 0? 2: 1);
    }

    $sum = 0;

    foreach ($doubles as $double) {
        for ($i = 0, $t = strlen($double); $i < $t; ++$i) {
            $sum += (int) substr($double, $i, 1);
        }
    }

    return substr($cardNumber, -1, 1) == (10-$sum%10);
}
```

Para utilizá-la, basta testar o número do cartão enviado pelo cliente:

```php
if (cardIsValid($customerCardNumber)) {
  // o cartão é válido e podemos dar andamento na integração
}
```
