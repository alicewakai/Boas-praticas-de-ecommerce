---
title: eCommerce Good Practices

language_tabs:
  - php: PHP

search: true
---

# eCommerce Good Practices

Because of curiousity or necessity, you must have wondered about how you define the number of a credit card. Some people even find that
are random numbers, or sequential, awarded by card issuers or by issuing banks, but the reality is: the card number follows a 
specified standard and it is possible to know what the card issuer, card type and account carrier. In fact, in some cases, you can learn 
what is the country of the origin of the card, just watching their number.

In order to contribute to reducing the transaction denied of code 14 - Invalid card, Cielo recommends using Luhn algorithm to verify 
the number sequence of credit and debit cards used in your store. From this measure, the retailer will be able to prevent a transaction 
with the mistyped card is sent for processing.

Therefore, Cielo recommends at the time the card holder is entering the card number and the Luhn algorithm detects a typing error, 
the retailer should display a clear information to the card holder, requesting to enter the data again or try another card.

## Card numbers 

![Cartão Visa](//developercielo.github.io/Cartoes-e-validacao/images/cartao.png)

Basically, the card number is composed of three parts:

1. **Bin or Inn** - Bank identification number, or Issuer identification number, is the number that identifies the issuing bank Visa, Mastercard, Amex, among others, through the first digits of the card. In case of the example above design, the bin is 4, which is the Visa identifier.
2. **Customer account** - After bin, the next digit identifies the holder of the account number with the bank. Soon after the bin, the next 14 digits are the customer's account identifier: **012 0010 3714 111**.
3. **Check digit** - This last digit is used to verify that the credit card number is valid. To get the check digit is used an algorithm called Luhn. For the card example above, the check digit is **2**

## Card information

After validating the card number through the check digit (obtained through the Luhn algorithm), we can verify that the card number 
is correct according to the card issuer chosen. Cielo does not recommend card to make a validation of card issuers by BIN - first digits 
of the card; this recommendation is important because there may be collision of the same number of BINs for different card issuers. 
Some card issuers have 13, 15 or 16 digits and the CVV has 3 or 4 digits. The table below shows the number of digits of each respective
card issuer and CVV. Use this information in conjunction with the Luhn algorithm to complete validation of the customer's card number.

|Card issuer|Number of digits|CVV Digits|
|--------|---|-----------------|--------------|
|Visa|13 or 16 digits|3 digits|
|Mastercard|16 digits|3 digits|
|Amex|15 digits|4 digits|
|Diners Club International|14 digits|3 digits|
|JCB|16 digits|3 digits|
|ELO|16 digits|3 digits|

# Validation of card numbers

<aside class="warning"> Cielo does not perform support or responsible for the care of such implementation. This tutorial is purely 
informational, with the purpose of helping the retailer to reduce the number of denied transactions with code 14.</aside>

## Validation of card numbers

The card number validation of most card issuers is made through a so-called Luhn algorithm, also known as module 10. 
Let's assume that the customer has informed the following number of credit card: 4012001037141112.

The first step is to temporarily remove the last digit, in this case digit 2. The new number will look like this: 401200103714111.

|Position| 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 |
|----------------|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|Card number|4|0|1|2|0|0|1|0|3|7|1|4|1|1|1|*x*|

The second step is to multiply, starting from the first digit, all digits in the even position by 2 and all the digits in unique position for 1:

|Position| 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 |
|----------------|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|Card number| 4 | 0 | 1 | 2 | 0 | 0 | 1 | 0 | 3 | 7 | 1 | 4 | 1 | 1 | 1 | - |
|Multiplicações|x2|x1|x2|x1|x2|x1|x2|x1|x2|x1|x2|x1|x2|x1|x2||-|
|Resultados|8|0|2|2|0|0|2|0|6|7|2|4|2|1|2||-|

The third step is to take the results of multiplications and add up all the digits:

|Dígitos|Calculation|Results|
|-------|-------|---------|
|802200206724212|8+0+2+2+0+0+2+0+6+7+2+4+2+1+2|38|

The fourth step is to obtain the remainder of the Euclidean division of the result by the third step by 10: 38/10 = 3, with remainder 8. The fifth step is to subtract the rest: 10 10 - 8 = 2

|Calculation IV|Quotient|Rest|Calculation V|Result|
|----------|---------|-----|---------|---------|
|38/10|3|8|10-8|2|

The number 2 is the check digit. To verify that the client informed by the card number is valid, just check if the number of the first step + the check digit is equal to the card number informed by the customer.

Another example of the calculation, this time with a Martercard card:

|Card| 5 | 4 | 5 | 3 | 0 | 1 | 0 | 0 | 0 | 0 | 0 | 6 | 6 | 1 | 6 | 7 |Result|
|------|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---------|
|Step 1|5|4|5|3|0|1|0|0|0|0|0|6|6|1|6|||
|Step 2|10|4|10|3|0|1|0|0|0|0|0|6|12|1|12|||
|Stepo 3|1+0|4|1+0|3|0|1|0|0|0|0|0|6|1+2|1|1+2||23|
|Step 4||||||||||||||||23%10|3|
|Step 5||||||||||||||||10-3|**7**|

## Backend validation

But even with the tester plugin in the frontend, it is essential, even for security considerations, a fact validation occurs on the
backend. The tool below will help you to do this validation:

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

To use it, just test the card number sent by the client:

```php
if (cardIsValid($customerCardNumber)) {
  // o cartão é válido e podemos dar andamento na integração
}
```
