---
layout: post
title: "파라미터(Parameter)와 콜백(Callback)"
date: 2026-06-03
categories: ROS2
tags:
  - parameter
  - c++
  - ROS2
---

# 파라미터(Parameter)와 콜백(Callback)

애플리케이션을 개발할 때 속도 제한 값이나 센서 임계값 같은 설정치를 코드 내부에 **하드코딩(Hard-coding)**해 두면, 설정 하나를 바꿀 때마다 코드를 수정하고 매번 다시 빌드(Build)해야 하는 번거로움이 발생합니다.

이러한 문제를 해결하기 위해 사용하는 것이 바로 **실행 인자(Arguments)와 파라미터(Parameters)**입니다. 실행 인자를 사용하면 다음과 같은 강력한 이점을 얻을 수 있습니다.

1. **재빌드 없는 설정 변경**: 소스 코드를 건드리지 않고, 노드를 실행할 때 터미널 명령어(명령행 인자)나 YAML 설정 파일만 변경하여 동작을 유연하게 제어할 수 있습니다.
2. **다양한 환경으로의 높은 이식성**: 동일한 노드 바이너리를 수정 없이 물리적 환경이 다른 다양한 로봇(예: 최대 속도가 다른 로봇 모델)에 바로 적용할 수 있습니다.
3. **런타임 시 동적 설정**: 본문에서 다룰 **파라미터 콜백(Parameter Callback)**을 함께 활용하면, 노드가 작동하고 있는 도중에도 시스템 중단 없이 제어 파라미터를 실시간으로 안전하게 변경할 수 있습니다.

이러한 실행 인자들이 코드 내부로 올바르게 파싱되어 노드에 주입되려면, C++ `main` 함수에서 수신한 명령행 인자(`argc`, `argv`)를 ROS 2 초기화 함수인 `rclcpp::init(argc, argv)`에 그대로 넘겨주어야 합니다.

예를 들어, 터미널에서 노드를 실행할 때 다음과 같이 `--ros-args`와 `-p` 옵션을 지정하여 특정 파라미터의 초기 설정값을 변경해 기동할 수 있습니다.

```bash
ros2 run <package_name> <executable_name> --ros-args -p max_linear_velocity:=0.8 -p control_frequency:=30.0
```

아래는 실행 인자를 통해 주입받은 파라미터를 선언하고, 실시간 변경(Callback) 시의 유효성 검사 및 안전한 업데이트를 수행하는 C++ 예제 코드입니다.

