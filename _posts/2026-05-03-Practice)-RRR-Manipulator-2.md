---
layout: post
title: "Practice) RRR Manipulator 2"
date: 2026-04-22
categories: robotics
tags:
  - robotics
  - manipulator
---

<!-- 여기에 포스트 내용을 작성하세요 -->

#practice

![Pasted image 20260424113131.png](/assets/images/Pasted-image-20260424113131.png)

## 구조정의

1. joint
   1. joint1: Revoute - $\theta_1$
   2. joint2: Revoute - $\theta_2$
   3. joint3: Revoute - $\theta_3$
2. link
   1. link1: $\hat{Z_2}$ - $\hat{Z_3}$ - $l_1$
   2. link2: $\hat{Z_3}$ - $\hat{Z_{EE}}$ - $l_2$

## DH parameter - standard

1. DH table

| $i$ | $\alpha_i$       | $a_i$ | $d_i$ | $\theta_i$ |
| --- | ---------------- | ----- | ----- | ---------- |
| 1   | $\frac{\pi}{2}$  | 0     | 0     | $\theta_1$ |
| 2   | $-\frac{\pi}{2}$ | $l_1$ | 0     | $\theta_2$ |
| 3   | 0                | $l_2$ | 0     | $\theta_3$ |

2. 일반식
   $$
   ^{i-1}_iT = R_z(\theta_i)T_z(d_i)T_x(a_i)R_x(\alpha_i)
   $$
   $$ = \begin{bmatrix} \cos \theta_i & -\sin \theta_i \cos \alpha_i & \sin \theta_i \sin \alpha_i & a_i \cos \theta_i \\ \sin \theta_i & \cos \theta_i \cos \alpha_i & -\cos \theta_i \sin \alpha_i & a_i \sin \theta_i \\ 0 & \sin \alpha_i & \cos \alpha_i & d_i \\ 0 & 0 & 0 & 1 \end{bmatrix}$$
3. 변환행렬 도출
   $$
   ^0_3T = ^0_1T^1_2T^2_3T
   $$

$i=1$일때, $\alpha_1$ = $\frac{\pi}{2}$ , $\cos\alpha_1=0$ , $\sin\alpha_1=1$ , $a_1=0$ , $d_1=0$ , $\theta_1=\theta_1$

$$
^0_1T =
\begin{bmatrix}
\cos\theta_1 & 0 & \sin\theta_1 & 0 \\
\sin\theta_1 & 0 & -\cos\theta_1 & 0 \\
0 & 1 & 0 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

$i=2$일때, $\alpha_2$ = $-\frac{\pi}{2}$ , $\cos\alpha_2=0$ , $\sin\alpha_2=-1$ , $a_2=l_1$ , $d_2=0$ , $\theta_2=\theta_2$

$$
^1_2T= \begin{bmatrix}
\cos\theta_2 & 0 & -\sin\theta_2 & l_1\cos\theta_2 \\
\sin\theta_2 & 0 & \cos\theta_2 & l_1\sin\theta_2 \\
0 & -1 & 0 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

$i=3$일때, $\alpha_3$ = $0$ , $\cos\alpha_3=1$ , $\sin\alpha_3=0$ , $a_3=l_2$ , $d_3=0$ , $\theta_3=\theta_3$

$$
^2_3T = \begin{bmatrix}
\cos\theta_3 & -\sin\theta_3& 0 & l_2\cos\theta_3 \\
\sin\theta_3 & \cos\theta_3 & 0 & l_2\sin\theta_3 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

$$
^0_1T^1_2T^2_3T =
\begin{bmatrix}
\cos\theta_1 & 0 & \sin\theta_1 & 0 \\
\sin\theta_1 & 0 & -\cos\theta_1 & 0 \\
0 & 1 & 0 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
\cos\theta_2 & 0 & -\sin\theta_2 & l_1\cos\theta_2 \\
\sin\theta_2 & 0 & \cos\theta_2 & l_1\sin\theta_2 \\
0 & -1 & 0 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
\cos\theta_3 & -\sin\theta_3& 0 & l_2\cos\theta_3 \\
\sin\theta_3 & \cos\theta_3 & 0 & l_2\sin\theta_3 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

