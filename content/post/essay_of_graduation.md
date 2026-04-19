---
title: "毕设代码"
date: 2026-04-19
#image: "cover.jpg"   # 封面图（可选）
math: true            # 如果有公式，记得开启
hidden: true
categories:
    - "本科毕业设计"      # 这里填你想划分的板块名
tags:
    - "毕业设计"
---


```python
import pulp

# ==========================================
# 1. 模拟数据 (简化版地图与需求)
# ==========================================
nodes = [0, 1, 2, 3, 4]  # 0为调度中心(Depot)，1-4为共享单车站点
stations = [1, 2, 3, 4]
time_limit = 100   # 车辆最大行驶时间 (对应 T_max)
cap_limit = 20    # 车辆最大载重量 (对应 Q_max)
K_vehicles = 4   # 可用调度车数量

# 行驶时间/距离矩阵 (对称简化)
dist = {
    (0,1):10, (0,2):15, (0,3):20, (0,4):25,
    (1,0):10, (1,2):5,  (1,3):15, (1,4):20,
    (2,0):15, (2,1):5,  (2,3):10, (2,4):15,
    (3,0):20, (3,1):15, (3,2):10, (3,4):5,
    (4,0):25, (4,1):20, (4,2):15, (4,3):5,
}

# 站点收益与单车调度需求量 (模拟)
revenues = {1: 10, 2: 12, 3: 15, 4: 10}
demands = {1: 5, 2: 8, 3: 10, 4: 6}

# ==========================================
# 2. 动态规划标签算法 (求解定价子问题)
# ==========================================
class Label:
    def __init__(self, node, cost, time, net_load, max_l, min_l, visited, parent=None):
        self.node = node
        self.cost = cost
        self.time = time
        self.net_load = net_load  # 当前累计的净负载变化（送货为负，取货为正）
        self.max_l = max_l        # 路径至今出现的最高 net_load
        self.min_l = min_l        # 路径至今出现的最低 net_load
        self.visited = visited
        self.parent = parent

    def to_path(self):
        path = []
        curr = self
        while curr:
            path.append(curr.node)
            curr = curr.parent
        return path[::-1]
    
def dominates(l1, l2):
    """
    支配准则 (Dominance Rule)：
    如果 l1 在所有资源消耗和成本上都不劣于 l2，且 l1 访问的节点数更少或相等，
    则 l1 支配 l2，l2 会被剪枝。
    """
    if l1.cost <= l2.cost and l1.time <= l2.time and l1.load <= l2.load:
        if l1.visited.issubset(l2.visited):
            return True
    return False

def solve_pricing_dp(duals, pi_0):
    # 初始化：起点 net_load, max_l, min_l 均为 0
    initial_label = Label(0, 0.0, 0, 0, 0, 0, frozenset({0}))
    unprocessed = [initial_label]
    pareto_labels = {i: [] for i in nodes}
    pareto_labels[0].append(initial_label)
    best_final_label = None

    while unprocessed:
        curr = unprocessed.pop(0)

        for next_node in nodes:
            if next_node == curr.node: continue
            if next_node in curr.visited and next_node != 0: continue
            if next_node == 0 and len(curr.visited) == 1: continue

            # --- 1. 时间约束 ---
            travel_time = dist[(curr.node, next_node)]
            new_time = curr.time + travel_time
            if new_time + (dist[(next_node, 0)] if next_node != 0 else 0) > time_limit:
                continue

            # --- 2. 改进后的载重约束 (关键修改) ---
            # 约定：demands[i] > 0 代表需要取走(溢出)，车内增加； < 0 代表需要送入(缺失)，车内减少
            # 如果你的数据定义相反，只需把下面的 demand 符号反过来即可
            demand = demands.get(next_node, 0)
            new_net_load = curr.net_load + demand
            new_max_l = max(curr.max_l, new_net_load)
            new_min_l = min(curr.min_l, new_net_load)

            # 只要最高和最低的差值在容量范围内，路径就合法
            if (new_max_l - new_min_l) > cap_limit:
                continue

            # --- 3. 边权与支配检查 ---
            if next_node == 0:
                edge_cost = 0
            else:
                # 无论取还是送，只要服务了该站，就获得对偶值奖励
                edge_cost = - revenues[next_node] + duals[next_node]

            new_cost = curr.cost + edge_cost
            new_visited = frozenset(curr.visited | {next_node})
            new_label = Label(next_node, new_cost, new_time, new_net_load, new_max_l, new_min_l, new_visited, curr)

            # 简化的支配检查 (增加了对 max/min 差值的考虑)
            is_dominated = False
            for existing in pareto_labels[next_node]:
                # 如果 existing 在成本、时间上更优，且对载荷的占用（差值）也更小，则支配
                if (existing.cost <= new_label.cost and 
                    existing.time <= new_label.time and 
                    (existing.max_l - existing.min_l) <= (new_label.max_l - new_label.min_l) and
                    existing.visited.issubset(new_label.visited)):
                    is_dominated = True
                    break
            
            if not is_dominated:
                pareto_labels[next_node].append(new_label)
                unprocessed.append(new_label)
                if next_node == 0:
                    if best_final_label is None or new_cost < best_final_label.cost:
                        best_final_label = new_label

    if best_final_label and best_final_label.cost < -pi_0 - 1e-5:
        return best_final_label.to_path(), best_final_label.cost
    return None, 0

# ==========================================
# 3. 列生成主循环
# ==========================================
# 初始路径池：赋予每辆车最基础的单点往返方案
routes_pool = []
for i in stations:
    real_revenue = revenues[i]
    if real_revenue > 0: #先把所有的有单车需求的站点的路径加路径池
        routes_pool.append({'path': [0, i, 0], 'revenue': real_revenue, 'covers': {i: 1}})

iteration = 0
# 1. 在 while 循环开始前，初始化一个已存在路径的集合
existing_paths = set(tuple(r['path']) for r in routes_pool)
while True:
    iteration += 1
    print(f"\n--- 受限主问题的迭代次数： {iteration} ---")
    
    # A. 求解受限主问题 (RMP)
    prob = pulp.LpProblem("RMP", pulp.LpMaximize)
    
    # 决策变量 Zr (线性松弛为连续变量)
    Z = [pulp.LpVariable(f"Z_{r}", lowBound=0, upBound=1, cat='Continuous') for r in range(len(routes_pool))]
    
    prob += pulp.lpSum([Z[r] * routes_pool[r]['revenue'] for r in range(len(routes_pool))])
    
    # 约束：各站点覆盖约束与车辆数限制
    cover_cons = {}
    for i in stations:
        cover_cons[i] = pulp.lpSum([Z[r] for r in range(len(routes_pool)) if routes_pool[r]['covers'].get(i, 0) == 1]) <= 1
        prob += cover_cons[i]
        
    fleet_con = pulp.lpSum(Z) <= K_vehicles
    prob += fleet_con
    
    prob.solve(pulp.PULP_CBC_CMD(msg=0))
    print(f"当前主问题目标收益: {pulp.value(prob.objective):.2f}")
    
    # B. 获取对偶变量传递给子问题
    duals = {i: cover_cons[i].pi for i in stations}
    pi_0 = fleet_con.pi
    
    # C. 求解定价子问题
    new_path, best_cost = solve_pricing_dp(duals, pi_0)
    
    if new_path:
        # --- 新增查重逻辑 ---
        path_tuple = tuple(new_path)
        if path_tuple in existing_paths:
            print(f"警告：子问题返回了已存在的路径 {new_path}，停止迭代以防死循环。")
            break

        print(f"引入新路径: {new_path}")
        new_revenue = sum(revenues[n] for n in new_path if n != 0)
        routes_pool.append({
            'path': new_path,
            'revenue': new_revenue,
            'covers': {n: 1 for n in new_path if n != 0}
        })
        # --- 更新查重集合 ---
        existing_paths.add(path_tuple)
    else:
        print("未发现负检验数路径，列生成收敛完毕。")
        break

# ==========================================
# 4. 恢复整数约束，获得最终解 (Price-and-Branch)
# ==========================================
print("\n===主问题及定价子问题已得到最优解===")
print("\n=== 最终 0-1 整数规划求解 ===")
prob_ip = pulp.LpProblem("Final_IP", pulp.LpMaximize)
Z_ip = [pulp.LpVariable(f"Zip_{r}", cat='Binary') for r in range(len(routes_pool))]

prob_ip += pulp.lpSum([Z_ip[r] * routes_pool[r]['revenue'] for r in range(len(routes_pool))])

for i in stations:
    prob_ip += pulp.lpSum([Z_ip[r] for r in range(len(routes_pool)) if routes_pool[r]['covers'].get(i, 0) == 1]) <= 1
prob_ip += pulp.lpSum(Z_ip) <= K_vehicles

prob_ip.solve(pulp.PULP_CBC_CMD(msg=0))

print(f"最终敲定的总收益: {pulp.value(prob_ip.objective)}")
print("选中的最优调度路径:")
for r in range(len(routes_pool)):
    if pulp.value(Z_ip[r]) > 0.5:
        print(f" -> 路线: {routes_pool[r]['path']} (对应收益: {routes_pool[r]['revenue']})")
```