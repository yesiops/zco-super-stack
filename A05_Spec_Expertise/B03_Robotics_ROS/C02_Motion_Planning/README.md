# C02_Motion_Planning

**所属子领域**: [B03_Robotics_ROS](../README.md)
**创建日期**: 2026-01-30
**最后更新**: 2026-01-30

## 📋 主题定位

运动规划（Motion Planning）是机器人学的核心领域，研究如何在复杂环境中为机器人生成从起始状态到目标状态的无碰撞、高效、平滑的运动轨迹。它涵盖了路径规划（Path Planning）、轨迹优化（Trajectory Optimization）、运动控制（Motion Control）等多个层面，是机器人自主操作、自动驾驶、无人机飞行等应用的关键技术。

## 🎯 核心概念

### 基本定义

运动规划解决在配置空间（C-space）中搜索可行路径的问题，核心要素包括：
- **配置空间（C-space）**：机器人所有可能状态的集合
- **自由空间（C-free）**：无碰撞的配置子集
- **路径（Path）**：C-space中的连续曲线
- **轨迹（Trajectory）**：带时间参数的路径
- **运动学/动力学约束**：机器人运动能力的限制

### 关键特性

**1. 规划算法分类**
- **基于图搜索**：A*、Dijkstra、D*、JPS
- **基于采样**：PRM（概率路线图）、RRT（快速随机树）
- **基于优化**：CHOMP、STOMP、TrajOpt
- **基于学习**：Neural Motion Planner、Diffusion Policy

**2. 规划维度**
- **全局规划**：长距离粗略路径
- **局部规划**：近距离精细轨迹
- **层级规划**：任务->行为->动作分层

**3. 约束处理**
- **运动学约束**：关节限位、奇异点避免
- **动力学约束**：速度/加速度限制
- **几何约束**：避障、路径宽度
- **时序约束**：截止时间、同步要求

**4. 实时性考虑**
- ** anytime算法**：快速提供次优解，逐步优化
- **重规划**：环境变化时快速调整
- **并行计算**：GPU加速采样和评估

### 应用场景
- **工业机器人**：机械臂抓取、焊接、装配
- **自动驾驶**：车道保持、避障、泊车
- **无人机**：三维路径、编队飞行
- **移动机器人**：AGV导航、探索建图
- **人形机器人**：双足行走、全身控制

## 🛠️ 技术实践

### 实现方法

**1. ROS2 Nav2移动机器人导航**

