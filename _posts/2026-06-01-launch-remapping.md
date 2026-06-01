---
layout: post
title: "launch remapping"
date: 2026-06-01
categories: ROS2
tags:
---

# Launch - Remapping

ROS 2로 개발하다 보면 시스템의 규모가 커지고 노드 간의 통신이 복잡해지기 쉽습니다. 초기에는 각 노드에 토픽 이름을 하드코딩하며 개발을 진행할 수 있지만, 프로젝트가 커지고 서드파티 패키지를 통합하는 단계에 이르면 복잡도가 증가합니다.

ROS 2 Launch 시스템의 Remapping 기능은 이러한 문제를 해결하기 위해 고안된 기능입니다.

## 하드코딩된 토픽

먼저 아래의 간단한 Publisher 노드 코드를 살펴보겠습니다.

```python
import rclpy as rp
from rclpy.node import Node
from geometry_msgs.msg import Twist

class CmdVelPuslisher(Node):
    def __init__(self):
        super().__init__('cmd_vel_publisher_node')
        # 토픽 이름이 '/cmd_vel'로 하드코딩 되어 있음
        self.publisher = self.create_publisher(Twist, '/cmd_vel', 10)
        self.timer = self.create_timer(1.0, self.timer_callback)

    def timer_callback(self):
        msg = Twist()
        msg.linear.x = 1.0
        msg.angular.z = 0.5
        self.publisher.publish(msg)
        self.get_logger().info('publish velocity command to /cmd_vel')

def main(args=None):
    rp.init()
    cmd_vel_publisher = CmdVelPuslisher()
    rp.spin(cmd_vel_publisher)
    cmd_vel_publisher.destroy_node()
    rp.shutdown()

if __name__ == '__main__':
    main()

```

위 코드는 잘 작동하지만, 단점이 있습니다. 토픽 이름이 `/cmd_vel`로 고정되어 있다는 것입니다.

만약 이 노드를 이용해 ROS 2의 대표적인 예제인 `turtlesim`의 거북이를 움직이고 싶다면 어떻게 될까요? `turtlesim` 노드는 `/cmd_vel`이 아니라 `/turtle1/cmd_vel` 토픽을 구독(Subscribe)하여 움직입니다. 통신이 엇갈리게 되는 것이죠.

이 문제를 해결하기 위해 노드의 소스 코드를 열어 `self.create_publisher(Twist, '/turtle1/cmd_vel', 10)`로 수정해야 할까요? 만약 거북이가 2마리, 10마리 라면요? 그때마다 코드를 복사해서 새로 빌드하는 것은 소프트웨어 공학적으로 비효율적입니다.

## Remapping이란?

ROS 2 아키텍처에서 노드 내부의 비즈니스 로직과 외부 통신 네트워크(Graph)의 구조는 분리되어야 합니다. Remapping은 이 '디커플링(Decoupling)'을 가능하게 합니다.

### 노드의 재사용성

노드를 개발할 때는 해당 노드가 '어떤 특정 로봇'이나 '특정 네임스페이스'에서 실행될지 고민할 필요가 없습니다. 그저 표준화된 이름(예: `cmd_vel`, `scan`, `odom`)을 사용하여 로직을 구현하면 됩니다. 실제 환경에 맞게 토픽 이름을 연결해 주는 것은 실행 시점(Launch)에 Remapping이 담당하므로, 하나의 노드를 여러 프로젝트나 다양한 로봇에서 그대로 재사용할 수 있습니다.

### 소스 코드 수정 없는 유연한 네트워크 재구성

다중 로봇 시스템이나 복잡한 센서 퓨전 환경에서는 동일한 노드를 여러 개 띄워야 하는 경우가 많습니다. Remapping을 사용하면 소스 코드(Python이나 C++)를 단 한 줄도 건드리지 않고, Launch 파일의 설정만으로 데이터의 흐름을 제어할 수 있습니다. 이는 시스템의 유지보수성을 크게 향상시킵니다.

## 3. Launch 파일을 통한 Remapping 해결책

앞서 제기된 `turtlesim` 문제를 Launch 파일의 Remapping을 통해 어떻게 해결하는지 보겠습니다.

```python
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    cmd_vel_publisher_node = Node(
        package='my_first_package',
        executable='cmd_vel_publisher',
        output='screen',
        # Remapping 규칙 적용: 노드의 '/cmd_vel'을 '/turtle1/cmd_vel'로 변경
        remappings=[
            ('/cmd_vel', '/turtle1/cmd_vel')
        ]
    )

    return LaunchDescription([
        cmd_vel_publisher_node
    ])

```

`Node` 액션을 정의할 때 `remappings` 인자를 리스트 형태로 전달합니다.
규칙은 간단합니다: `('기존 토픽 이름', '새로운 토픽 이름')`

위 Launch 파일을 실행하면, ROS 2 미들웨어는 `cmd_vel_publisher` 노드가 `/cmd_vel`로 퍼블리시하려는 데이터를 가로채어 실제로는 `/turtle1/cmd_vel`로 흐르게 만듭니다. 노드 자신은 여전히 `/cmd_vel`로 데이터를 보내고 있다고 생각하지만, 시스템 전체의 통신망에서는 `turtlesim`을 향해 데이터가 전달됩니다.
