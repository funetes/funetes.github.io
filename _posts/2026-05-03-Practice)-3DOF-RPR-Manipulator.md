---
layout: post
title: "Practice) 3DOF RPR Manipulator"
date: 2026-04-24
categories: Kinematics
tags:
  - robotics
  - manipulator
  - Kinematics
---

<!-- 여기에 포스트 내용을 작성하세요 -->

#practice
![스크린샷 2026-04-23 오후 2.41.40.png](/assets/images/스크린샷-2026-04-23-오후-2.41.40.png)

## 구조정의

1. joint
   1. $\hat Z_1$: revolute -> $\theta_1$ -> $\theta_1$
   2. $\hat Z_2$: prismatic -> $\theta_2 = 0$
   3. $\hat Z_3$: revolute -> $\theta_3 = \theta_3$
2. link
   1. $\hat X_1$ : $l_1$ => $a_1$
   2. $\hat X_2$: 0 => $d_2$
   3. $\hat X_3$: $l_3$

## DH parameter - standard

1.  DH table
    #Realize $d_2$, $-\theta_3$

| $i$ |    $\alpha_i$    |                            $a_i$                             | $d_i$ |                            $\theta_i$                            |
| :-: | :--------------: | :----------------------------------------------------------: | :---: | :--------------------------------------------------------------: |
|  1  | $-\frac{\pi}{2}$ |                            $l_1$                             |   0   |                            $\theta_1$                            |
|  2  | $\frac{\pi}{2}$  | $\color{red}{0}$ - 링크전체를 사용하는것이 아님, $d_2$만큼만 | $d_2$ |                                0                                 |
|  3  |        0         |                            $l_3$                             |   0   | ${\color{red}{-}} \theta_3$ -> EE 각이 오른손법칙에서 반대방향임 |

2. 일반식

   $$
   ^{i-1}_iT = R_z(\theta_i)T_z(d_i)T_x(a_i)R_x(\alpha_i)
   $$

   $$
   =\begin{bmatrix} \cos \theta_i & -\sin \theta_i \cos \alpha_i & \sin \theta_i \sin \alpha_i & a_i \cos \theta_i \\ \sin \theta_i & \cos \theta_i \cos \alpha_i & -\cos \theta_i \sin \alpha_i & a_i \sin \theta_i \\ 0 & \sin \alpha_i & \cos \alpha_i & d_{i} \\ 0 & 0 & 0 & 1
   \end{bmatrix}
   $$

3. 변환행렬 도출

$$
^0_1T= \begin{bmatrix}
  C_{1} & 0 & -S_{1} & l_{1} C_{1} \\
  S_{1} & 0 & C_{1} & l_{1} S_{1} \\
  0 & -1 & 0 & 0 \\
  0 & 0 & 0 & 1
\end{bmatrix}
{^1_2T=\begin{bmatrix}
  1 & 0 & 0 & 0 \\
  0 & 0 & -1 & 0 \\
  0 & 1 & 0 & d_{2} \\
  0 & 0 & 0 & 1
\end{bmatrix}}
$$

$$
^2_3T= \begin{bmatrix}
  C(-\theta _{3}) & -S(-\theta _{3}) & 0 & l_{3} C(-\theta _{3}) \\
  S(-\theta _{3}) & C(-\theta _{3}) & 0 & l_{3} S(-\theta _{3}) \\
  0 & 0 & 1 & 0 \\
  0 & 0 & 0 & 1
\end{bmatrix}
= \begin{bmatrix}
  C(\theta _{3}) & S(\theta _{3}) & 0 & l_{3} C(\theta _{3}) \\
  -S(\theta _{3}) & C(\theta _{3}) & 0 & -l_{3} S(\theta _{3}) \\
  0 & 0 & 1 & 0 \\
  0 & 0 & 0 & 1
\end{bmatrix}
$$

