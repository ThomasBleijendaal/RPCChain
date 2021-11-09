# RPCChain

## Syntax idea

```c#
var userId = "some-id";
var transactionId = "transaction-id";

var requestChain = Chain.Create<TransactionDetails>()
    .StartWith(() => AccountService.GetAccountByUserId(userId)) // returns object like { AccountNumber = "x" }
    .DoNext((step1) => TransactionService.GetTransactionByTransactionId(From.Step(step1, x => x.AccountNumber), userId)) // returns object like { TargetAccountNumber = "y" }
    .Returns((step1, step2) => new TransactionDetails { AccountNumber = From.Step(step1, x => x.AccountNumber), TargetAccountNumber = From.Step(step2, x => x.TargetAccountNumber) });

var result = await Chain.Execute(); // returns object with { AccountNumber = "x", TargetAccountNumber = "y" }

```

## Requirements

- All nodes should understand each others RPC calls. Should be based on gRPC so every step can rely on each others code.
- Logic in `Returns` will be performed in the chain creators process, to facilitate projection.

## Basic working

- Chain expressions get parsed and put in some serializable object: `{ ServiceMethod = AccountService.GetAccountByUserId, Arguments = new[] { "some-id" } }` and `{ ServiceMethod = TransactionService.GetTransactionByTransactionId, Arguments = new [] { step1.AccountNumber, "some-id" }`. 
- Returns expression is parsed but doesn't have to go somewhere.
- When Execute() is called, the chain expression is shipped to the Chain handler. This chain handlers handles locally originated chains, as well as remotely originated chains. 

- Chain handler gets first action to do, finds the service and posts it to the gRPC service. (Should every gRPC endpoint support the rpc chain object, or should there be a chain endpoint, and every service call is done locally?)
- After service call, next step is made concrete by resolving pending actions (could be more then only getting properties from models, could be basic logic).
- Chain handler posts it to next gRPC service.

## Security considerations

- Each chain handler should verify if the originated is allowed to do those actions. That should come from some auth server.
- 
