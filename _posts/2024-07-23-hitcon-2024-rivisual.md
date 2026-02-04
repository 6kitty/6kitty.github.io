---
layout: post
title: "[HITCON 2024] rivisual"
categories: [System Hacking]
tags: [JavaScript, Deobfuscation, CTF]
last_modified_at: 2024-07-23
---

[Deobfuscate Tool](https://deobfuscate.io/)

![Image 1](https://blog.kakaocdn.net/dna/mOylU/btsIx5kYs8K/AAAAAAAAAAAAAAAAAAAAAO38oXX1vEsHkBvv48DQY7E82b_-gnYEkg3v-Z2ZeU1a/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=Jxzi0QyPuz7bNYGSSjVzTNJKeis%3D)
![Image 2](https://blog.kakaocdn.net/dna/KUkJv/btsIx5FgTmf/AAAAAAAAAAAAAAAAAAAAACKtxZvPLpPRfARQ-xMVNFEN5Jg1_dABpU6q_zqdTBTj/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=ne%2BJoNxrKd%2Fn2qBYS6x1I145u8M%3D)

너무 많아서 일단 접어놨음  
아래서부터 함수 분석해보자..

```javascript
function _0x52fd86(_0x1f947c) {
    const _0x52daff = {
      'sMjKe': _0xfbe68(0x306) + _0xfbe68(0x268) + _0xfbe68(0x343) + '7c',
      'eyMvd': _0xfbe68(0x371) + _0xfbe68(0x1cd) + _0xfbe68(0x347) + '601473cb42' + _0xfbe68(0x387) + _0xfbe68(0x249) + _0xfbe68(0x313) + _0xfbe68(0x2f9) + _0xfbe68(0x2dc) + '37b04dcef8' + _0xfbe68(0x2b4) + _0xfbe68(0x394) + 'ef873ce857' + _0xfbe68(0x2b0) + _0xfbe68(0x31d) + _0xfbe68(0x288) + _0xfbe68(0x2ce) + _0xfbe68(0x2fd) + _0xfbe68(0x322) + _0xfbe68(0x3ab) + _0xfbe68(0x3b0) + _0xfbe68(0x241) + _0xfbe68(0x229)
    };
    let _0x4a7c67 = CryptoJS.enc[_0xfbe68(0x3a7)][_0xfbe68(0x25e)](CryptoJS[_0xfbe68(0x3af)](_0x1f947c)[_0xfbe68(0x37d)](CryptoJS[_0xfbe68(0x349)][_0xfbe68(0x3a7)]));
    let _0x1f1504 = CryptoJS[_0xfbe68(0x349)][_0xfbe68(0x3a7)][_0xfbe68(0x25e)](_0x52daff[_0xfbe68(0x2d4)]);
    let _0x289540 = CryptoJS.enc[_0xfbe68(0x3a7)][_0xfbe68(0x25e)](_0x52daff.eyMvd);
    let _0x21f6cd = CryptoJS[_0xfbe68(0x22a)].decrypt({
      'ciphertext': _0x289540
    }, _0x4a7c67, {
      'iv': _0x1f1504,
      'padding': CryptoJS[_0xfbe68(0x242)][_0xfbe68(0x231)],
      'mode': CryptoJS[_0xfbe68(0x363)].CBC,
      'hasher': CryptoJS.algo[_0xfbe68(0x3af)]
    });
    return _0x21f6cd[_0xfbe68(0x37d)](CryptoJS[_0xfbe68(0x349)][_0xfbe68(0x1eb)]);
}
```

복호화 함수  
상수 지정해주는데 _0xfbe68이 자꾸 쓰인다.  
이건 사용자 정의 function이 아니다  
풀어줘야 한다. 이외에도..

![Image 3](https://blog.kakaocdn.net/dna/cpvSng/btsIzqurZ2n/AAAAAAAAAAAAAAAAAAAAAPaV7pFKjkDwHrvHW2cX_FfgF7AgBJZ_fljivWfC455P/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=hpbS8UMnNGYkJCg4F1FqSUzqgG4%3D)

background가 red이면 lose이고, green되어야 함..?

[Obfuscator Tool](https://obfuscator.io/#code)  
문제 제작한 사람이 사용한 obfuscator이다.  
여기 deobfuscator을 더 찾아보다가...

[GitHub Repository](https://github.com/j4k0xb/webcrack?tab=readme-ov-file)  
해당 깃허브 발견

![Image 4](https://blog.kakaocdn.net/dna/bepnZw/btsIAduAYfQ/AAAAAAAAAAAAAAAAAAAAACUOmNIjrEMgfXsC6AFWGw1526dnyZ_EG-hyh92TT7kZ/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=uvAPKLaEuouqZSYNl5aqoN2xt%2Fg%3D)

```javascript
i.addEventListener("mousemove", a => {
    if (!p) {
      return;
    }
    if (k && m) {
      m.setAttribute("x2", a.clientX - i.getBoundingClientRect().left);
      m.setAttribute("y2", a.clientY - i.getBoundingClientRect().top);
    }
});
i.addEventListener("mouseup", a => {
    if (k && m) {
      m.setAttribute("x2", k.offsetLeft + k.offsetWidth / 2);
      m.setAttribute("y2", k.offsetTop + k.offsetHeight / 2);
      k.classList.add("selected");
      k = null;
      let a = f(q.map(a => parseInt(a.dataset.number)));
      if (a !== null) {
        q.forEach(a => {
          a.classList.add("win");
        });
        let b = document.getElementById("flag");
        b.innerText = a;
      } else {
        q.forEach(a => {
          a.classList.add("lose");
        });
      }
    }
    p = false;
});
```

이부분이 좀 중요한 거 같은데..  
조건문 1. k && m이 0이 아님  
조건문 2. f의 값이 null이 아님  

마우스 이벤트 정리하자면  
mousedown: 누른 시점  
mouseover: 움직인 시점인데 아마 밖에서 해당 별로 왔을 때 말하는듯  
mousemove: 움직인 시점, 이건 over과 반대인듯  
mouseup: 뗀 시점  

let a = f(q.map(a => parseInt(a.dataset.number)));  
여기서 q.map이 아래 함수의 a로 감  

```javascript
function f(a) {
  // 다양한 b.wtf 연산의 결과를 변수 c부터 y에 저장
  let c = b.wtf(a[19], a[3], a[5]) * 25;
  let e = b.wtf(a[7], a[20], a[18]) * 25;
  let f = b.wtf(a[11], a[22], a[18]) * 25;
  let h = b.wtf(a[5], a[17], a[2]) * 25;
  let i = b.wtf(a[20], a[13], a[5]) * 25;
  let j = b.wtf(a[11], a[1], a[21]) * 25;
  let k = b.wtf(a[8], a[11], a[1]) * 25;
  let l = b.wtf(a[9], a[5], a[4]) * 25;
  let m = b.wtf(a[17], a[9], a[21]) * 25;
  let n = b.wtf(a[23], a[9], a[20]) * 25;
  let o = b.wtf(a[16], a[5], a[4]) * 25;
  let p = b.wtf(a[16], a[14], a[13]) * 25;
  let q = b.wtf(a[5], a[6], a[10]) * 25;
  let r = b.wtf(a[2], a[11], a[5]) * 25;
  let t = b.wtf(a[11], a[3], a[1]) * 25;
  let u = b.wtf(a[12], a[3], a[10]) * 25;
  let v = b.wtf(a[14], a[1], a[9]) * 25;
  let w = b.wtf(a[18], a[11], a[17]) * 25;
  let x = b.wtf(a[12], a[15], a[2]) * 25;
  let y = b.wtf(a[22], a[0], a[19]) * 25;

  // 변수 z 초기화
  let z = 0;

  // 변수 z에 다양한 b.gtfo 연산의 결과를 누적
  z += d(0.3837876686390533 - b.gtfo(j, v, m, 16, 21));
  z += d(0.21054889940828397 - b.gtfo(t, j, k, 8, 2));
  z += d(0.475323349112426 - b.gtfo(j, w, q, 0, 20));
  z += d(0.6338370887573964 - b.gtfo(h, e, q, 8, 4));
  z += d(0.4111607928994082 - b.gtfo(f, t, u, 23, 1));
  z += d(0.7707577751479291 - b.gtfo(w, h, p, 20, 6));
  z += d(0.7743081420118344 - b.gtfo(n, r, h, 9, 10));
  z += d(0.36471487573964495 - b.gtfo(m, c, i, 18, 8));
  z += d(0.312678449704142 - b.gtfo(u, n, w, 0, 17));
  z += d(0.9502808165680473 - b.gtfo(x, n, h, 22, 10));
  z += d(0.5869052899408282 - b.gtfo(q, l, f, 14, 10));
  z += d(0.9323389467455623 - b.gtfo(w, f, q, 12, 7));
  z += d(0.4587118106508875 - b.gtfo(k, r, f, 4, 21));
  z += d(0.14484472189349107 - b.gtfo(u, n, t, 7, 15));
  z += d(0.7255550059171598 - b.gtfo(j, w, x, 9, 23));
  z += d(0.5031261301775147 - b.gtfo(h, f, t, 7, 1));
  z += d(0.1417352189349112 - b.gtfo(k, t, m, 16, 14));
  z += d(0.5579334437869822 - b.gtfo(t, f, x, 19, 11));
  z += d(0.48502262721893485 - b.gtfo(o, i, l, 23, 18));
  z += d(0.5920916568047336 - b.gtfo(l, m, e, 19, 6));
  z += d(0.7222713017751479 - b.gtfo(v, f, i, 8, 16));
  z += d(0.12367382248520711 - b.gtfo(o, u, q, 9, 5));
  z += d(0.4558028402366864 - b.gtfo(p, o, f, 10, 2));
  z += d(0.8537692426035504 - b.gtfo(w, n, r, 4, 11));
  z += d(0.9618170650887574 - b.gtfo(q, x, w, 15, 2));
  z += d(0.22088933727810647 - b.gtfo(c, l, v, 10, 5));
  z += d(0.4302783550295858 - b.gtfo(v, p, j, 14, 2));
  z += d(0.6262803313609467 - b.gtfo(y, t, f, 17, 22));

  // z가 특정 값을 초과하면 null 반환
  if (z > 0.00001) {
    return null;
  }

  // 조건을 만족하면 문자열 s 생성
  let s = "";
  s += Math.round(b.wtf(a[4], a[2], a[22]) * 100000).toString();
  s += Math.round(b.wtf(a[17], a[9], a[14]) * 100000).toString();
  s += Math.round(b.wtf(a[4], a[13], a[7]) * 100000).toString();
  s += Math.round(b.wtf(a[4], a[20], a[23]) * 100000).toString();
  s += Math.round(b.wtf(a[5], a[7], a[12]) * 100000).toString();
  s += Math.round(b.wtf(a[20], a[19], a[4]) * 100000).toString();
  s += Math.round(b.wtf(a[17], a[6], a[19]) * 100000).toString();
  s += Math.round(b.wtf(a[6], a[21], a[18]) * 100000).toString();
  s += Math.round(b.wtf(a[4], a[3], a[8]) * 100000).toString();
  s += Math.round(b.wtf(a[11], a[7], a[14]) * 100000).toString();
  s += Math.round(b.wtf(a[9], a[2], a[13]) * 100000).toString();
  s += Math.round(b.wtf(a[22], a[10], a[3]) * 100000).toString();
  s += Math.round(b.wtf(a[15], a[22], a[13]) * 100000).toString();
  s += Math.round(b.wtf(a[16], a[12], a[9]) * 100000).toString();
  s += Math.round(b.wtf(a[14], a[8], a[17]) * 100000).toString();
  s += Math.round(b.wtf(a[1], a[18], a[6]) * 100000).toString();
  s += Math.round(b.wtf(a[10], a[11], a[3]) * 100000).toString();
  s += Math.round(b.wtf(a[8], a[12], a[5]) * 100000).toString();
  s += Math.round(b.wtf(a[1], a[3], a[12]) * 100000).toString();
  s += Math.round(b.wtf(a[9], a[13], a[7]) * 100000).toString();
}
```

너무 길어서 이것보다 더 많은데 암튼..  
위쪽에 string 처리 된 거 코드 풀어야 했음  

```glsl
attribute vec3 position;
uniform   mat4 mvpMatrix;
varying   vec2 vPosition;

void main(void) {
	gl_Position = mvpMatrix * vec4(position, 1.0);
	vPosition = position.xy;
}
#ifdef GL_ES
precision mediump float;
#endif 

uniform float u_time;
uniform vec2 u_resolution;
varying vec2 vPosition;

vec3 hash(vec2 seed){
	vec3 p3 = fract(float(seed.x + seed.y*86.) * vec3(.1051, .1020, .0983));
	
	p3 += dot(p3, p3.yzx + 33.33);
	return fract(p3);
}

vec3 layer(float scale, vec2 uv, float time){
	// uv coord in cell
	vec2 scaled_uv = uv * scale - 0.5;
	vec2 uv0 = fract(scaled_uv) - 0.5;
	// cell id
	vec2 cell_id = scaled_uv - fract(scaled_uv);
	vec3 col = vec3(0);
	float speed = 1.5;
	// distance to a spinning random point in the cell (also surrounding cells)
	vec3 seed = hash(cell_id);
	
	float radiance = seed.x + time * seed.y;
	vec2 center_of_star = vec2(sin(radiance), cos(radiance)) * 0.3;
	
	// radial distort effect for star shine
	vec2 v_to_star = uv0 - center_of_star;
	float star_radiance = atan(v_to_star.x/v_to_star.y);
	float star_spark_1 = sin(star_radiance*14.+radiance*6.);
	float star_spark_2 = sin(star_radiance*8.-radiance*2.);
	float stars = length(v_to_star) * (5.+star_spark_1+star_spark_2) * 0.03;
	col += smoothstep(length(seed) * 0.01, 0., stars);
	return col;
}
void main() {
	// center global uv from -1 to 1
	vec2 virtual_resolution = vec2(2.0, 2.0);
	vec2 uv = (vPosition * 2. - virtual_resolution.xy) / virtual_resolution.y;
	vec3 col = vec3(0.);//vColor.xyz;
	
	const float layer_count = 6.5;
	for(float i = 0.0; i < layer_count; i+=1.){
		float rotate_speed = u_time*0.4;
		float scale = mod(i - rotate_speed, layer_count)*1.5;
		vec2 offseted_uv = uv + vec2(sin(rotate_speed), cos(rotate_speed));
		vec3 layer_col = layer(scale, offseted_uv, u_time + i*1.5);
		
		// we want the star to smoothly show uㅔ
		float max_scale = layer_count * 1.5;
		float color_amp = smoothstep(0., 1., smoothstep(max_scale, 0., scale));
		col += layer_col * color_amp;
	}
	// blue background
	col += vec3(0., 0., -0.15) * (uv.y - 0.7) * pow(length(uv), 0.5);
	gl_FragColor = vec4(col, 1.);
}
attribute vec3 position;
varying   float owO;

void main(){
	gl_Position = vec4(position.xy, 0.0, 1.0);
	owO = position.z;
}

#ifdef GL_ES
precision highp float;
#endif            
varying float owO;
#define OvO 255.0
#define Ovo 128.0
#define OVO 23.0

float OwO (float Owo, float OWO, float owO) { 
	OWO = floor(OWO + 0.5); owO = floor(owO + 0.5); 
	return mod(floor((floor(Owo) + 0.5) / exp2(OWO)), floor(1.0*exp2(owO - OWO) + 0.5)); 
}
vec4 oWo (float Ow0) { 
	if (Ow0 == 0.0) return vec4(0.0); 
	float Owo = Ow0 > 0.0 ? 0.0 : 1.0; 
	Ow0 = abs(Ow0); 
	float OWO = floor(log2(Ow0)); 
	float oWo = OWO + OvO - Ovo; 
	OWO = ((Ow0 / exp2(OWO)) - 1.0) * pow(2.0, OVO);
	float owO = oWo / 2.0; 
	oWo = fract(owO) + fract(owO); 
	float oWO = floor(owO); 
	owO = OwO(OWO, 0.0, 8.0) / OvO; 
	Ow0 = OwO(OWO, 8.0, 16.0) / OvO; 
	OWO = (oWo * Ovo + OwO(OWO, 16.0, OVO)) / OvO; 
	Owo = (Owo * Ovo + oWO) / OvO; 
	return vec4(owO, Ow0, OWO, Owo);
}
void main()
{
	gl_FragColor = oWo(owO);
}
```

어... 어렵다