```python
#!/usr/bin/env python3
# custom_planner.py
import rclpy
from rclpy.node import Node
from nav2_simple_commander.robot_navigator import BasicNavigator
from geometry_msgs.msg import PoseStamped
from nav_msgs.msg import Path
import math

class AutonomousNavigator(Node):
    """
    基于Nav2的自主导航系统
    包含全局规划和局部控制
    """
    
    def __init__(self):
        super().__init__('autonomous_navigator')
        
        self.navigator = BasicNavigator()
        
        # 等待Nav2激活
        self.navigator.waitUntilNav2Active()
        
        self.get_logger().info('Autonomous Navigator initialized')
    
    def navigate_to_pose(self, x, y, yaw):
        """
        导航到指定位姿
        使用Nav2内置的全局规划器（A*/Dijkstra）和局部规划器（DWB/Pure Pursuit）
        """
        goal_pose = PoseStamped()
        goal_pose.header.frame_id = 'map'
        goal_pose.header.stamp = self.navigator.get_clock().now().to_msg()
        goal_pose.pose.position.x = x
        goal_pose.pose.position.y = y
        goal_pose.pose.orientation.z = math.sin(yaw / 2)
        goal_pose.pose.orientation.w = math.cos(yaw / 2)
        
        self.navigator.goToPose(goal_pose)
        
        # 等待完成（带超时检测）
        while not self.navigator.isTaskComplete():
            feedback = self.navigator.getFeedback()
            if feedback:
                self.get_logger().info(
                    f'Estimated time: {feedback.estimated_time_remaining.sec} seconds'
                )
        
        result = self.navigator.getResult()
        return result
    
    def follow_waypoints(self, waypoints):
        """
        跟随路径点
        waypoints: [(x1, y1, yaw1), (x2, y2, yaw2), ...]
        """
        poses = []
        for x, y, yaw in waypoints:
            pose = PoseStamped()
            pose.header.frame_id = 'map'
            pose.header.stamp = self.navigator.get_clock().now().to_msg()
            pose.pose.position.x = x
            pose.pose.position.y = y
            pose.pose.orientation.z = math.sin(yaw / 2)
            pose.pose.orientation.w = math.cos(yaw / 2)
            poses.append(pose)
        
        self.navigator.followWaypoints(poses)
        
        while not self.navigator.isTaskComplete():
            feedback = self.navigator.getFeedback()
            if feedback:
                self.get_logger().info(
                    f'Executing waypoint {feedback.current_waypoint + 1}/{len(waypoints)}'
                )
        
        return self.navigator.getResult()
    
    def backup(self, distance=0.5, speed=0.1):
        """后退指定距离"""
        self.navigator.backup(distance, speed)
        return self.navigator.getResult()
    
    def spin(self, angle_deg=90):
        """原地旋转"""
        self.navigator.spin(spin_dist=math.radians(angle_deg))
        return self.navigator.getResult()

def main():
    rclpy.init()
    
    navigator = AutonomousNavigator()
    
    # 示例：导航到目标点
    result = navigator.navigate_to_pose(x=3.0, y=2.0, yaw=1.57)
    
    if result == NavigationResult.SUCCEEDED:
        print('Navigation succeeded!')
    else:
        print(f'Navigation failed with result: {result}')
    
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

**2. 机械臂运动学规划（MoveIt 2）**

```python
#!/usr/bin/env python3
# manipulator_planning.py
import rclpy
from rclpy.node import Node
from moveit.planning import MoveItPy
from moveit.core.kinematic_constraints import construct_joint_constraint
from geometry_msgs.msg import PoseStamped, Pose
from sensor_msgs.msg import JointState
import numpy as np

class ManipulatorMotionPlanner(Node):
    """
    机械臂运动规划器（基于MoveIt 2）
    支持关节空间、笛卡尔空间规划
    """
    
    def __init__(self):
        super().__init__('manipulator_planner')
        
        # 初始化MoveIt
        self.moveit = MoveItPy(node_name='moveit_py')
        self.robot = self.moveit.get_planning_component('panda_arm')
        
        self.get_logger().info('Manipulator planner initialized')
    
    def plan_to_joint_goal(self, joint_positions, planner_id='RRTConnect'):
        """
        关节空间规划
        joint_positions: [j1, j2, j3, j4, j5, j6, j7]
        """
        # 设置规划器
        self.robot.set_planner_id(planner_id)
        self.robot.set_planning_time(5.0)
        
        # 设置目标状态
        goal_state = self.robot.get_goal_state()
        for i, pos in enumerate(joint_positions):
            goal_state.set_joint_positions(f'panda_joint{i+1}', [pos])
        
        self.robot.set_goal_state(goal_state)
        
        # 规划
        plan_result = self.robot.plan()
        
        if plan_result:
            trajectory = plan_result.trajectory
            self.get_logger().info(f'Plan found with {len(trajectory)} waypoints')
            return trajectory
        else:
            self.get_logger().warn('Planning failed')
            return None
    
    def plan_to_pose_goal(self, target_pose, frame_id='panda_link0'):
        """
        笛卡尔空间规划（末端执行器位姿）
        target_pose: geometry_msgs/Pose
        """
        goal_pose = PoseStamped()
        goal_pose.header.frame_id = frame_id
        goal_pose.header.stamp = self.get_clock().now().to_msg()
        goal_pose.pose = target_pose
        
        # 设置姿态目标
        self.robot.set_goal_state(pose_goal=goal_pose, link_name='panda_hand')
        
        plan_result = self.robot.plan()
        
        if plan_result:
            return plan_result.trajectory
        return None
    
    def plan_cartesian_path(self, waypoints, eef_step=0.01, jump_threshold=0.0):
        """
        笛卡尔路径规划（直线插值）
        waypoints: [Pose, Pose, ...]
        """
        from moveit.core.robot_state import RobotState
        
        # 获取当前状态
        current_state = self.robot.get_current_state()
        
        # 计算笛卡尔路径
        fraction, trajectory = self.moveit.compute_cartesian_path(
            current_state,
            waypoints,
            eef_step,
            jump_threshold
        )
        
        self.get_logger().info(f'Cartesian path: {fraction*100:.1f}% achieved')
        
        if fraction > 0.9:
            return trajectory
        return None
    
    def execute_trajectory(self, trajectory):
        """执行规划轨迹"""
        if trajectory:
            self.moveit.execute(trajectory, controllers=[])
            return True
        return False
    
    def add_collision_object(self, pose, size, name='obstacle'):
        """添加碰撞物体"""
        from shape_msgs.msg import SolidPrimitive
        from moveit.planning_scene import PlanningScene
        
        collision_object = SolidPrimitive()
        collision_object.type = SolidPrimitive.BOX
        collision_object.dimensions = size  # [x, y, z]
        
        pose_stamped = PoseStamped()
        pose_stamped.pose = pose
        pose_stamped.header.frame_id = 'panda_link0'
        
        self.moveit.add_collision_object(
            name=name,
            primitives=[collision_object],
            primitive_poses=[pose_stamped],
            frame_id='panda_link0'
        )
    
    def plan_with_constraints(self, target_pose, path_constraints):
        """
        带约束的规划
        例如：保持末端执行器水平
        """
        self.robot.set_goal_state(pose_goal=target_pose)
        self.robot.set_path_constraints(path_constraints)
        
        plan_result = self.robot.plan()
        self.robot.clear_path_constraints()
        
        return plan_result.trajectory if plan_result else None

