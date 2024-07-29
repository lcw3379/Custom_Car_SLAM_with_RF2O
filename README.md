# custom_car

1. yd lidar

ros2 humble에서 yd lidar_ros2_driver를 설치할 때엔 humble용 브랜치로 설정하고 설치
https://github.com/YDLIDAR/ydlidar_ros2_driver/tree/humble

rviz에서 시각화할땐 qos의 reliability가 서로 다르므로 rviz의 토픽 설정에서 reliable을 best_effort로 바꿔주면 됨

Node name: ydlidar_ros2_driver_node
Node namespace: /
Topic type: sensor_msgs/msg/LaserScan
Endpoint type: PUBLISHER
GID: 01.0f.61.7c.9f.50.39.10.01.00.00.00.00.00.12.03.00.00.00.00.00.00.00.00
QoS profile:
  Reliability: BEST_EFFORT
  History (Depth): UNKNOWN
  Durability: VOLATILE
  Lifespan: Infinite
  Deadline: Infinite
  Liveliness: AUTOMATIC
  Liveliness lease duration: Infinite

Subscription count: 1

Node name: rviz
Node namespace: /
Topic type: sensor_msgs/msg/LaserScan
Endpoint type: SUBSCRIPTION
GID: 01.0f.61.7c.7e.50.10.9a.01.00.00.00.00.00.1b.04.00.00.00.00.00.00.00.00
QoS profile:
  Reliability: RELIABLE
  History (Depth): UNKNOWN
  Durability: VOLATILE
  Lifespan: Infinite
  Deadline: Infinite
  Liveliness: AUTOMATIC
  Liveliness lease duration: Infinite




2. mpu6050
