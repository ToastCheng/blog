### Read concern

`db.collection.find().readConcern(<level>)`

> For multi-document transactions, you set the read concern at the transaction level, not at the individual operation level.
>


Concern about consistency and isolation level?
Then you can set read concern properly to meet your needs.

#### local 
Read data from the local node.
Not guaranteed that data is copied to the relicas, and the data you read MIGHT BE ROLLED BACK.

#### available
Read data whenever is available.
Not guaranteed that data is copied to the relicas, and the data you read MIGHT BE ROLLED BACK.

#### majority
Read the result that is acknowledge by the majorities.
Guarantees:
- Read your write
- Monotonic reads

#### linearizable
Read the result that is acknowledge by all nodes.
May wait til exectution propagates to all nodes.
 
#### snapshot



### Write concern
