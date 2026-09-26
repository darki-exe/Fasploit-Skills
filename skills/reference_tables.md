# Quick Reference — Risk Levels & Full Function Index

> Use as a lookup table: risk level (low/medium/high) per technique, and a complete A-to-Z index of every executor function with its category and the section that covers it. Scan this first when unsure which section to read.

**Source:** `skill.md v3` — sections §30, §31  
**Executor reference:** Delta v73801 (semi-universal, UNC-compliant naming)

---

## 30. RISK REFERENCE TABLE

| Technique | Risk Level | Notes |
|---|---|---|
| Table overwrite (unfrozen) | 🟢 Low | Client memory only |
| Proxy table (frozen bypass) | 🟢 Low | Client-side only |
| cache.replace instance swap | 🟢 Low | Purely local reference trick |
| Module lazy patch (§2.3) | 🟡 Medium | Side effects possible on some modules |
| Remote spy / logging | 🟡 Medium | Passive only — no server interaction |
| Remote replay | 🟡 Medium | Server validates args — analyze structure first |
| Metatable hijack | 🟡 Medium | Executor-dependent; AC checks __namecall |
| Upvalue patching | 🟡 Medium | Requires debug library access |
| BodyVelocity speed | 🟡 Medium | Less flagged than WalkSpeed but still visible |
| RakNet desync | 🟡 Medium | Very powerful, very detectable on most games |
| FireServer spamming | 🔴 High | Rate limits → instant ban |
| Direct WalkSpeed write | 🔴 High | Server-side validation catches it |
| Direct Health write | 🔴 High | Server re-validates — caught fast |
| setthreadidentity(8) | 🔴 High | CoreScript level — very visible |
| loadstring from URL | 🔴 High | Flagged by Hyperion's behavioral heuristics |

---

---

## 31. FULL FUNCTION QUICK REFERENCE

