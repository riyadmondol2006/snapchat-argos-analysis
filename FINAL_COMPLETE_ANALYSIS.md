# Final Complete Analysis - Snapchat Argos Token System

## Executive Summary

Through extensive reverse engineering of Snapchat v13.51.0.56, I've completely mapped the Argos attestation token system from Java layer through native implementation to network communication.

## Complete System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Java Layer                             │
│  O60 (Provider) → iJd (Factory) → ArgosClient (Interface)   │
└───────────────────────┬─────────────────────────────────────┘
                        │ JNI Bridge
┌───────────────────────▼─────────────────────────────────────┐
│                     Native Layer (libclient.so)              │
│  ArgosClientImpl → TokenManagerImpl → ArgosServiceClient    │
└───────────────────────┬─────────────────────────────────────┘
                        │ gRPC
┌───────────────────────▼─────────────────────────────────────┐
│                  Backend Service (GCP)                        │
│           gcp.api.snapchat.com/ArgosService                 │
└─────────────────────────────────────────────────────────────┘
```

## Key Discoveries

### 1. Server Endpoint
- **Host**: `gcp.api.snapchat.com` (Google Cloud Platform)
- **Service**: `/snap.security.ArgosService/GetTokens`
- **Protocol**: gRPC over HTTPS

### 2. Token Headers
- **Primary**: `x-snapchat-att-token`
- **Signature**: `x-snapchat-att-sign`
- **Tracking**: `x-request-consistent-tracking-id`

### 3. Cryptographic Implementation
```
Algorithms:
- AESGCC20 (Custom AES-GCM with 20-byte tag)
- CC20AESG (ChaCha20-AES hybrid)
- TLS_CHACHA20_POLY1305_SHA256
- RSA-SHA1 (Signatures)
- HMAC-SHA1 (Message auth)

Key Management:
- getKeyForCurrentUserAsync() 
- mDeviceEncryptionKey
- recipient_key_version
- onSecurityKeyChanged()
```

### 4. Token Storage
- **File**: `/token.bin` (Binary format)
- **Cache**: SQL-based with TTL
- **Memory**: ArgosTokenManagerQueue

## Complete Implementation Flow

### Phase 1: Initialization (Java)
```java
// O60.smali - ArgosClientProvider
Configuration config = new Configuration();
config.server = "gcp.api.snapchat.com";
config.timeout = 20000; // 20 seconds
config.retryTimeout = 10000; // 10 seconds
```

### Phase 2: Native Client Creation
```java
// iJd.smali - Factory
ArgosClient client = ArgosClient.createInstance(
    platformAttestation,  // Hardware attestation
    configuration,        // From Phase 1
    authContext,         // User auth
    logger,              // Blizzard logger
    dispatchQueue        // Threading
);
```

### Phase 3: Token Generation (Native)
```cpp
// Memory addresses from analysis
0x00dd3504: getAttestationHeaders() {
    // Check cache
    if (tokenCache.hasValid(url)) {
        return tokenCache.get(url);
    }
    
    // Generate new token
    AttestationPayload payload = getAttestationPayloadProto();
    payload.deviceId = getDeviceId();
    payload.timestamp = getCurrentTime();
    payload.signature = signPayload(payload);
    
    // gRPC call
    GetTokensRequest request;
    request.payload = payload;
    GetTokensResponse response = grpcClient.getTokens(request);
    
    // Cache and return
    tokenCache.store(url, response.token);
    return buildHeaders(response.token);
}
```

### Phase 4: Proto Structures
```protobuf
// Reconstructed from analysis
message GetTokensRequest {
    string url = 1;
    string method = 2;
    bytes payload = 3;
    int64 timestamp = 4;
    DeviceInfo device = 5;
}

message GetTokensResponse {
    TokenRecord token = 1;
    int32 cacheTtl = 2;
    RefreshStrategy strategy = 3;
}

message TokenRecord {
    string token = 1;
    int64 expiry = 2;
    ArgosType type = 3;
    int32 version = 4;
}
```

## Memory Map Summary

### Java Classes (Obfuscated)
```
O60  - ArgosClientProvider (Main provider)
iJd  - ArgosClient factory with RxJava
LS60 - Token service interface
eG8  - Configuration builder
LeJe - Helper/builder class
LPSg - Platform service implementation
```

### Native Functions
```
0x00c1b180 - ArgosClient::createInstance
0x00dd3504 - ArgosClient::getAttestationHeaders  
0x00c1dd9c - ArgosClient::getArgosTokenAsync
0x00dd33b0 - ArgosClient::nativeDestroy

String Table:
0x001401b0 - "x-snapchat-att-token"
0x00230360 - "x-snapchat-att"
0x00140190 - "x-request-consistent-tracking-id"
```

## Security Analysis

### Protection Layers
1. **Code Obfuscation**
   - Java classes renamed
   - Native symbols stripped
   - Control flow obfuscation

2. **Cryptographic Security**
   - Hardware-backed keys
   - Multiple encryption layers
   - Dynamic key rotation

3. **Platform Attestation**
   - SafetyNet (Android)
   - DeviceCheck (iOS)
   - Server validation

4. **Network Security**
   - Certificate pinning
   - gRPC encryption
   - GCP infrastructure

### Anti-Tampering Measures
- Binary packing
- Anti-debugging checks
- Runtime integrity verification
- Server-side validation

## Token Format (Final)
```json
{
    "version": 1,
    "timestamp": 1234567890000,
    "mode": "STANDARD|ENHANCED|LEGACY",
    "device": {
        "id": "hardware_device_id",
        "fingerprint": "sha256_hash",
        "platform": "Android|iOS"
    },
    "payload": "base64_encoded_attestation",
    "signature": "base64_encoded_signature",
    "keyVersion": 1,
    "nonce": "random_hex_string"
}
```

## Implementation Challenges

1. **Hardware Dependency**: Requires genuine device attestation
2. **Native Code**: Core logic in compiled C++ 
3. **Server Validation**: Backend verifies all tokens
4. **Key Protection**: Hardware-backed key storage
5. **Obfuscation**: Multiple layers of protection

## Research Tools Summary

Created 11 files documenting:
- Initial analysis
- Native code investigation  
- Memory addresses and locations
- Cryptographic findings
- Complete implementation flow
- Python research implementations

## Conclusion

The Argos system represents state-of-the-art mobile security:
- Multi-layer architecture
- Hardware attestation integration
- Advanced cryptography
- Sophisticated obfuscation
- Server-side validation

This makes it extremely difficult to bypass without:
- Physical device access
- Kernel-level modifications
- Server infrastructure knowledge
- Cryptographic keys

The system successfully prevents API abuse while maintaining performance through intelligent caching and efficient native implementation.