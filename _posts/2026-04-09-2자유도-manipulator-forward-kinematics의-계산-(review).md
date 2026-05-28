---
layout: post
title: "2자유도 manipulator forward kinematics의 계산 (review)"
date: 2026-04-09
categories: Kinematics
tags:
  - robotics
  - manipulator
  - Kinematics
---

![Pasted image 20260409101830.png](/assets/images/Pasted-image-20260409101830.png)

![Pasted image 20260409103850.png](/assets/images/Pasted-image-20260409103850.png)

![Pasted image 20260409104317.png](/assets/images/Pasted-image-20260409104317.png)
지금 배운건 D-H parameter의 *modified 된 방법*임 원래 방식은 약간 다름.

#### D-H parameter (standard)

![Pasted image 20260409164443.png](/assets/images/Pasted-image-20260409164443.png)

- 붉은색 글자를 따라가면됨. (i축이 시작임)

![스크린샷 2026-04-09 오전 10.54.29.png](/assets/images/스크린샷-2026-04-09-오전-10.54.29.png)

![스크린샷 2026-04-09 오후 1.56.53.png](/assets/images/스크린샷-2026-04-09-오후-1.56.53.png)

![스크린샷 2026-04-09 오후 2.07.08.png](/assets/images/스크린샷-2026-04-09-오후-2.07.08.png)

- **$\theta_{i}$ (Joint Angle):** $z_{i-1}$ 축을 중심으로 $x_{i-1}$ 축이 $x_{i}$ 축과 일치하도록 회전한 각도
- **$d_{i}$ (Joint Offset):** $z_{i-1}$ 축을 따라 $x_{i-1}$ 축에서 $x_{i}$ 축까지의 거리
- **$a_{i}$ (Link Length):** $x_{i}$ 축을 따라 $z_{i-1}$ 축에서 $z_{i}$ 축까지의 거리
- **$\alpha_{i}$ (Link Twist):** $x_{i}$ 축을 중심으로 $z_{i-1}$ 축이 $z_{i}$ 축과 일치하도록 회전한 각도

### [[Forwrad Kinematics 최종 Flow]]

#### D-H parameter (standard)

##### 일반식

$$
^{i-1}_iT = R_z(\theta_i)T_z(d_i)T_x(a_i)R_x(\alpha_i)
$$

$$
=\begin{bmatrix} C_{\theta_i} & -S_{\theta_i} & 0 & 0 \\ S_{\theta_i} & C_{\theta_i} & 0 & 0 \\ 0 & 0 & 1 & 0 \\ 0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix} 1 & 0 & 0 & 0 \\ 0 & 1 & 0 & 0 \\ 0 & 0 & 1 & d_i \\ 0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix} 1 & 0 & 0 & a_i \\ 0 & 1 & 0 & 0 \\ 0 & 0 & 1 & 0 \\ 0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix} 1 & 0 & 0 & 0 \\ 0 & C_{\alpha_i} & -S\alpha_i & 0 \\ 0 & S\alpha_i & C\alpha_i & 0 \\ 0 & 0 & 0 & 1
\end{bmatrix}
$$

$$ = \begin{bmatrix} \cos \theta*{i} & -\sin \theta*{i} \cos \alpha*{i} & \sin \theta*{i} \sin \alpha*{i} & a*{i} \cos \theta*{i} \\ \sin \theta*{i} & \cos \theta*{i} \cos \alpha*{i} & -\cos \theta*{i} \sin \alpha*{i} & a*{i} \sin \theta*{i} \\ 0 & \sin \alpha*{i} & \cos \alpha*{i} & d\_{i} \\ 0 & 0 & 0 & 1 \end{bmatrix}$$

##### 예제

#practice
![스크린샷 2026-04-09 오후 5.22.07.png](/assets/images/스크린샷-2026-04-09-오후-5.22.07.png)

joint parameter: ${\theta}_i$ , $d_i$
link parameter: $\alpha_{i}$ , $l_i$

\*joint -> link 순서임

변수를 구해야 한다. #yet

| i   | ${\theta}_i$ | $d_i$ | $\alpha_{i}$ | $l_i$ |
| --- | ------------ | ----- | ------------ | ----- |
| 1   | ${\theta_1}$ | 0     | 0            | $l_1$ |
| 2   | ${\theta_2}$ | 0     | 0            | $l_2$ |

