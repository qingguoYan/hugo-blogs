---
title: "使用 Axios AbortController 优雅解决重复请求竞态问题"
description: 在前端开发中，处理重复请求就像指挥交通 - 你需要确保最新的请求总能优先通过，而陈旧的请求会被及时取消，避免造成数据混乱
date: 2024-12-27
draft: true
tags: [javascript]
categories: [JS]
---

**效果图**

![alt text](/images/js-unique-request.webp)

## 问题背景：为什么需要取消重复请求？

当用户快速连续触发相同 API 请求时（如频繁点击搜索按钮），我们会遇到两个关键问题：

1. 竞态条件（Race Condition）：先发起的请求可能比后发起的请求更晚返回
2. 资源浪费：不必要的请求继续占用网络资源和服务器负载

## 解决方案核心：AbortController 的威力

代浏览器提供的 **AbortController** 接口是我们的终极武器，它允许我们：

✅ 主动取消正在进行的网络请求  
✅ 避免过时响应污染应用状态  
✅ 显著提升应用性能和用户体验

axios 支持 AbortController 功能,创建一个 AbortController 实例，将实例对象的 signal 作为 axios 请求参数，调用 abort 方法可以终止请求

```js
// 魔法发生的核心代码
const controller = new AbortController();
axios.get("/api/data", { signal: controller.signal });
controller.abort(); // 随时取消请求！
```

## 完整实现方案

> 基于 axios，我们可以为每一个请求生成唯一的请求标识 requestId，并使用一个 map 存储 requestId 和该请求 AbortController 的映射关系。当我们发送请求时，判断 map 是否存在该请求的 id，如果存在则调用该请求 AbortController 的终止方法，并重新生成一个 AbortController。

### 请求管理中心实现

我们创建一个智能请求管理器，它会自动追踪和管理所有进行中的请求：

```js
// 请求管理中心 - 全局单例
const requestControlCenter = new Map<symbol, AbortController>();

export const smartRequest = async <T>({
  url,
  requestId,
  ...config
}: SmartRequestConfig): Promise<T> => {
  // 检查并取消重复请求
  const existingController = requestControlCenter.get(requestId);
  if (existingController) {
    existingController.abort('取消重复请求'); // 发送取消信号
    requestControlCenter.delete(requestId); // 清理旧控制器
  }

  // 创建新控制器
  const freshController = new AbortController();
  requestControlCenter.set(requestId, freshController);

  try {
    const response = await axios.request({
      url,
      ...config,
      signal: freshController.signal // 绑定取消信号
    });

    return response.data;
  } finally {
    // 请求完成后清理
    requestControlCenter.delete(requestId);
  }
};
```

### React Hook 集成

专为 React 设计的智能请求 Hook，自动处理组件生命周期：

```js
export const useSmartRequester = () => {
  const requestId = useRef(Symbol()).current;

  // 组件卸载时自动取消未完成请求
  useEffect(() => {
    return () => {
      const controller = requestControlCenter.get(requestId);
      if (controller) {
        controller.abort("组件卸载取消请求");
        requestControlCenter.delete(requestId);
      }
    };
  }, [requestId]);

  return useCallback(
    (config) => {
      return smartRequest({ ...config, requestId });
    },
    [requestId]
  );
};
```

### 实际使用示例

```js
function SearchComponent() {
  const [results, setResults] = useState([]);
  const smartRequest = useSmartRequester();

  const handleSearch = async (query) => {
    try {
      const data = await smartRequest({
        url: "/api/search",
        method: "GET",
        params: { q: query },
      });
      setResults(data);
    } catch (err) {
      if (!axios.isCancel(err)) {
        // 处理真实错误（非取消错误）
        console.error("搜索失败:", err);
      }
    }
  };

  return (
    <div>
      <input
        onChange={(e) => handleSearch(e.target.value)}
        placeholder="输入搜索关键词..."
      />
      <SearchResults data={results} />
    </div>
  );
}
```
