# HCaptcha reverse engineered

```
this repo (and more generally all those linked to implex) are the result of countless hours of work, a lot of learning and new things.
Take the time to look and learn, instead of copying without thinking, because it won't work. If you need a developer, contact me.
```

```
⚠️ PS: I see more and more people copying/stealing this script and reuploading it without credit on github, that's why I'm posting an update, be careful who you pay services to, especially towards those people who are just thieves and don't know how to do anything on their own.
```

## Hcaptcha update logs !

- `1.40.16` They added "obfuscation" to the wasm, but hey!
- `1.40.20?` They removed `version` params (*+ fixed rand?*) (now ima count 1.40.X)
- `1.40.21` They renamed wasm bindgen binding name !!
- `1.40.22` new param `ardata`
- `1.40.23` dynamic numbers for `fingerprint_events` + fix `ardata` always null
- `1.40.25` `fingerprint_events` order change ??
- `1.60.0` New VM
- `1.80.0?` New obfuscation + VM for motiondata which gets pushed into N data 

## String integrity check (outdated)

[This script](https://gist.github.com/nikolahellatrigger/a8856463170fbe3596569977148ebaf4) is used to "encode" somes data into `fingerprint_event` field such as:
    
    - webgl vendor + renderer
    - browser performance
    - browser timezone
    
I think it's used to verify the data is authentic / non duplicated (output is different each time you run the function)

## Lib used by WASM

- https://crates.io/crates/rand_chacha/0.2.2 (encryption)
- https://crates.io/crates/cipher/0.3.0 (encryption)
- https://crates.io/crates/ctr/0.8.0 (encryption)
- https://crates.io/crates/rust-hashcash/0.3.3 (stamp)
- https://crates.io/crates/aes/0.7.5 (encryption)
- https://crates.io/crates/js-sys/0.3.52 (javascript)
- https://crates.io/crates/twox-hash/1.6.0 (hash)

## Stamp (proof of work)

[Hashcash](https://crates.io/crates/rust-hashcash/0.3.3) algorithm is used to generate stamp value as a POW with custom date format (`2006-01-02`), bits is set by using the difficulty present into the JWT 

## Fingerprint hash

[XxHash3 (sixty_four.rs)](https://crates.io/crates/twox-hash/1.6.0) algorithm is used with custom seed (`5575352424011909552`) to create unique hash of 15 unique properties such as:

    - Html DOM
    - Webgl properties
    - Css properties
    - Javascript window functions
    - ...
    

## Rand

Rand is a `CRC-32` checksum hash of the N payload in json format, it's used to check the payload integrity if you edited it from memory etc...
Format: `[math.random, crc-32 * 2.3283064365386963e-10]` (`table: 79764919`)

## Encryptions

There are two encryptions in wasm `AES-256-GCM` (3 different keys)

And one encryption in the JS `AES-128-CBC`

- [N data encryption](https://github.com/Implex-ltd/hcaptcha-reverse/blob/main/encryptions/main.py)
- [Request payload and response encryption](https://github.com/Implex-ltd/hcaptcha-reverse/blob/main/encryptions/request.py)
- [Fingerprint blob encryption](https://github.com/Implex-ltd/hcaptcha-reverse/blob/main/encryptions/blob.py)

- [AES key fetcher for N data encryption](https://github.com/Implex-ltd/hcaptcha-reverse/blob/main/encryptions/fetcher.py)

## Fingerprint events

> `fingerprint_events` is parsed output of fingerprinting script, somes data are hashed.
> Final output is used into n data.
> Hash algorithm is xxHash3 (sixty_four.rs). 

### Raw javascript fp output (outdated)

- You can use [fingerprint_dumper.js](https://github.com/Implex-ltd/hcaptcha-reverse/blob/main/versions/fingerprint_dumper.js) to to dump the current raw fp before they got parsed by WASM

- [1.40.10](https://gist.github.com/nikolahellatrigger/65ff078faa990db653adb2d6052be6b0)
- [1.39.0](https://gist.github.com/nikolahellatrigger/b34456fdc7383ffbb26246bb9db28b7e)

| id     | type                                                                   | type      | hashed    | fp_raw                                                                              |
| ------ | ---------------------------------------------------------------------- | --------- | --------- | ----------------------------------------------------------------------------------- |
| `3`    |                                                                        | `float64` | **false** | [x](https://x.com)                                                                  |
| `1902` | `57`                                                                   | `int`     | **false** | [x](https://x.com)                                                                  |
| `1901` | math fingerprint wich give different result + err between device       | `u64`     | **true**  | [x](https://x.com)                                                                  |
| `1101` | canvas fingerprint hash of the image (`data:image/png;base64,...`)     | `u64`     | **true**  | [x](https://x.com)                                                                  |
| `1103` | `[255,255,255,255,192,192,192,255,244,244,244,255,53,53,53,255]`       | `array`   | **true**  | [x](https://x.com)                                                                  |
| `1105` | `[14,4,1,41.3203125,17,4,44.2890625]`                                  | `array`   | **true**  | [x](https://x.com)                                                                  |
| `1107` | `[274.609375,266,274.609375,266,274.609375,266,274.609375,....]`       | `array`   | **false** | [x](https://x.com)                                                                  |
| `201`  | hash of `1107`                                                         |           | **false** | [x](https://x.com)                                                                  |
| `211`  | audio fingerprint                                                      | `array`   | **true**  | [x](https://x.com)                                                                  |
| `3401` | page HTML Tree                                                         |           | **true**  | [x](https://x.com)                                                                  |
| `3403` | Link of hcaptcha.js                                                    |           | **false** | [x](https://x.com)                                                                  |
| `803`  | `[1,4,5,7,9,12,20,21,24,25,29]`                                        | `array`   | **false** | [x](https://x.com)                                                                  |
| `604`  | `[n.appv,n.ua,n.mem,n.hwconc,n.lang,n.langs,n.platform,n.cpu,versin]`  | `array`   | **false** | [link](https://gist.github.com/nikolahellatrigger/c4d6cf4ddb0ab219c38ddd133dc772eb) |
| `2801` | probably webgl related                                                 | `u8`      | **true**  | [x](https://x.com)                                                                  |
| `2805` | hash of `2801`                                                         | `array`   | **false** | [x](https://x.com)                                                                  |
| `107`  | `[s.w,s.h,s.aw,s.ah,s.cd,s.pd,event,n.maxtp,w.dpr,w.ow,w.oh...]`       | `array`   | **false** | [link](https://gist.github.com/nikolahellatrigger/ea00832b010c0db8f0a0d5ca0d467072) |
| `302`  | css default colors                                                     | `u64`     | **true**  | [x](https://x.com)                                                                  |
| `303`  | `fonts`                                                                | `array`   | **false** | [x](https://x.com)                                                                  |
| `301`  | CSS properties list                                                    | `u64`     | **true**  | [x](https://x.com)                                                                  |
| `304`  | length of CSS properties                                               | `int`     | **false** | [x](https://x.com)                                                                  |
| `1401` | `timezone`                                                             | `string`  | **false** | [x](https://x.com)                                                                  |
| `1402` | `[timezone,x,x,new Date("1/1/1970").getTimezoneOffset(),x,n.lang]`     | `array`   | **false** | [x](https://x.com)                                                                  |
| `1403` | timezone "encrypted"                                                   | `array`   | **false** | [x](https://x.com)                                                                  |
| `3504` |                                                                        | `float64` | **false** | [x](https://x.com)                                                                  |
| `3501` | navigation timestamp                                                   | `array`   | **false** | [x](https://x.com)                                                                  |
| `3503` | current unix timestamp                                                 | `int`     | **false** | [x](https://x.com)                                                                  |
| `3502` |                                                                        | `float64` | **false** | [x](https://x.com)                                                                  |
| `3505` |                                                                        | `float64` | **false** | [x](https://x.com)                                                                  |
| `401`  | browser properties hash                                                | `u64`     | **true**  | [x](https://x.com)                                                                  |
| `402`  | length of windows properties                                           | `int`     | **false** | [x](https://x.com)                                                                  |
| `407`  | browser keys?                                                          | `array`   | **false** | [x](https://x.com)                                                                  |
| `412`  | `[true,true,true,true,true,true,true,true,true,true,...]`              | `u64`     | **true**  | [x](https://x.com)                                                                  |
| `2402` | `[webgl_vendor, webgl_renderer]`                                       | `array`   | **false** | [x](https://x.com)                                                                  |
| `2420` | `[encrypt_webgl_vendor, encrypt_webgl_renderer]` "encrypted"           | `array`   | **false** | [x](https://x.com)                                                                  |
| `2403` | `[webgl2_vendor, webgl2_renderer]`                                     | `array`   | **false** | [x](https://x.com)                                                                  |
| `2401` | WebGL properties hash                                                  | `u64`     | **true**  | [x](https://x.com)                                                                  |
| `2408` | `!!navigator.webdriver`                                                | `bool`    | **false** | [x](https://x.com)                                                                  |
| `2407` | Math fingerprint of 28 first fibonacci number                          | `u64`     | **true**  | [x](https://x.com)                                                                  |
| `2409` | probably webgl related                                                 | `array`   | **false** | [x](https://x.com)                                                                  |
| `2410` | `[16,1024,4096,7,12,120,[23,127,127]]`                                 | `array`   | **false** | [x](https://x.com)                                                                  |
| `2411` | `[32767,32767,16384,8,8,8]`                                            | `array`   | **false** | [x](https://x.com)                                                                  |
| `2412` | `[1,1024,1,1,4]`                                                       | `array`   | **false** | [x](https://x.com)                                                                  |
| `2413` | probably webgl related                                                 | `array`   | **false** | [x](https://x.com)                                                                  |
| `2414` | `[16384,32,16384,2048,2,2048]`                                         | `array`   | **false** | [x](https://x.com)                                                                  |
| `2415` | `[4,120,4]`                                                            | `array`   | **false** | [x](https://x.com)                                                                  |
| `2416` | `[24,24,65536,212988,200704]`                                          | `array`   | **false** | [x](https://x.com)                                                                  |
| `2417` | `[16,4095,30,16,16380,120,12,120,[23,127,127]]`                        | `array`   | **false** | [x](https://x.com)                                                                  |
| `3800` | CSP (disabled)                                                         | `error`   | **false** | [x](https://x.com)                                                                  |
| `1302` | `[0,1,2,3,4]`                                                          | `array`   | **false** | [x](https://x.com)                                                                  |
| `901`  | All browser voices hash                                                | `u64`     | **true**  | [x](https://x.com)                                                                  |
| `905`  | Browser voice enabled                                                  | `array`   | **false** | [x](https://x.com)                                                                  |
| `3210` | `[143254600089,143254600089,null,null,4294705152,true,true,true,null]` | `array`   | **false** | [x](https://x.com)                                                                  |
| `3211` | first arg of performance **3210** "encrypted"                          | `array`   | **false** | [x](https://x.com)                                                                  |
| `702`  | `[os.name, os.version, null, os.bits, os.arch, navigator.version]`     | `array`   | **false** | [x](https://x.com)                                                                  |
| `2001` | Permissions hash                                                       | `u64`     | **true**  | [x](https://x.com)                                                                  |
| `2002` | Notifications permissions                                              | `array`   | **false** | [x](https://x.com)                                                                  |
| `0`    |                                                                        | `float64` | **false** | [x](https://x.com)                                                                  |

## Sandbox

Sandbox is fast way to encrypt own HSW without retrieving stuff as encryption-key, xxHash nonce and stuff that change, or if new update happen and you don't reversed it yet.
You can build custom HSW wasm using [builder](https://github.com/Implex-ltd/hcaptcha-reverse/blob/main/src/main.py) / [WABT](https://github.com/WebAssembly/wabt) tools.

HSW usualy take less than 10ms to execute. (you have to remove all fingerprints from array)

### how sandbox work ?

the sandbox executes a custom hsw containing a hand-modified WASM which adds the payload to be encrypted to the end of memory and returns the pointer to encrypt our payload and not the one generated by hcaptcha. It's kinda smart isn't it ?

### Wasm hook
```wasm
;; new modules import
(func $./client_bg.js.inject (;31;) (import "./client_bg.js" "inject") (param i32 i32))
(func $./client_bg.js.getLen (;56;) (import "./client_bg.js" "getLen") (result i32))
(func $./client_bg.js.getPtr (;75;) (import "./client_bg.js" "getPtr") (result i32))

;; edited function: func 150 (1.39) // func 152 (1.40.10)

;; JSON is built above...
local.set $var7
local.get $var5
i32.const 32
i32.add

local.get $var6 ;; load len of the JSON
local.get $var7 ;; load ptr of the JSON
call $./client_bg.js.inject ;; append custom payload into memory
      
call $./client_bg.js.getLen
local.set $var6 ;; ^+ get the payload len and overwrite original one
      
call $./client_bg.js.getPtr
local.set $var7 ;; ^+ get the payload ptr and overwrite original one

call $func211 ;; continue wasm with out custom payload...
```

### Hsw hook
```js

let jlen = 0
let jptr = 0
let fp_json_curr = {}

// this append over and over and can lead to memory leak // 100% RAM but it's working
function appendJsonToMemory(pp) {
    const to_inject = new TextEncoder().encode(pp);
    const buffer = M.memory.buffer;

    const currentSize = buffer.byteLength;
    const requiredSize = currentSize + to_inject.length;

    M.memory.grow(Math.ceil((requiredSize - currentSize) / 65536));

    const updatedBuffer = M.memory.buffer;
    const memoryView = new Uint8Array(updatedBuffer);

    memoryView.set(to_inject, currentSize);

    return {
        ptr: currentSize,
        len: to_inject.length
    };
}

inject: function (len, ptr) {
    try {
        /*
            - This part was used to get the stamp + rand when it was not fully reversed

            let parsed = JSON.parse(__getStrFromWasm(ptr, len))
            fp_json_curr.stamp = parsed.stamp
            fp_json_curr.rand = parsed.rand
        */

        console.log(JSON.stringify(fp_json_curr))
        const data = appendJsonToMemory(JSON.stringify(fp_json_curr));

        // save new ptr + len
        jlen = data.len
        jptr = data.ptr
        } catch (err) { console.log(err) }
},

getPtr: function () {
    return jptr
},

getLen: function () {
    return jlen
},
```

## Key Building Algo

the 32 byte key is generated as follows:

1. the first 2 bytes are taken directly from the `key_seed` (in little format)
2. the remaining 30 bytes are generated iteratively:

   for each step (0-29):
   
   a. if not the first step, update the seed using an LCG:
      ```
      seed = (seed * 6364136223846793005) & 0xFFFFFFFFFFFFFFFF
      seed = (seed ± key_factor1) & 0xFFFFFFFFFFFFFFFF  // + or - depending on operator
      ```
   
   b. calculate memory access positions:
      ```
      base_index = memory + step
      memory_position = base_index + key_factor2
      segment_address = (((memory_position // 320) << 3) + memory_position + 1032 - 1075552) % len(memory)
      mask_address = (memory_position % 96) + 8
      ```
   
   c. extract values from memory:
      ```
      segment_value = 32 bit little value from memory[segment_address]
      mask_value = 64 bit little value from memory[mask_address]
      ```
   
   d. calculate hash value:
      ```
      hash_value = (segment_value ^ (mask_value & 0xFFFFFFFF)) & 0xFF
      ```
   
   e. extract and process bit positions from the seed:
      ```
      bit45 = (seed >> 45) & 0xFFFFFFFF
      bit27 = (seed >> 27) & 0xFFFFFFFF
      bit59 = (seed >> 59) & 0xFFFFFFFF

      if bit45 & 0x80000000: bit45 = bit45 - 0x100000000
      if bit27 & 0x80000000: bit27 = bit27 - 0x100000000
      if bit59 & 0x80000000: bit59 = bit59 - 0x100000000
      ```
   
   f. combine and rotate bits:
      ```
      combined = bit45 ^ bit27
      shift = bit59 % 32
      rotated = ((combined >> shift) | (combined << (32 - shift))) & 0xFFFFFFFF

      if rotated & 0x80000000: rotated = rotated - 0x100000000
      ```
   
   g. calculate final key byte and add to key:
      ```
      key_byte = (hash_value ^ rotated) & 0xFF
      key_bytes.append(key_byte)
      ```

3. The final key is the hex representation of all 32 bytes

### Linear Congruential Generator

the algorithm uses an LCG with the following parameters:
- multiplier: 6364136223846793005
- increment: different based on `key_factor1` and `operator`
- modulus: 2^64

## Notes (from Cyrus)

- This community is a shitty community filled with arrogant people *ehm ehm dort* who can't take anything seriously
- This is going to be the last time you are seeing me, good luck to everyone who is on this path (The VM is pretty fun). 
- If you urgently need me contact me (Cyrus) at telegram: @hcaptcha_staff
