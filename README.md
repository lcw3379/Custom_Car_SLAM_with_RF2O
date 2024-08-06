# custom_car

1. yd lidar

ros2 humble에서 yd lidar_ros2_driver를 설치할 때엔 humble용 브랜치로 설정하고 설치
https://github.com/YDLIDAR/ydlidar_ros2_driver/tree/humble

rviz에서 시각화할땐 qos의 reliability가 서로 다르므로 rviz의 토픽 설정에서 reliable을 best_effort로 바꿔주면 됨



라즈베리파이에서 시리얼 포트 사용할 땐 sudo usermod -a -G dialout $USER 해야 접근 가능


2. mpu6050

현재 가지고 있는 로봇 자동차 바퀴에 엔코더가 달려있자 않았다.
그래서 처음에는 엔코더 없이 LiDAR센서와 imu센서만으로 cartographer slam을 하는 방법을 찾았다.

그러기 위해서 imu센서를 필터링하는 한편, odometry 방법을 찾아보던 중에 LiDAR 레이저 센서의 데이터로 odometry를 생성하는 기술을 알게 되었다.

아니 이러면, 달랑 2D lidar센서 하나만으로도 odometry를 생성하고, 그걸 토대로 slam까지 꽤 정확하게 할 수 있지 않을까? 라는 생각이 들었다.

LOAM의 논문엔 3D lidar 센서의 포인트클라우드를 토대로 odometry를 만든다. 이걸 2d lidar로 만들어놓은 것이 없나 찾아보다가 이미 패키지로 만들어놓은 것을 발견했다.

https://github.com/MAPIRlab/rf2o_laser_odometry

실제로 odometry가 정확히 만들어지는 걸 확인할 수 있었다. 이걸 토대로 실내 환경에서 직접 cartographer slam와 navigation을 해 보고, 직접 짠 path planning 알고리즘을 적용해 보아야 겠다.
