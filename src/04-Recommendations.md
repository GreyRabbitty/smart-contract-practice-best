
## Recommendations for Smart Contract Security in Solidity


### External Calls

#### Avoid external calls when possible
<a name="avoid-external-calls"></a>

Calls to untrusted contracts can introduce several unexpected risks or errors. External calls may execute malicious code in that contract _or_ any other contract that it depends upon. As such, every external call should be treated as a potential security risk, and removed if possible. When it is not possible to remove external calls, use the recommendations in the rest of this section to minimize the danger.

<a name="avoid-call-value"></a>

#### Use `send()`, avoid `call.value()`

When sending Ether, use `someAddress.send()` and avoid `someAddress.call.value()()`.

External calls such as `someAddress.call.value()()` can trigger malicious code. While `send()` also triggers code, it is safe because it only has access to gas stipend of 2,300 gas. Currently, this is only enough to log an event, not enough to launch an attack.

```
// bad
if(!someAddress.call.value(100)()) {
    // Some failure code
}

// good
if(!someAddress.send(100)) {
    // Some failure code
}
```

<a name="handle-external-errors"></a>

#### Handle errors in external calls

Solidity offers low-level call methods that work on raw addresses: `address.call()`, `address.callcode()`, `address.delegatecall()`, and `address.send`. These low-level methods never throw an exception, but will return `false` if the call encounters an exception. On the other hand, *contract calls* (e.g., `ExternalContract.doSomething()`) will automatically propagate a throw (for example, `ExternalContract.doSomething()` will also `throw` if `doSomething()` throws).

If you choose to use the low-level call methods, make sure to handle the possibility that the call will fail, by checking the return value. Note that the [Call Depth Attack](https://github.com/ConsenSys/smart-contract-best-practices/#call-depth-attack) can cause *any* call to fail, even if the external contract's code is working and non-malicious.

```
// bad
someAddress.send(55);
someAddress.call.value(55)(); // this is doubly dangerous, as it will forward all remaining gas and doesn't check for result
someAddress.call.value(100)(bytes4(sha3("deposit()"))); // if deposit throws an exception, the raw call() will only return false and transaction will NOT be reverted

// good
if(!someAddress.send(55)) {
    // Some failure code
}

ExternalContract(someAddress).deposit.value(100);
```

<a name="expect-control-flow-loss"></a>

#### Don't make control flow assumptions after external calls

Whether using *raw calls* or *contract calls*, assume that malicious code will execute if `ExternalContract` is untrusted. Even if `ExternalContract` is not malicious, malicious code can be executed by any contracts *it* calls. One particular danger is malicious code may hijack the control flow, leading to race conditions. (See [Race Conditions](https://github.com/ConsenSys/smart-contract-best-practices/#race-conditions) for a fuller discussion of this problem).

<a name="favor-pull-over-push-payments"></a>

#### Favor *pull* over *push* for external calls

As we've seen, external calls can fail for a number of reasons, including external errors and malicious [Call Depth Attacks](https://github.com/ConsenSys/smart-contract-best-practices/#call-depth-attack). To minimize the damage caused by such failures, it is often better to isolate each external call into its own transaction that can be initiated by the recipient of the call. This is especially relevant for payments, where it is better to let users withdraw funds rather than push funds to them automatically. (This also reduces the chance of [problems with the gas limit](https://github.com/ConsenSys/smart-contract-best-practices/#dos-with-block-gas-limit).)

```
// bad
contract auction {
    address highestBidder;
    uint highestBid;

    function bid() {
        if (msg.value < highestBid) throw;

        if (highestBidder != 0) {
            if (!highestBidder.send(highestBid)) { // if this call consistently fails, no one else can bid
                throw;
            }
        }

       highestBidder = msg.sender;
       highestBid = msg.value;
    }
}

// good
contract auction {
    address highestBidder;
    uint highestBid;
    mapping(address => uint) refunds;

    function bid() external {
        if (msg.value < highestBid) throw;

        if (highestBidder != 0) {
            refunds[highestBidder] += highestBid; // record the refund that this user can claim
        }

        highestBidder = msg.sender;
        highestBid = msg.value;
    }

    function withdrawRefund() external {
        uint refund = refunds[msg.sender];
        refunds[msg.sender] = 0;
        if (!msg.sender.send(refund)) {
            refunds[msg.sender] = refund; // reverting state because send failed
        }
    }
}
```

<a name="mark-untrusted-contracts"></a>

#### Mark untrusted contracts

When interacting with external contracts, name your variables, methods, and contract interfaces in a way that makes it clear that interacting with them is potentially unsafe. This applies to your own functions that call external contracts.

```
// bad
Bank.withdraw(100); // Unclear whether trusted or untrusted

function makeWithdrawal(uint amount) { // Isn't clear that this function is potentially unsafe
    UntrustedBank.withdraw(amount);
}

// good
UntrustedBank.withdraw(100); // untrusted external call
TrustedBank.withdraw(100); // external but trusted bank contract maintained by XYZ Corp

function makeUntrustedWithdrawal(uint amount) {
    UntrustedBank.withdraw(amount);
}
```

<a name="beware-rounding-with-integer-division"></a>

### Beware rounding with integer division

All integer divison rounds down to the nearest integer. If you need more precision, consider using a multiplier, or store both the numerator and denominator.

(In the future, Solidity will have a fixed-point type, which will make this easier.)

```
// bad
uint x = 5 / 2; // Result is 2, all integer divison rounds DOWN to the nearest integer

// good
uint multiplier = 10;
uint x = (5 * multiplier) / 2;

uint numerator = 5;
uint denominator = 2;
```

<a name="keep-fallback-functions-simple"></a>

### Keep fallback functions simple

[Fallback functions](http://solidity.readthedocs.io/en/latest/contracts.html#fallback-function) are called when a contract is sent a message with no arguments (or when no function matches), and only has access to 2,300 gas when called from a `.send()`. If you wish to be able to receive Ether from a `.send()`, the most you can do in a fallback function is log an event. Use a proper function if a computation or more gas is required.

```
// bad
function() { balances[msg.sender] += msg.value; }

// good
function() { throw; }
function deposit() external { balances[msg.sender] += msg.value; }

function() { LogDepositReceived(msg.sender); }
```

<a name="mark-visibility"></a>

### Explicitly mark visibility in functions and state variables

Explicitly label the visibility of functions and state variables. Functions can be specified as being `external`, `public`, `internal` or `private`. For state variables, `external` is not possible. Labeling the visibility explicitly will make it easier to catch incorrect assumptions about who can call the function or access the variable.

```
// bad
uint x; // the default is private for state variables, but it should be made explicit
function transfer() { // the default is public
    // public code
}

// good
uint private y;
function transfer() public {
    // public code
}

function internalAction() internal {
    // internal code
}
```


<a name="beware-division-by-zero"></a>

### Beware division by zero

Currently, Solidity [returns zero](https://github.com/ethereum/solidity/issues/670) and does not `throw` an exception when a number is divided by zero. You therefore need to check for division by zero manually.

```
// bad
function divide(uint x, uint y) returns(uint) {
    return x / y;
}

// good
function divide(uint x, uint y) returns(uint) {
   if (y == 0) { throw; }

   return x / y;
}
```

<a name="differentiate-functions-events"></a>

### Differentiate functions and events

Favor capitalization and a prefix in front of events (we suggest *Log*), to prevent the risk of confusion between functions and events. For functions, always start with a lowercase letter, except for the constructor.

```
// bad
event Transfer() {}
function transfer() {}

// good
event LogTransfer() {}
function transfer() external {}
```
