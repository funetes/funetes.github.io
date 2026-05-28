---
layout: post
title: "Practice) 3DOF Spatial type robot"
date: 2026-04-21
categories: Kinematics
tags:
  - robotics
  - manipulator
  - Kinematics
---

<!-- 여기에 포스트 내용을 작성하세요 -->

#practice

![Pasted image 20260424114450.png](/assets/images/Pasted-image-20260424114450.png)

## 구조정의

1. 공간형 type RRR robot
2. joint
   1. joint1: $z_1$축 회전 $\theta_1$ (Base Yaw)
   2. joint2: $z_2$축 회전 $\theta_2$ (Shoulder Pitch)
   3. joint3: $z_3$축 회전 $\theta_3$ (Elbow Pitch)
3. link
   1. link1: $l_1$ : j2 ->j3
   2. link2: $l_2$ : j3 -> End-Effector

## DH parameter - standard

1. DH table

| i   | $\alpha_i$      | $a_i$ | $d_i$ | $\theta_i$ |
| --- | --------------- | ----- | ----- | ---------- |
| 1   | $\frac{\pi}{2}$ | 0     | 0     | $\theta_1$ |
| 2   | 0               | $l_1$ | 0     | $\theta_2$ |
| 3   | 0               | $l_2$ | 0     | $\theta_3$ |

2. 일반식

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

$i=2$일때, $\alpha_2=0$ , $\cos\alpha_2=1$ , $\sin\alpha_2=0$ , $a_2=l_1$ , $d_2=0$ , $\theta_2=\theta_2$

$$
^1_2T =
\begin{bmatrix}
\cos\theta_2 & -\sin\theta_2 & 0 & l_1\cos\theta_2 \\
\sin\theta_2 & \cos\theta_2 & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

$i=3$일때, $\alpha_3=0$ , $\cos\alpha_3=1$ , $\sin\alpha_3=0$ , $a_3=l_2$ , $d_3=0$ , $\theta_3=\theta_3$

$$
^2_3T =
\begin{bmatrix}
\cos\theta_3 & -\sin\theta_3 & 0 & l_2\cos\theta_3 \\
\sin\theta_3 & \cos\theta_3 & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

$$
^0_3T =
\begin{bmatrix}
\cos\theta_1 & 0 & \sin\theta_1 & 0 \\
\sin\theta_1 & 0 & -\cos\theta_1 & 0 \\
0 & 1 & 0 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
\cos\theta_2 & -\sin\theta_2 & 0 & l_1\cos\theta_2 \\
\sin\theta_2 & \cos\theta_2 & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
\cos\theta_3 & -\sin\theta_3 & 0 & l_2\cos\theta_3 \\
\sin\theta_3 & \cos\theta_3 & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

## DH parameter - modified

1. DH Table

| $i$ | $\alpha_{i-1}$  | $a_{i-1}$ | $d_i$ | $\theta_i$ |
| --- | --------------- | --------- | ----- | ---------- |
| 1   | 0               | 0         | 0     | $\theta_1$ |
| 2   | $\frac{\pi}{2}$ | 0         | 0     | $\theta_2$ |
| 3   | 0               | $l_1$     | 0     | $\theta_3$ |

2. 일반식
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

3. 변환행렬 도출
   $i=1$일때, $\alpha_0=0$ , $\cos\alpha_0=1$ , $\sin\alpha_0=0$ , $a_0=0$ , $d_1=0$ , $\theta_1=\theta_1$

$$
^0_1T=\begin{bmatrix}
C\theta_1 & -S\theta_1 & 0 & 0 \\
S\theta_1 & C\theta_1 & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

$i=2$일때, $\alpha_1=\frac{\pi}{2}$ , $\cos\alpha_1=0$ , $\sin\alpha_1=1$ , $a_1=0$ , $d_2=0$ , $\theta_2=\theta_2$

$$
^1_2T=\begin{bmatrix}
C\theta_2 & -S\theta_2 & 0 & 0 \\
0 & 0 & -1 & 0 \\
S\theta_2 & C\theta_2 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

$i=3$일때, $\alpha_2=0$ , $\cos\alpha_2=1$ , $\sin\alpha_2=0$ , $a_2=l_1$ , $d_3=0$ , $\theta_3=\theta_3$

$$
^2_3T=\begin{bmatrix}
C\theta_3 & -S\theta_3 & 0 & l_1 \\
S\theta_3 & C\theta_3 & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

$$
^0_3T=\begin{bmatrix}
C\theta_1C(\theta_2+\theta_3) & -C\theta_1S(\theta_2+\theta_3) & S\theta_1 & l_1C\theta_1C\theta_2 \\
S\theta_1C(\theta_2+\theta_3) & -S\theta_1S(\theta_2+\theta_3) & -C\theta_1 & l_1S\theta_1C\theta_2 \\
S(\theta_2+\theta_3) & C(\theta_2+\theta_3) & 1 & l_1S\theta_2 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

#Realize 위 변환식은 ${l_2}$ 의 길이가 포함되지 않은 식임. $l_2$를 포함하려면 $T_x(l_2)$ 를 곱해주면 된다.

$$
^0_3T^3_{EE}T=\begin{bmatrix}
C\theta_1C(\theta_2+\theta_3) & -C\theta_1S(\theta_2+\theta_3) & S\theta_1 & l_1C\theta_1C\theta_2 \\
S\theta_1C(\theta_2+\theta_3) & -S\theta_1S(\theta_2+\theta_3) & -C\theta_1 & l_1S\theta_1C\theta_2 \\
S(\theta_2+\theta_3) & C(\theta_2+\theta_3) & 1 & l_1S\theta_2 \\
0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
1 & 0 & 0 & l_2 \\
0 & 1 & 0 & 0 \\
0 & 0 & 1 & 0  \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

$$
=\begin{bmatrix}
C\theta_1C(\theta_2+\theta_3) & -C\theta_1S(\theta_2+\theta_3) & S\theta_1 & l_2C\theta_1(\theta_2+\theta_3)+l_1C\theta_1C\theta_2 \\
S\theta_1C(\theta_2+\theta_3) & -S\theta_1S(\theta_2+\theta_3) & -C\theta_1 & l_2S\theta_1(\theta_2+\theta_3)+l_1S\theta_1C\theta_2 \\
S(\theta_2+\theta_3) & C(\theta_2+\theta_3) & 1 & l_2S(\theta_2+\theta_3)+l_1S\theta_2 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

최종 end-effector의 위치벡터는

$$
P_{EE} = \begin{bmatrix}
l_2C\theta_1(\theta_2+\theta_3)+l_1C\theta_1C\theta_2 \\
l_2S\theta_1(\theta_2+\theta_3)+l_1S\theta_1C\theta_2 \\
l_2S(\theta_2+\theta_3)+l_1S\theta_2
\end{bmatrix}
$$

---

![Pasted image 20260421133034.png](/assets/images/Pasted-image-20260421133034.png)

![Pasted image 20260421140509.png](/assets/images/Pasted-image-20260421140509.png)
![스크린샷 2026-04-21 오후 2.06.03.png](/assets/images/스크린샷-2026-04-21-오후-2.06.03.png)
![스크린샷 2026-04-21 오후 2.06.29.png](/assets/images/스크린샷-2026-04-21-오후-2.06.29.png)
