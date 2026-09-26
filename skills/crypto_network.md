# Crypto, Compression, HTTP, WebSocket & RakNet

> Use when handling encrypted data, compressing payloads, making HTTP requests from the executor, opening WebSocket connections, or using low-level RakNet desync. Covers base64, AES, hash/HMAC, lz4/zstd, request(), WebSocket.connect(), RakNet.desync().

**Source:** `skill.md v3` — sections §13, §14, §15  
**Executor reference:** Delta v73801 (semi-universal, UNC-compliant naming)

---

## 13. CRYPTO, COMPRESSION & ENCODING

```lua
-- Base64
local encoded = base64.encode("hello world")        -- "aGVsbG8gd29ybGQ="
local decoded = base64.decode(encoded)               -- "hello world"
-- Aliases: base64_encode, base64encode, crypt.base64.encode

-- AES encryption (crypt module)
local key       = crypt.generatekey()               -- random 32-byte key
local encrypted = crypt.encrypt("secret data", key, nil, "CBC")
local decrypted = crypt.decrypt(encrypted, key, nil, "CBC")
print(decrypted)  -- "secret data"

-- Hash
local hash = crypt.hash("hello", "sha256")
print(hash)

-- HMAC
local hmac = crypt.hmac("message", key, "sha256")
print(hmac)

-- Random bytes
local bytes = crypt.generatebytes(16)  -- 16 random bytes as string

-- LZ4 compression (fast, lower ratio)
local compressed   = lz4compress("large string data here")
local decompressed = lz4decompress(compressed)
-- Aliases: lz4_compress, crypt.lz4compress

-- Zstandard compression (slower, better ratio)
local zcompressed   = zstdcompress("large string data here")
local zdecompressed = zstddecompress(zcompressed)
-- Aliases: zstd_compress, zstdcompress
```

---

---

## 14. WEBSOCKET & HTTP

```lua
-- HTTP request (full)
local res = request({
    Url     = "https://api.example.com/data",
    Method  = "POST",
    Headers = {
        ["Content-Type"]  = "application/json",
        ["Authorization"] = "Bearer token123",
    },
    Body = '{"value": 42}',
})
print(res.StatusCode)   -- 200
print(res.Body)         -- response body string
print(res.Headers)      -- response headers table

-- WebSocket (bidirectional)
local ws = WebSocket.connect("ws://localhost:8080")
ws.OnMessage:Connect(function(msg) print("Received:", msg) end)
ws.OnClose:Connect(function()      print("Closed")         end)
ws:Send("ping")
ws:Close()
```

---

---

## 15. RAKNET (LOW-LEVEL NETWORK)

```lua
-- RakNet gives access to the raw packet layer — below RemoteEvents.
-- Delta exposes it as: RakNet, Raknet, rnet (all aliases).

-- Check if RakNet interception is enabled
print(RakNet.is_enabled())  -- bool

-- desync() — desynchronizes the client from the server's network tick
-- Can be used for position desync exploits (server sees you in one place, you're elsewhere).
-- Use with extreme caution — very detectable on games with server-side position validation.
RakNet.desync()
```

---
