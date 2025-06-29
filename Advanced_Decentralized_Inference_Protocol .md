[ NODE OPERATOR / LLM PROVIDER ]
    |
    |--[1] Generate Identity:
    |     - node_pk, node_sk = Curve25519 keypair
    |
    |--[2] Publish Full Node Profile (off-chain):
    |     - JSON metadata: model_type, specs, latency, policy, etc.
    |     - Signed hash: H = SHA256(profile)
    |     - Save: profile.json → IPFS → CID = QmX...
    |
    |--[3] Register Node (on-chain):
    |     - Action: register_node({cid: QmX..., hash: H, signature})
    |     - Emitted: NodeRegistered event
    ↓
[ BLOCKCHAIN (FulaChain / Solana Subnet) ]
    |
    |-- Maintains anchor of profile: CID + signature
    |-- Indexable for discovery by model_type, reputation, etc.
    |-- Publicly verifiable signature
    |
    |--[4] Payment Contract
    |     - Function: pay_for_inference(node_id, tokens)
    |     - Emits: PaymentRecord(tx_hash, tokens, client_id)
    |
    |--[5] Reputation / SLA Tracker Contract
    |     - Tracks: inference_count, avg_latency, uptime %
    |     - Binds to: Oracle input + Validator reports
    ↓

[ ORACLE BRIDGE (Performance Watchdog) ]
    |
    |--[6] External Verifier or Bot:
    |     - Periodically queries nodes with test prompts
    |     - Measures: response time, availability, correctness
    |--[7] Signs report + submits on-chain
    |
    |-- Smart Contract Updates:
    |     - If node violates SLA → trigger `slash(node_id)`
    |     - Updates `reputation_score[node_id]`
    ↓

[ VERIFIER / AUDITOR NODES ]
    |
    |--[8] Optional ZK Challenge System:
    |     - Send encrypted blind input to node
    |     - Node returns commitment hash of output
    |     - Auditor decrypts and checks → submit zk-proof on-chain
    |
    |--[9] Used for:
    |     - Zero-Knowledge Quality Auditing
    |     - Privacy-preserving trust scoring
    |
    |-- If mismatch or inconsistency → Reputation slashing
    ↓

[ USER CLIENT ]
    |
    |--[10] Query Model Registry (on-chain):
    |     - Get CID → fetch profile.json from IPFS
    |
    |--[11] Show Models + SLA Score + Policy
    |--[12] Select Node, Fetch node_pk
    |--[13] Generate Ephemeral AES Session + Encrypt Message
    |--[14] Pay via `pay_for_inference` (on-chain tx)
    |--[15] Send RPC with ZK Token (or JWT) to node
    ↓

[ MODEL NODE ]
    |
    |--[16] Verify token or payment proof
    |--[17] Decrypt Message → Execute Model → Encrypt Response
    |--[18] Return Output + Optional Log Ref (IPFS)
    ↓

[ USER CLIENT ]
    |
    |--[19] Decrypt & Display Output
    |--[20] Optional: Store locally or publish encrypted to IPNS