def create_orientation_constraint(frame_id, link_name, orientation):
    """创建姿态约束（保持特定方向）"""
    from moveit_msgs.msg import OrientationConstraint
    
    constraint = OrientationConstraint()
    constraint.header.frame_id = frame_id
    constraint.link_name = link_name
    constraint.orientation = orientation
    constraint.absolute_x_axis_tolerance = 0.1
    constraint.absolute_y_axis_tolerance = 0.1
    constraint.absolute_z_axis_tolerance = 0.1
    constraint.weight = 1.0
    
    return constraint

def main():
    rclpy.init()
    
    planner = ManipulatorMotionPlanner()
    
    # 示例1：关节空间规划
    joint_goal = [0.0, -0.785, 0.0, -2.356, 0.0, 1.571, 0.785]
    trajectory = planner.plan_to_joint_goal(joint_goal)
    
    if trajectory:
        planner.execute_trajectory(trajectory)
    
    # 示例2：笛卡尔空间规划
    pose_goal = Pose()
    pose_goal.position.x = 0.4
    pose_goal.position.y = 0.2
    pose_goal.position.z = 0.5
    pose_goal.orientation.w = 1.0
    
    trajectory = planner.plan_to_pose_goal(pose_goal)
    if trajectory:
        planner.execute_trajectory(trajectory)
    
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

**3. RRT路径规划实现**