$$
=\begin{bmatrix}
C\theta_1C\theta_2C\theta_3-S\theta_1S\theta_3 & -C\theta_1C\theta_2S\theta_3-S\theta_1C\theta_3 & -C\theta_1S\theta_2 & l_2C\theta_1C\theta_2C\theta_3-l_2S\theta_1S\theta_3+l_1C\theta_1C\theta_2 \\
S\theta_1C\theta_2C\theta_3+C\theta_1S\theta_3 & -S\theta_1C\theta_2S\theta_3+C\theta_1C\theta_3 & -S\theta_1S\theta_2 & l_2S\theta_1C\theta_2C\theta_3+l_2C\theta_1S\theta_3+l_1S\theta_1C\theta_2 \\
S\theta_2C\theta_3 & -S\theta_2S\theta_3 & C\theta_2 & l_2S\theta_2C\theta_3+l_1S\theta_2 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

## DH parameter - modified

1. DH Table

| $i$               | $\alpha_{i-1}$   | $a_{i-1}$          | $d_i$ | $\theta_i$ |
| ----------------- | ---------------- | ------------------ | ----- | ---------- |
| 1                 | 0                | 0                  | 0     | $\theta_1$ |
| 2                 | $\frac{\pi}{2}$  | 0                  | 0     | $\theta_3$ |
| 3                 | $-\frac{\pi}{2}$ | $l_1$              | 0     | $\theta_3$ |
| $\color{red}{EE}$ | 0                | $\color{red}{l_2}$ | 0     | 0          |

#Realize modified 방식은 ee 전까지만 계산 최종 ee의 위치는 따로 변환식을 한번 더 적용해야한다. 2. 일반식

$$
^{i-1}_iT = R_x(\alpha_{i-1})T_x(a_{i-1})R_z({\theta}_i)T_z(d_i)
$$

$$
= \begin{bmatrix}
C{\theta_i} & -S{\theta_i} & 0 & a_{i-1} \\
S{\theta_i}C{\alpha_{i-1}} & C{\theta_i}C{\alpha_{i-1}} & -S{\alpha_{i-1}} & -d_{i}S{\alpha_{i-1}} \\
S{\theta_i}S{\alpha_{i-1}} & C{\theta_i}S{\alpha_{i-1}} & C{\alpha_{i-1}} & d_iC{\alpha_{i-1}} \\
 0 & 0 & 0 & 1
\end{bmatrix}
$$

2. 변환행렬 도출
   $$
   ^0_3T = ^0_1T^1_2T^2_3T
   $$
   $$
   i=1 , \alpha_0=0, \cos\alpha_0=1, \sin\alpha_0=0, a_0=0, d_1=0,
   \theta_1 = \theta_1
   $$
   $$
   ^0_1T= \begin{bmatrix}
   C{\theta_1} & -S{\theta_1} & 0 & 0 \\
   S{\theta_1} & C{\theta_1} & 0 & 0 \\
   0 & 0 & 1 & 0 \\
    0 & 0 & 0 & 1
   \end{bmatrix}
   $$
   $$
   i=2 , \alpha_1=\frac{\pi}{2}, \cos\alpha_1=0, \sin\alpha_1=1, a_1=0, d_2=0, \theta_2 = \theta_2
   $$
   $$
   ^1_2T= \begin{bmatrix}
   C{\theta_2} & -S{\theta_2} & 0 & 0 \\
   0 & 0 & -1 & 0 \\
   S{\theta_2} & C{\theta_2} & 0 & 0 \\
    0 & 0 & 0 & 1
   \end{bmatrix}
   $$
   $$
   i=3 , \alpha_2=-\frac{\pi}{2}, \cos\alpha_2=0, \sin\alpha_2=-1, a_2=l_1, d_3=0, \theta_3 = \theta_3
   $$
   $$
   ^2_3T= \begin{bmatrix}
   C{\theta_3} & -S{\theta_3} & 0 & l_1 \\
   0 & 0 & 1 & 0 \\
   -S{\theta_3} & -C{\theta_3} & 0 & 0 \\
    0 & 0 & 0 & 1
   \end{bmatrix}
   $$
   $$
   ^0_1T^1_2T^2_3T= \begin{bmatrix}
   C{\theta_1} & -S{\theta_1} & 0 & 0 \\
   S{\theta_1} & C{\theta_1} & 0 & 0 \\
   0 & 0 & 1 & 0 \\
    0 & 0 & 0 & 1
   \end{bmatrix}
   \begin{bmatrix}
   C{\theta_2} & -S{\theta_2} & 0 & 0 \\
   0 & 0 & -1 & 0 \\
   S{\theta_2} & C{\theta_2} & 0 & 0 \\
    0 & 0 & 0 & 1
   \end{bmatrix}
   \begin{bmatrix}
   C{\theta_3} & -S{\theta_3} & 0 & l_1 \\
   0 & 0 & 1 & 0 \\
   -S{\theta_3} & -C{\theta_3} & 0 & 0 \\
    0 & 0 & 0 & 1
   \end{bmatrix}
   $$
   $$
   = \begin{bmatrix}
   C\theta_1C\theta_2C\theta_3-S\theta_1S\theta_3 & -C\theta_1C\theta_2S\theta_3-S\theta_1C\theta_3 & -C\theta_1S\theta_2 & l_1C\theta_1C\theta_2 \\
   S\theta_1C\theta_2C\theta_3+C\theta_1S\theta_3 & -S\theta_1C\theta_2S\theta_3+C\theta_1C\theta_3 & -S\theta_1S\theta_2 & l_1S\theta_1C\theta_2 \\
   S\theta_2C\theta_3 & -S\theta_2S\theta_3 & C\theta_2 & l_1S\theta_2 \\
   0 & 0 & 0 & 1
   \end{bmatrix}
   $$

