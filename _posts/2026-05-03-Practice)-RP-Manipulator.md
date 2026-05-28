---
layout: post
title: "Practice) RP Manipulator"
date: 2026-04-20
categories: Kinematics
tags:
  - robotics
  - manipulator
  - Kinematics
---

<!-- 여기에 포스트 내용을 작성하세요 -->

# Practice) RP Manipulator

#practice

![Pasted image 20260424114855.png](/assets/images/Pasted-image-20260424114855.png)![스크린샷 2026-04-20 오후 3.21.27.png](/assets/images/스크린샷-2026-04-20-오후-3.21.27.png)

## 구조 정의

1. joint 1. {1} - Revolute joint 2. {2} - prismatic joint
2. link 1. 직렬링크 $Z_1$축 ~ $Z_2$축
3. End-Effector정의 1. 그리퍼

## DH parameter - standard

1. DH Table
   #Realize
   stadard way는 1부터 (이전 축 안봄)

| i       | $\alpha_{i}$     | $a_{i}$            | $\theta_{i}$ | $d_{i}$ |
| ------- | ---------------- | ------------------ | ------------ | ------- |
| 1: 1~2  | -$\frac{\pi}{2}$ | $l_{1}Sin(\alpha)$ | $\theta^ʹ_1$ | 0       |
| 2: 2~EE | 0                | 0                  | 0            | $d_2$   |

2. 일반식
   #Realize Z축 부터 변환한 후 X축 변환
   $$
   ^{i-1}_iT = R_z(\theta_i)T_z(d_i)T_x(a_i)R_x(\alpha_i)
   $$

$$
=\begin{bmatrix} C{\theta_i} & -S{\theta_i} & 0 & 0 \\ S{\theta_i} & C{\theta_i} & 0 & 0 \\ 0 & 0 & 1 & 0 \\ 0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix} 1 & 0 & 0 & 0 \\ 0 & 1 & 0 & 0 \\ 0 & 0 & 1 & d_i \\ 0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix} 1 & 0 & 0 & a_i \\ 0 & 1 & 0 & 0 \\ 0 & 0 & 1 & 0 \\ 0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix} 1 & 0 & 0 & 0 \\ 0 & C{\alpha_i} & -S{\alpha_i} & 0 \\ 0 & S\alpha_i & C\alpha_i & 0 \\ 0 & 0 & 0 & 1
\end{bmatrix}
$$

$$
= \begin{bmatrix}
\cos \theta_{i} & -\sin \theta_{i} \cos \alpha_{i} & \sin \theta_{i} \sin \alpha_{i} & a_{i} \cos \theta_{i} \\
\sin \theta_{i} & \cos \theta_{i} \cos \alpha_{i} & -\cos \theta_{i} \sin \alpha_{i} & a_{i} \sin \theta_{i} \\
0 & \sin \alpha_{i} & \cos \alpha_{i} & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

3. 변환행렬 계산 $^0T_1$ ~ $^1T_2$

$$
^0_1T = \begin{bmatrix}
\cos \theta^ʹ_{i} & -\sin\theta^ʹ_{i} \cos (-\frac{\pi}{2}) & \sin \theta^ʹ_{i} \sin (-\frac{\pi}{2}) & a_{i} \cos \theta^ʹ_{i} \\
\sin \theta^ʹ_{i} & \cos \theta^ʹ_{i} \cos (-\frac{\pi}{2}) & -\cos \theta^ʹ_{i} \sin (-\frac{\pi}{2}) & a_{i} \sin \theta^ʹ_{i} \\
0 & \sin (-\frac{\pi}{2}) & \cos (-\frac{\pi}{2}) & d_{i} \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

$\cos (-\frac{\pi}{2})$ = $0$, $\sin (-\frac{\pi}{2})$ = $-1$ 이므로,

$$
^0_1T = \begin{bmatrix}
\cos \theta^ʹ_{i} & 0 & -\sin \theta^ʹ_{i} & a_{i} \cos \theta^ʹ_{i} \\
\sin \theta^ʹ_{i} & 0 & \cos \theta^ʹ_{i} & a_{i} \sin \theta^ʹ_{i} \\
0 & -1 & 0 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

$\theta^ʹ_i$ 는 $\theta_1$ + $\alpha$ - $\frac{\pi}{2}$ 로 나타낼수 있다,

