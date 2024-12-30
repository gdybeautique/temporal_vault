# Temporal Message Vault

## Overview
The **Temporal Message Vault** is a Clarity-based smart contract that allows users to securely store encrypted messages that can only be accessed after a specified block height. The system leverages the blockchain's immutability and time-based features to ensure messages remain inaccessible until the defined unlock height is reached.

### Key Features:
- **Secure Storage**: Messages are stored securely on the blockchain.
- **Time-locked Access**: Messages can only be revealed after the specified block height is reached.
- **Ownership**: Only the creator (owner) of a capsule can manage or access its details.
- **Immutable Records**: Stored messages and their unlock criteria are tamper-proof.

---

## Contract Constants

### `contract-owner`
Defines the address of the contract owner, which is the deploying principal (`tx-sender`).

### Error Codes:
- **`err-owner-only (u100)`**: Raised when a non-owner tries to perform owner-only actions.
- **`err-already-exists (u101)`**: Raised if a capsule ID already exists (not currently used).
- **`err-not-found (u102)`**: Raised if a capsule ID is not found.
- **`err-too-early (u103)`**: Raised if an attempt is made to reveal a message before its unlock height.
- **`err-invalid-blocks-locked (u104)`**: Raised if the `blocks-locked` parameter is invalid (<= 0).
- **`err-unauthorized (u105)`**: Raised if a non-owner tries to access or modify a capsule.

---

## Data Structures

### `capsules`
A map that stores the following details for each capsule:
- **`capsule-id (uint)`**: Unique identifier for the capsule.
- **`owner (principal)`**: The principal address of the capsule creator.
- **`message (string-utf8 500)`**: The encrypted message.
- **`unlock-height (uint)`**: The block height after which the message can be revealed.
- **`is-revealed (bool)`**: A flag indicating whether the capsule has been revealed.

### `next-capsule-id`
A data variable that tracks the next available capsule ID.

---

## Public Functions

### 1. `create-capsule`
Creates a new time-locked message capsule.

#### Parameters:
- **`message (string-utf8 500)`**: The message to store.
- **`blocks-locked (uint)`**: The number of blocks to lock the message for.

#### Returns:
- **`capsule-id (uint)`**: The unique ID of the created capsule.

#### Logic:
1. Ensure `blocks-locked > 0`.
2. Calculate the `unlock-height` as `block-height + blocks-locked`.
3. Store the capsule details in the `capsules` map.
4. Increment the `next-capsule-id`.

---

### 2. `reveal-capsule`
Reveals the message of a capsule if the unlock height has been reached.

#### Parameters:
- **`capsule-id (uint)`**: The ID of the capsule to reveal.

#### Returns:
- **`message (string-utf8 500)`**: The revealed message.

#### Logic:
1. Fetch the capsule details using the provided `capsule-id`.
2. Ensure the current block height (`block-height`) is greater than or equal to the capsule's `unlock-height`.
3. Ensure the caller is the owner of the capsule.
4. Return the message.

---

### 3. `get-capsule-details`
Retrieves details of a specific capsule. This function is read-only.

#### Parameters:
- **`capsule-id (uint)`**: The ID of the capsule to fetch.

#### Returns:
- **`unlock-height (uint)`**: The block height at which the capsule can be revealed.
- **`is-revealed (bool)`**: Whether the capsule has been revealed.

#### Logic:
1. Fetch the capsule details using the `capsule-id`.
2. Ensure the caller is the owner of the capsule.
3. Return the capsule details.

---

### 4. `get-total-capsules`
Provides the total number of capsules created. This function is read-only.

#### Returns:
- **`total-capsules (uint)`**: The total number of capsules stored in the contract.

#### Logic:
1. Subtract 1 from the value of `next-capsule-id` to get the count.

---

## Private Functions

### `store-capsule`
An internal function to insert a new capsule into the `capsules` map.

#### Parameters:
- **`capsule-id (uint)`**: The unique ID for the capsule.
- **`message (string-utf8 500)`**: The message to store.
- **`unlock-height (uint)`**: The block height after which the message can be revealed.

#### Returns:
- **`true`**: If the capsule is stored successfully.

#### Logic:
1. Insert the capsule details into the `capsules` map.

---

## Usage Example

### 1. Create a Capsule
```clarity
(create-capsule "Secret Message" u100)
```
This creates a capsule with the message "Secret Message" that will unlock after 100 blocks.

### 2. Reveal a Capsule
```clarity
(reveal-capsule u1)
```
This reveals the message of the capsule with ID `1` if the unlock height has been reached.

### 3. Get Capsule Details
```clarity
(get-capsule-details u1)
```
This retrieves details for the capsule with ID `1`, including the unlock height and whether it has been revealed.

### 4. Get Total Capsules
```clarity
(get-total-capsules)
```
This returns the total number of capsules created in the contract.

---

## Security Considerations
- **Access Control**: Only the owner of a capsule can reveal its message or fetch its details.
- **Time-lock Enforcement**: Messages cannot be accessed before the specified block height.
- **Data Integrity**: Capsules are immutable once created.

---

## Deployment
To deploy the contract, use a Clarity-compatible blockchain platform (e.g., Stacks). Ensure the deploying principal has sufficient balance for contract deployment.

---

## Future Enhancements
- **Message Encryption**: Add optional encryption/decryption mechanisms for enhanced security.
- **Capsule Deletion**: Implement a feature to allow owners to delete capsules.
- **Event Logging**: Emit events on capsule creation and reveal for better tracking.

