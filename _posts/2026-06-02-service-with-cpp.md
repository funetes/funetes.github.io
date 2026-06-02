---
layout: post
title: "service with cpp"
date: 2026-06-02
categories: ROS2
tags:
  - C++
  - Service
---

ROS2에서 Custom Interface(srv)를 정의하고, 이를 활용해 **C++로 Service Server와 Client**를 구현하는 과정을 블로그 포스팅 형식으로 정리해 보았습니다.

---

## Service with C++

ROS2에서 노드 간 동기적인 통신을 위해 사용하는 **Service** 모델을 직접 구현해 봅시다. 두 숫자와 연산자를 전달하면 계산 결과를 반환하는 간단한 'Math Service' 예제입니다.

---

### 1. Custom Interface 정의 (.srv)

가장 먼저 데이터의 형식을 정의해야 합니다. `my_first_package_msgs` 패키지 내의 `srv/` 디렉토리에 `CalculateTwonumbers.srv` 파일을 생성합니다.

```text
# Request
float64 x
float64 y
string arithmetic_operator
---
# Response
float64 result
bool success
string message

```

**중요:** 인터페이스 정의 후에는 반드시 빌드를 통해 헤더 파일(`.hpp`)을 생성해야 소스 코드에서 include 할 수 있습니다.

```bash
$ colcon build --packages-select my_first_package_msgs

```

---

### 2. CMakeLists.txt 설정

사용자 정의 인터페이스를 사용할 때는 `CMakeLists.txt`에 의존성을 명확히 기술해야 합니다.

- `find_package`에 인터페이스 패키지 추가
- `ament_target_dependencies`에 패키지 이름 등록

> **[!주의]** 인터페이스를 빌드하면 소스 파일에서 `#include "패키지명/srv/파일명.hpp"` 형태로 사용할 수 있게 됩니다.

---

### 3. Service Server 구현 (C++)

클라이언트의 요청을 받아 연산을 수행하고 응답을 보내는 서버 노드입니다.

```cpp
#include <memory>
#include <string>
#include <functional>

#include "rclcpp/rclcpp.hpp"
// 빌드 후 생성된 인터페이스 헤더 포함
#include "my_first_package_msgs/srv/calculate_twonumbers.hpp"

class MathServer : public rclcpp::Node {
public:
    using CalculateTwoNumbers = my_first_package_msgs::srv::CalculateTwonumbers;

    MathServer() : Node("math_server") {
        // 서비스 서버 생성: 서비스명 "Calculate_two_numbers"
        service_ = this->create_service<CalculateTwoNumbers>(
            "Calculate_two_numbers",
            std::bind(&MathServer::handle_request, this, std::placeholders::_1, std::placeholders::_2)
        );
        RCLCPP_INFO(this->get_logger(), "Math service server is ready.");
    };

private:
    void handle_request(
        const std::shared_ptr<CalculateTwoNumbers::Request> request,
        std::shared_ptr<CalculateTwoNumbers::Response> response)
    {
        const double x = request->x;
        const double y = request->y;
        const std::string op = request->arithmetic_operator;

        response->success = true;
        response->message = "Calculation completed.";

        // 사칙연산 로직
        if (op == "add") {
            response->result = x + y;
        } else if (op == "subtract") {
            response->result = x - y;
        } else if (op == "multiply") {
            response->result = x * y;
        } else if (op == "divide") {
            if (y == 0) {
                response->result = 0.0;
                response->success = false;
                response->message = "Division by zero is not allowed.";
            } else {
                response->result = x / y;
            }
        } else {
            response->result = 0.0;
            response->success = false;
            response->message = "Unknown operator";
        }

        RCLCPP_INFO(this->get_logger(),
            "Request: %.2f %s %.2f -> Result: %.2f, Success: %s",
            x, op.c_str(), y, response->result, response->success ? "true" : "false");
    }

    rclcpp::Service<CalculateTwoNumbers>::SharedPtr service_;
};

int main(int argc, char * argv[]) {
    rclcpp::init(argc, argv);
    auto node = std::make_shared<MathServer>();
    rclcpp::spin(node);
    rclcpp::shutdown();
    return 0;
}

```

---

### 4. Service Client 구현 (C++)

서버에 요청을 보내고 응답이 올 때까지 기다리는 클라이언트 노드입니다.

```cpp
#include <memory>
#include <string>
#include <chrono>

#include "rclcpp/rclcpp.hpp"
#include "my_first_package_msgs/srv/calculate_twonumbers.hpp"

using namespace std::chrono_literals;

class MathClient : public rclcpp::Node {
public:
    using CalculateTwonumbers = my_first_package_msgs::srv::CalculateTwonumbers;

    MathClient() : Node("math_client") {
        client_ = this->create_client<CalculateTwonumbers>("Calculate_two_numbers");
    };

    void send_request(double x, double y, const std::string & op) {
        // 서비스 서버가 활성화될 때까지 대기
        while (!client_->wait_for_service(1s)) {
            if (!rclcpp::ok()) {
                RCLCPP_ERROR(this->get_logger(), "Interrupted while waiting for service.");
                return;
            }
            RCLCPP_INFO(this->get_logger(), "Waiting for service to appear...");
        };

        auto request = std::make_shared<CalculateTwonumbers::Request>();
        request->x = x;
        request->y = y;
        request->arithmetic_operator = op;

        // 비동기 요청 전송
        auto future = client_->async_send_request(request);

        // 응답이 올 때까지 spin
        if (rclcpp::spin_until_future_complete(this->get_node_base_interface(), future) == rclcpp::FutureReturnCode::SUCCESS) {
            auto response = future.get();
            RCLCPP_INFO(this->get_logger(),
                "Result: %.2f, Success: %s, Message: %s",
                response->result, response->success ? "true" : "false", response->message.c_str());
        } else {
            RCLCPP_ERROR(this->get_logger(), "Failed to call service.");
        }
    };

private:
    rclcpp::Client<CalculateTwonumbers>::SharedPtr client_;
};

int main(int argc, char * argv[]) {
    rclcpp::init(argc, argv);
    auto node = std::make_shared<MathClient>();

    node->send_request(12.0, 5.0, "divide");

    rclcpp::shutdown();
    return 0;
}

```

---

### 5. 빌드 및 실행

작성이 완료되었다면 패키지를 빌드합니다.

```bash
$ colcon build --packages-select my_cpp_package
$ source install/setup.bash
# 터미널 1: 서버 실행
$ ros2 run my_cpp_package math_server
# 터미널 2: 클라이언트 실행
$ ros2 run my_cpp_package math_client

```
