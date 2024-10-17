# Almost in all smart contracts
- `65535` - unknown opcode. In smartcontracts looks like `throw(0xffff)`. Custom error, is being used in almost all smartcontracts existing. Meaning that this smartcontract doesn't know opcode that came to it. E.g. sending *op::transfer* to *jetton_master* will trigger 65535 error.
# Jetton
#### Jetton Master Contract
- `73` - Ownership check. Sender of the message must be admin.
- `74` - Sender of the *burn_notification* message must be jetton_wallet of this master contract.
- `75` - Provide wallet address gas check error. Incoming TON value must be `> fwd_fee + 0.01 TON`.
#### Jetton Wallet Contract
- `705` - Ownership check. Sender of the message must be owner of this *jetton_wallet*.
- `706` - Not enough balance. Balance on this *jetton_wallet* must be >= 0.
- `707` - Sender of the *internal_transfer* message must be either *jetton_wallet* of the same token or *jetton_master* of this *jetton_wallet* (when *jetton_master* is minting tokens it sends *internal_transfer* message in its body)
- `707` (in *burn_tokens*) - Gas check error. Incoming value must be `fwd_fee + 2 * 0.015`.
- `708` - Forward payload either bit is missing.
  - ***Forward payload tip:***
    If you don't want to store forward payload, just do `.store_uint(0, 1)` and finish your cell. If you want to add forward payload (e.g. some comment or any other custom operation) and your forward payload is in the same cell, do `.store_uint(0, 1).store_...<your forward payload here>`. If your forward payload is in reference cell, do `.store_uint(1, 1).store_ref(<your forward payload cell here>)`    
- `709` - Gas check error. Incoming TON value must be `> forward_ton_amount + fwd_count (= 1 if forward_ton_amount is 0, = 2 if foward_ton_amount > 0) * fwd_fee + (2 * 0.015 + 0.01)`.
   - ***forward_ton_amount*** **tip:** Basically it's recommended to send value that is twice as much as *forward_ton_amount*, if it's > 0.
- **Alternative** `709` **(not to be confused with 709 Gas check error)** - Pretty rare error, but nevertheless can occur. When *op::internal_transfer* or *op::burn_notification* are bounced (something went wrong) the tokens should be returned to *jetton_wallet* that initiated those opcodes according to the jetton logic. Any other opcodes when bounced will trigger this `709` error.
# NFT
#### NFT Collection Contract
- `401` - Ownership check. Sender of this message must be owner (admin) of this NFT Collection.
- `402` - Minting error. Invalid NFT item_index. NFT item_index must be <= next_item_index.
- `399` - Batch mint error. Total NFTs in batch mint must be <= 250 nft_items due to limits in action list size. **However on practice you can mint 120-130 in batch per transaction due to computation fee limit of 1 TON**.
- `403 + counter (0 <= counter <= n)` - Batch mint error. One of NFTs in the list has invalid index. E.g. 0,1,2,3,*9*.
#### NFT Item Contract
- `401` - Ownership check. Sender of this message must be owner of this NFT.
- `402` - Rest amount error. *Rest amount = balance of nft_item contract - 0.05 (min_storage)*. Meaning that when *transfer_ownership* or *transfer_editorship* are invoked, *Rest amount* should remain positive after deducting fees (forward_amount, fwd_fee).
- `405` - If NFT is not initialized, `init?` field is false (0) the sender of initialization message must be NFT Collection that this NFT belongs to.
- `410` **(in nft_item_editable)** - Sender of *edit_content* message must be *editor_address*.
  
# Coming soon...
