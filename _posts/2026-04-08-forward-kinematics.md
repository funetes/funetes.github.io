---
layout: post
title: "forward-kinematics"
date: 2026-04-08
categories: Kinematics
tags:
  - robotics
  - manipulator
  - Kinematics
---

<!-- 여기에 포스트 내용을 작성하세요 -->

# Forward Kinematics (정방향 기구학)

## 관절 변수(_Joint variables_)가 주어졌을 때, 말단(_End-Effector_)의 ==위치와 자세를 계산==하는 문제

![Pasted image 20260408101443.png](/assets/images/Pasted-image-20260408101443.png)

- 링크와 링크사이의 각($\theta$)을 알 때 // 말단의 위치, 방위가 어떻게 되니?
  $$
  q\ -> ^W_ET\ -> EE\ position\ and\ orientation
  $$
- 함수적 표현
  - input: 관절각 / 변위(q)
  - output: end-effector pose(position + orientation)

![Pasted image 20260408104140.png](/assets/images/Pasted-image-20260408104140.png)

---

## 관절의 종류

[[자유도 (DOF)]]

### Revolute Joint (1 DOF)

![Pasted image 20260408104502.png](/assets/images/Pasted-image-20260408104502.png)

### Prismatic Joint(2DOF)

![스크린샷 2026-04-08 오전 10.48.00.png](/assets/images/스크린샷-2026-04-08-오전-10.48.00.png)

### Cylindrial Joint(2DOF)

![스크린샷 2026-04-08 오전 10.49.00.png](/assets/images/스크린샷-2026-04-08-오전-10.49.00.png)

### Spherical Joint(3DOF)

![스크린샷 2026-04-08 오전 10.55.35.png](/assets/images/스크린샷-2026-04-08-오전-10.55.35.png)

### Planar Joint(3DOF)

![스크린샷 2026-04-08 오전 10.56.07.png](/assets/images/스크린샷-2026-04-08-오전-10.56.07.png)

### Screw Joint(1DOF)

![스크린샷 2026-04-08 오전 11.00.02.png](/assets/images/스크린샷-2026-04-08-오전-11.00.02.png)

## 기본 정의

### 링크의 정의

- 2개이상의 절점(관절(joint))이 있는 강체

#### \*Link Length(a)

- ![스크린샷 2026-04-08 오전 11.43.53.png](/assets/images/스크린샷-2026-04-08-오전-11.43.53.png)
- ![스크린샷 2026-04-08 오전 11.46.40.png](/assets/images/스크린샷-2026-04-08-오전-11.46.40.png)
  - ![스크린샷 2026-04-08 오전 11.45.50.png](/assets/images/스크린샷-2026-04-08-오전-11.45.50.png)

#### \*Twist Angle($\alpha$)

- Link의 양단에 부착된 Joint Axis 간의 \*비틀림각
- ![스크린샷 2026-04-08 오전 11.50.14.png](/assets/images/스크린샷-2026-04-08-오전-11.50.14.png)
  > Denavit-Hartenburg parameters 에서 ${\alpha}$ 이고
  > 회전행렬 계산할때의 로테이션이 여기서 계산됨.

### 관절의 정의

- 두 링크(강체 세그먼트)를 연결해 특정한 운동(회전, 선형 등)을 가능하게 하는 _관절_

#### \*Joint Angle(${\theta}$)

- 링크에 대한 그 다음(인접한) 링크의 회전각
- ${\theta}$ : Relative Angle
- ![스크린샷 2026-04-08 오전 11.54.48.png](/assets/images/스크린샷-2026-04-08-오전-11.54.48.png)

#### \*Joint Displacement(d)

- 링크에 대한 그 다음(인접한)링크의 미끄럼 변위(d)
-
- ![스크린샷 2026-04-08 오전 11.56.16.png](/assets/images/스크린샷-2026-04-08-오전-11.56.16.png)

### \*Denavit-Hartenburg parameters