cos( $\theta_1$ + $\alpha$ - $\frac{\pi}{2}$) , sin( $\theta_1$ + $\alpha$ - $\frac{\pi}{2}$) 을 [[덧셈정리]]를 이용하여 정리하면,

cos$\theta^ʹ_i$ = sin($\theta_1$ + $\alpha$) , sin$\theta^ʹ_i$ = -cos($\theta_1$ + $\alpha$) 가 된다.

정리하면 $^0_1T$ 는

$$
^0_1T = \begin{bmatrix}
sin(\theta_1 + \alpha) & 0 & -cos(\theta_1 + \alpha)  & a_{i} sin(\theta_1 + \alpha) \\
-cos(\theta_1 + \alpha) & 0 & sin(\theta_1 + \alpha) & -a_{i} cos(\theta_1 + \alpha) \\
0 & -1 & 0 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

$^1_2T$ 는 $d_2$값만 남으므로,

$$
^1_2T = \begin{bmatrix}
1 & 0 & 0 & 0 \\
0 & 1 & 0 & 0 \\
0 & 0 & 1 & d_2 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

$$
^0_1T^1_2T =
\begin{bmatrix}
\cos \theta^ʹ_{i} & 0 & -\sin \theta^ʹ_{i} & a_{i} \cos \theta^ʹ_{i} \\
\sin \theta^ʹ_{i} & 0 & \cos \theta^ʹ_{i} & a_{i} \sin \theta^ʹ_{i} \\
0 & -1 & 0 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
1 & 0 & 0 & 0 \\
0 & 1 & 0 & 0 \\
0 & 0 & 1 & d_2 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

cos$\theta^ʹ_i$ = sin($\theta_1$ + $\alpha$) , sin$\theta^ʹ_i$ = -cos($\theta_1$ + $\alpha$) 가 된다.

$$
= \begin{bmatrix}
\sin(\theta_1 + \alpha) & 0 & \cos(\theta_1 + \alpha) & \cos(\theta_1 + \alpha)+l_1\sin{\alpha}\sin(\theta_1 + \alpha) \\
-\cos(\theta_1 + \alpha) & 0 & \sin(\theta_1 + \alpha) & \sin(\theta_1 + \alpha)d_2 - l_1\sin{\alpha}\cos(\theta_1 + \alpha) \\
0 & -1 & 0 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

## DH parameter - modified

1. DH Table
   #Realize modified way는 0부터

| i   | $\alpha_{i-1}$   | $a_{i-1}$       | $\theta_i$   | $d_i$ |
| --- | ---------------- | --------------- | ------------ | ----- |
| 1   | 0                | 0               | $\theta^ʹ_1$ | 0     |
| 2   | $-\frac{\pi}{2}$ | $l_1\sin\alpha$ | 0            | $d_2$ |

2. 일반식
   #Realize X축 부터 변환한 후 Z축 변환

$$
^{i-1}_iT = R_x(\alpha_{i-1})T_x(a_{i-1})R_z({\theta}_i)T_z(d_i)
$$

$$
= \begin{bmatrix}
1 & 0 & 0 & 0 \\
0 & C{\alpha_{i-1}} & -S{\alpha_{i-1}} & 0 \\
0 & S{\alpha_{i-1}} & C{\alpha_{i-1}} & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
1 & 0 & 0 & a_{i-1} \\
0 & 1 & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
 C{\theta_{i}} & -S{\theta_{i}} & 0 & 0 \\
 S{\theta_{i}} & C{\theta_{i}} & 0 & 0 \\
 0 & 0 & 1 & 0 \\
 0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
1 & 0 & 0 & 0 \\
0 & 1 & 0 & 0 \\
0 & 0 & 1 & d_i \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

$$
= \begin{bmatrix}
C{\theta_i} & -S{\theta_i} & 0 & a_{i-1} \\
S{\theta_i}C{\alpha_{i-1}} & C{\theta_i}C{\alpha_{i-1}} & -S{\alpha_{i-1}} & -d_{i}S{\alpha_{i-1}} \\
S{\theta_i}S{\alpha_{i-1}} & C{\theta_i}S{\alpha_{i-1}} & C{\alpha_{i-1}} & d_iC{\alpha_{i-1}} \\
 0 & 0 & 0 & 1
\end{bmatrix}
$$

