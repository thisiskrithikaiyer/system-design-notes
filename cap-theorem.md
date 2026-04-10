Consistency, Availability, Partition Tolerance. 

In a distributed network when parts of network or clusters fail. Choose between consistency and avaialibilty. 

ACID - When you place an order and payment has to be made to that order. You need to rollback order/item state if payment is interupted. You need
to make sure after payment the database is in a valid state. Two concurrent transcations should not interfere each other. When data is commited it should be mainrtained after power loss or restarts. 

BASE - Basic Avaialiblity, Soft State and Eventual Consistency. When you like a user's profile picture then one server may show immediate increment some might show previous and eventually after a while show same. 



