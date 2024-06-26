### [H-1] Storing the password on-chain makes it visible to anyone, and no longer private

**Description:** All data stored on-chain is visible to anyone, and can be read directly fromthe blockchain. The `PasswordStore::s_password` variable is intended to be a private variable and only accessed through `PasswordStore::getPassword` function, which is intended to be only called by the owner of the contract.

**Impact:** Anyone can read the private password, severly breaking the functionality of the protocol.

**Proof of Concept:** (Proof of Code)
The below test case shows how anyone can read the password directly from the blockchain.

1. Create a locally running chain

```bash
anvil
```

2. Deploy the contract to the chain

```bash
make deploy
```

3. Read the storage of the contract

```bash
cast storage <ADDRESS_HERE> 1 --rpc-url http://127.0.0.1:8545
```
You will get an output that looks like this
`0x6d7950617373776f726400000000000000000000000000000000000000000014`

4. Parse the hex to string

```bash
cast parse-bytes32-string 0x6d7950617373776f726400000000000000000000000000000000000000000014
```
to get an output of:
`myPassword`

**Recommended Mitigation:** Due to this the overall architecture of the contract should be rethought. One could encrypt the password off-chain, then store the encrypted password on-chain. This would require the user to remember another password off-chain to decrypt the password.


### [H-2] `PasswordStore::setPassword` has no access control, meaning anyone can change the password

**Description:** The `PasswordStore::setPassword` function is set to `external` and doesn't have any checks to check if the one calling it is actually the owner

**Impact:** Anyone can set/change the password stored in the contract, severely breaking the contract intended functionality. 

**Proof of Concept:** Add the following to `PasswordStore.t.sol` test file

<details>
<summary>code</summary>

```javascript
    function test_anyone_can_set_password(address randomAddress, string memory newPassword) public {
        vm.assume(randomAddress != owner);
        vm.startPrank(randomAddress);
        passwordStore.setPassword(newPassword);
        vm.stopPrank();
        vm.startPrank(owner);
        string memory actualPassword = passwordStore.getPassword();
        vm.stopPrank();
        assertEq(actualPassword, newPassword);
    }
```

</details>

**Recommended Mitigation:** Add an access control conditional to `setPassword` function.
```javascript
if (msg.sender != s_owner) {
    revert PasswordStore__NotOwner();
}
```


### [I-1] The `PasswordStore::getPassword` natspec indicates a parameter that doesn't exist causing the natspecto be incorrect

**Description:** 
```javascript
    /*
     * @notice This allows only the owner to retrieve the password.
     * @param newPassword The new password to set.
     */
    function getPassword() external view returns (string memory) {
```
The `PasswordStore::getPassword` function signature is `getPassword()` while the natspec say it should be `getPassword(string)`.

**Impact:** natspec is incorrect

**Recommended Mitigation:** Remove the incorrect natspec line

```diff
-     * @param newPassword The new password to set.
```