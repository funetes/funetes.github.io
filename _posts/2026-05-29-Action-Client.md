---
layout: post
title: "Action Client"
date: 2026-05-29
categories: ROS2
tags:
  - ROS2
  - Action
  - Client
  - robotics
---

# Action Client

Action은 **서비스(Service)**와 **토픽(Topic)**의 중간 형태로, **시간이 걸리는 작업을 실행하고 그 진행 상황을 계속 주고받으며, 최종 결과를 돌려받는 방식**입니다.

## 1. Action 구조

Action은 크게 3가지 요소로 구성됩니다.

| 요소                  | 역할                                                                                                                                            |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| **Goal (목표)**       | 클라이언트가 서버에 보내는 **요구사항**. (예: “10m 앞으로 가줘.”)<br>이 요청이 서버에 도착해야 작업이 시작됩니다.                               |
| **Feedback (피드백)** | 서버가 클라이언트에게 **실시간으로 보내는 진행 상황**. (예: “지금 3m 갔어.”)<br>클라이언트는 이를 통해 목표 달성 중임을 모니터링할 수 있습니다. |
| **Result (결과)**     | 서버가 작업을 완료한 후 클라이언트에게 돌려주는 **최종 결과값**. (예: “10m 주행 완료. 로봇 정지.”)                                              |

## 2. Action 실행 과정

1. **Action Client 생성**: 서버의 액션 이름을 지정하여 클라이언트를 생성합니다.<br>
2. **Goal 전송**: 클라이언트가 목표를 서버로 전송하고 응답을 기다립니다.<br>
3. **수락/거절 확인**: 서버가 목표를 받아들일지(Accepted) 거절할지(Rejected) 클라이언트에게 응답합니다.<br>
   - **수락**되었다면, 클라이언트는 **결과를 받을 준비**를 합니다. 이때부터 **피드백**을 받을 수도 있습니다.<br>
4. **결과 수신**: 서버가 작업을 끝내면, 클라이언트는 최종 결과를 받습니다.<br>
5. **Action Client 종료**: 서버와의 연결을 끊습니다.

## 3. Action Client 코드 예시

아래 코드는 `DistTurtle` 액션을 생성하고, 실행 후 약 2초 뒤에 목표를 취소(Cancel)하는 예제입니다.

```python

import rclpy as rp
from rclpy.action import ActionClient
from rclpy.node import Node

from my_first_package_msgs.action import DistTurtle

class DistTurtleCancelClient(Node):
    def __init__(self):
        super().__init__('dist_turtle_cancel_client')

        # action client 설정
        self._action_client = ActionClient(
            self,
            DistTurtle,
            'dist_turtle'
        )

        self.goal_handle = None
        self.timer = None

    def send_goal(self):
        goal_msg = DistTurtle.Goal()
        goal_msg.linear_x = 1.0
        goal_msg.angular_z = 0.0
        goal_msg.dist = 10.0

        # wait for server
        self._action_client.wait_for_server()

        self.get_logger().info('sending goal')

        # send goal request
        self._send_goal_future = self._action_client.send_goal_async(
            goal=goal_msg,
            feedback_callback=self.feedback_callback # feedback callback
        )

        # 비동기 요청을 하면 수락 여부를 받는 함수를 콜백으로 받는다.
        self._send_goal_future.add_done_callback(
            self.goal_response_callback
        )

    def feedback_callback(self, feedback_msg):
        feedback = feedback_msg.feedback

        self.get_logger().info(
            f'Received feedback: remained_dist = {feedback.remained_dist}'
        )

    def goal_response_callback(self, future):
      # 여기서 받는 것은 '작업 수락 여부'를 담은 핸들입니다.
      # 서버로부터 Goal ID를 포함한 핸들을 반환받으며, 이 시점부터 해당 목표를 취소(`cancel_goal_async`)할 수 있는 권한이 생김."
        self.goal_handle = future.result()

        # 수락 여부 판단
        if not self.goal_handle.accepted:
            self.get_logger().info('Goal rejected')
            return

        self.get_logger().info('Goal accepted')

        # 목표가 수락되었으므로, 실제 작업이 끝날 때까지 기다리는 '최종 결과용 Future'를 생성함.
        self._get_result_future = self.goal_handle.get_result_async()
        # 호출 결과를 콜백으로 받음
        self._get_result_future.add_done_callback(
            self.get_result_callback
        )

        # 2초 후에 취소 요청을 보내기 위한 타이머 설정
        self.timer = self.create_timer(2.0, self.cancel_goal)

    def get_result_callback(self, future):
        # 서버의 execute_callback이 리턴한 최종 객체를 받음.
        result = future.result().result
        status = future.result().status

        self.get_logger().info(f'Action finished with status: {status}')
        self.get_logger().info(f'Result distance: {result.result_dist}')

        rp.shutdown()

    def cancel_goal(self):
        self.get_logger().info('sending cancel request')

        self.timer.cancel()

        # 취소 요청
        self._cancel_future = self.goal_handle.cancel_goal_async()
        # 취소 요청에 대한 응답 콜백
        self._cancel_future.add_done_callback(
            self.cancel_done_callback
        )

    def cancel_done_callback(self, future):
        # 취소 요청 자체에 대한 서버의 응답(CancelResponse)을 확인하며, 실제 취소가 성공했는지는 `get_result_callback`에서 `status` 값을 통해 최종 확인해야 함.
        cancel_response = future.result()

        if len(cancel_response.goals_canceling) > 0:
            self.get_logger().info('Cancel request accepted')
        else:
            self.get_logger().info('Cancel request rejected')

def main(args=None):
    rp.init(args=args)
    dist_turtle_action_client = DistTurtleCancelClient()
    dist_turtle_action_client.send_goal()

    rp.spin(dist_turtle_action_client)

if __name__ == "__main__":
    main()



```

- **`send_goal_async`**: 서버에 목표를 던지고 "수락/거절" 응답을 기다리는 Future를 만듭니다.
- **`goal_response_callback`**: 서버가 "알았어, 해볼게"라고 하면 `goal_handle`을 저장하고 바로 결과(Result)를 비동기로 요청해 둡니다.
- **`create_timer`**: 2초 뒤에 무조건 취소 요청을 보내도록 예약합니다.
- **`cancel_goal`**: 2초가 지나면 실행되며, 저장해둔 `goal_handle`을 이용해 서버에 "하던 거 멈춰!"라고 요청합니다.
- **`cancel_done_callback`**: 서버가 "취소 요청 확인했어"라고 답합니다.
- **`get_result_callback`**: (취소 요청에 의해) 작업이 중단되면 서버는 최종 결과를 반환하며 종료됩니다. 이때 `status` 값은 `STATUS_CANCELED`가 됩니다.

---

## Future(퓨처)란?

`self._send_goal_future`에서 등장하는 **Future**는 "지금은 없지만, 미래에 채워질 데이터의 상자"라고 생각하면 쉽습니다.

- **비동기 처리:** `send_goal_async` 함수는 서버로부터 응답이 올 때까지 프로그램을 멈추게(Blocking) 하지 않습니다. 대신 "응답이 오면 여기 담아줄게"라며 `Future` 객체를 즉시 반환하고 다음 코드로 넘어갑니다.
- **상태 추적:** 이 상자가 채워졌는지(Done), 아니면 아직 기다리는 중인지 상태를 추적할 수 있게 해줍니다.
