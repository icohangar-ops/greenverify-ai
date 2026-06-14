# Guardian Integration Study

## Source: hashgraph/guardian (140 ★)
**Apache-2.0 | Hedera Hashgraph | TypeScript**

## Key Patterns Extracted for GreenProof

### 1. Verifiable Credential (VC) Document Pipeline

Guardian's verification flow follows a block-based policy engine:

```
Request VC → Verify VC → Issue VC → Register VC
    ↓              ↓            ↓            ↓
 Schema        External      Hedera       HCS Topic
 Validation    Data Block    Consensus    Logging
```

**GreenProof adaptation:**
- Replace PSP34 NFT metadata with Verifiable Credential (VC) format
- Each verification produces a VC with: issuer, subject (project), claims (scores), proof (AI model hash)
- VC is minted on-chain as a credential, not just an NFT

### 2. Policy Workflow Engine (PWE)

Guardian uses a DAG of blocks where each block processes a verification step:

```typescript
// Block types relevant to GreenProof:
- RequestVcDocumentBlock:   Submit project documentation
- HttpRequestBlock:         Call AI verification API
- ExternalDataBlock:        Fetch on-chain project data
- CustomLogicBlock:         Run Qwen LLM scoring
- UploadVcDocumentBlock:    Mint verification credential
```

**GreenProof adaptation:**
- Define verification as a policy workflow: Submit → AI Score → Multi-verifier Consensus → Mint VC
- Each block is composable — other protocols can define their own verification policies

### 3. External Data Oracles

Guardian's `ExternalDataBlock` connects to off-chain data sources:

```typescript
// Pattern for fetching verification data from external sources
{
  blockType: 'externalData',
  config: {
    url: 'https://api.greenproof.io/verify/{projectId}',
    method: 'POST',
    headers: { 'Authorization': 'Bearer {token}' }
  }
}
```

**GreenProof adaptation:**
- Use the same oracle pattern to fetch real-world carbon project data
- Connect to registries (Verra, Gold Standard) for existing verification data
- AI verification supplements, not replaces, registry data

### 4. Registry HCS Topic

Guardian logs every verification to a Hedera Consensus Service topic, creating an immutable audit trail:

```
Policy Topic → Registry Topic → Project Topic
( methodology )  ( all credits )  ( per-project )
```

**GreenProof adaptation:**
- Use Portaldot's equivalent for audit trail
- Each verification event is logged to a dedicated topic
- Anyone can replay the verification history for any project

## Recommended GreenProof Changes

| Current | After Guardian Study |
|---------|---------------------|
| PSP34 NFT with metadata | Verifiable Credential (VC) format |
| Single AI verification | Multi-block policy workflow |
| Off-chain verification result | On-chain VC with proof |
| No audit trail | HCS-equivalent topic logging |
| Monolithic verification | Composable block-based pipeline |

## Reference Implementation

```python
# GreenProof verification as a policy workflow
verification_policy = PolicyWorkflow([
    SubmitDocumentation(),      # Block 1: Receive project docs
    FetchRegistryData(),        # Block 2: Check Verra/Gold Standard
    AIVerification(),           # Block 3: Qwen LLM scoring
    MultiVerifierConsensus(),   # Block 4: 3 verifiers must agree
    MintVerifiableCredential(), # Block 5: On-chain VC
    LogToAuditTopic(),          # Block 6: Immutable audit trail
])
```
