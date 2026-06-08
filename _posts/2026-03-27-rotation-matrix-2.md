---
layout: post
title: "Mapping"
date: 2026-03-27 12:00:00 +0900
---

# 3D 공간에서의 매핑(Mapping)과 회전 행렬

## 매핑(Mapping)

공학에서의 **매핑(Mapping)**은 한 좌표계에서 표현된 위치나 벡터를 다른 좌표계에서의 표현으로 바꾸는 것을 말합니다.

방 안에 가만히 놓여있는 사과(점 P)가 있다고 가정하면,

Frame A (방 밖에 있는 관찰자): "사과는 내 기준에서 앞으로 5m, 오른쪽으로 3m에 있어."

Frame B (방 안에서 돌아가고 있는 로봇): "사과는 내 기준에서 바로 정면에 있어."

사과의 실제 위치는 우주 공간에서 변함이 없지만, 누가(어떤 좌표계가) 관찰하느냐에 따라 위치를 표현하는 값(좌표)은 달라집니다. 이렇게 '로봇의 시선(Frame B)'에서 본 좌표를 '관찰자의 시선(Frame A)'에서 본 좌표로 번역해 주는 과정이 바로 매핑(좌표계 변환)입니다.

## 3D 공간의 회전 변환 행렬 (Rotation Matrices)

3차원 공간에서 좌표계를 회전시킬 때는 3x3 크기의 **회전 행렬(Rotation Matrix)**을 곱하여 매핑을 수행합니다. 기준이 되는 회전 축(X, Y, Z)에 따라 행렬의 형태가 조금씩 달라집니다.

① X축 기준 회전 ($R_x$)

X축을 꼬치처럼 꽂아두고, 그 축을 중심으로 $\theta$만큼 회전하는 경우입니다. X축의 좌표는 변하지 않으므로 행렬의 첫 번째 행과 열은 각각 $(1, 0, 0)$의 형태를 띱니다.

$$R_x(\theta) = \begin{bmatrix} 1 & 0 & 0 \\ 0 & \cos(\theta) & -\sin(\theta) \\ 0 & \sin(\theta) & \cos(\theta) \end{bmatrix}$$

② Y축 기준 회전 ($R_y$)

Y축을 중심으로 $\theta$만큼 회전하는 행렬입니다. 다른 축과 달리 $\sin(\theta)$의 부호 위치가 반대입니다.

$$R_y(\theta) = \begin{bmatrix} \cos(\theta) & 0 & \sin(\theta) \\ 0 & 1 & 0 \\ -\sin(\theta) & 0 & \cos(\theta) \end{bmatrix}$$

③ Z축 기준 회전 ($R_z$)

Z축을 중심으로 $\theta$만큼 회전하는 행렬입니다. 2차원 평면에서의 회전 변환 행렬과 가장 유사한 형태를 가지고 있어 이해하기 쉽습니다.

$$R_z(\theta) = \begin{bmatrix} \cos(\theta) & -\sin(\theta) & 0 \\ \sin(\theta) & \cos(\theta) & 0 \\ 0 & 0 & 1 \end{bmatrix}$$

## 시각화

위 행렬들이 어떻게 나왔는지 이해하기위한 시각화입니다.

<iframe src="/lab/rotation.html" style="width:100%; height:800px; border:none; margin-top:20px;"></iframe>
