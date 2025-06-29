[ USER CLIENT (dApp - Web/Mobile) ]
    |
    |--[1] Connect Wallet (Metamask / WalletConnect)
    |     - Identity = Public Key (ECDSA / Ed25519)
    |     - Optional: Sign session token
    |
    |--[2] Model Selection:
    |     - Fetch metadata from blockchain (Model Registry)
    |     - Includes: Node URI + Public Key (RSA-2048 / Curve25519)
    |
    |--[3] Message Preparation:
    |     - Generate Ephemeral AES-256 Session Key (K_session)
    |     - Encrypt message: E_payload = AES(K_session, Message)
    |     - Encrypt session key: E_key = RSA/EC_Encrypt(PK_Model, K_session)
    |     - Create Payload:
    |         {
    |           encrypted_key: E_key,
    |           encrypted_payload: E_payload,
    |           client_id: wallet_address,
    |           nonce: random,
    |           timestamp: T
    |         }
    |
    |--[4] Send Encrypted Payload to Model RPC Endpoint
          (REST or gRPC over TLS + Signature Header)
    |
[ ROUTER (Box Local Proxy) ]
    |-- Checks if model is hosted locally:
    |     - YES → Dispatch to Local Container
    |     - NO → Forward to remote node via encrypted channel
    |
    |-- Optionally: Verify payment via on-chain tx or signed token
    |
    ↓

[ MODEL EXECUTION NODE (Local or Remote) ]
    |
    |--[5] Decrypt session key: K_session = RSA/EC_Decrypt(SK_Model, E_key)
    |--[6] Decrypt payload: Message = AES_Decrypt(K_session, E_payload)
    |
    |--[7] Launch model inside sandbox:
    |     - Docker / Firecracker microVM
    |     - Allocated memory (quota-limited)
    |     - Time-based execution (e.g. 10s max)
    |
    |--[8] Generate Response: R
    |
    |--[9] Encrypt Response:
    |     - R_payload = AES(K_session, R)
    |
    |--[10] (Optional) Response Metadata:
    |     - ipfs_ref = Encrypted(IPFS_CID of log)
    |     - hash = SHA256(Message + R)
    |
    |--[11] Return Encrypted Response:
    |         {
    |           encrypted_payload: R_payload,
    |           proof: [signature or hash],
    |           storage_ref: ipfs_ref (optional)
    |         }
    ↓

[ USER CLIENT (dApp) ]
    |
    |--[12] Decrypt Response: AES_Decrypt(K_session, R_payload)
    |--[13] Show to user
    |
    |--[14] Optional: Store session log (chat + response)
    |     - Encrypted by user’s own public key
    |     - Pinned to IPFS
    |     - Stored CID on local Box or public IPNS
