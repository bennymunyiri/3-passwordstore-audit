### [H-1] Storing onchain makes it visible to anyone.

**Description:** All data stored on-chain is visible to anyone, and can be read directly from the blockchain, The `PasswordStore::s_password `is intended to be private variable and only accessed through the `PasswordStore::getPassword` function, which is intended to be only called by the owner of the contract.


We show one such method of reading any data off chain below.

**Impact:** Anyone can read the private password, Breaking the functionality of the protocol.

**Proof of Concept:**
The below data shows how one can read directly from the blockchain

1. Create a locally running chain
```bash
make anvil
```

2. Deploy the contract to the chain

```bash
make deploy
```

3. Run the storage tool
we use `1` because the storage slot of `s_password` in the contract.

you will get an output that looks like a byte32
`0x6d7950617373776f726400000000000000000000000000000000000000000014`
you can then parse that hex to a string with 

```bash
cast parse-bytes32-string 0x6d7950617373776f726400000000000000000000000000000000000000000014
```
and you will get an output of 
`my password`

**Recommended Mitigation:** 
You can try to implement a new way of encrypting the password maybe using the sha256


### [] TITLE has no access control, meaning a non-owner could change the password.

**Description:** The `PasswordStore::setPassword `function set to be an external function and there is no way to mitigate a non-owner from setting a new password. The natspec of the function and overall purpose of the smart contract is that `This function allows only the owneer to set a new password`

```javascript
    function setPassword(string memory newPassword) external {
            // there is no revert function
            s_password = newPassword;
            emit SetNetPassword();
        }

```

**Impact:** Anyone can set/change the password of the contract, severly breaking the contract intended functionality.

**Proof of Concept:** Add the following to the `PasswordStore.t.sol` test file.

<details>

```javascript
        function test_anyone_can_set_password(address randomAddress) public {
        vm.assume(randomAddress != owner);
        vm.prank(randomAddress);
        string memory expectedPassword = "mynewpassword";
        passwordStore.setPassword(expectedPassword);
        vm.prank(owner);
        string memory actualPassword = passwordStore.getPassword();
        assertEq(actualPassword, expectedPassword);
    }
```

</details>

**Recommended Mitigation:** Add an access control conditional to the `setPassword` function

```javascript
    if (msg.sender) != s_owner{
        revert PasswordStore__NotOwner();
    }
```

### [I-1] TITLE The `PasswordStore::getPassword` natspec indicates a parameter that doesn't exist, fix the natspec to be correct

**Description:** 
```javascript

    /*
     * @notice This allows only the owner to retrieve the password.
     * @param newPassword The new password to set.
     * @audit newPassword the new password to set.
     */
    function getPassword() external view returns (string memory) 
```
**Impact:** The natspec is incorrect



**Recommended Mitigation:** remove the incorrect natspec line

```diff
-   * @param newPassword The new password to set.
```