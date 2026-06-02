---
layout: post
title: "action with cpp"
date: 2026-06-02
categories: ROS2
tags:
  - action
  - c++
---

# Action with C++

## 1. Action Interface 정의 및 빌드 설정

클라이언트와 서버가 주고받을 메시지 규격을 정의해야 합니다. Action은 `Goal`, `Result`, `Feedback` 세 가지 영역으로 나뉩니다.

**`move_distance.action`**

```yaml
# Request (Goal)
float32 target_distance
---
# Result
bool success
string message
---
# Feedback
float32 current_distance
float32 progress
```

`colcon build --packages-select <package name>` 을 통해 빌드를 수행하여 C++ 헤더 파일을 생성합니다.

---

## 2. Action Server

액션 서버는 클라이언트의 요청을 받고, 거절/수락 여부를 판단하며, 실제 작업을 수행합니다. 서버 구현에서 가장 핵심적인 부분은 **작업 수행 시 메인 스레드를 블로킹하지 않도록**하는 것입니다.

### 서버의 3가지 핵심 콜백

`rclcpp_action::create_server`를 통해 서버를 생성할 때, 다음 세 가지 상태를 처리할 콜백 함수를 바인딩합니다.

1. **`handle_goal`**: 목표 수락 여부 결정
2. **`handle_cancel`**: 취소 요청 처리
3. **`handle_accepted`**: 실제 비즈니스 로직 실행

### 서버 구현 코드 살펴보기 (`server.cpp`)

```cpp
#include <chrono>
#include <memory>
#include <thread>
#include "rclcpp/rclcpp.hpp"
#include "rclcpp_action/rclcpp_action.hpp"
#include "my_first_package_msgs/action/move_distance.hpp"

using namespace std::chrono_literals;

class MoveDistanceServer : public rclcpp::Node {
    public:
        using MoveDistance = my_first_package_msgs::action::MoveDistance;
        using GoalHandleMoveDistance = rclcpp_action::ServerGoalHandle<MoveDistance>;

        MoveDistanceServer() : Node("move_distance_server") {
            action_server_ = rclcpp_action::create_server<MoveDistance>(
                this, "move_distance",
                std::bind(&MoveDistanceServer::handle_goal, this, std::placeholders::_1, std::placeholders::_2),
                std::bind(&MoveDistanceServer::handle_cancel, this, std::placeholders::_1),
                std::bind(&MoveDistanceServer::handle_accepted, this, std::placeholders::_1)
            );
            RCLCPP_INFO(this->get_logger(), "Move Distance Action server started.");
        }

    private:
        rclcpp_action::Server<MoveDistance>::SharedPtr action_server_;

        rclcpp_action::GoalResponse handle_goal(
            const rclcpp_action::GoalUUID &uuid, std::shared_ptr<const MoveDistance::Goal> goal) {
            (void)uuid;
            if (goal->target_distance <= 0.0) {
                RCLCPP_WARN(this->get_logger(), "Rejected: Distance must be greater than 0.");
                return rclcpp_action::GoalResponse::REJECT;
            }
            return rclcpp_action::GoalResponse::ACCEPT_AND_EXECUTE;
        }

        rclcpp_action::CancelResponse handle_cancel(
            const std::shared_ptr<GoalHandleMoveDistance> goal_handle) {
            (void)goal_handle;
            RCLCPP_INFO(this->get_logger(), "Goal canceled requested");
            return rclcpp_action::CancelResponse::ACCEPT;
        }

        void handle_accepted(const std::shared_ptr<GoalHandleMoveDistance> goal_handle) {
            // 핵심 로직: 별도의 스레드로 분리하여 메인 스레드 블로킹 방지
            std::thread{
                std::bind(&MoveDistanceServer::execute, this, std::placeholders::_1),
                goal_handle
            }.detach();
        }

        void execute(const std::shared_ptr<GoalHandleMoveDistance> goal_handle) {
            const auto goal = goal_handle->get_goal();
            auto feedback = std::make_shared<MoveDistance::Feedback>();
            auto result = std::make_shared<MoveDistance::Result>();
            rclcpp::Rate loop_rate(2);

            float current_distance = 0.0;
            const float step_distance = 0.1;

            while (current_distance < goal->target_distance) {
                // 실행 중 지속적인 취소 요청 체크
                if(goal_handle->is_canceling()) {
                    result->success = false;
                    result->message = "Goal canceled.";
                    goal_handle->canceled(result);
                    return;
                }

                current_distance = std::min(current_distance + step_distance, goal->target_distance);
                feedback->current_distance = current_distance;
                feedback->progress = (current_distance / goal->target_distance) * 100.0;

                goal_handle->publish_feedback(feedback);
                loop_rate.sleep();
            }

            if(rclcpp::ok()) {
                result->success = true;
                result->message = "Target distance reached.";
                goal_handle->succeed(result);
            }
        }
};

int main(int argc, char* argv[]) {
    rclcpp::init(argc, argv);
    rclcpp::spin(std::make_shared<MoveDistanceServer>());
    rclcpp::shutdown();
    return 0;
}

```

**`std::thread`**

