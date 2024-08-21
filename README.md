
![makers](https://github.com/user-attachments/assets/6f040cfa-538c-4613-9866-843d29bf81d0)



이전에 자율주행차 교육용으로 받은 메이커스의 자율주행차 모델이 있다. 이 모델을 개조해서 직접 slam을 해보려 한다.

하지만 현재 가지고 있는 로봇 자동차 바퀴에 엔코더가 달려있지 않았다. 몇 달 전에 지능로봇 강의에서 공부할 때엔 Gmapping 등의 SLAM엔 odometry 데이터가 꼭 필요하다고 들어서, 이 로봇으로는 SLAM은 무리인가 하고 잠시 진행을 멈추었다. 그리고 다시 엔코더 없이 LiDAR센서와 imu센서만으로 Cartographer SLAM을 하는 방법을 찾고 있었다.

# 1. yd lidar x4

ros2 humble에서 yd lidar_ros2_driver를 설치할 때엔 humble용 브랜치로 설정하고 설치
https://github.com/YDLIDAR/ydlidar_ros2_driver/tree/humble

rviz에서 시각화할땐 qos의 reliability가 서로 다르므로 rviz의 토픽 설정에서 reliable을 best_effort로 바꿔주었다.

라즈베리파이에서 시리얼 포트를 처음 사용할 땐 sudo usermod -a -G dialout $USER 해야 접근이 가능하다

라이다 설정을 한 이후 imu센서를 필터링하는 한편, odometry 방법을 찾아보던 중에 LiDAR 레이저 센서의 데이터로 odometry를 생성하는 기술을 알게 되었다.

# 2. RF2O

아니 이러면, 달랑 저가의 2D lidar센서 하나만으로도 odometry를 생성하고, 그걸 토대로 SLAM까지 꽤 정확하게 할 수 있지 않을까? 라는 생각이 들었다.

LOAM의 논문엔 3D lidar 센서의 포인트클라우드를 토대로 odometry를 만든다. 2d lidar에서도 odometry를 생성하도록 만들어놓은 것이 없나 찾아보다가 이미 ROS와 ROS2에서 모두 사용 가능하도록 패키지로 만들어놓은 것을 발견했다.

https://github.com/MAPIRlab/rf2o_laser_odometry

Planar Odometry from a Radial Laser Scanner. A Range Flow-based Approach 이라는 논문을 기반으로 작성된 패키지이니 논문을 먼저 분석하였다.

# 3. 논문 분석

스캔 범위 R(t,α) 를 정의한다. 이때 t는 시간, α는 scan coordinate 이다.

![image](https://github.com/user-attachments/assets/3f331ff4-8fa1-4248-a5dc-e8d20d948c63)

![image](https://github.com/user-attachments/assets/accd11d9-1250-4a39-ac8b-94f00d57d123)

첫 스캔에서 R의 위치와 다음 스캔에서 스캔 범위 R은 테일러 전개에 의해 다음과 같이 표현된다.

![image](https://github.com/user-attachments/assets/434380ef-1cdb-4ad2-893e-597f504caa0a)

여기서 뒤쪽 항인 O는 무시하고, 양변을 Δt로 나누고 정리하면 다음과 같이 나타낼 수 있다.

![image](https://github.com/user-attachments/assets/74896bac-4429-4bc7-a34a-c67e0c08ea59)


단, ΔR, Rt, Ra는 다음과 같다.

![image](https://github.com/user-attachments/assets/b11553a9-731e-42ff-8978-1781fd37a080)

이때 $\dot{r} = ∆R/∆t, \dot{α} = ∆α/∆t$ 로 나타낸다. $\dot{r}$는 범위 내 점들의 평균 속도, $\dot{α}$는 스캔 좌표계의 평균 속도이다.

![image](https://github.com/user-attachments/assets/bf7448c3-f119-4c10-9a09-60413a0c7b4d)

이 식이 Gonzalez&Gutierrez 에 의해서 처음 고안된 range flow constraint equation 이라고 한다.

한편, 위의 그림에서 점 P의 위치를 극좌표 r과 theta로 표현했다. 그럼 r과 theta의 속도는 다음과 같이 나타냈다.

![image](https://github.com/user-attachments/assets/7d99656c-eda8-4396-af0d-ef3a10f0e87f)

마지막으로 모든 점이 센서에 대해 속도는 같지만 부호가 반대인 강체의 일부로 가정하였다.

![image](https://github.com/user-attachments/assets/8cd0b12d-f2a0-4b48-9084-fc825d9f02da)

이때 ![image](https://github.com/user-attachments/assets/67e2ad89-f621-47a9-9b49-679023973ac6) 을 x,y축에서의 속도와 z축 각속도로 정의했다.

이 식들을 range flow constraint equation 에 넣고 정리하면 다음과 같이 나온다.

![image](https://github.com/user-attachments/assets/b64e0763-8dfc-419a-a64a-6f8844166cdf)

이때, 3개의 선형으로 독립된 제한요소(x방향 속도, y방향 속도, 각속도)들이 있으면 이론적으로는 추정하기에 충분하다. 

하지만 센서의 노이즈와 테일러 급수의 근사 등의 이유로 실제로는 추정이 불가능하다고 한다.

논문에서는 기하적 잔여? (geometric residuals) 라고 하는 함수를 위에서 구한 식으로 정의했다.

![image](https://github.com/user-attachments/assets/f166edf8-1451-4d13-9e23-0db3a50b39b8)

그리고 모든 geometric residuals를 최소화 하기 위해, Robust Function인 F라고 하는 함수를 정의했다. 이때 F는 Cauchy M-estimator 라 하고, k는 조정 가능한 값이다.

![image](https://github.com/user-attachments/assets/56660a6f-e26d-4ce8-8ffd-a1ab6442eb52)

이것의 최적화 문제는 이미 Iteratively Reweighted Least Squares (IRLS)로 풀려 있고, 가중치는 다음과 같다.

![image](https://github.com/user-attachments/assets/647653f8-d256-4fca-859d-0f60ab2170a6)

하지만 이렇게 만든 Cauchy M-estimator도 오차를 완전히 없앨 수는 없어서 Pre-weighting strategy 라는 방법을 제안했다. 2번에서의 테일러 전개를 2차로 확장하였다.

![image](https://github.com/user-attachments/assets/da1c8fbd-71a1-4e56-8d2a-2c19a6cfffd0)


이때 ![image](https://github.com/user-attachments/assets/39634410-883c-4284-83cb-fbdf497788be) 에서 R2o의 2차 도함수로 (3)의 선형 편차를 감지할 수 있다고 한다.



# (추가!)

비선형성과 불연속성에 패널티를 주기 위해 해당 논문은 다음과 같은 사전 가중치 함수를 정의했다.

![image](https://github.com/user-attachments/assets/b2ca00e5-b412-48e7-8032-7d820f47da2e)



이때 Kd는 1차와 2차도함수의 상대적 연관도, ![image](https://github.com/user-attachments/assets/b62c980f-bf10-4e85-9e6a-290772844e03) 는 singular case를 피하기 위한 상수라 한다.


따라서 이 residuals들에 가중치를 다음과 같이 부여했다.

![image](https://github.com/user-attachments/assets/e08ba269-8031-4418-871c-62fda9b5decf)

이것이 (10)과 (11)에 따라 최소화 된다.

다음으로 R0과 R1을 두 개의 연속 레이저 스캔이라고 가정하면, 가장 먼저 R0와 R1을 연속적으로 다운샘플링 해서 가우시안 피라미드를 만들어야 한다.

이때 가우시안 마스크를 필터로 주로 사용하는데, 이번과 같은 범위 데이터에선 이게 최적의 선택이 아니다. 따라서 bilateral filter(쌍방 필터)를 대안으로 사용한다.



간단히 2D Laserscan 데이터로 오도메트리 데이터를 생성하기에 이론도 간단할 줄 알았는데, 배우지 못한 여러 수학적 기법들이 많이 사용될 줄은 몰랐다. 역시 논문은 어렵다...

그럼 논문의 이해를 위해 rf2o 코드를 살펴보았다.

![image](https://github.com/user-attachments/assets/65ddde30-350e-4e61-9a92-cdd536b6a1b6)

![image](https://github.com/user-attachments/assets/f362a833-acc7-459a-85e2-7ab23c6679dc)

핵심 함수인  CLaserOdometry2D::odometryCalculation 함수이다.

순서대로 




![Screenshot2](https://github.com/user-attachments/assets/07b8cbe4-ca91-486f-ac73-da04c0b3b5ba)

실제로 odometry가 지금까지 만든 것 중에서 가장 정확히 만들어지는 걸 확인할 수 있었다. 이걸 토대로 실내 환경에서 직접 자동차에 라이다 센서를 장착해서 cartographer slam을 해 보아야 겠다.

이미 필터링 중인 imu는 나중에 다른 프로젝트를 하면 사용하도록 하고, 현재는 rf2o 노드에서 생성된 odometry를 활용할 계획이다.

# 3. custom car

![customcar](https://github.com/user-attachments/assets/9551cdc7-a489-4b5d-ae72-5bcc4c1c343e)


기존의 자동차 모델을 개조하여 오렌지파이 보드를 빼고 라즈베리파이 보드와 lidar를 장착한 모습이다.




![room](https://github.com/user-attachments/assets/8bcfd0a5-6964-42b8-a913-5b8f4f706528)




확실히 실내에서 작동하는 로봇은 간단한 2d lidar 데이터만으로도 위치/방향 추종이 굉장히 잘 되어 odometry와 SLAM이 가능했다.

TF 데이터는 SLAM의 시작지점에서부터 현재의 위치/방향을 나타낸다.

Map까지 생성을 했으니 이걸 토대로 훗날 Navigation을 할 수 있다고 기대한다.
 
   