```cpp
#include <rclcpp/rclcpp.hpp>
// 파라미터 변경 결과(성공 여부, 실패 이유 등)를 보고하기 위해 필요한 ROS2 인터페이스 헤더
#include <rcl_interfaces/msg/set_parameters_result.hpp>

class MobileBaseController : public rclcpp::Node {
    public:
        MobileBaseController() : Node("mobile_base_controller") {
            // 1. 파라미터 선언 (Declaration)
            // declare_parameter<T>(이름, 기본값)을 통해 노드가 기동될 때 사용할 파라미터를 등록합니다.
            // 외부(yaml 파일 등)에서 값을 제공하지 않으면 기본값이 적용됩니다.
            this->declare_parameter<double>("max_linear_velocity", 0.6);
            this->declare_parameter<double>("max_angular_velocity", 1.2);
            this->declare_parameter<double>("obstacle_stop_distance", 0.5);
            this->declare_parameter<double>("control_frequency", 50.0);
            this->declare_parameter<std::string>("base_frame", "base_link");
            this->declare_parameter<std::string>("odom_frame", "odom");

            // 2. 파라미터 값 가져오기 (Get)
            // 노드가 처음 실행될 때 설정된 값을 읽어와서 클래스의 멤버 변수에 저장합니다.
            max_linear_velocity_ = this->get_parameter("max_linear_velocity").as_double();
            max_angular_velocity_ = this->get_parameter("max_angular_velocity").as_double();
            obstacle_stop_distance_ = this->get_parameter("obstacle_stop_distance").as_double();
            control_frequency_ = this->get_parameter("control_frequency").as_double();
            base_frame_ = this->get_parameter("base_frame").as_string();
            odom_frame_ = this->get_parameter("odom_frame").as_string();

            // 3. 파라미터 변경 콜백 등록 (Parameter Callback Registration)
            // 외부(CLI `ros2 param set`, `rqt_reconfigure` 등)에서 파라미터 값이 실시간으로 변경될 때
            // 자동으로 실행될 콜백 함수를 연결(bind)해 줍니다.
            // parameter_callback_handle_은 콜백이 소멸되지 않고 노드 수명 동안 유지되도록 생명주기를 관리하는 역할을 합니다.
            parameter_callback_handle_ = this->add_on_set_parameters_callback(
                std::bind(
                    &MobileBaseController::onParameterChanged, // 실행할 콜백 멤버 함수
                    this,                                      // 현재 객체 포인터
                    std::placeholders::_1                      // 콜백 함수에 전달될 첫 번째 인자
                )
            );

            // 초기 로딩된 파라미터 값을 터미널에 출력
            printParameters();
        }

    private:
        // 노드 내부에서 사용할 파라미터 값들을 저장
        double max_linear_velocity_;
        double max_angular_velocity_;
        double obstacle_stop_distance_;
        double control_frequency_;
        std::string base_frame_;
        std::string odom_frame_;

        // 콜백 핸들 변수: 이 핸들이 유지되어야 파라미터 변경 이벤트가 계속 수신됩니다.
        OnSetParametersCallbackHandle::SharedPtr parameter_callback_handle_;

        // 현재 파라미터 상태를 출력하는 헬퍼 함수
        void printParameters() {
            RCLCPP_INFO(this->get_logger(), "max_linear_velocity: %.2f" ,max_linear_velocity_);
            RCLCPP_INFO(this->get_logger(), "max_angular_velocity: %.2f" ,max_angular_velocity_);
            RCLCPP_INFO(this->get_logger(), "obstacle_stop_distance: %.2f" ,obstacle_stop_distance_);
            RCLCPP_INFO(this->get_logger(), "control_frequency: %.2f" ,control_frequency_);
            RCLCPP_INFO(this->get_logger(), "base_frame: %s" ,base_frame_.c_str());
            RCLCPP_INFO(this->get_logger(), "odom_frame: %s" ,odom_frame_.c_str());
        }

        // 4. 파라미터 변경 콜백 함수 (Callback Function)
        // 외부에서 하나 혹은 여러 개의 파라미터가 동시에 변경될 때 이 함수가 호출됩니다.
        // 매개변수: 변경 요청이 들어온 파라미터들의 목록 (const std::vector<rclcpp::Parameter> &)
        // 반환값: 변경 허용 여부 및 실패 사유를 담은 SetParametersResult 구조체
        rcl_interfaces::msg::SetParametersResult onParameterChanged(const std::vector<rclcpp::Parameter> &parameters) {
            rcl_interfaces::msg::SetParametersResult result;
            result.successful = true; // 기본적으로 변경이 성공했다고 설정해둡니다.

            // 들어온 변경 요청들을 루프를 돌며 검증 및 반영합니다.
            for (const auto &param : parameters) {
                // 변경 요청이 온 파라미터의 이름을 가져옵니다.
                const std::string &param_name = param.get_name();

                // 선형 속도 최대값 검증 및 변경
                if(param_name == "max_linear_velocity") {
                    double value = param.as_double();

                    // 유효성 검사 (음수 값 차단)
                    if(value < 0.0) {
                        result.successful = false;
                        result.reason = "max_linear_velocity must be greater than or equal to 0.0";
                        return result;
                    }

                    max_linear_velocity_ = value;
                    RCLCPP_INFO(
                        this->get_logger(),
                        "Updated max_linear_velocity: %.2f",
                        max_linear_velocity_
                    );
                }

                // 각속도 최대값 검증 및 변경
                if(param_name == "max_angular_velocity") {
                    double value = param.as_double();

                    if(value < 0.0) {
                        result.successful = false;
                        result.reason = "max_angular_velocity must be greater than or equal to 0.0";
                        return result;
                    }

                    max_angular_velocity_ = value;
                    RCLCPP_INFO(
                        this->get_logger(),
                        "Updated max_angular_velocity: %.2f",
                        max_angular_velocity_
                    );
                }

                // 장애물 감지 정지 거리 검증 및 변경
                if(param_name == "obstacle_stop_distance") {
                    double value = param.as_double();

                    if(value < 0.0) {
                        result.successful = false;
                        result.reason = "obstacle_stop_distance must be greater than or equal to 0.0";
                        return result;
                    }

                    obstacle_stop_distance_ = value;
                    RCLCPP_INFO(
                        this->get_logger(),
                        "Updated obstacle_stop_distance: %.2f",
                        obstacle_stop_distance_
                    );
                }

                // 제어 주기(주파수) 검증 및 변경
                if(param_name == "control_frequency") {
                    double value = param.as_double();

                    // 주파수는 0보다 커야 함
                    if(value <= 0.0) {
                        result.successful = false;
                        result.reason = "control_frequency must be greater than 0.0";
                        return result;
                    }

                    control_frequency_ = value;
                    RCLCPP_INFO(
                        this->get_logger(),
                        "Updated control_frequency: %.2f",
                        control_frequency_
                    );
                }
            }

            return result; // 루프를 통과한 경우(모든 검증을 성공적으로 마친 경우) true가 담긴 result를 반환
        }
};

int main(int argc, char *argv[]) {
    rclcpp::init(argc, argv);
    // 노드 생성 후 이벤트를 계속 처리할 수 있도록 스핀(대기)
    rclcpp::spin(std::make_shared<MobileBaseController>());
    rclcpp::shutdown();
    return 0;
}
```

## 마무리

실행 인자를 사용하면 **소스를 수정하거나 재빌드하지 않고도 로봇 노드의 설정을 유연하게 변경**할 수 있습니다. 하지만 런타임 시에 검증되지 않은 잘못된 값이 들어오면 오작동을 유발할 수 있으므로, **콜백 함수 내에서 적절한 유효성 검사(Validation)**를 결합하여 안정성을 확보하는 것이 매우 중요합니다.
이처럼 파라미터 선언과 콜백을 잘 구성하면 다양한 로봇 환경에 유연하게 대응할 수 있는 안정적인 노드를 구축할 수 있습니다.