`handle_accepted`에서 `execute` 함수를 `std::thread`로 실행하고 `.detach()`하는 이유는 메인 콜백에서 무거운 루프를 돌릴 경우 클라이언트의 취소 요청(`handle_cancel`)이나 다른 토픽의 메시지를 수신할 수 없게 되기 때문입니다. 비동기 환경에서 자원의 독립적인 제어권을 잃지 않기 위해 실행 컨텍스트를 분리하는 것입니다.

---

## 3. Action Client: 비동기 흐름 제어

클라이언트는 `async_send_goal`을 호출하여 비동기적으로 서버에 작업을 던지고 자기 할 일을 계속합니다. 결과를 기다리느라 프로세스가 멈추지 않으며, 서버의 응답은 미리 등록해 둔 세 가지 콜백을 통해 처리됩니다.

### 클라이언트 구현 코드 살펴보기 (`client.cpp`)

```cpp
#include <chrono>
#include <memory>
#include "rclcpp/rclcpp.hpp"
#include "rclcpp_action/rclcpp_action.hpp"
#include "my_first_package_msgs/action/move_distance.hpp"

using namespace std::chrono_literals;

class MoveDistanceClient : public rclcpp::Node {
    public:
        using MoveDistance = my_first_package_msgs::action::MoveDistance;
        using GoalHandleMoveDistance = rclcpp_action::ClientGoalHandle<MoveDistance>;

        MoveDistanceClient(): Node("move_distance_client") {
            action_client_ = rclcpp_action::create_client<MoveDistance>(this, "move_distance");
        }

        void send_goal(float target_distance) {
            // 서버 연결 대기
            if(!action_client_->wait_for_action_server(5s)) {
                RCLCPP_ERROR(this->get_logger(), "Action server not available.");
                return;
            }

            auto goal_msg = MoveDistance::Goal();
            goal_msg.target_distance = target_distance;

            auto send_goal_options = rclcpp_action::Client<MoveDistance>::SendGoalOptions();
            send_goal_options.goal_response_callback = std::bind(&MoveDistanceClient::goal_response_callback, this, std::placeholders::_1);
            send_goal_options.feedback_callback = std::bind(&MoveDistanceClient::feedback_callback, this, std::placeholders::_1, std::placeholders::_2);
            send_goal_options.result_callback = std::bind(&MoveDistanceClient::result_callback, this, std::placeholders::_1);

            // 비동기 목표 전송
            action_client_->async_send_goal(goal_msg, send_goal_options);
        }

    private:
        rclcpp_action::Client<MoveDistance>::SharedPtr action_client_;

        // 1. Goal Response: 서버가 내 요청을 받았는가?
        void goal_response_callback(const GoalHandleMoveDistance::SharedPtr &goal_handle) {
            if (!goal_handle) RCLCPP_ERROR(this->get_logger(), "Goal was rejected.");
            else RCLCPP_INFO(this->get_logger(), "Goal accepted.");
        }

        // 2. Feedback: 진행 상황 수신
        void feedback_callback(
            GoalHandleMoveDistance::SharedPtr, const std::shared_ptr<const MoveDistance::Feedback> feedback) {
            RCLCPP_INFO(this->get_logger(), "Feedback: %.2f m, %.1f %%", feedback->current_distance, feedback->progress);
        }

        // 3. Result: 최종 결과 확인
        void result_callback(const GoalHandleMoveDistance::WrappedResult &result) {
            switch (result.code) {
                case rclcpp_action::ResultCode::SUCCEEDED:
                    RCLCPP_INFO(this->get_logger(), "Result: success"); break;
                case rclcpp_action::ResultCode::ABORTED:
                    RCLCPP_ERROR(this->get_logger(), "Result: aborted"); break;
                case rclcpp_action::ResultCode::CANCELED:
                    RCLCPP_WARN(this->get_logger(), "Result: canceled"); break;
                default:
                    RCLCPP_ERROR(this->get_logger(), "Unknown result code"); break;
            }
            rclcpp::shutdown();
        }
};

int main(int argc, char* argv[]) {
    rclcpp::init(argc, argv);
    float target_distance = (argc >= 2) ? std::atof(argv[1]) : 1.0;

    auto action_client = std::make_shared<MoveDistanceClient>();
    action_client->send_goal(target_distance);

    rclcpp::spin(action_client);
    return 0;
}

```

클라이언트 노드가 실행되는 시점에 서버 노드가 완전히 준비되지 않았을 수 있습니다. 분산 시스템 환경에서`wait_for_action_server`를 통한 동기화 대기는 선택이 아닌 필수입니다.

---

ROS 2 Action의 생명주기는 크게 `Goal 요청` ➔ `Feedback 수신` ➔ `Result 확인`의 비동기 흐름으로 이어집니다.

- **서버 사이드**에서는 실행 로직을 별도의 스레드로 빼내어 이벤트 루프(spin)가 멈추지 않도록 하고, 언제든 클라이언트의 Cancellation 요청에 대응할 수 있도록 구조를 짜는 것이 중요합니다.
- **클라이언트 사이드**에서는 `async_send_goal`을 통해 멈춤 없는 흐름을 만들고, 3가지 콜백(`Goal Response`, `Feedback`, `Result`)을 통해 상태 변화에 대응합니다.