3. 변환행렬 도출
   $$
   ^0_1T=
   \begin{bmatrix}
   C{\theta^ʹ_1} & -S{\theta^ʹ_1} & 0 & 0 \\
   S{\theta^ʹ_1} & C{\theta^ʹ_1} & 0 & 0 \\
   0 & 0 & 1 & 0 \\
   0 & 0 & 0 & 1 \\
   \end{bmatrix}
   $$

cos$\theta^ʹ_i$ = sin($\theta_1$ + $\alpha$) , sin$\theta^ʹ_i$ = -cos($\theta_1$ + $\alpha$) 가 된다. [[덧셈정리]]

$$
^0_1T=
\begin{bmatrix}
\sin(\theta_1 + \alpha) & cos(\theta_1 + \alpha) & 0 & 0 \\
-cos(\theta_1 + \alpha) & \sin(\theta_1 + \alpha) & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1 \\
\end{bmatrix}
$$

$$
^1_2T=
\begin{bmatrix}
1 & 0 & 0 & l_1\sin\alpha \\
0 & 0 & 1 & d_2 \\
0 & -1 & 0 & 0 \\
0 & 0 & 0& 1
\end{bmatrix}
$$

$$
^0_1T^1_2T =
\begin{bmatrix}
\sin(\theta_1 + \alpha) & cos(\theta_1 + \alpha) & 0 & 0 \\
-cos(\theta_1 + \alpha) & \sin(\theta_1 + \alpha) & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1 \\
\end{bmatrix}
\begin{bmatrix}
1 & 0 & 0 & l_1\sin\alpha \\
0 & 0 & 1 & d_2 \\
0 & -1 & 0 & 0 \\
0 & 0 & 0& 1
\end{bmatrix}
$$

$$
= \begin{bmatrix}
\sin(\theta_1 + \alpha) & 0 & \cos(\theta_1 + \alpha) & \cos(\theta_1 + \alpha)d_2+l_1\sin{\alpha}\sin(\theta_1 + \alpha) \\
-\cos(\theta_1 + \alpha) & 0 & \sin(\theta_1 + \alpha) & \sin(\theta_1 + \alpha)d_2 - l_1\sin{\alpha}\cos(\theta_1 + \alpha) \\
0 & -1 & 0 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

최종 end-effector의 위치

$$
x= \cos(\theta_1 + \alpha)d_2+l_1\sin{\alpha}\sin(\theta_1 + \alpha)
$$

$$
y= \sin(\theta_1 + \alpha)d_2 - l_1\sin{\alpha}\cos(\theta_1 + \alpha)
$$

$$
z=0
$$

최종 end-effector의 방향

$$
X_2= \begin{bmatrix} \sin(\theta_1 + \alpha) \\ -\cos(\theta_1 + \alpha) \\ 0 \end{bmatrix}
Y_2= \begin{bmatrix} 0 \\ 0 \\ -1 \end{bmatrix}
Z_2= \begin{bmatrix} \cos(\theta_1 + \alpha) \\ \sin(\theta_1 + \alpha) \\ 1 \end{bmatrix}
$$

---

![Pasted image 20260420105909.png](/assets/images/Pasted-image-20260420105909.png)

![스크린샷 2026-04-20 오후 12.17.26.png](/assets/images/스크린샷-2026-04-20-오후-12.17.26.png)

![Pasted image 20260420122232.png](/assets/images/Pasted-image-20260420122232.png)

$\theta^ʹ_i$ 는 $\theta_1$ 와 $\alpha$로 표현해서 변수를 줄인다.

$\theta^ʹ_i$ = ($\theta_1$ + $\alpha$) - $\frac{\pi}{2}$ [[덧셈정리]] 를 이용해서 정리하면 됨.

$d_2$ => z축 성분이 x , y 성분으로 분리됨.

![Pasted image 20260420134452.png](/assets/images/Pasted-image-20260420134452.png)

---

![Pasted image 20260410144417.png](/assets/images/Pasted-image-20260410144417.png)

![Pasted image 20260410150118.png](/assets/images/Pasted-image-20260410150118.png)

![스크린샷 2026-04-10 오후 4.33.18.png](/assets/images/스크린샷-2026-04-10-오후-4.33.18.png)

![스크린샷 2026-04-10 오후 4.46.27.png](/assets/images/스크린샷-2026-04-10-오후-4.46.27.png)

![Pasted image 20260410165000.png](/assets/images/Pasted-image-20260410165000.png)

![스크린샷 2026-04-10 오후 4.53.52.png](/assets/images/스크린샷-2026-04-10-오후-4.53.52.png)
