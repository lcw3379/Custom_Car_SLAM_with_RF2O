
![makers](https://github.com/user-attachments/assets/6f040cfa-538c-4613-9866-843d29bf81d0)



이전에 자율주행차 교육용으로 받은 메이커스의 자율주행차 모델이 있다. 이 모델을 개조해서 직접 slam을 해보려 한다.

하지만 현재 가지고 있는 로봇 자동차 바퀴에 엔코더가 달려있지 않았다. 그래서 몇 달 전에 지능로봇 강의에서 공부할 때엔 Gmapping 등의 SLAM엔 odometry 데이터가 꼭 필요하다고 들어서, 이 로봇으로는 SLAM은 무리인가 하고 잠시 진행을 멈추었다가, 다시 엔코더 없이 LiDAR센서와 imu센서만으로 Cartographer SLAM을 하는 방법을 찾고 있었다.

# 1. yd lidar

ros2 humble에서 yd lidar_ros2_driver를 설치할 때엔 humble용 브랜치로 설정하고 설치
https://github.com/YDLIDAR/ydlidar_ros2_driver/tree/humble

rviz에서 시각화할땐 qos의 reliability가 서로 다르므로 rviz의 토픽 설정에서 reliable을 best_effort로 바꿔주었다.

라즈베리파이에서 시리얼 포트를 처음 사용할 땐 sudo usermod -a -G dialout $USER 해야 접근이 가능하다

라이다 설정을 한 이후 imu센서를 필터링하는 한편, odometry 방법을 찾아보던 중에 LiDAR 레이저 센서의 데이터로 odometry를 생성하는 기술을 알게 되었다.

# 2. RF2O

아니 이러면, 달랑 저가의 2D lidar센서 하나만으로도 odometry를 생성하고, 그걸 토대로 SLAM까지 꽤 정확하게 할 수 있지 않을까? 라는 생각이 들었다.

LOAM의 논문엔 3D lidar 센서의 포인트클라우드를 토대로 odometry를 만든다. 2d lidar에서도 odometry를 생성하도록 만들어놓은 것이 없나 찾아보다가 이미 패키지로 만들어놓은 것을 발견했다.

https://github.com/MAPIRlab/rf2o_laser_odometry

소스코드과 논문을 분석해서 어떤 방식으로 돌아가는 건지 알아야겠다.

![Screenshot from 2024-08-06 21-26-31](https://github.com/user-attachments/assets/07b8cbe4-ca91-486f-ac73-da04c0b3b5ba)


실제로 odometry가 지금까지 만든 것 중에서 가장 정확히 만들어지는 걸 확인할 수 있었다. 이걸 토대로 실내 환경에서 직접 cartographer slam

이미 필터링 중인 imu는 나중에 다른 프로젝트를 하면 사용하도록 하고, 현재는 rf2o 노드에서 생성된 odometry를 활용할 계획이다.


# 3. custom car

![customcar](https://github.com/user-attachments/assets/9551cdc7-a489-4b5d-ae72-5bcc4c1c343e)


기존의 자동차 모델을 개조하여 오렌지파이 보드를 빼고 라즈베리파이 보드와 lidar를 장착한 모습이다.




![room](https://github.com/user-attachments/assets/8bcfd0a5-6964-42b8-a913-5b8f4f706528)




확실히 실내에서 작동하는 로봇은 간단한 2d lidar 데이터만으로도 위치/방향 추종이 굉장히 잘 되어 odometry와 SLAM이 가능했다.

TF 데이터는 SLAM의 시작지점에서부터 현재의 위치/방향을 나타낸다.

 
   
