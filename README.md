# BitBay Trading API #

###  Connecting with API ###

To connect with API you have to send POST request on address below:

https://bitbay.net/API/Trading/tradingApi.php

Request must have **method** parameter, which calls proper operation. 
Next parameter of each request is **moment** - it is current time in unix timestamp format.

### API Keys ###

When you want to call API operation you need authentication key. 
Key generation is in account page, under “**API Keys**” tab

https://bitbay.net/account

Each key consist of 3 properties:

- **public key** : you can see it on keys list, it is used for recognizing user and key
- **secret key** :  this one is visible only after generating; after this operation you cannot see it anywhere. Secret key is used to authenticate user, which calls API operation
- **permissions** : when generating new key you can set permissions for specific operations

### Signing requests ###

POST requests have to be signed by authentication code. 
You have to use secret key for that, which you use to sign parameters with SHA512 algorithm.
It is neccessary to send public key in header as API-Key and also hashed post message as API-Hash.

Example:

```php

        $params["method"] = “info”;    
        $params["moment"] = time();
        $post = http_build_query($params, "", "&");
        $sign = hash_hmac("sha512", $post, $secret);
        $headers = array(
            "API-Key: " . $key,
            "API-Hash: " . $sign,
        );
```

### Moment parameter ###

It is request execution unix timestamp. This value is compared with server time. If difference is bigger than 5 seconds - operation won’t be executed.
It is important to have synchronized time with NTP public time server - in other case server time can differ a lot than time of request machine.

### Example PHP execution code ###

```php
function BitBay_Trading_Api($method, $params = array())
{
    $key = "123";
    $secret = "321";
    
    $params["method"] = $method;        
    $params["moment"] = time();

    $post = http_build_query($params, "", "&");
    $sign = hash_hmac("sha512", $post, $secret);
    $headers = array(
        "API-Key: " . $key,
        "API-Hash: " . $sign,
    );
    $curl = curl_init();
    curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($curl, CURLOPT_URL, "https://bitbay.net/API/Trading/tradingApi.php");
    curl_setopt($curl, CURLOPT_POST, true);
    curl_setopt($curl, CURLOPT_POSTFIELDS, $post);
    curl_setopt($curl, CURLOPT_HTTPHEADER, $headers);
    $ret = curl_exec($curl);

    return $ret;
}
```

### API call rate limit  ###

The rate limit is 1 req / second.

###  API methods ###

List of methods, which you have to send in method parameter.

####  `info` -  returns information about account balances ####

Input:

-  _(optional)_ **currency** :  currency shortcut if you want to display only specific        balance (e.g. “BTC”)

Output: 

- **currency** :  shortcut
- **available**  : amount of available money/cryptocurrency
- **locked** : amount of locked money/cryptocurrency

#### `trade` - places offer at the stock market ####

Input:

- **type** : offer type bid/buy or ask/sell
- **currency**  : shortcut of main currency for offer (e.g. “BTC”)
- **amount** : quantity of main currency
- **payment_currency** : shortcut of currency used to pay for offer (e.g. “PLN”)
- **rate** : rate for offer
    
Output:

- **order_id** :  id of placed order
- **error** :  if any took place

####  `cancel` - removes offer from the stock market ####

Input:

- **id** : id used to recognize offer; you get it from trade method output

Output:

- **success** : if cancelation was successful
- **error** : if any



#### `orderbook` - list of all open offers at the stock market ####

Input:

- **order_currency** : shortcut of offer main currency (e.g. “BTC”)
- **payment_currency** : shortcut of currency used to pay (e.g. EUR)

Output:

- **bids** : object with bids details
- **currency** : shortcut of payment currency
- **price** : amount of currency for whole offer
- **quantity** : amount of offer main currency
- **asks** : object with asks details
- **currency** : shortcut of payment currency
- **price** : amount of currency for whole offer
- **quantity** : amount of offer main currency

#### `orders` - list of all your offers ####

Input:
- _(optional)_  **limit** : number of rows to show; if no specified, returns latest 50 orders

Output:

- **order_id** : id of offer
- **order_currency** : main currency (e.g. “LTC”)
- **order_date** : time, when offer was changed recently
- **payment_currency** : shortcut of currency used to pay for offer
- **type** : bid/ask
- **status** : “active” if order is active, “inactive” if order is unactive
- **units** : current amount of main currency in order
- **start_units** : amount of main currency when order was added
- **current_price** : price for whole amount of main currency
- **start_price** : starting price for whole amount when offer was added

#### `transfer` - transfers cryptocurrency to other wallet ####

Input:

- **currency** : cryptocurrency to transfer
- **quantity** : amount of cryptocurrency, which will be transferred
- **address** : wallet address of receiver

Output:

- **success** - if transfer succeeded
- **error** - if any

#### `withdraw` - withdraws money into bank account ####

Input:

- **currency** : currency to withdraw (e.g. USD)
- **quantity** : amount of money to withdraw
- **account** : account number on which money would be transferred
- **express** : true/false
- **bic** : swift/bic number

Output:
 
- **success** : if withdrawal succeeded
- **error** :  if any

#### `history` - history of all operations on user account ####

Input:

- **currency** : name of the currency in which history will be displayed
- **limit** : the limit for results

Output:

- **amount** : amount of currency in operation
- **balance_after** : total balance after operation (does not contain substracted   locked balance)
- **currency** : currency of operation
- **time** : commission date
- **comment** : comment for operation (usually it is type of operation)

### Error codes ###

<table>
	<tr><td>Code</td><td>Description</td></tr>
	<tr><td>400</td><td>At least one parameter wasn't set</td></tr>
	<tr><td>401</td><td>Invalid order type</td></tr>
	<tr><td>402</td><td>No orders with specified currencies</td></tr>
	<tr><td>403</td><td>Invalid payment currency name</td></tr>
	<tr><td>404</td><td>Error. Wrong transaction type</td></tr>
	<tr><td>405</td><td>Order with this id doesn't exist</td></tr>
	<tr><td>406</td><td>No enough money or crypto</td></tr>
	<tr><td>408</td><td>Invalid currency name</td></tr>
	<tr><td>501</td><td>Invalid public key</td></tr>
	<tr><td>502</td><td>Invalid sign</td></tr>
	<tr><td>503</td><td>Invalid moment parameter. Request time doesn't match current server time</td></tr>
	<tr><td>504</td><td>Invalid method</td></tr>
	<tr><td>505</td><td>Key has no permission for this action</td></tr>
	<tr><td>506</td><td>Account locked. Please contact with customer service</td></tr>
	<tr><td>509</td><td>The BIC/SWIFT is required for this currency</td></tr>
</table>

