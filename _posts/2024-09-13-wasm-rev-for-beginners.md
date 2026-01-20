---
layout: post
title: "Wasm rev for beginners"
categories: [SWING, Writeup]
tags: [Wasm, Reverse Engineering, CTF]
last_modified_at: 2024-09-13
---

대놓고 웹어셈블리에 대한 문제라고 명시해놨다  

![Image](https://blog.kakaocdn.net/dna/cmYVYA/btsJyrS8MKK/AAAAAAAAAAAAAAAAAAAAAP1RlfGDJFae2k9kXjEl2kT5MNz1yn_R1NLM3hT5yqv9/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=Kko3k2bvvaEFp0FXdjVFSVTRIFs%3D)

![Image](https://blog.kakaocdn.net/dna/97G8v/btsJxESZCaJ/AAAAAAAAAAAAAAAAAAAAAHgtwzwIRtNXZAU7L1OV1necT938eDK88KztJqnZ6XpY/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=wYmnX8wNlnEtpSk0GVE6CIB1U68%3D)

```html
<!doctype html>
<html lang="en-US">

<head>
    <meta charset="utf-8" />
    <title>Wasm rev for beginners</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@picocss/pico@2/css/pico.min.css" />
</head>

<body>
    <main class="container">
        <form onsubmit="return false;">
            <fieldset>
                <label for="input"> Input </label>
                <input name="input" id="input" placeholder="Put your flag here" required>
            </fieldset>
            <input type="button" id="check_button" value="Check!">
        </form>
    </main>
    <script type="module">
        import init, { check } from "./pkg/challenge.js";
        await init();
        document.getElementById('check_button').addEventListener('click', function() {
          check(document.getElementById('input').value);
        });
    </script>
</body>

</html>
```

script를 따라 pkg/challenge.js로 가보면... 뭐 길게 js 코드가 있다.  
일단 문제설명에 있는 WASM 링크 먼저 공부하고 다시 봐야할 거 같다  

웹어셈블리란? 프로그래밍 언어와 실행 환경 사이 중간 계층...?  
여러 언어로 작성된 코드를 .wasm 파일로 컴파일하면 브라우저 등에서 실행 가능 ㄷ  
대충 머 C파일 -> wasm거치면 -> 웹브라우저에서도 실행가능 한 줄 요약인 거 같다  

초기에 웹에서 코드를 빠르게 실행하기 위해 개발됨  
```javascript
import init, { check } from "./pkg/challenge.js";
```
이므로 pkg/challenge.js에서 check 함수만 우선적으로 보자  

[MDN WebAssembly JavaScript API](https://developer.mozilla.org/ko/docs/WebAssembly/Using_the_JavaScript_API)

HTML 파일에 `<script></script>` 요소를 만들고  
```javascript
var importObject = { 
  imports: { imported_func: (arg) => console.log(arg) }, 
};
```

```javascript
/**
* @param {string} flag
*/
export function check(flag) {
    const ptr0 = passStringToWasm0(flag, wasm.__wbindgen_malloc, wasm.__wbindgen_realloc);
    const len0 = WASM_VECTOR_LEN;
    wasm.check(ptr0, len0);
}
```

passStringToWasm0  
```javascript
function passStringToWasm0(arg, malloc, realloc) {
    if (realloc === undefined) {
        const buf = cachedTextEncoder.encode(arg);
        const ptr = malloc(buf.length, 1) >>> 0;
        getUint8Memory0().subarray(ptr, ptr + buf.length).set(buf);
        WASM_VECTOR_LEN = buf.length;
        return ptr;
    }

    let len = arg.length;
    let ptr = malloc(len, 1) >>> 0;

    const mem = getUint8Memory0();

    let offset = 0;

    for (; offset < len; offset++) {
        const code = arg.charCodeAt(offset);
        if (code > 0x7F) break;
        mem[ptr + offset] = code;
    }

    if (offset !== len) {
        if (offset !== 0) {
            arg = arg.slice(offset);
        }
        ptr = realloc(ptr, len, len = offset + arg.length * 3, 1) >>> 0;
        const view = getUint8Memory0().subarray(ptr + offset, ptr + len);
        const ret = encodeString(arg, view);

        offset += ret.written;
        ptr = realloc(ptr, len, offset, 1) >>> 0;
    }

    WASM_VECTOR_LEN = offset;
    return ptr;
}
```
자세히 보진 않았지만 뭔가 처리하고 wasm으로 보내는.. 고런 거 같다  
대충 js에서 받은 거 -> 뭔가 메모리 할당, 재할당 하고 -> wasm에서 맞는지 check하는..  

![Image](https://blog.kakaocdn.net/dna/efEB9w/btsJAP8zZRj/AAAAAAAAAAAAAAAAAAAAAPxROKrLCcozWMuzO4rUzfs6bvkkgM_Q0ixipATRP9qO/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=42kjuM%2FZzZUHaZS9%2FrXWEE8ijoo%3D)

online 사이트로 wat 변환 했는데  
음 클론해서 디컴파일을 써보는 게 나을 거 같다  

![Image](https://blog.kakaocdn.net/dna/qUyx2/btsJBPUeSpU/AAAAAAAAAAAAAAAAAAAAAEd6hkZB1w-NoEueE0GiZ30GIbtebx3pjAKDSveI6w9q/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=LCZm8oCbV1MaM%2FurLlhNpGmED1E%3D)

설치가 안된다  
아마.. WSL이라서 그런 거 같은데 vm으로 다시 시도해봐야 할 거 같다..  