$$
^0_1T^1_2T^2_3T = \begin{bmatrix}
  C_{3} C_{1} + S_{3} S_{1} & S_{3} C_{1} - C_{3} S_{1} & 0 & C_{3} l_{3} C_{1} + l_{3} S_{3} S_{1} - d_{2} S_{1} + l_{1} C_{1} \\
  C_{3} S_{1} - S_{3} C_{1} & S_{3} S_{1} + C_{3} C_{1} & 0 & C_{3} l_{3} S_{1} - l_{3} S_{3} C_{1} + d_{2} C_{1} + l_{1} S_{1} \\
  0 & 0 & 1 & 0 \\
  0 & 0 & 0 & 1
\end{bmatrix}
$$

$$ =\begin{bmatrix} C*{1-3} & -S*{1-3} & 0 & l*{3} C*{1-3} - d*{2} S*{1} + l*{1} C*{1} \\ S*{1-3} & C*{1-3} & 0 & l*{3} S*{1-3} + d*{2} C*{1} + l*{1} S*{1} \\ 0 & 0 & 1 & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix} $$

---

## DH parameter - modified

1. DH table

| $i$ |  $\alpha_{i-1}$  | $a_{i-1}$ | $d_i$ | $\theta_i$  |
| :-: | :--------------: | :-------: | :---: | :---------: |
|  1  |        0         |     0     |   0   | $\theta_1$  |
|  2  | $-\frac{\pi}{2}$ |   $l_1$   | $d_2$ |      0      |
|  3  | $\frac{\pi}{2}$  |     0     |   0   | $-\theta_3$ |
| EE  |        0         |   $l_3$   |   0   |      0      |

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

2. 변환행렬 도출

$$
^0_1T= \begin{bmatrix}
  C_{1} & -S_{1} & 0 & 0 \\
  S_{1} & C_{1} & 0 & 0 \\
  0 & 0 & 1 & 0 \\
  0 & 0 & 0 & 1
\end{bmatrix}
{^1_2T= \begin{bmatrix}
  1 & 0 & 0 & l_{1} \\
  0 & 0 & 1 & d_{2} \\
  0 & -1 & 0 & 0 \\
  0 & 0 & 0 & 1
\end{bmatrix}}
$$

음각공식 적용,

$$
^2_3T= \begin{bmatrix}
  C(-\theta _{3}) & -S(-\theta _{3}) & 0 & 0 \\
  0 & 0 & -1 & 0 \\
  S(-\theta _{3}) & C(-\theta _{3}) & 0 & 0 \\
  0 & 0 & 0 & 1
\end{bmatrix} = \begin{bmatrix}
  C_{3} & S_{3} & 0 & 0 \\
  0 & 0 & -1 & 0 \\
  -S_{3} & C_{3} & 0 & 0 \\
  0 & 0 & 0 & 1
\end{bmatrix}
$$

$$
^3_{EE}T= \begin{bmatrix}
  1 & 0 & 0 & l_{3} \\
  0 & 1 & 0 & 0 \\
  0 & 0 & 1 & 0 \\
  0 & 0 & 0 & 1
\end{bmatrix}
$$

$$
^0_1T^1_2T^2_3T^3_{EE}T= \begin{bmatrix}
  C_{3} C_{1} + S_{3} S_{1} & S_{3} C_{1} - C_{3} S_{1} & 0 & C_{3} l_{3} C_{1} + l_{3} S_{3} S_{1} + l_{1} C_{1} - d_{2} S_{1} \\
  C_{3} S_{1} - S_{3} C_{1} & S_{3} S_{1} - C_{3} C_{1} & 0 & C_{3} l_{3} S_{1} - l_{3} S_{3} C_{1} + l_{1} S_{1} + d_{2} C_{1} \\
  0 & 0 & 1 & 0 \\
  0 & 0 & 0 & 1
\end{bmatrix}
$$

$$^0_{EE}T = \begin{bmatrix} C_{1-3} & -S_{1-3} & 0 & l_3 C_{1-3} + l_1 C_1 - d_2 S_1 \\ S_{1-3} & C_{1-3} & 0 & l_3 S_{1-3} + l_1 S_1 + d_2 C_1 \\ 0 & 0 & 1 & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix}$$

---

## 최종 kinematics 도출

1.  End-Effector 위치 1. X좌표
    $$
    x= l_3 C_{1-3} + l_1 C_1 - d_2 S_1
    $$
        2. Y좌표
        $$
    y= l*3 S*{1-3} + l_1 S_1 + d_2 C_1
    $$
    	3. Z좌표
    $$
    z=0
    $$