- ![스크린샷 2026-04-08 오후 12.00.03.png](/assets/images/스크린샷-2026-04-08-오후-12.00.03.png)
- [[Denavit-Hartenberg (D-H) Parameters]] - gemini
- ![Pasted image 20260408120358.png](/assets/images/Pasted-image-20260408120358.png)
- ![Pasted image 20260409164547.png](/assets/images/Pasted-image-20260409164547.png)
  - 색이 들어있는 좌표계를 보면 됨.
- 4개의 변수를 이용해서 제어.
- ![Pasted image 20260408120603.png](/assets/images/Pasted-image-20260408120603.png)

- 원래 3차원 공간에서 물체의 위치와 방향을 나타내려면 6개의 파라미터(위치 3개, 방향 3개)가 필요합니다. 하지만 D-H 방식은 특정 규칙(z축을 회전축으로 잡는 등)을 정해놓음으로써 **최소한의 정보(4개)**만으로 로봇의 움직임을 완벽하게 설명할 수 있게 해줍니다.
- [행렬표현](https://wisepubliclife.tistory.com/8)
- [Denavit-Hartenberg Reference Frame Layout](https://www.youtube.com/watch?v=rA9tm0gTln8&pp=ygUdRGVuYXZpdC1IYXJ0ZW5idXJnIHBhcmFtZXRlcnPSBwkJ2QoBhyohjO8%3D)
- [Denavit - Hartenberg (DH) Tables For Robotic Systems - Direct Kinematics II](https://www.youtube.com/watch?v=DPO9Se6ZqN0)
- Link Parameters
  - ![Pasted image 20260408140950.png](/assets/images/Pasted-image-20260408140950.png)
- Joint parameters
  - ![스크린샷 2026-04-08 오후 2.35.30.png](/assets/images/스크린샷-2026-04-08-오후-2.35.30.png)
  - ![스크린샷 2026-04-08 오후 2.45.33.png](/assets/images/스크린샷-2026-04-08-오후-2.45.33.png)

### Manipulator Kinematics

다관절 매니퓰레이터를 위한 기구학 계산을 하는데 사용함.

![스크린샷 2026-04-08 오후 2.49.56.png](/assets/images/스크린샷-2026-04-08-오후-2.49.56.png)

- 상대변환 (반대면 inverse kinematics)
- ![Pasted image 20260408150425.png](/assets/images/Pasted-image-20260408150425.png)
- ![스크린샷 2026-04-08 오후 3.35.12.png](/assets/images/스크린샷-2026-04-08-오후-3.35.12.png)
- \*\*자주사용함
  $$
  ^{i-1}_{i}T =
  \underbrace{\begin{bmatrix} 1 & 0 & 0 & 0 \\ 0 & c\alpha_{i-1} & -s\alpha_{i-1} & 0 \\ 0 & s\alpha_{i-1} & c\alpha_{i-1} & 0 \\ 0 & 0 & 0 & 1
  \end{bmatrix}}_{^{i-1}_R T}
  \underbrace{\begin{bmatrix} 1 & 0 & 0 & a_{i-1} \\ 0 & 1 & 0 & 0 \\ 0 & 0 & 1 & 0 \\ 0 & 0 & 0 & 1
  \end{bmatrix}}_{^R_Q T}
  \underbrace{\begin{bmatrix} c\theta_{i} & -s\theta_{i} & 0 & 0 \\ s\theta_{i} & c\theta_{i} & 0 & 0 \\ 0 & 0 & 1 & 0 \\ 0 & 0 & 0 & 1
  \end{bmatrix}}_{^Q_P T}
  \underbrace{\begin{bmatrix} 1 & 0 & 0 & 0 \\ 0 & 1 & 0 & 0 \\ 0 & 0 & 1 & d_{i} \\ 0 & 0 & 0 & 1
  \end{bmatrix}}_{^P_i T}
  $$
  [[회전행렬 (축기준)]]

D-H parameter 일반식

$$= \begin{bmatrix} c\theta_{i} & -s\theta_{i} & 0 & a_{i-1} \\ s\theta_{i}c\alpha_{i-1} & c\theta_{i}c\alpha_{i-1} & -s\alpha_{i-1} & -s\alpha_{i-1}d_{i} \\ s\theta_{i}s\alpha_{i-1} & c\theta_{i}s\alpha_{i-1} & c\alpha_{i-1} & c\alpha_{i-1}d_{i} \\ 0 & 0 & 0 & 1 \end{bmatrix}$$
#tip
2행 4열은 2행 3열에 d를 곱한것임.
3행 4열은 3행 3열에 d를 곱한것임.

#### **동차 변환 행렬의 구조**

$${}^{i-1}_{i}T = \left[ \begin{array}{ccc|c} c\theta_{i} & -s\theta_{i} & 0 & a_{i-1} \\ s\theta_{i}c\alpha_{i-1} & c\theta_{i}c\alpha_{i-1} & -s\alpha_{i-1} & -s\alpha_{i-1}d_{i} \\ s\theta_{i}s\alpha_{i-1} & c\theta_{i}s\alpha_{i-1} & c\alpha_{i-1} & c\alpha_{i-1}d_{i} \\ \hline 0 & 0 & 0 & 1 \end{array} \right] = \begin{bmatrix} {}^{i-1}_{i}R & {}^{i-1}p_{iORG} \\ \underline{0} & 1 \end{bmatrix}$$

- 회전 행렬 (Rotation Matrix)
  - 왼쪽 상단의 $3 \times 3$ 영역으로, 프레임 $i$의 프레임 $i-1$에 대한 회전을 나타냅니다.

$${}^{i-1}_{i}R = \begin{bmatrix} c\theta_{i} & -s\theta_{i} & 0 \\ s\theta_{i}c\alpha_{i-1} & c\theta_{i}c\alpha_{i-1} & -s\alpha_{i-1} \\ s\theta_{i}s\alpha_{i-1} & c\theta_{i}s\alpha_{i-1} & c\alpha_{i-1} \end{bmatrix}$$

- 위치 벡터 (Position Vector) - 오른쪽 상단의 $3 \times 1$ 영역으로, 프레임 $i$의 원점이 프레임 $i-1$에서 어디에 위치하는지 나타냅니다.
  $${}^{i-1}p_{iORG} = \begin{bmatrix} a_{i-1} \\ -s\alpha_{i-1}d_{i} \\ c\alpha_{i-1}d_{i} \end{bmatrix}$$

---

예제
![Pasted image 20260408170316.png](/assets/images/Pasted-image-20260408170316.png)

![Pasted image 20260408170358.png](/assets/images/Pasted-image-20260408170358.png)

- DH convention - 링크를 x축으로 봐야함. #Realize
- 링크방향 x축, 회전 중심축: z축 #Realize
- d는 뒤틀림(displacement)이 존재할때 생김(구체적으로 언제 이런게 생기는지 알아봐야할듯) 여기서는 뒤틀림이 안생긴다 #Realize

![스크린샷 2026-04-08 오후 5.34.37.png](/assets/images/스크린샷-2026-04-08-오후-5.34.37.png)

![스크린샷 2026-04-08 오후 5.48.27.png](/assets/images/스크린샷-2026-04-08-오후-5.48.27.png)

![스크린샷 2026-04-08 오후 6.34.08.png](/assets/images/스크린샷-2026-04-08-오후-6.34.08.png)
https://gemini.google.com/share/b0cfda8a9f0f
#Realize $l_1$ 이 뒤에 붙는 이유

![스크린샷 2026-04-08 오후 6.35.21.png](/assets/images/스크린샷-2026-04-08-오후-6.35.21.png)
축약형

![Pasted image 20260408190217.png](/assets/images/Pasted-image-20260408190217.png)

![Pasted image 20260408190339.png](/assets/images/Pasted-image-20260408190339.png)
EE 의 x,y 기하학적 의미 직각삼각형 2개로 이해할수 있다. (단순한경우만임.)