에서 $^0_2T$ 를 구하면,

$$
^0_2T = ^0_1T ^1_2T
$$

아래 식은 일반식을 이용해서 풀면 나옴

$$
^0_1T=
\begin{bmatrix}
C{\theta_1} & -S{\theta_1} & 0 & l_1C{\theta_1} \\
S{\theta_1} & C{\theta_1} & 0 & l_1S{\theta_1} \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

$$
^1_2T=
\begin{bmatrix}
C{\theta_2} & -S{\theta_2} & 0 & l_2C{\theta_2} \\
S{\theta_1} & C{\theta_1} & 0 & l_2S{\theta_2} \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

$$
^0_1T ^1_2T=
\begin{bmatrix}
C{\theta_1} & -S{\theta_1} & 0 & l_1C{\theta_1} \\
S{\theta_1} & C{\theta_1} & 0 & l_1S{\theta_1} \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
C{\theta_2} & -S{\theta_2} & 0 & l_2C{\theta_2} \\
S{\theta_1} & C{\theta_1} & 0 & l_2S{\theta_2} \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

$$
= \begin{bmatrix}
C{\theta_1}C{\theta_2}-S{\theta_1}S{\theta_2} &
-(C{\theta_1}S{\theta_2}+C{\theta_1}S{\theta_2}) &
0 &
l_2(C{\theta_1}C{\theta_2}-S{\theta_1}S{\theta_2}) \\
S{\theta_1}C{\theta_2}+C{\theta_1}S{\theta_2} &
-S{\theta_1}S{\theta_2}+C{\theta_1}C{\theta_2} &
0 &
l_2(S{\theta_1}C{\theta_2}+C{\theta_1}S{\theta_2}) \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

-> #todo 위 식 덧셈정리 적용 만들어놓기

---

#### D-H parameter (modified)

##### 일반식 #Realize

link parameter: $\alpha_{i}$ , $l_i$
joint parameter: ${\theta}_i$ , $d_i$

\*link -> joint 순서임

$$
^{i-1}_iT = R_x(\alpha_{i-1})T_x(a_{i-1})R_z({\theta}_i)T_z(d_i)
$$

$$
\begin{bmatrix}
1 & 0 & 0 & 0 \\
0 & C_{\alpha_{i-1}} & -S_{\alpha_{i-1}} & 0 \\
0 & S_{\alpha_{i-1}} & C_{\alpha_{i-1}} & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
1 & 0 & 0 & a_{i-1} \\
0 & 1 & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
 C_{\theta_{i}} & -S_{\theta_{i}} & 0 & 0 \\
 S_{\theta_{i}} & C_{\theta_{i}} & 0 & 0 \\
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

#todo 중간계산식 만들기

최종 일반식 (=일반 변환행렬)

$$
\begin{bmatrix}
C{\theta_i} & -S{\theta_i} & 0 & a_{i-1} \\
S{\theta_i}C{\alpha_{i-1}} & C{\theta_i}C{\alpha_{i-1}} & -S{\alpha_{i-1}} & -d_{i}S{\alpha_{i-1}} \\
S{\theta_i}S{\alpha_{i-1}} & C{\theta_i}S{\alpha_{i-1}} & C{\alpha_{i-1}} & d_iC{\alpha_{i-1}} \\
 0 & 0 & 0 & 1
\end{bmatrix}
$$

##### 예제

#practice
DH table #Realize #yet

modified 와 standard는 table이 서로 다르다 - 어떻게 다른가 설명할수 있어야함

| i   | ${\alpha}_{i-1}$ | $a_{i-1}$ | $d_{i}$ | ${\theta_i}$ |
| --- | ---------------- | --------- | ------- | ------------ |
| 1   | 0                | 0         | 0       | ${\theta_1}$ |
| 2   | 0                | $l_1$     | 0       | ${\theta_2}$ |
| EE  | 0                | $l_2$     | 0       | 0            |

변수를 바탕으로 일반식을 도출하자

$$
^0_1T= R_x(\alpha_0)T_x(a_0)R_z({\theta}_1)T_z(d_1)
$$

이고, 이 식을 행렬로 만들면,