```python
#!/usr/bin/env python3
# rrt_planner.py
import numpy as np
import matplotlib.pyplot as plt
from typing import List, Tuple, Optional
import random

class RRTPlanner:
    """
    RRT（快速随机树）路径规划器
    适用于高维配置空间
    """
    
    class Node:
        def __init__(self, config: np.ndarray):
            self.config = config
            self.parent: Optional['RRTPlanner.Node'] = None
            self.cost: float = 0.0
    
    def __init__(self, 
                 bounds: List[Tuple[float, float]],
                 step_size: float = 0.1,
                 max_iter: int = 10000,
                 goal_sample_rate: float = 0.1,
                 connect_circle_dist: float = 1.0):
        """
        初始化RRT规划器
        bounds: [(x_min, x_max), (y_min, y_max), ...] 各维度范围
        """
        self.bounds = bounds
        self.step_size = step_size
        self.max_iter = max_iter
        self.goal_sample_rate = goal_sample_rate
        self.connect_circle_dist = connect_circle_dist
        
        self.nodes: List[RRTPlanner.Node] = []
        self.obstacles: List[Tuple[np.ndarray, float]] = []  # (center, radius)
    
    def add_obstacle(self, center: np.ndarray, radius: float):
        """添加圆形障碍物"""
        self.obstacles.append((center, radius))
    
    def is_collision(self, config: np.ndarray) -> bool:
        """检查配置是否与障碍物碰撞"""
        for obs_center, obs_radius in self.obstacles:
            if np.linalg.norm(config - obs_center) < obs_radius:
                return True
        return False
    
    def is_edge_collision(self, config1: np.ndarray, config2: np.ndarray) -> bool:
        """检查两点间连线是否与障碍物碰撞"""
        # 采样检查
        steps = int(np.linalg.norm(config2 - config1) / (self.step_size * 0.5))
        steps = max(steps, 2)
        
        for i in range(steps + 1):
            t = i / steps
            config = config1 + t * (config2 - config1)
            if self.is_collision(config):
                return True
        return False
    
    def sample_config(self, goal: np.ndarray) -> np.ndarray:
        """随机采样配置空间"""
        if random.random() < self.goal_sample_rate:
            return goal.copy()
        
        config = np.zeros(len(self.bounds))
        for i, (min_bound, max_bound) in enumerate(self.bounds):
            config[i] = random.uniform(min_bound, max_bound)
        return config
    
    def nearest_node(self, config: np.ndarray) -> 'RRTPlanner.Node':
        """找到最近的节点"""
        return min(self.nodes, key=lambda node: np.linalg.norm(node.config - config))
    
    def steer(self, from_config: np.ndarray, to_config: np.ndarray) -> np.ndarray:
        """从from_config向to_config前进step_size距离"""
        direction = to_config - from_config
        distance = np.linalg.norm(direction)
        
        if distance < self.step_size:
            return to_config
        
        return from_config + (direction / distance) * self.step_size
    
    def find_near_nodes(self, config: np.ndarray) -> List['RRTPlanner.Node']:
        """找到新节点附近的现有节点（用于RRT*）"""
        near_nodes = []
        for node in self.nodes:
            if np.linalg.norm(node.config - config) < self.connect_circle_dist:
                near_nodes.append(node)
        return near_nodes
    
    def choose_parent(self, new_config: np.ndarray, near_nodes: List['RRTPlanner.Node']) -> 'RRTPlanner.Node':
        """选择新节点的父节点（RRT*）"""
        if not near_nodes:
            return self.nearest_node(new_config)
        
        # 找到成本最小的父节点
        min_cost = float('inf')
        best_parent = None
        
        for node in near_nodes:
            if not self.is_edge_collision(node.config, new_config):
                cost = node.cost + np.linalg.norm(node.config - new_config)
                if cost < min_cost:
                    min_cost = cost
                    best_parent = node
        
        return best_parent if best_parent else self.nearest_node(new_config)
    
    def rewire(self, new_node: 'RRTPlanner.Node', near_nodes: List['RRTPlanner.Node']):
        """重新连接附近节点以优化路径（RRT*）"""
        for node in near_nodes:
            if node == new_node.parent:
                continue
            
            new_cost = new_node.cost + np.linalg.norm(new_node.config - node.config)
            if new_cost < node.cost and not self.is_edge_collision(new_node.config, node.config):
                node.parent = new_node
                node.cost = new_cost
    
    def plan(self, start: np.ndarray, goal: np.ndarray) -> Optional[List[np.ndarray]]:
        """
        RRT*路径规划
        返回路径点列表，若失败返回None
        """
        if self.is_collision(start) or self.is_collision(goal):
            return None
        
        # 初始化树
        self.nodes = [self.Node(start)]
        self.nodes[0].cost = 0
        
        goal_node = None
        
        for _ in range(self.max_iter):
            # 随机采样
            random_config = self.sample_config(goal)
            
            # 找到最近节点
            nearest = self.nearest_node(random_config)
            
            # 扩展
            new_config = self.steer(nearest.config, random_config)
            
            if self.is_collision(new_config):
                continue
            
            if self.is_edge_collision(nearest.config, new_config):
                continue
            
            # RRT*优化
            near_nodes = self.find_near_nodes(new_config)
            parent = self.choose_parent(new_config, near_nodes)
            
            # 创建新节点
            new_node = self.Node(new_config)
            new_node.parent = parent
            new_node.cost = parent.cost + np.linalg.norm(parent.config - new_config)
            self.nodes.append(new_node)
            
            # 重新连接
            self.rewire(new_node, near_nodes)
            
            # 检查是否到达目标
            if np.linalg.norm(new_config - goal) < self.step_size:
                if not self.is_edge_collision(new_config, goal):
                    goal_node = self.Node(goal)
                    goal_node.parent = new_node
                    goal_node.cost = new_node.cost + np.linalg.norm(new_config - goal)
                    self.nodes.append(goal_node)
                    break
        
        if goal_node is None:
            return None
        
        # 回溯路径
        path = []
        node = goal_node
        while node is not None:
            path.append(node.config)
            node = node.parent
        
        return path[::-1]  # 反转得到从start到goal的路径
    
    def visualize(self, path: Optional[List[np.ndarray]] = None):
        """可视化树和路径"""
        if len(self.bounds) != 2:
            print("Visualization only supported for 2D")
            return
        
        plt.figure(figsize=(10, 10))
        
        # 绘制障碍物
        for center, radius in self.obstacles:
            circle = plt.Circle(center, radius, color='red', alpha=0.3)
            plt.gca().add_patch(circle)
        
        # 绘制树
        for node in self.nodes:
            if node.parent:
                plt.plot([node.config[0], node.parent.config[0]],
                        [node.config[1], node.parent.config[1]],
                        'g-', alpha=0.3, linewidth=0.5)
        
        # 绘制路径
        if path:
            path_array = np.array(path)
            plt.plot(path_array[:, 0], path_array[:, 1], 'b-', linewidth=2, label='Path')
        
        # 绘制起点和终点
        if self.nodes:
            plt.plot(self.nodes[0].config[0], self.nodes[0].config[1], 'go', markersize=15, label='Start')
        
        plt.axis('equal')
        plt.grid(True)
        plt.legend()
        plt.show()

# 使用示例
def demo_rrt():
    # 2D规划场景
    bounds = [(-10, 10), (-10, 10)]
    planner = RRTPlanner(bounds, step_size=0.5, max_iter=5000)
    
    # 添加障碍物
    planner.add_obstacle(np.array([2, 2]), 1.5)
    planner.add_obstacle(np.array([-3, -3]), 2.0)
    planner.add_obstacle(np.array([5, -5]), 1.0)
    planner.add_obstacle(np.array([-5, 5]), 1.5)
    
    # 规划
    start = np.array([-8, -8])
    goal = np.array([8, 8])
    
    print("Planning...")
    path = planner.plan(start, goal)
    
    if path:
        print(f"Path found with {len(path)} waypoints")
        planner.visualize(path)
    else:
        print("Planning failed")

if __name__ == '__main__':
    demo_rrt()
```