2.  End-Effector 방향
    $$
    X_3=\begin{bmatrix}
    C_{1-3} \\
    S_{1-3} \\
    0
    \end{bmatrix}
    Y_3=\begin{bmatrix}
    -S_{1-3} \\
    C_{1-3} \\
    0
    \end{bmatrix}
    Z_3=\begin{bmatrix}
    0 \\
    0 \\
    1
    \end{bmatrix}
    $$
3.  최종 Forward Kinematics 요약
    $$
    x= l_3 C_{1-3} + l_1 C_1 - d_2 S_1
    $$
    $$
    y= l_3 S_{1-3} + l_1 S_1 + d_2 C_1
    $$
    $$
    z=0
    $$
4.  구조해석

![Pasted image 20260424110112.png](/assets/images/Pasted-image-20260424110112.png)

---

# Practice RPR Manipulator

![Pasted image 20260424160023.png](/assets/images/Pasted-image-20260424160023.png)

## 구조정의

1. joint
   1. 원통형 revolute $z_1$ -> $z_2$: $\theta_1$
   2. revolute $z_2$ -> $z_3$: $\theta_2$
   3. prismatic
2. link
   1. $\hat X_1:l_1$
   2. $\hat X_2:l_2$
   3. $\hat X_3:d_3$

## DH parameter - Standard

1. DH Table

| $i$ | $\alpha_i$       | $a_i$ | $d_i$ | $\theta_i$ |
| --- | ---------------- | ----- | ----- | ---------- |
| 1   | $\frac{\pi}{2}$  | $l_1$ | 0     | $\theta_1$ |
| 2   | $-\frac{\pi}{2}$ | $l_2$ | 0     | $\theta_2$ |
| 3   | 0                | 0     | $d_3$ | 0          |

2. 일반식

   $$
   ^{i-1}_iT = R_z(\theta_i)T_z(d_i)T_x(a_i)R_x(\alpha_i)
   $$

   $$ = \begin{bmatrix} \cos \theta*i & -\sin \theta_i \cos \alpha_i & \sin \theta_i \sin \alpha_i & a_i \cos \theta_i \\ \sin \theta_i & \cos \theta_i \cos \alpha_i & -\cos \theta_i \sin \alpha_i & a_i \sin \theta_i \\ 0 & \sin \alpha_i & \cos \alpha_i & d*{i} \\ 0 & 0 & 0 & 1 \end{bmatrix}$$

3. 변환행렬 도출
   $$
   ^{0}_{1}T=
   \begin{bmatrix}
     C_{1} & 0 & S_{1} & C_{1}l_{1} \\
     S_{1} & 0 & -C_{1} & S_{1}l_{1} \\
     0 & 1 & 0 & 0 \\
     0 & 0 & 0 & 1
   \end{bmatrix}
   {^{1}_{2}T= \begin{bmatrix}
   C_{1} & 0 & S_{1} & C_{1}l_{1} \\
   S_{1} & 0 & -C_{1} & S_{1}l_{1} \\
   0 & 1 & 0 & 0 \\
   0 & 0 & 0 & 1
   \end{bmatrix}}
   {^{2}_{3}T= \begin{bmatrix}
   1 & 0 & 0 & 0 \\
     0 & 1 & 0 & 0 \\
     0 & 0 & 1 & d_{3} \\
     0 & 0 & 0 & 1
   \end{bmatrix}}
   $$

$$
^{0}_{3}T=
\begin{bmatrix}
  C_{1}C_{2} & -S_{1} & -C_{1}S_{2} & -C_{1}S_{2}d_{3} + C_{1}C_{2}l_{2} + C_{1}l_{1} \\
  C_{2}S_{1} & C_{1} & -S_{1}S_{2} & -S_{1}S_{2}d_{3} + C_{2}S_{1}l_{2} + S_{1}l_{1} \\
  S_{2} & 0 & C_{2} & C_{2}d_{3} + S_{2}l_{2} \\
  0 & 0 & 0 & 1
\end{bmatrix}
$$
