<a name="fusion_module" />

## fusion
fusion传感器数据融合，整体的融合有硬件融合和数据融合，主要的工作是时间同步和数据融合。

## Multi-sensor fusion

### common
#### base_filter
base_filter 作为一个基类，被各种滤波器继承，以实现特定的滤波器。
滤波器的作用是对时序信息（状态）进行建模和滤波，目的是滤除量测带来的随机噪声。在 Apollo 代码中，主要考虑白噪声模型。

作为滤波器的基类，base_filter.h 实现了滤波器需要维护的属性以及基础接口。

属性：
1. bool _init; // 表示滤波器对象是否被初始化过
2. std::string name_; // 表示滤波器对象的实例化名称
3. int state_num_; // 表示系统状态维度
4. Eigen::MatrixXd transform_matrix_; // 系统转移矩阵
5. Eigen::VectorXd global_states_; // 命名为全局状态，实际上指用来初始化滤波器对象的状态向量
6. Eigen::MatrixXd global_uncertainty_; // 全局状态对应的协方差矩阵
7. Eigen::MatrixXd env_uncertainty_; // 滤波器在预测过程的噪声
8. Eigen::MatrixXd cur_observation_; // 当前时刻的观测值
9. Eigen::MatrixXd cur_observation_uncertainty_; // 当前时刻的量测噪声
10. Eigen::MatrixXd c_matrix_; // 控制矩阵

方法接口：
1. virtual bool Init(const Eigen::VectorXd &global_states,
                    const Eigen::MatrixXd &global_uncertainty) = 0; // 初始化接口
2. virtual bool Predict(const Eigen::MatrixXd &transform_matrix,
                       const Eigen::MatrixXd &env_uncertainty_matrix) = 0; // 预测接口
3. virtual bool Correct(const Eigen::VectorXd &cur_observation,
                       const Eigen::MatrixXd &cur_observation_uncertainty) = 0; // 更新接口
4. virtual bool SetControlMatrix(const Eigen::MatrixXd &c_matrix) = 0; // 设置控制矩阵的接口
5. virtual Eigen::VectorXd GetStates() const = 0; // 获得滤波后系统状态的接口
6. std::string Name() { return name_; } // 获得滤波器对象名称的接口

## app
增添一句话。
## lib
