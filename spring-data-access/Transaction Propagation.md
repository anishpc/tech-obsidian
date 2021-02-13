## Propagation
- In Spring-managed transactions, be aware of the difference between physical and logical transactions, and how the propagation setting applies to this difference.

### Propagation.REQUIRED (default)
![tx prop required](https://docs.spring.io/spring-framework/docs/current/reference/html/images/tx_prop_required.png)
- Physical transaction : Creates a new physical transaction if one doesn't exist. If there's an existing transaction then it participates in the existing 'outer' transaction
- Logical transaction : 
	- a logical transaction scope is created for each method
	- such a logical transaction scope can determine rollback-only status individually with an outer transaction scope being logically independent from the inner transaction scope
- Outer transaction affected by Inner transaction :
	- a rollback-only marker set in the inner transaction scope DOES affect the outer transaction's chance to commit, since there's only one physical transaction
	- in the case where an inner transaction scope sets the rollback-only marker, the outer transaction has not decided on the rollback itself, so the rollback (silently triggered by the inner transaction scope) is unexpected. A corresponding `UnexpectedRollbackException` is thrown at that point.
- Outer calls commit for inner rollback : 
	- So, if an inner transaction (of which the outer caller is not aware) silently marks a transaction as rollback-only, the outer caller still calls commit. The outer caller needs to receive an `UnexpectedRollbackException` to indicate clearly that a rollback was performed instead.
> By default, a participating transaction joins the characteristics of the outer scope, silently ignoring the local isolation level, timeout value, or read-only flag (if any).

### Propagation.REQUIRES_NEW
![tx prop requires new](https://docs.spring.io/spring-framework/docs/current/reference/html/images/tx_prop_requires_new.png)
- Physical transaction : `PROPAGATION_REQUIRES_NEW` always uses an independent physical transaction for each affected transaction scope, never participating in an existing transaction for an outer scope.
- Outer not affected by inner : 
	- an outer transaction not affected by an inner transaction’s rollback status and with an inner transaction’s locks released immediately after its completion
	- Such an independent inner transaction can also declare its own isolation level, timeout, and read-only settings and not inherit an outer transaction’s characteristics

### Propagation.NESTED
- Physical : `PROPAGATION_NESTED` uses a single physical transaction with multiple savepoints that it can roll back to. 
- Outer is parts of inner & not totally affected by inner : 
	- partial rollbacks let an inner transaction scope trigger a rollback for its scope, with the outer transaction being able to continue the physical transaction despite some operations having been rolled back. 
- JDBC resource : This setting is typically mapped onto JDBC savepoints, so it works only with JDBC resource transactions