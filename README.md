omise-node
==========

Omise Node.js bindings.

## Installation

```
$ npm install omise
```

The library has been tested with Node version 0.10.32+.

## Usage

First, you have to configure the library by passing the public key and secret key from `https://dashboard.omise.co/` to `omise` export, for example:

```
var omise = require('omise')({
  'publicKey': 'pkey_test_...',
  'secretKey': 'skey_test_...'
});
```

Please see [Omise Documentation](https://docs.omise.co/) for more information on how to use the library.


**Full Credit Card data should never touch or go through your servers. That means, Do not send the credit card data to Omise from your servers directly.**

The token creation method in the library should only be used either with fake data in test mode (e.g.: quickly creating some fake data, testing our API from a terminal, etc.), or if you do and you are PCI-DSS compliant, sending card data from server requires a valid PCI-DSS certification.
that said, you must achieve, maintain PCI ompliance at all times and do following a Security Best Practices https://www.pcisecuritystandards.org/documents/PCI_DSS_V3.0_Best_Practices_for_Maintaining_PCI_DSS_Compliance.pdf

So, we recommended you to create a token using [Omise.JS](https://github.com/omise/omise.js) library which runs on browser side. It uses javascript to send the credit card data on client side, send it to Omise, and then you can populate the form with a unique one-time used token which can be used later on with `omise-node`
or [Card.js](https://docs.omise.co/card-js/), by using it you can let it builds a credit card payment form window and creates a card token that you can use to create a charge with `omise-node`.
For both methods, the client will directly send the card information to Omise gateway, your servers don't have to deal with card information at all and you don't need to deal with credit card data hassle, it reduces risk.

**Please read https://docs.omise.co/collecting-card-information/ regarding how to collecting card information.**

## Examples

### Create a customer with card associated to it

Creating a customer could be done by using `omise.customers.create` which accept optional `card` argument. When you pass in a `tokenId` retrieve from `omise.tokens` or [Omise.js](https://docs.omise.co/omise-js/), the card associated to that token will be associated to the customer.

```
var customer = {
  'email': "john.doe@example.com",
  'description': "John Doe (id: 30)",
  'card': tokenId
};

omise.customers.create(customer, function(err, customer) {
  var customerId = customer.id;
  console.log(customerId);
});
```

### List all customers

After customers are created, you can list them with `customer.customers.list` and passing a callback to it. The object returned from a list API will be a `list` object, which you can access the raw data via `data` attribute:

```
omise.customers.list(function(err, list) {
  console.log(list.data);
});
```

### Retrieve a customer

You can retrieve a created customer by using `omise.customers.retrieve` and passing a customer ID to it, e.g.

```
omise.customers.retrieve(customerId, function(err, resp) {
  console.log(resp.description);
});
```

### Updating a Customer

The same with customer updating, which could be done using `omise.customers.update` with a customer ID and an object containing changes:

```
var changes = {
  description: "Customer for john.doe@example.com"
}

omise.customers.update(customerId, changes, function(err, resp) {
  console.log(resp.description);
});
```

## Promise support

The library also supports the Promise/A+ interface that shares the same API method as the callback one, for example:

```
var cardDetails = {
  card: {
    'name': 'JOHN DOE',
    'city': 'Bangkok',
    'postal_code': 10320,
    'number': '4242424242424242',
    'expiration_month': 2,
    'expiration_year': 2017
  }
};

// First, we start creating a token with `omise.tokens.create` to store the card data.
omise.tokens.create(cardDetails).then(function(token) {

  // Then, create a customer using token returned the API.
  console.log(token.id);
  return omise.customers.create({
    email: "john.doe@example.com",
    description: "John Doe (id: 30)",
    card: token.id
  });

}).then(function(customer) {

  // And we make a charge to actually charge the customer for something.
  console.log(customer.id);
  return omise.charges.create({
    amount: 10000,
    currency: 'thb',
    customer: customer.id
  });

}).then(function(charge) {

  // This function will be called after a charge is created.

}).error(function(err) {

  // Put error handling code here.

}).done();
```

##Error Handling

To handle an invalid request, it is required to check any error via an `Error` object
that includes `code` and `message` attributes as stated in https://docs.omise.co/api/errors.
But, for any valid request, checking `failure_code` and `failure_message` is required, for example:
If you'd like to create a `Charge` or a `Transfer` with a valid request,
A sucessfully charge or tranfer happens only when none of failure exists that means both `failure_code` and `failure_message` must be `null`.

## Resource methods

The following API methods are available. Please see [https://docs.omise.co](https://docs.omise.co) for more details.

* account
  * `retrieve()`
* balance
  * `retrieve()`
* charges
  * `create(data)`
  * `list()`
  * `retrieve(chargeId)`
  * `capture(chargeId)`
  * `createRefund(chargeId[, data])`
  * `update(chargeId[, data])`
* customers
  * `create(data)`
  * `list()`
  * `update(customerId[, data])`
  * `destroy(customerId)`
  * `retrieve(customerId)`
  * `listCards(customerId)`
  * `retrieveCard(customerId, cardId)`
  * `updateCard(customerId, cardId[, data])`
  * `destroyCard(customerId, cardId)`
* tokens
  * `create(data)`
  * `retrieve(tokenId)`
* transfers
  * `create(data)`
  * `list()`
  * `retrieve(transferId)`
  * `update(transferId[, data])`
* transactions
  * `list()`
  * `retrieve(transactionId)`
* disputes
  * `list()`
  * `listClosed()`
  * `listOpen()`
  * `listPending()`
  * `retrieve(disputeId)`
  * `update(disputeId[, data])`
* recipients
  * `create(data)`
  * `list()`
  * `update(recipientId[, data])`
  * `destroy(recipientId)`
  * `retrieve(recipientId)`

## Testing

There are two modes of testing, to test without connecting to remote API server:

```
$ npm test
```

If you want to test by connecting to actual API server, you must first obtain a public and secret keys and export it:

```
$ export OMISE_PUBLIC_KEY=<test public key>
$ export OMISE_SECRET_KEY=<test secret key>
$ NOCK_OFF=true npm test
```

## Contributions

Before submitting a pull request, please run jscs to verify coding styles and ensure all test passed:

```
$ npm run jscs
$ npm test
```

You could use also use a git pre-commit hook to do this automatically by aliasing the `pre-commit.sh` to Git pre-commit hook:

```
ln -s ../../pre-commit.sh .git/hooks/pre-commit
```

### Adding new resources

Resources are handled via `apiResource`. Adding new resource could be done by creating a new resource file as `lib/resourceName.js` with the following content:

```
var resource = require('../apiResources');
var resourceName = function(config) {
  return resource.resourceActions(
    'resourceName',
    ['create', 'list', 'retrieve', 'destroy', 'update'],
    {'key': config['secretKey']}
  );
}

module.exports = resourceName;
```

Then register the newly created resource to `lib/apiResources.js` as e.g. `resourceName('resourceName')`. Pre-built actions are: create, list, retrieve, destroy and update.

### Requests mocking

Request mocks are stored as `test/mocks/<resource>_<action>.js`.
