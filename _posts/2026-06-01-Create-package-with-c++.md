---
layout: post
title: "Create package with c++"
date: 2026-06-01
categories: ROS2
tags:
  - build system
  - c++
---

# ROS2 C++ 패키지 생성하기

## Package 생성하기

```bash
ros2 pkg create <package_name> --build-type ament_cmake --dependencies rclcpp std_msgs
```

- 빌드 타입으로 `ament_cmake`를 사용합니다.
- C++의 대표적인 빌드 시스템인 **CMake**를 기반으로 동작합니다.

### Python 패키지와 차이점

- **Python (`setup.py` 중심):** Python은 의존성과 패키지 정보를 `setup.py`에 작성합니다. 별도의 컴파일 과정 없이 스크립트를 바로 실행할 수 있기 때문에 구조가 직관적이고 수정 후 테스트가 매우 빠릅니다.

- **C++ (`CMakeLists.txt` 중심):** C++ 패키지는 `CMakeLists.txt`라는 파일이 핵심입니다. 이곳에 실행 파일(Executable)을 어떻게 만들 것인지, 어떤 외부 라이브러리(Dependencies)를 링킹(Linking)할 것인지 세밀하게 명시해야 합니다. 코드(src)와 헤더(include) 폴더가 분리되는 것도 특징입니다.

- Python은 `colcon build --symlink-install`을 사용하면 코드를 수정할 때마다 다시 빌드할 필요 없이 즉시 반영됩니다. C++은 코드를 한 줄만 수정해도 **반드시 다시 `colcon build`를 통해 컴파일**을 거쳐야만 변경 사항이 반영됩니다.

### 폴더구조

```bash
my_cpp_package/
├── CMakeLists.txt
├── include/
│ └── my_cpp_package/
├── package.xml
└── src/
```

### 의존성 추가 확인

```xml
...
<buildtool_depend>ament_cmake</buildtool_depend>

<depend>rclcpp</depend>
<depend>std_msgs</depend>

<export>
	<build_type>ament_cmake</build_type>
</export>
...
```

### publisher node 작성하기

`src/simple_publisher.cpp`

```cpp

#include <chrono>     // 시간 관련 기능 (예: 1s) 사용을 위한 헤더
#include <functional> // std::bind 등 함수 객체 처리를 위한 헤더
#include <memory>     // 스마트 포인터 (SharedPtr 등) 사용을 위한 헤더
#include <string>     // 문자열 처리를 위한 헤더

#include "rclcpp/rclcpp.hpp"       // ROS 2 핵심 C++ 라이브러리
#include "std_msgs/msg/string.hpp" // 표준 문자열 메시지 타입(String)

using namespace std::chrono_literals; // '1s', '500ms' 와 같은 시간 리터럴을 쉽게 쓰기 위해 사용

// rclcpp::Node를 상속받아 SimplePublisher라는 노드 클래스 정의
class SimplePublisher : public rclcpp::Node {
public:
    // 생성자: 노드의 이름을 "Simplepubisher"로 지정하고, count_ 변수를 0으로 초기화
    SimplePublisher(): Node("Simplepubisher"), count_(0) {

        // 퍼블리셔 생성: 메시지 타입(String), 토픽 이름("mobile_robot_status"), 통신 큐 사이즈(10)
        publisher_ = this->create_publisher<std_msgs::msg::String>("mobile_robot_status", 10);

        // 타이머 생성: 1초(1s)마다 timer_callback 함수를 반복적으로 실행하도록 설정
        timer_ = this->create_wall_timer(1s, std::bind(&SimplePublisher::timer_callback, this));
    };

private:
    // 타이머 콜백 함수: 지정된 주기(1초)마다 실행되어 실제로 메시지를 발행하는 역할
    void timer_callback()
    {
        auto message = std_msgs::msg::String(); // 보낼 메시지 객체 생성

        // 메시지 데이터 안에 문자열과 카운트 값을 결합하여 저장하고, 카운트 1 증가
        message.data = "Mobile robot is alive: " + std::to_string(count_++);

        // 콘솔 창에 현재 발행하는 메시지의 내용을 출력 (로깅)
        RCLCPP_INFO(this->get_logger(), "Publishing: '%s'", message.data.c_str());

        // 생성한 메시지를 "mobile_robot_status" 토픽으로 발행
        publisher_->publish(message);
    }

    // 멤버 변수 선언부
    rclcpp::Publisher<std_msgs::msg::String>::SharedPtr publisher_; // 퍼블리셔의 스마트 포인터
    rclcpp::TimerBase::SharedPtr timer_;                            // 타이머의 스마트 포인터
    size_t count_;                                                  // 발행 횟수를 세는 변수
};

int main(int argc, char* argv[]) {
    // ROS 2 시스템 초기화 (모든 ROS 2 프로그램의 시작점)
    rclcpp::init(argc, argv);

    // SimplePublisher 노드를 생성하고 스핀(spin) 상태로 진입
    // spin은 노드가 종료될 때까지 대기하며 콜백 함수(여기서는 타이머)를 지속적으로 실행하게 해줍니다.
    rclcpp::spin(std::make_shared<SimplePublisher>());

    // 노드 종료 및 할당된 리소스 정리
    rclcpp::shutdown();

    return 0;
}
```