| Function | Category | Notes |
|---|---|---|
| `safe_hook(fn, rep)` | Hook (custom) | pcall wrapper — §2.2 |
| `hookfunction(fn, rep)` | Hook | UNC standard |
| `hookmetamethod(obj, mm, rep)` | Hook | Hooks metamethods |
| `hookproto(fn, rep)` | Hook | Proto-level hook |
| `oth.hook(fn, rep)` | Hook | Off-thread hook — §19 |
| `newcclosure(fn)` | Closure | Bypasses islclosure |
| `newlclosure(fn)` | Closure | Lua closure wrapper |
| `clonefunction(fn)` | Closure | Different pointer, same body |
| `replaceclosure(fn, rep)` | Closure | In-place body swap |
| `restoreclosure(fn)` | Closure | Restore original |
| `restoreproto(fn)` | Closure | Restore proto |
| `getgenv()` | Environment | Executor global |
| `getrenv()` | Environment | Roblox global |
| `getsenv(script)` | Environment | Script env |
| `getmenv(module)` | Environment | Module env (Delta) |
| `getreg()` | Registry | Raw Lua registry |
| `getgc(bool?)` | GC | All live GC objects |
| `filtergc(type, opts)` | GC | Filtered GC scan |
| `getinstances()` | Instance | All instances |
| `getnilinstances()` | Instance | Nil-parented |
| `getinstancecache()` | Instance | Internal cache |
| `gethui()` | Instance | Hidden CoreGui |
| `cloneref(obj)` | Instance | New reference, same object |
| `compareinstances(a,b)` | Instance | True if same underlying obj |
| `cache.replace(old, new)` | Cache | Redirect instance references |
| `cache.invalidate(obj)` | Cache | Clear from cache |
| `cache.iscached(obj)` | Cache | Check cache presence |
| `getscripts()` | Script | All scripts |
| `getloadedmodules()` | Script | Required modules |
| `getscriptbytecode(s)` | Script | Raw bytecode |
| `getscriptclosure(s)` | Script | Script closure |
| `getscriptfromthread(t)` | Script | Script from thread |
| `getrunningscripts()` | Script | Currently running |
| `getcallingscript()` | Script | Caller script |
| `getconnections(sig)` | Signal | Signal connections |
| `firesignal(sig, ...)` | Signal | Fire directly |
| `cansignalreplicate(sig)` | Signal | Check replicate |
| `replicatesignal(sig,...)` | Signal | Force replicate |
| `fireclickdetector(cd, d)` | Instance | Trigger click |
| `fireproximityprompt(pp)` | Instance | Trigger prompt |
| `firetouchinterest(p,t,i)` | Instance | Fake touch |
| `gethiddenproperty(o,k)` | Property | Read hidden |
| `get_hidden_properties(o)` | Property | All hidden props |
| `sethiddenproperty(o,k,v)` | Property | Write hidden |
| `setscriptable(o,k,bool)` | Property | Unlock non-scriptable |
| `getrenderproperty(o,k)` | Property | Render-side read |
| `setrenderproperty(o,k,v)` | Property | Render-side write |
| `getthreadidentity()` | Thread | Current level |
| `setthreadidentity(n)` | Thread | Set level |
| `debug.getupvalue(fn,i)` | Debug | Read upvalue |
| `debug.setupvalue(fn,i,v)` | Debug | Write upvalue |
| `debug.getupvalues(fn)` | Debug | All upvalues (table) |
| `debug.getconstants(fn)` | Debug | Bytecode constants |
| `debug.setconstant(fn,i,v)` | Debug | Patch constant |
| `debug.getprotos(fn)` | Debug | Sub-function protos |
| `debug.getstack(l,i)` | Debug | Read stack |
| `debug.getcallstack()` | Debug | Traceback string |
| `decompile(fn/script)` | Debug | Source decompile |
| `dumpbytecode(fn)` | Debug | Raw bytecode |
| `getrawmetatable(obj)` | Metatable | Raw metatable |
| `setreadonly(t,bool)` | Metatable | Toggle freeze |
| `isreadonly(t)` | Metatable | Check freeze |
| `islclosure(fn)` | Closure | Is Lua closure |
| `iscclosure(fn)` | Closure | Is C closure |
| `isexecutorclosure(fn)` | Closure | Is executor closure |
| `isfunctionhooked(fn)` | Hook | Is hooked |
| `isprotohooked(fn)` | Hook | Proto hooked |
| `RakNet.desync()` | Network | Desync client |
| `RakNet.is_enabled()` | Network | RakNet active |
| `WebSocket.connect(url)` | Network | WS connection |
| `request(config)` | Network | HTTP request |
| `crypt.encrypt(d,k,iv,m)` | Crypto | AES encrypt |
| `crypt.decrypt(d,k,iv,m)` | Crypto | AES decrypt |
| `crypt.hash(d, algo)` | Crypto | Hash string |
| `crypt.generatekey()` | Crypto | Random key |
| `lz4compress(data)` | Compress | LZ4 compress |
| `lz4decompress(data)` | Compress | LZ4 decompress |
| `zstdcompress(data)` | Compress | Zstd compress |
| `zstddecompress(data)` | Compress | Zstd decompress |
| `base64.encode(s)` | Encoding | Base64 encode |
| `base64.decode(s)` | Encoding | Base64 decode |
| `writefile(p, s)` | File | Write file |
| `readfile(p)` | File | Read file |
| `appendfile(p, s)` | File | Append to file |
| `isfile(p)` | File | File exists |
| `isfolder(p)` | File | Folder exists |
| `makefolder(p)` | File | Create folder |
| `listfiles(p)` | File | List directory |
| `delfile(p)` | File | Delete file |
| `loadfile(p)` | File | Load as Lua chunk |
| `dofile(p)` | File | Load + execute |
| `setclipboard(s)` | Clipboard | Copy to clipboard |
| `rconsolecreate()` | Console | Open console |
| `rconsoleprint(s)` | Console | Print to console |
| `rconsoleinput()` | Console | Read user input |
| `rconsoledestroy()` | Console | Close console |
| `mousemoveabs(x,y)` | Input | Move mouse (abs) |
| `mousemoverel(x,y)` | Input | Move mouse (rel) |
| `mouse1click()` | Input | Left click |
| `keypress(vk)` | Input | Press key |
| `keyclick(vk)` | Input | Press + release |
| `setfpscap(n)` | Misc | Set FPS cap |
| `getfpscap()` | Misc | Get FPS cap |
| `setfflag(n, v)` | Misc | Override FFlag |
| `getfflag(n)` | Misc | Read FFlag |
| `gethwid()` | Misc | Hardware ID |
| `getexecutorname()` | Misc | Executor name |
| `queue_on_teleport(s)` | Misc | Run after teleport |
| `messagebox(t, title, f)` | Misc | Dialog box |
| `isnetworkowner(part)` | Misc | Network ownership |
| `setsimulationradius(n)` | Misc | Sim radius |
| `get_actors()` | Actor | All actors |
| `run_on_actor(a, code)` | Actor | Execute on actor |
| `create_comm_channel()` | Actor | Actor IPC channel |
| `isparallel()` | Actor | In parallel context |
| `_G.UNLOAD()` | Cleanup | Full teardown — §2.1 |

---

*— skill.md v3: all English, Delta API dump integrated, Fun Facts & History section added, oth/RakNet/Actor/Crypto/Console/Input sections added, full function reference expanded*