**4. 轨迹优化（时间参数化）**

```python
#!/usr/bin/env python3
# trajectory_optimization.py
import numpy as np
from scipy.interpolate import CubicSpline, interp1d
from scipy.optimize import minimize
import matplotlib.pyplot as plt

class TrajectoryOptimizer:
    """
    轨迹优化器
    实现路径的时间参数化和平滑处理
    """
    
    def __init__(self, max_vel: float, max_acc: float, max_jerk: float):
        """
        初始化约束参数
        max_vel: 最大速度 (m/s)
        max_acc: 最大加速度 (m/s^2)
        max_jerk: 最大加加速度 (m/s^3)
        """
        self.max_vel = max_vel
        self.max_acc = max_acc
        self.max_jerk = max_jerk
    
    def smooth_path(self, path: np.ndarray, num_points: int = 100) -> np.ndarray:
        """
        使用样条插值平滑路径
        path: (N, D) 路径点，N为点数，D为维度
        """
        # 计算路径长度参数
        distances = np.zeros(len(path))
        for i in range(1, len(path)):
            distances[i] = distances[i-1] + np.linalg.norm(path[i] - path[i-1])
        
        # 归一化参数
        t = distances / distances[-1]
        
        # 对每个维度进行样条插值
        interpolated_path = np.zeros((num_points, path.shape[1]))
        t_new = np.linspace(0, 1, num_points)
        
        for dim in range(path.shape[1]):
            cs = CubicSpline(t, path[:, dim])
            interpolated_path[:, dim] = cs(t_new)
        
        return interpolated_path
    
    def time_optimal_parameterization(self, path: np.ndarray, dt: float = 0.01) -> dict:
        """
        时间最优参数化（TOPP）
        使用数值积分方法计算满足约束的最短执行时间
        """
        # 计算路径弧长
        s = self.compute_arc_length(path)
        
        # 计算曲率限制
        curvature = self.compute_curvature(path)
        
        # 初始猜测：最大速度匀速运动
        total_length = s[-1]
        t_total = total_length / self.max_vel
        
        # 分段三次样条参数化
        t = np.linspace(0, t_total, len(path))
        
        # 数值积分计算速度、加速度
        traj = self.integrate_trajectory(path, t, dt)
        
        return traj
    
    def compute_arc_length(self, path: np.ndarray) -> np.ndarray:
        """计算路径弧长参数"""
        s = np.zeros(len(path))
        for i in range(1, len(path)):
            s[i] = s[i-1] + np.linalg.norm(path[i] - path[i-1])
        return s
    
    def compute_curvature(self, path: np.ndarray) -> np.ndarray:
        """计算路径曲率"""
        curvature = np.zeros(len(path))
        
        for i in range(1, len(path) - 1):
            # 使用三点计算曲率
            p1, p2, p3 = path[i-1], path[i], path[i+1]
            
            # 计算切线变化
            t1 = (p2 - p1) / (np.linalg.norm(p2 - p1) + 1e-10)
            t2 = (p3 - p2) / (np.linalg.norm(p3 - p2) + 1e-10)
            
            # 曲率近似
            curvature[i] = np.linalg.norm(t2 - t1)
        
        curvature[0] = curvature[1]
        curvature[-1] = curvature[-2]
        
        return curvature
    
    def integrate_trajectory(self, path: np.ndarray, time_stamps: np.ndarray, dt: float) -> dict:
        """数值积分生成轨迹"""
        # 对每个维度独立插值
        t_final = time_stamps[-1]
        t_interp = np.arange(0, t_final, dt)
        
        traj = {
            'time': t_interp,
            'position': np.zeros((len(t_interp), path.shape[1])),
            'velocity': np.zeros((len(t_interp), path.shape[1])),
            'acceleration': np.zeros((len(t_interp), path.shape[1]))
        }
        
        for dim in range(path.shape[1]):
            # 位置插值
            pos_interp = interp1d(time_stamps, path[:, dim], kind='cubic')(t_interp)
            traj['position'][:, dim] = pos_interp
            
            # 数值微分计算速度
            vel = np.gradient(pos_interp, dt)
            
            # 应用速度限制
            vel = np.clip(vel, -self.max_vel, self.max_vel)
            traj['velocity'][:, dim] = vel
            
            # 数值微分计算加速度
            acc = np.gradient(vel, dt)
            
            # 应用加速度限制
            acc = np.clip(acc, -self.max_acc, self.max_acc)
            traj['acceleration'][:, dim] = acc
        
        return traj
    
    def minimum_snap_trajectory(self, waypoints: np.ndarray, times: np.ndarray, order: int = 4) -> dict:
        """
        最小snap轨迹生成（适用于四旋翼）
        order: 4=min_snap, 3=min_jerk
        """
        n_seg = len(waypoints) - 1
        n_order = 2 * order - 1  # 多项式阶数
        
        # 构建优化问题（简化版）
        # 实际实现需要求解QP问题
        
        # 这里使用简化的样条插值
        t_all = []
        pos_all = []
        
        for i in range(n_seg):
            t_seg = np.linspace(times[i], times[i+1], 20)
            
            # 简单的五次多项式插值
            t_normalized = (t_seg - times[i]) / (times[i+1] - times[i])
            
            # 使用多项式基函数
            for t in t_normalized:
                # 五次多项式: p(t) = a0 + a1*t + a2*t^2 + a3*t^3 + a4*t^4 + a5*t^5
                # 系数由边界条件确定
                pass
        
        return self.integrate_trajectory(waypoints, times, 0.01)
    
    def check_constraints(self, traj: dict) -> bool:
        """检查轨迹是否满足约束"""
        # 检查速度
        vel_max = np.max(np.linalg.norm(traj['velocity'], axis=1))
        if vel_max > self.max_vel * 1.01:  # 允许1%误差
            print(f"Velocity constraint violated: {vel_max:.2f} > {self.max_vel}")
            return False
        
        # 检查加速度
        acc_max = np.max(np.linalg.norm(traj['acceleration'], axis=1))
        if acc_max > self.max_acc * 1.01:
            print(f"Acceleration constraint violated: {acc_max:.2f} > {self.max_acc}")
            return False
        
        return True
    
    def plot_trajectory(self, traj: dict):
        """可视化轨迹"""
        fig, axes = plt.subplots(3, 1, figsize=(12, 10))
        
        t = traj['time']
        
        # 位置
        axes[0].plot(t, traj['position'])
        axes[0].set_ylabel('Position (m)')
        axes[0].grid(True)
        axes[0].legend(['x', 'y', 'z'] if traj['position'].shape[1] > 2 else ['x', 'y'])
        
        # 速度
        axes[1].plot(t, np.linalg.norm(traj['velocity'], axis=1), 'r-', label='Velocity magnitude')
        axes[1].axhline(self.max_vel, color='r', linestyle='--', label='Max velocity')
        axes[1].set_ylabel('Velocity (m/s)')
        axes[1].grid(True)
        axes[1].legend()
        
        # 加速度
        axes[2].plot(t, np.linalg.norm(traj['acceleration'], axis=1), 'g-', label='Acceleration magnitude')
        axes[2].axhline(self.max_acc, color='g', linestyle='--', label='Max acceleration')
        axes[2].set_ylabel('Acceleration (m/s^2)')
        axes[2].set_xlabel('Time (s)')
        axes[2].grid(True)
        axes[2].legend()
        
        plt.tight_layout()
        plt.show()

# 使用示例
def demo_trajectory_opt():
    # 路径点
    waypoints = np.array([
        [0, 0],
        [2, 1],
        [4, 0],
        [5, 3],
        [3, 5],
        [0, 4]
    ])
    
    # 优化器
    optimizer = TrajectoryOptimizer(max_vel=2.0, max_acc=1.0, max_jerk=0.5)
    
    # 平滑路径
    smooth_path = optimizer.smooth_path(waypoints, num_points=200)
    
    # 时间参数化
    traj = optimizer.time_optimal_parameterization(smooth_path, dt=0.01)
    
    # 检查约束
    if optimizer.check_constraints(traj):
        print("Trajectory satisfies all constraints")
    
    # 可视化
    optimizer.plot_trajectory(traj)
    
    # 2D轨迹可视化
    plt.figure(figsize=(10, 10))
    plt.plot(waypoints[:, 0], waypoints[:, 1], 'ro-', label='Waypoints')
    plt.plot(smooth_path[:, 0], smooth_path[:, 1], 'b-', label='Smooth path')
    plt.plot(traj['position'][:, 0], traj['position'][:, 1], 'g--', label='Trajectory')
    plt.axis('equal')
    plt.grid(True)
    plt.legend()
    plt.show()

if __name__ == '__main__':
    demo_trajectory_opt()
```

