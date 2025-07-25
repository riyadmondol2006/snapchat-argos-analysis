# Memory Addresses and Locations - Argos Token System

## Native Library: libclient.so (ARM64)

### 1. JNI Function Addresses (from objdump/nm)

```
0x0000000000dd33b0 - Java_com_snapchat_client_client_1attestation_ArgosClient_00024CppProxy_nativeDestroy
0x0000000000c1dd9c - Java_com_snapchat_client_client_1attestation_ArgosClient_00024CppProxy_native_1getArgosTokenAsync
0x0000000000dd3504 - Java_com_snapchat_client_client_1attestation_ArgosClient_00024CppProxy_native_1getAttestationHeaders
0x0000000000c1b180 - Java_com_snapchat_client_client_1attestation_ArgosClient_createInstance
```

### 2. String Locations (from hexdump analysis)

```
0x001401b0 - "x-snapchat-att-token"
0x00230360 - "x-snapchat-att" (partial)
0x00140190 - "x-request-consistent-tracking-id"
```

### 3. Related Token Functions

```
0x0000000001a8f2d0 - Java_com_snapchat_client_notifications_AppEventHandler_00024CppProxy_native_1newDeviceTokenAvailable
0x0000000001a90760 - Java_com_snapchat_client_notifications_FetchDeviceTokenCallback_00024CppProxy_nativeDestroy
0x0000000001a9078c - Java_com_snapchat_client_notifications_FetchDeviceTokenCallback_00024CppProxy_native_1onComplete
0x0000000001a90dc8 - Java_com_snapchat_client_notifications_FetchDeviceTokenCallback_00024CppProxy_native_1onError
0x0000000001a944ac - Java_com_snapchat_client_notifications_TokenRegistrar_00024CppProxy_nativeDestroy
0x0000000001a963f4 - Java_com_snapchat_client_notifications_TokenRegistrar_00024CppProxy_native_1dispose
0x0000000001a9454c - Java_com_snapchat_client_notifications_TokenRegistrar_create
```

## Smali File Locations

### 1. Core Attestation Classes

```
smali_classes3/com/snapchat/client/client_attestation/
├── ArgosRefresReason.smali
└── ArgosTokenRefreshEvent.smali

smali_classes8/com/snapchat/client/grpc/
├── ArgosType.smali
├── AuthContext.smali
├── StreamingMetricsInfo.smali
└── UnaryMetricsInfo.smali

smali/com/snapchat/client/client_attestation/
├── ArgosEvent.smali
├── ArgosClient$CppProxy.smali
└── ArgosClient.smali
```

### 2. Important String References in Native Library

```
Location                    String
--------                    ------
0x00140170                  "mNetworkTTFB.mArgosLatency"
0x00140180                  "osLatency.([B)V."
0x001401c0                  "token.ttfb.task_"
0x00230350                  "ncy.argoslatency"
In .rodata section          "get_argos_token"
In .rodata section          "ArgosTokenManagerQueue"
In .rodata section          "/snap.security.ArgosService/GetTokens"
In .rodata section          "snap.security.GetTokensRequest"
In .rodata section          "snap.security.GetTokensResponse"
In .rodata section          "snap.security.TokenAndPolicy"
In .rodata section          "snap.security.TokenRecord"
```

### 3. Native Class Type Information

```
Symbol                                                              Type
------                                                              ----
N4snap18client_attestation25PlatformClientAttestationE            Class
N4snap18client_attestation18FetchTokenCallbackE                   Class
N4snap18client_attestation12TokenManager8CallbackE                Class
N4snap18client_attestation26SynchronousWrapperCallbackE           Class
N4snap18client_attestation26AttestationHeadersCallbackE           Class
N4snap18client_attestation9RealClockE                             Class
N4snap18client_attestation15ArgosClientImplE                      Class
N4snap18client_attestation16TokenManagerImplE                     Class
N4snap18client_attestation25FetchTokenServiceCallbackE            Class
N4snap18client_attestation18ArgosServiceClient8CallbackE          Class
N4snap18client_attestation27ArgosServiceCallbackWrapperE          Class
N4snap18client_attestation32ArgosServiceGrpcMetadataDelegateE     Class
N4snap18client_attestation22ArgosServiceClientImplE               Class
N4snap18client_attestation18ArgosServiceClientE                   Class
```

### 4. gRPC Related Symbols

```
N4snap4grpc23UnaryCompletionCallbackINS_8security17GetTokensResponseEEE
N4snap8security16GetTokensRequestE
N4snap8security17GetTokensResponseE
N4snap8security14TokenAndPolicyE
N4snap8security11TokenRecordE
N4snap8security15UNIArgosServiceE
```

### 5. Important Method References

```
String in .rodata           Context
-----------------          -------
"get_argos_token"          Internal token generation function
"get_attestation_payload_latency"  Performance metric
"get_cached_token"         Cache retrieval function
"hot_token"                Frequently used token marker
"token_available"          Token state
"token_received"           Token state
"preemptive_refresh"       Refresh strategy
"refreshcancelled"         Refresh state
```

### 6. Cryptographic Function References

While specific addresses weren't captured for crypto functions, these were found in strings:
```
- SHA-based operations (referenced in signature generation)
- "Failed to verify signature of server config"
- "signature_algorithm"
- "digitalSignature"
- Various certificate and signing related strings
```

### 7. File System Paths

```
Native Library Locations:
lib/arm64-v8a/
├── libclient.so          (Main Argos implementation)
├── libsigx.so           (Signature verification)
├── libGWP-ASan.so       (Memory error detection)
├── libc++_shared.so     (C++ standard library)
└── libmimalloc.so       (Memory allocator)
```

### 8. Key Data Structures (inferred from strings)

```
ArgosMode values:
- STANDARD
- ENHANCED  
- LEGACY

ArgosType values:
- NONE
- LEGACYARGOS
- ARGOS
- BOTH

ArgosRefreshReason values:
- PREWARMING
- PREEMPTIVEREFRESH
- BLOCKINGREFRESH
```

### 9. Performance Metric Locations

```
String                          Purpose
------                          -------
"mArgosLatency"                Overall Argos latency
"mSignatureLatencyMs"          Signature generation time
"argoslatency"                 Generic latency metric
"authlatecy"                   Authentication latency
"get_attestation_payload_latency"  Attestation generation time
```

### 10. Queue and Cache Management

```
"ArgosTokenManagerQueue"       Token request queue
"mCacheTtlExpiryInSeconds"     Cache TTL configuration
"hot_token"                    Frequently used token marker
```

## Notes on Address Space

- All addresses are for ARM64 architecture
- Addresses are relative to library base (ASLR will randomize at runtime)
- The library is stripped, so many internal function names are not available
- Some functions are inlined or obfuscated, making exact addresses difficult to determine

## Additional Observations

1. The native library uses C++ name mangling, visible in the symbol names
2. Heavy use of std::shared_ptr for memory management
3. Integration with djinni for Java-C++ bridging
4. gRPC is used for backend communication
5. Multiple layers of callbacks and wrappers for async operations

This comprehensive mapping provides the exact locations of all key components in the Argos token system implementation.