$$
^0_1T=
\begin{bmatrix}
1 & 0 & 0 & 0 \\
0 & C{\alpha_0} & -S{\alpha_0} & 0 \\
0 & S{\alpha_0} & C{\alpha_0} & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
1 & 0 & 0 & a_0 \\
0 & 1 & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
C{\theta_1} & -S{\theta_1} & 0 & 0 \\
S{\theta_1} & C{\theta_1} & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
1 & 0 & 0 & 0 \\
0 & 1 & 0 & 0 \\
0 & 0 & 1 & d_1 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

$\alpha_0 = 0$, 따라서 cos(0) = 1, sin(0) = 0
$a_0=0$,
$d_1=0$ 임.

정리하면,

$$
^0_1T=
\begin{bmatrix}
C{\theta_1} & -S{\theta_1} & 0 & 0 \\
S{\theta_1} & C{\theta_1} & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1 \\
\end{bmatrix}
$$

$$
^1_2T= R_x(\alpha_1)T_x(a_1)R_z({\theta}_2)T_z(d_2)
$$

$$
^1_2T=
\begin{bmatrix}
1 & 0 & 0 & 0 \\
0 & C{\alpha_1} & -S{\alpha_1} & 0 \\
0 & S{\alpha_1} & C{\alpha_1} & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
1 & 0 & 0 & a_1 \\
0 & 1 & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
C{\theta_2} & -S{\theta_2} & 0 & 0 \\
S{\theta_2} & C{\theta_2} & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
1 & 0 & 0 & 0 \\
0 & 1 & 0 & 0 \\
0 & 0 & 1 & d_2 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

$\alpha_1 = 0$, 따라서 cos(0) = 1, sin(0) = 0
$a_1=l_1$,
$d_1=0$ 임.

정리하면,

$$
^1_2T=
\begin{bmatrix}
C{\theta_2} & -S{\theta_2} & 0 & l_1 \\
S{\theta_2} & C{\theta_2} & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1 \\
\end{bmatrix}
$$

따라서,

$$
^0_2T=^0_1T^1_2T
$$

$$
^0_2T=
\begin{bmatrix}
C{\theta_1} & -S{\theta_1} & 0 & 0 \\
S{\theta_1} & C{\theta_1} & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1 \\
\end{bmatrix}
\begin{bmatrix}
C{\theta_2} & -S{\theta_2} & 0 & l_1 \\
S{\theta_2} & C{\theta_2} & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1 \\
\end{bmatrix}
$$

아직 최종 EE 까지 계산된것이 아님

$$
^2T_{EE} = R_x(\alpha_2)T_x(a_2)R_z({\theta}_{EE})T_z(d_{EE})
$$

$\alpha_2 = 0$, 따라서 cos(0) = 1, sin(0) = 0
$a_2=l_2$,
$d_{EE}=0$ 임.

$T_x(a_2)$ 만 남는다.

$$
T_x(l_2)=
\begin{bmatrix}
1 & 0 & 0 & l_2 \\
0 & 1 & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

최종,

$$
^0_1T^1_0T^2T_{EE} =
\begin{bmatrix}
C{\theta_1} & -S{\theta_1} & 0 & 0 \\
S{\theta_1} & C{\theta_1} & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1 \\
\end{bmatrix}
\begin{bmatrix}
C{\theta_2} & -S{\theta_2} & 0 & l_1 \\
S{\theta_2} & C{\theta_2} & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1 \\
\end{bmatrix}
\begin{bmatrix}
1 & 0 & 0 & l_2 \\
0 & 1 & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

회전행렬과 최종위치를 도출 (3x3) (3x1)

$$
=\begin{bmatrix}
C{\theta_1}C{\theta_2}-S{\theta_1}S{\theta_2} &
-(C{\theta_1}S{\theta_2}+C{\theta_1}S{\theta_2}) &
0 &
l_1C_1+l_2(C{\theta_1}C{\theta_2}-S{\theta_1}S{\theta_2}) \\
S{\theta_1}C{\theta_2}+C{\theta_1}S{\theta_2} &
-S{\theta_1}S{\theta_2}+C{\theta_1}C{\theta_2} &
0 &
l_1S_1+l_2(S{\theta_1}C{\theta_2}+C{\theta_1}S{\theta_2}) \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

#todo 덧셈정리식으로 정리한거 넣기