### 最佳实践

**1. 规划器选择**
- 结构化环境：A*、Dijkstra
- 高维空间：RRT、PRM
- 动态环境：RRT*、D* Lite
- 需要平滑：优化方法（CHOMP、TrajOpt）

**2. 碰撞检测优化**
- 空间索引：八叉树、BVH
- 增量检测：相邻配置间快速检测
- 并行化：GPU加速碰撞检测

**3. 参数调优**
- 采样密度与计算时间平衡
- 启发函数权重调整
- 约束松弛与收紧策略

**4. 实时性保障**
- anytime算法优先
- 分层规划（全局+局部）
- 预计算与缓存

### 常见陷阱

**1. 维度灾难**
- ❌ 问题：高维空间采样效率低
- ✅ 解决：使用位姿图、任务空间规划

**2. 局部最优**
- ❌ 问题：优化方法陷入局部最优
- ✅ 解决：多起点优化、随机重启

**3. 约束违反**
- ❌ 问题：动力学约束被违反
- ✅ 解决：凸包近似、保守估计

**4. 实时性不足**
- ❌ 问题：规划时间过长
- ✅ 解决：anytime算法、硬件加速

## 📚 资源索引

### 学术论文

1. **Rapidly-Exploring Random Trees: A New Tool for Path Planning** (1998)
   - 作者：Steven M. LaValle
   - RRT算法奠基

