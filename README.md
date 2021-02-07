# SWR-Hooks

## 什么是swr
1. 既可以作为请求库
2. 也可以作为状态管理的缓存
3. 全称 stale while revalidate
4. http标准中的缓存策略：
    - 首先从缓存中取数据
    - 然后真实请求响应的数据
    - 最后将缓存值和返回值做对比
    - 相同不更新，否则更新并刷新ui展示

## 基础数据加载
```
import useSWR from 'swr'

function Profile () {
    
  const { data, error } = useSWR('/api/user', fetch)
  
  // 保镖模式（the Bouncer Pattern）， 后面处理正确业务逻辑
  if (error) return <div>failed to load</div>
  
  // 没有错误，且没有数据 只有可能是正常业务流程中的等待取数据
  if (!data) return <div>loading...</div>
  
  // 没有错误有数据，进行渲染
  return <div>hello {data.name}!</div>
}
```

## 多窗口同步功能
1. 在使用 SWR 之后，如果我们在当前应用打开多个窗口或者选项卡。重新聚焦当前页面时候，无需手动或者在代码中重新刷新。SWR 会自动取得数据然后基于 React diff 进行渲染
2. 可以通过配置来决定是否使用该功能

## 快速导航
1. 使用 SWR,我们如果在系统内部进行导航或者按下后退按钮，
2. 直接会取得缓存版本数据。
3. 然后系统为了一致性，呈现了数据之后，会继续请求服务端，重新拉去数据

## 局部突变 mutate
```
import useSWR, { mutate } from 'swr'

function Profile () {
  const { data } = useSWR('/api/user', fetcher)

  return (
    <div>
      <h1>My name is {data.name}.</h1>
      <button onClick={async () => {
        const newName = data.name.toUpperCase()
        // 请求更新名称
        await requestUpdateUsername(newName)
        // 先更新名称，后重新拉去数据验证
        mutate('/api/user', { ...data, name: newName })
      }}>Uppercase my name!</button>
    </div>
  )
}

// requestUpdateUsername 返回 200 无需验证。填写 new User
// 不过该方案仅仅只能修改无乐观锁的数据
mutate('/api/user', newUser, false)

// promise 返回更新的 user。直接更新
mutate('/api/user', requestUpdateUsername(newUser)) 

// 也可以返回 id 与乐观锁
const modifiedUser =  requestUpdateUsername(newUser).then(res => {
   return Object.assign({}, newUser, res)
})
// promise 返回更新的 user。直接更新
mutate('/api/user', modifiedUser) 
```