---
layout: post
title: "launch LaunchConfiguration"
date: 2026-06-01
categories: ROS2
tags:
  - ROS2
  - Launch
  - LaunchConfiguration
  - robotics
---

# LaunchConfiguration

동일한 웹 서버 이미지를 개발 서버와 운영 서버에 배포할 때, 매번 이미지를 새로 빌드하지 않고 `docker run -e NODE_ENV=production` 또는 `docker-compose.yml`을 통해 실행 시점(Runtime)에 환경 변수를 주입합니다. ROS 2에서도 노드를 실행할 때마다 코드를 수정하는 대신, 런치 파일을 유연하게 설계하여 실행 시점(Runtime)에 값을 주입해줄수가 있습니다. `LaunchConfiguration`은 바로 이 역할을 수행합니다.

## LaunchConfiguration 원리

Python 스크립트가 파싱되는 시점에 값이 결정되는 것이 아니라, 사용자가 터미널에서 `ros2 launch ... key:=value` 형태로 값을 입력하면 **실행 시점(LaunchContext)에 비로소 값을 치환**해 줍니다. 이를 통해 하나의 런치 파일이 다양한 환경에 대응할 수 있는 유연성을 갖게 됩니다.

## 기본 활용 패턴

### Namespace 분리를 통한 Multi-Robot 제어

동일한 노드(예: `turtlesim`)를 여러 개 띄울 때 토픽 충돌을 막기 위해 네임스페이스를 동적으로 주입합니다.

```python
namespace = LaunchConfiguration('namespace')
DeclareLaunchArgument('namespace', default_value='robot1', description='Robot namespace')

Node(
    package='turtlesim',
    executable='turtlesim_node',
    namespace=namespace, # 터미널 입력값에 따라 /robot1, /robot2 등으로 동적 할당
)

```

### 리소스 최적화를 위한 조건부 실행 (IfCondition)

테스트 환경에서는 디버깅용 카메라 노드를 켜고, 실제 운영 환경에서는 리소스 절약을 위해 끄는 등의 분기 처리가 가능합니다.

```python
use_camera = LaunchConfiguration('use_camera')
DeclareLaunchArgument('use_camera', default_value='true')

Node(
    package='my_package',
    executable='camera_node',
    condition=IfCondition(use_camera) # use_camera:=false 입력 시 노드 실행 생략
)

```

## 활용

### 시뮬레이션(Gazebo)과 실제 하드웨어 전환 (`use_sim_time`)

로봇을 시뮬레이터에서 돌릴 때는 가상 시간을, 실제 로봇에서는 시스템 시간을 써야 합니다. 이를 `LaunchConfiguration`으로 제어하여 하나의 런치 파일로 두 환경을 모두 지원합니다.

```python
use_sim_time = LaunchConfiguration('use_sim_time')

DeclareLaunchArgument(
    'use_sim_time',
    default_value='false',
    description='Use simulation (Gazebo) clock if true'
)

Node(
    package='robot_state_publisher',
    executable='robot_state_publisher',
    # 파라미터 딕셔너리 내부에 LaunchConfiguration 객체를 그대로 전달
    parameters=[{'use_sim_time': use_sim_time}]
)

```

### 동적 파라미터(YAML) 파일 경로 주입

로봇의 기구학(Kinematics) 설정값이나 모터 튜닝 파라미터 등은 보통 외부 `yaml` 파일로 관리합니다. 로봇의 모델명이나 설정 버전에 따라 로드해야 할 파일이 다르므로, 파일 경로 자체를 동적으로 만듭니다.

```python
import os
from ament_index_python.packages import get_package_share_directory

# 패키지 기본 경로
pkg_dir = get_package_share_directory('my_robot_bringup')

# 환경에 따라 다른 파라미터 파일을 로드하도록 설정
param_file_name = LaunchConfiguration('param_file')
DeclareLaunchArgument('param_file', default_value='default_bot.yaml')

Node(
    package='my_controller_pkg',
    executable='controller_node',
    # os.path.join 대신 리스트로 묶어주면 런타임에 경로를 안전하게 조합해줌
    parameters=[[pkg_dir, '/config/', param_file_name]]
)

```