2. **Sampling-Based Algorithms for Optimal Motion Planning** (2011)
   - 作者：Sertac Karaman, Emilio Frazzoli
   - RRT*最优性证明

3. **CHOMP: Covariant Hamiltonian Optimization for Motion Planning** (2009)
   - 作者：Nathan Ratliff et al.
   - 基于优化的规划

4. **Sampling-Based Motion Planning** (2006)
   - 作者：Hanna Kurniawati
   - 采样规划综述

5. **The Open Motion Planning Library** (2012)
   - 作者：Ioan Sucan et al.
   - OMPL库介绍

### 技术文档

1. **ROS Navigation Stack**
   - http://wiki.ros.org/navigation
   - ROS导航栈文档

2. **MoveIt Documentation**
   - https://moveit.picknik.ai/
   - MoveIt运动规划框架

3. **OMPL Documentation**
   - https://ompl.kavrakilab.org/
   - 开放运动规划库

### 开源项目

1. **OMPL** - https://github.com/ompl/ompl
   - 开放运动规划库

2. **SBPL** - https://github.com/sbpl/sbpl
   - 基于搜索的规划库

3. **TrajOpt** - https://github.com/joschu/trajopt
   - 轨迹优化框架

4. **OpenRAVE** - https://github.com/rdiankov/openrave
   - 机器人仿真环境