최종 EE 위치까지 계산.

$$
^0_3T^3_{EE}T = \begin{bmatrix}
C\theta_1C\theta_2C\theta_3-S\theta_1S\theta_3 & -C\theta_1C\theta_2S\theta_3-S\theta_1C\theta_3 & -C\theta_1S\theta_2 & l_1C\theta_1C\theta_2 \\
S\theta_1C\theta_2C\theta_3+C\theta_1S\theta_3 & -S\theta_1C\theta_2S\theta_3+C\theta_1C\theta_3 & -S\theta_1S\theta_2 & l_1S\theta_1C\theta_2 \\
S\theta_2C\theta_3 & -S\theta_2S\theta_3 & C\theta_2 & l_1S\theta_2 \\
0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
1 & 0 & 0 & l_2 \\
0 & 1 & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

$$
=\begin{bmatrix}
C\theta_1C\theta_2C\theta_3-S\theta_1S\theta_3 & -C\theta_1C\theta_2S\theta_3-S\theta_1C\theta_3 & -C\theta_1S\theta_2 & l_2(C\theta_1C\theta_2C\theta_3-S\theta_1S\theta_3)+l_1C\theta_1C\theta_2 \\
S\theta_1C\theta_2C\theta_3+C\theta_1S\theta_3 & -S\theta_1C\theta_2S\theta_3+C\theta_1C\theta_3 & -S\theta_1S\theta_2 & l_2(S\theta_1C\theta_2C\theta_3+C\theta_1S\theta_3)+l_1S\theta_1C\theta_2 \\
S\theta_2C\theta_3 & -S\theta_2S\theta_3 & C\theta_2 & l_2S\theta_2C\theta_3+l_1S\theta_2 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

## 최종 위치 도출

최종 end-effector 위치

$$
P_{EE} = \begin{bmatrix}
l_2C\theta_1C\theta_2C\theta_3-l_2S\theta_1S\theta_3+l_1C\theta_1C\theta_2  \\
l_2S\theta_1C\theta_2C\theta_3+l_2C\theta_1S\theta_3+l_1S\theta_1C\theta_2
\\
l_2S\theta_2C\theta_3+l_1S\theta_2
\end{bmatrix}
$$

최종 end-effector 방향

$$
X_3= \begin{bmatrix}
C\theta_1C\theta_2C\theta_3-S\theta_1S\theta_3 \\
S\theta_1C\theta_2C\theta_3+C\theta_1S\theta_3 \\
S\theta_2C\theta_3
\end{bmatrix}
Y_3= \begin{bmatrix}
-C\theta_1C\theta_2S\theta_3-S\theta_1C\theta_3 \\
-S\theta_1C\theta_2S\theta_3+C\theta_1C\theta_3 \\
-S\theta_2S\theta_3
\end{bmatrix}
Z_3= \begin{bmatrix}
-C\theta_1S\theta_2 \\
-S\theta_1S\theta_2 \\
C\theta_2
\end{bmatrix}
$$

![Pasted image 20260422093507.png](/assets/images/Pasted-image-20260422093507.png)
![스크린샷 2026-04-22 오후 12.01.57.png](/assets/images/스크린샷-2026-04-22-오후-12.01.57.png)