### subscriber node 작성하기

`src/simple_subscriber.cpp`

```cpp
#include <functional> // std::bind, std::placeholders 등 콜백 함수 연결을 위한 헤더
#include <memory>     // 스마트 포인터 사용을 위한 헤더
#include <string>     // 문자열 처리를 위한 헤더

#include "rclcpp/rclcpp.hpp"       // ROS 2 핵심 C++ 라이브러리
#include "std_msgs/msg/string.hpp" // 표준 문자열 메시지 타입(String)

// rclcpp::Node를 상속받아 SimpleSubscriber라는 노드 클래스 정의
class SimpleSubscriber : public rclcpp::Node {
    public:
        // 생성자: 노드의 이름을 "simple_subscriber"로 지정
        SimpleSubscriber() : Node("simple_subscriber") {

            // 서브스크라이버 생성:
            // 구독할 토픽 이름("mobile_robot_status"), 큐 사이즈(10),
            // 메시지가 수신될 때마다 실행될 콜백 함수(topic_callback)를 연결
            subscription_ = this->create_subscription<std_msgs::msg::String>(
                "mobile_robot_status",
                10,
                std::bind(&SimpleSubscriber::topic_callback, this, std::placeholders::_1)
            );
        };

    private:
        // 토픽 콜백 함수: 퍼블리셔로부터 메시지가 도착할 때마다 자동으로 호출됨
        // 파라미터(msg)는 수신된 메시지 데이터를 담고 있는 스마트 포인터
        void topic_callback(const std_msgs::msg::String::SharedPtr msg) const {
            // 수신한 메시지의 데이터를 콘솔 창에 출력 (로깅)
            RCLCPP_INFO(this->get_logger(), "Received: '%s'", msg->data.c_str());
        }

        // 서브스크라이버의 스마트 포인터 멤버 변수
        rclcpp::Subscription<std_msgs::msg::String>::SharedPtr subscription_;
};

int main(int argc, char * argv[]) {
    // ROS 2 시스템 초기화
    rclcpp::init(argc, argv);

    // SimpleSubscriber 노드를 생성하고 스핀(spin) 상태로 진입
    // 메시지가 수신될 때마다 콜백 함수가 실행될 수 있도록 계속 대기합니다.
    rclcpp::spin(std::make_shared<SimpleSubscriber>());

    // 노드 종료 및 리소스 정리
    rclcpp::shutdown();

    return 0;
}

```

## 무엇을 선택해야 할까?

은탄환은 없다는 말이 있듯 정답은 없는것 같습니다.

- **빠른 아이디어 검증이나 AI/딥러닝 연동**이 필요하다면 **Python**으로 먼저 패키지를 구성하는 것이 효율적입니다.
- 하지만 자율주행, 로봇 팔 제어, 대용량 센서 데이터(Point Cloud 등) 처리처럼 **0.01초의 반응 속도와 실시간성(Real-time)이 중요한 코어 모듈**이라면 반드시 C++로 패키지를 작성해야 합니다.
