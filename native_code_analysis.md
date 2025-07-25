# Native Code Analysis - libclient.so

## Key Native Functions Found

### 1. Token Generation Functions
- `get_argos_token` - Core token generation function
- `Java_com_snapchat_client_client_1attestation_ArgosClient_00024CppProxy_native_1getAttestationHeaders`
- `Java_com_snapchat_client_client_1attestation_ArgosClient_00024CppProxy_native_1getArgosTokenAsync`

### 2. JNI Bindings
The native library uses JNI (Java Native Interface) to communicate with Java code:
```
Java_com_snapchat_client_client_1attestation_ArgosClient_createInstance
```

### 3. Token Header Format
- Header name: `x-snapchat-att-token`
- The token is generated in native code and returned to Java layer

### 4. Dependencies
The native library depends on:
- `libc++_shared.so` - C++ standard library
- Platform-specific attestation APIs
- Cryptographic libraries (likely embedded)

### 5. Security Measures
The native implementation likely includes:
- Hardware attestation (SafetyNet on Android, DeviceCheck on iOS)
- Certificate pinning
- Anti-debugging techniques
- Obfuscation and packing
- Runtime integrity checks

### 6. Token Generation Flow
1. Java layer calls native method with request parameters
2. Native code performs:
   - Device attestation
   - Timestamp generation
   - Cryptographic signing
   - Token assembly
3. Token is returned to Java layer
4. Java adds token to HTTP headers

### 7. Additional Libraries
Other native libraries in the APK:
- `libsigx.so` - Likely for signature verification
- `libferrite-launcher-4fae2d0.so` - Unknown purpose
- `libferrite-tracer-4fae2d0.so` - Possibly for tracing/debugging
- `libGWP-ASan.so` - Memory error detection
- `libarcore_sdk_c.so` / `libarcore_sdk_jni.so` - AR Core (for AR features)

## Reverse Engineering Challenges
1. The actual algorithm is in compiled native code
2. Likely uses code obfuscation techniques
3. May include anti-debugging measures
4. Platform-specific security features are used
5. Cryptographic keys are likely embedded and protected

## Next Steps for Deep Analysis
To fully reverse engineer the token generation:
1. Use IDA Pro or Ghidra for static analysis of libclient.so
2. Use Frida for dynamic analysis and hooking
3. Analyze the JNI calls and parameters
4. Trace the execution flow during token generation
5. Identify cryptographic operations and key derivation

Note: Full reverse engineering would require significant effort and specialized tools.