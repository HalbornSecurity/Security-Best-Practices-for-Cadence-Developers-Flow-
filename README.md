# Security Best Practices for Cadence Developers (FLOW)

1. References are ephemeral values and cannot be stored. If persistence is required, store a capability and borrow it when needed.

2. Don't trust the users’ account storage. 

> **Warning**
> Users have full control over their data and may reorganize it as they see fit. Users may store values in any path, so paths may store values of “unexpected” types. These values may be instances of types in contracts that the user deployed.

> Always borrow with the specific type that is expected, or check if the value is an instance of the expected type.

3. Access to an AuthAccount gives full access to the storage, keys, and contracts. 

> Therefore, avoid using AuthAccount as a function parameter unless absolutely necessary.


4. Don’t store anything under the public capability storage unless strictly required. Anyone can access your public capability using getCapability. If something needs to be stored under public make sure only read functionality is provided by restricting its type using either a resource interface or struct interface.

5. Authorized references (references with the auth keyword) allow downcasting, e.g. a restricted type to its unrestricted type, so should only be used in some specific cases.

> **Warning**
> The subtype or unrestricted type could expose functionality that was not intended to be exposed.

> Do not use authorized references when exposing functionality. For example, the fungible token standard provides an interface to get the balance of a vault, without exposing the withdrawal functionality.

6. When linking a capability the link might be already present. In that case, Cadence will not panic with a runtime error but the link function will return nil. The documentation states that:

> The link function does not check if the target path is valid/exists at the time the capability is created and does not check if the target value conforms to the given type.

> It is a good practice to check if the link does already exist before creating it with getLinkTarget. This function will return nil if the link does not exist.


7. If it is necessary to handle the case where borrowing a capability (borrow) might fail, the check function can be used to verify that the target exists and has a valid type.


8. Signing a transaction gives access to the AuthAccount, i.e. full access to the account’s storage, keys, and contracts. 

> **Warning**
> Do not blindly sign a transaction. The transaction could for example change deployed contracts by upgrading it with malicious statements, revoke or add keys, transfer resources from storage, etc.


9. It is preferable to use capabilities over direct AuthAccount storage when exposing account data. Using capabilities allows the revocation of access by unlinking and limits the access to a single value with a certain set of functionality – access to an AuthAccount gives full access to the whole storage, as well as key and contract management.


10. Audits of Cadence code should also include transactions, as they may contain arbitrary code, just, like in contracts. In addition, they are given full access to the accounts of the transaction’s signers, i.e. the transaction is allowed to manipulate the signers’ account storage, contracts, and keys.


11. Always use the most specific type possible, following the principle of least privilege.
Types should always be as specific (restrictive) as possible, especially for resource-types. Use restricted types and interfaces.

> When exposing functionality, provide the least access necessary. Do not use authorized references, as they can be downcasted, potentially allowing a user to gain access to supposedly restricted functionality. For example, the fungible token standard provides an interface to get the balance of a vault, without exposing the withdrawal functionality.

> If given a less-specific type, cast to the more specific type that is expected. For example, when implementing the fungible token standard, a user may deposit any fungible token, so the implementation should cast to the expected concrete fungible token type.


12. Declaring a field as pub/access(all) only protects from replacing the field’s value, but the value itself can still be mutated if it is mutable. Remember that containers, like dictionaries, arrays, are mutable.

> Prefer non-public access to mutable state. 

> Such mutable state may also be nested. For example, a child may still be mutated even if its parent exposes it through a field with non-settable access.


13. Ensure capabilities cannot be accessed by unauthorized parties. For example, capabilities should not be accessible through a public field, including public dictionaries or arrays. Exposing a capability in such a way allows anyone to borrow it and perform all actions that the capability allows. 


14. Do not use the pub/access(all) modifier on fields and functions unless necessary. 

> Prefer priv/access(self), or access(contract) and access(account) when other types in the contract or account need to have access.

> See the design pattern: https://docs.onflow.org/cadence/design-patterns/#script-accessible-public-fieldfunction