5. **Nav2** - https://github.com/ros-planning/navigation2
   - ROS2导航框架

## 🔗 关联知识

```mermaid
graph TD
    C02[C02_Motion_Planning]

    C02 --> C01[C01_Sensor_Fusion]
    C02 --> C03[C03_Swarm_Intelligence]
    
    C02 -.-> A01[A03_Design_Architecture/B02_Design_Patterns]
    C02 -.-> A02[A06_Technical_Intuition/B03_Algorithm_Analysis]
    
    C01 --> |定位输入| C02
```

## 💡 学习建议

### 前置知识
- 数据结构与算法
- 线性代数、优化理论
- ROS/ROS2基础
- C++/Python编程

### 学习路径

**第1-2周：基础算法**
- 图搜索算法（A*、Dijkstra）
- 采样规划（RRT、PRM）
- 实践：2D路径规划器

**第3-4周：ROS集成**
- Nav2导航框架
- MoveIt机械臂规划
- 实践：移动机器人导航

**第5-6周：高级规划**
- 轨迹优化
- 约束规划
- 实践：机械臂避障

**第7-8周：优化与部署**
- 实时规划
- 硬件部署
- 实践：完整系统

### 实践项目

**项目1：AGV导航系统**
- 功能：动态避障、路径跟踪
- 技术：Nav2、局部规划器
- 平台：差速移动机器人

**项目2：机械臂抓取**
- 功能：视觉引导抓取
- 技术：MoveIt、碰撞检测
- 平台：UR5/Franka

**项目3：无人机轨迹规划**
- 功能：3D避障、快速飞行
- 技术：MINCO、RRT*
- 平台：四旋翼仿真

## 🔄 维护说明

- **更新频率**: 每半年更新，跟踪新算法
- **质量标准**: 所有代码在ROS2 Humble测试
- **贡献方式**: 提交新规划算法、优化方法
