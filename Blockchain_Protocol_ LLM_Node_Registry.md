[ LLM PROVIDER / NODE OPERATOR ]
    |
    |--[1] Generate Node Keys
    |     - node_pk, node_sk = EC keypair (Curve25519)
    |
    |--[2] Create Node Profile Metadata:
    |     {
    |       node_id: hash(node_pk),
    |       model_type: "mistral-7b-instruct",
    |       version: "1.0.0",
    |       performance: { latency: 200ms, rps: 10 },
    |       endpoint_uri: https://nodeX.ai:443/infer,
    |       pricing: { cost_per_token: 0.001 SOL },
    |       storage_policy: ["logs_disabled", "no_training"],
    |       public_key: node_pk,
    |       owner_wallet: <wallet_address>,
    |       signature: Sign(sk_node, hash(all_fields))
    |     }
    |
    |--[3] Submit Node Registration Transaction:
    |     - Chain: FulaChain / Solana
    |     - Action: `register_model_node(NodeProfile)`
    |
    ↓
[ BLOCKCHAIN (Custom Subnet / Fula-compatible) ]
    |
    |-- Stores model metadata in decentralized registry:
    |     - key = node_id
    |     - value = metadata, signature, timestamp
    |     - Verifiable by all users
    |
    |-- Enables lookups:
    |     - `get_model_list(filter)`
    |     - `get_model_info(node_id)`
    |
    ↓

[ USER CLIENT ]
    |
    |--[4] Discover available models:
    |     - Calls: `get_model_list({model_type: "mistral"})`
    |
    |--[5] Choose model → fetch public_key, uri, pricing
    |--[6] Encrypt session message with node_pk (as per secure RPC spec)
    |
    |--[7] Execute payment via smart contract:
    |     - `pay_for_inference(node_id, tokens)`
    |     - Emits event: `UsageRecord {client, node_id, tx_hash, token_count}`
    |
    ↓

[ SMART CONTRACT: Usage & Reputation ]
    |
    |--[8] Record usage logs:
    |     - Maps: client → node → usage count, avg latency
    |     - Stores hash of encrypted payload + time
    |
    |--[9] Update node stats:
    |     - InferenceCount++, LastSeen = now
    |     - Slash if node fails challenge (see below)
    |
    |--[10] Optional: Reputation challenge
    |     - Auditors or validators can send test queries
    |     - If node fails, reputation score drops / staking slashed

[ OPTIONAL: ZK Token Access Layer ]
    |
    |-- Users receive signed JWT / ZK Token as proof-of-payment
    |-- Used in RPC headers to access node securely
