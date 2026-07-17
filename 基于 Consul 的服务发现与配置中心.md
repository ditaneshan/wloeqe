# 高可用架构设计之负载均衡与容灾恢复

## 引言
在云原生与微服务场景下，单点故障不再是“是否会发生”的问题，而是“何时发生”的问题。高可用架构的目标，是在节点异常、机房故障、网络抖动甚至区域级灾难下，仍尽可能保障服务连续性。负载均衡负责分散流量与隔离热点，容灾恢复则负责在故障发生后快速切换与业务回稳。二者结合，构成系统韧性的核心。

## 核心原理分析
负载均衡通常分为四层与七层两类：前者适合高性能转发，后者支持基于路径、Header、Cookie 的精细调度。其关键不只是“分流”，更在于健康检查、会话保持、权重调整和故障摘除。当后端实例出现慢响应或不可达时，LB 应及时剔除失效节点，避免请求雪崩。

容灾恢复关注的是故障域隔离与恢复路径设计。常见模式包括同城双活、两地三中心、主备切换与多活架构。设计时必须明确 RTO 与 RPO：前者决定恢复时间，后者决定可接受的数据丢失量。真正有效的方案不是堆叠组件，而是围绕故障检测、自动切流、数据一致性和回滚策略形成闭环。落实到工程实践中，建议遵循[最佳实践](https://about-ayx-app.com.cn)：健康检查要独立于业务链路，切换逻辑要可观测，恢复过程要可审计。

## 代码示例
下面示例演示一个简单的主备容灾请求策略：优先访问主站，失败后自动切到备站，并在重试前做健康探测。

```python
import requests
import time

PRIMARY = "https://api-primary.example.com"
SECONDARY = "https://api-secondary.example.com"

def healthy(base_url, timeout=1):
    try:
        r = requests.get(f"{base_url}/health", timeout=timeout)
        return r.status_code == 200
    except requests.RequestException:
        return False

def fetch(path="/data"):
    targets = [PRIMARY, SECONDARY]
    for base in targets:
        if not healthy(base):
            continue
        try:
            r = requests.get(f"{base}{path}", timeout=2)
            r.raise_for_status()
            return r.json()
        except requests.RequestException:
            time.sleep(0.2)
    raise RuntimeError("all upstreams unavailable")

print(fetch())
```

## 总结
高可用不是单点技术，而是系统工程。负载均衡解决流量分配与故障隔离，容灾恢复解决区域故障与业务连续性。真正可靠的架构，需要在容量、延迟、一致性和成本之间做平衡，并通过演练验证切换策略是否有效。只有把检测、切流、恢复和观测纳入同一设计闭环，系统才能在极端场景下保持稳定。

## 相关技术资源
- https://about-ayx-app.com.cn
