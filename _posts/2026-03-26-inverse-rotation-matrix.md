---
layout: post
title: "Inverse Rotation Matrix"
date: 2026-03-26 12:00:00 +0900
categories: Kinematics
---

## Inverse Rotation Matrix

회전 행렬 ${}^B_A R$은 프레임 A의 주축 단위 벡터들($\hat{X}_A, \hat{Y}_A, \hat{Z}_A$)을 프레임 B의 성분으로 투영하여 표현한 것입니다.

$$
{}^B_A R = \begin{bmatrix} {}^B \hat{X}_A & {}^B \hat{Y}_A & {}^B \hat{Z}_A \end{bmatrix}
$$

각 열 벡터를 프레임 B의 기본 벡터들과의 내적으로 풀어서 쓰면 다음과 같습니다.

$$
= \begin{bmatrix}
\hat{X}_A \cdot \hat{X}_B & \hat{Y}_A \cdot \hat{X}_B & \hat{Z}_A \cdot \hat{X}_B \\
\hat{X}_A \cdot \hat{Y}_B & \hat{Y}_A \cdot \hat{Y}_B & \hat{Z}_A \cdot \hat{Y}_B \\
\hat{X}_A \cdot \hat{Z}_B & \hat{Y}_A \cdot \hat{Z}_B & \hat{Z}_A \cdot \hat{Z}_B
\end{bmatrix}
$$

### 내적의 교환 법칙을 이용한 변환

내적의 교환 법칙($\vec{a} \cdot \vec{b} = \vec{b} \cdot \vec{a}$)을 적용하여 순서를 바꾸면, 프레임 A를 기준으로 프레임 B의 벡터들을 투영한 형태가 됩니다.

$$
= \begin{bmatrix}
\hat{X}_B \cdot \hat{X}_A & \hat{X}_B \cdot \hat{Y}_A & \hat{X}_B \cdot \hat{Z}_A \\
\hat{Y}_B \cdot \hat{X}_A & \hat{Y}_B \cdot \hat{Y}_A & \hat{Y}_B \cdot \hat{Z}_A \\
\hat{Z}_B \cdot \hat{X}_A & \hat{Z}_B \cdot \hat{Y}_A & \hat{Z}_B \cdot \hat{Z}_A
\end{bmatrix}
$$

### 전치 행렬(Transpose)과의 관계

위 행렬은 프레임 A에 대한 프레임 B의 회전 행렬인 ${}^A_B R$의 행과 열을 바꾼 것(전치)과 같습니다.

$$
= \begin{bmatrix}
{}^A \hat{X}_B^T \\
{}^A \hat{Y}_B^T \\
{}^A \hat{Z}_B^T
\end{bmatrix} = {}^A_B R^T
$$

회전 행렬은 **직교 행렬(Orthogonal Matrix)**의 성질을 가지므로, 역행렬은 전치 행렬과 같습니다.

$${}^B_A R = {}^A_B R^T = {}^A_B R^{-1}$$
