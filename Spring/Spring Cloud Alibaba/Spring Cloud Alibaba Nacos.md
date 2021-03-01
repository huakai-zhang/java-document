# 1 什么是 Nacos

Nacos 用于发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，快速实现动态服务发现、服务配置、服务元数据及流量管理。

Nacos 是构建以“服务”为中心的现代应用架构 (例如微服务范式、云原生范式) 的服务基础设施。

# 2 Nacos 基本使用

下载 releases https://github.com/alibaba/nacos/releases

![image-20210226153646442](Spring Cloud Alibaba Nacos.assets/image-20210226153646442.png)



## 2.1 快速开始

```shell
# 源码目录运行
mvn -Prelease-nacos -Dmaven.test.skip=true clean install -U
cd distribution/target/nacos-server-$version/nacos/bin
# 启动服务器
sh startup.sh -m standalone
```

也可以直接从 [最新稳定版本](https://github.com/alibaba/nacos/releases) 下载 `nacos-server` 包，解压运行 bin 目录 startup.sh 启动服务器。

![image-20210226155449039](Spring Cloud Alibaba Nacos.assets/image-20210226155449039.png)

## 2.2 Nacos Spring Boot

```xml
<!-- 版本 0.2.x.RELEASE 对应的是 Spring Boot 2.x 版本，版本 0.1.x.RELEASE 对应的是 Spring Boot 1.x 版本 -->
<dependency>
    <groupId>com.alibaba.boot</groupId>
    <artifactId>nacos-config-spring-boot-starter</artifactId>
    <version>0.2.7</version>
</dependency>
```

```yml
nacos:
  config:
    server-addr: localhost:8848
```

```java
// dataId groupId 用于进行配置隔离
// dataId 是一个数据集
// group 是分组
@RestController
@NacosPropertySource(dataId = "spring", groupId = "DEFAULT_GROUP", autoRefreshed = true)
public class NacosController {
	// hello Nacos 表示本地属性，降级
    @NacosValue(value = "${info:hello Nacos}", autoRefreshed = true)
    private String info;

    @GetMapping("nacos")
    public String nacos() {
        return info;
    }
}
```

> Nacos 提供两种方式来访问和改变配置信息：
>
> Open API(上诉 Spring Boot 加载配置我呢见)
>
> SDK(如下)

## 2.3 SDK 方式

```xml
<dependency>
    <groupId>com.alibaba.nacos</groupId>
    <artifactId>nacos-client</artifactId>
    <version>1.4.1</version>
</dependency>
```

```java
public class NacosSdkDemo {
    private static final String SERVER_ADDR = "localhost:8848";
    private static final String DATA_ID = "spring";
    private static final String GROUP_ID = "DEFAULT_GROUP";

    public static void main(String[] args) {
        Properties properties = new Properties();
        properties.put("serverAddr", SERVER_ADDR);

        try {
            ConfigService configService = NacosFactory.createConfigService(properties);
            String info = configService.getConfig(DATA_ID, GROUP_ID, 3000);
            System.out.println(info);
		   // 监听
            configService.addListener(DATA_ID, GROUP_ID, new Listener() {
                @Override
                public Executor getExecutor() {
                    return null;
                }
                @Override
                public void receiveConfigInfo(String configInfo) {
                    System.out.println("配置改变==》" + configInfo);
                }
            });
            System.in.read();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## 2.4 配置数据库

Nacos 在单机模式下默认使用内置数据库 `derby`。

### 使用 MySQL

1. 创建数据库 nacos-config 并运行 `nacos/conf/nacos-mysql.sql` 的 SQL 文件

2. 修改 `nacos/conf/application.properties` 配置

   ```properties
   #*************** Config Module Related Configurations ***************#
   ### If use MySQL as datasource:
   spring.datasource.platform=mysql
   
   ### Count of DB:
   db.num=1
   
   ### Connect URL of DB:
   db.url.0=jdbc:mysql://127.0.0.1:3306/nacos-config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
   db.user.0=root
   db.password.0=root
   ```

3. 使用 MySQL，必须在集群模式下，所以这里在 nacos/conf/ 目录下创建一个 cluster.conf

   ```
   locahost
   locahost
   locahost
   ```

4. 集群模式启动 Nacos

   ```shell
   sh startup.sh -m cluster
   ```

# 3 源码分析

## 3.1 实例化 NacosConfigService

以 NacosFactory.createConfigService(properties) 为入口：

```java
// ConfigFactory.java
public static ConfigService createConfigService(Properties properties) throws NacosException {
    try {
        // 通过反射实例化一个 NacosConfigService
        Class<?> driverImplClass = Class.forName("com.alibaba.nacos.client.config.NacosConfigService");
        Constructor constructor = driverImplClass.getConstructor(Properties.class);
        ConfigService vendorImpl = (ConfigService) constructor.newInstance(properties);
        return vendorImpl;
    } catch (Throwable e) {
        throw new NacosException(NacosException.CLIENT_INVALID_PARAM, e);
    }
}
// NacosConfigService.java
public NacosConfigService(Properties properties) throws NacosException {
    ValidatorUtils.checkInitParam(properties);
    String encodeTmp = properties.getProperty(PropertyKeyConst.ENCODE);
    if (StringUtils.isBlank(encodeTmp)) {
        this.encode = Constants.ENCODE;
    } else {
        this.encode = encodeTmp.trim();
    }
    initNamespace(properties);
    // 通过装饰器模式初始化一个 Http 连接的代理对象，http的方式去发起请求
    // MetricsHttpAgent 中做了一个监控和统计
    this.agent = new MetricsHttpAgent(new ServerHttpAgent(properties));
    this.agent.start();
    this.worker = new ClientWorker(this.agent, this.configFilterChainManager, properties);
}

// clientWorker -> 具体的工作相关
public ClientWorker(final HttpAgent agent, final ConfigFilterChainManager configFilterChainManager,
            final Properties properties) {
		this.agent = agent;
    this.configFilterChainManager = configFilterChainManager;
        
    // Initialize the timeout parameter
    init(properties);
        
    this.executor = Executors.newScheduledThreadPool(1, new ThreadFactory() {
        @Override
        public Thread newThread(Runnable r) {
            Thread t = new Thread(r);
            // 自定义线了线程名称
            t.setName("com.alibaba.nacos.client.Worker." + agent.getName());
            t.setDaemon(true);
            return t;
        }
    });
        
    // 长轮询线程池
    this.executorService = Executors
        .newScheduledThreadPool(Runtime.getRuntime().availableProcessors(), new ThreadFactory() {
        @Override
        public Thread newThread(Runnable r) {
            Thread t = new Thread(r);
            t.setName("com.alibaba.nacos.client.Worker.longPolling." + agent.getName());
            t.setDaemon(true);
            return t;
        }
    });
    // 指定延迟执行，延迟 1 ms 每次执行的间隔是 10 ms
    // 不断检查有没有新的分组需要分配长轮询线程
    this.executor.scheduleWithFixedDelay(new Runnable() {
        @Override
        public void run() {
            try {
                // 检查配置
                checkConfigInfo();
            } catch (Throwable e) {
                LOGGER.error("[" + agent.getName() + "] [sub-check] rotate check error", e);
            }
        }
    }, 1L, 10L, TimeUnit.MILLISECONDS);
}
public void checkConfigInfo() {
    // Dispatch taskes. 分任务，解决了大数据量的长轮询
    int listenerSize = cacheMap.size();
    // Round up the longingTaskCount. 向上取整为整数
    int longingTaskCount = (int) Math.ceil(listenerSize / ParamUtil.getPerTaskConfigSize());
    if (longingTaskCount > currentLongingTaskCount) {
        for (int i = (int) currentLongingTaskCount; i < longingTaskCount; i++) {
            // 为每一个分组分配一个长轮询线程
            executorService.execute(new LongPollingRunnable(i));
        }
        currentLongingTaskCount = longingTaskCount;
    }
}
```

## 3.2 注册 listener

可以看到此处使用了一个内存缓存 cacheMap，cacheMap 会在添加 listener 的时候填充：

```java
// main 方法中在通过 configServer.addListener() 
public void addListener(String dataId, String group, Listener listener) throws NacosException {
    worker.addTenantListeners(dataId, group, Arrays.asList(listener));
}
// ClientWorker.java
public void addTenantListeners(String dataId, String group, List<? extends Listener> listeners) throws NacosException {
    group = null2defaultGroup(group);
    String tenant = agent.getTenant();
    CacheData cache = addCacheDataIfAbsent(dataId, group, tenant);
    for (Listener listener : listeners) {
        cache.addListener(listener);
    }
}
public CacheData addCacheDataIfAbsent(String dataId, String group, String tenant) throws NacosException {
    // 根据 dataId group tenant 生成 cacheMap 的 key
    // 不同 dataId 不同的 CacheData
    String key = GroupKey.getKeyTenant(dataId, group, tenant);
    CacheData cacheData = cacheMap.get(key);
    if (cacheData != null) {
        return cacheData;
    }

    cacheData = new CacheData(configFilterChainManager, agent.getName(), dataId, group, tenant);
    // 如果不存在（新的entry），那么会向map中添加该键值对，并返回null
	// 如果已经存在，那么不会覆盖已有的值，直接返回已经存在的值
    CacheData lastCacheData = cacheMap.putIfAbsent(key, cacheData);
    if (lastCacheData == null) {
        // 是否在监听器注册时主动去向远端拉取当前最新的配置信息
        if (enableRemoteSyncConfig) {
            String[] ct = getServerConfig(dataId, group, tenant, 3000L);
            cacheData.setContent(ct[0]);
        }
        int taskId = cacheMap.size() / (int) ParamUtil.getPerTaskConfigSize();
        cacheData.setTaskId(taskId);
        lastCacheData = cacheData;
    }
    ...
}
// CacheData.java
public CacheData(ConfigFilterChainManager configFilterChainManager, String name, String dataId, String group,
            String tenant) {
    if (null == dataId || null == group) {
        throw new IllegalArgumentException("dataId=" + dataId + ", group=" + group);
    }
    this.name = name;
    this.configFilterChainManager = configFilterChainManager;
    this.dataId = dataId;
    this.group = group;
    this.tenant = tenant;
    listeners = new CopyOnWriteArrayList<ManagerListenerWrap>();
    this.isInitializing = true;
	// 从本地文件读取配置信息  
    this.content = loadCacheContentFromDiskLocal(name, dataId, group, tenant);
    this.md5 = getMd5String(content);
}
```

## 3.3 创建长轮询线程

回到 checkConfigInfo 的 for 循环代码，其中通过 LongPollingRunnable 创建多个线程对缓存中的配置进行检查。

```java
class LongPollingRunnable implements Runnable {
    
    private final int taskId;
    
    public LongPollingRunnable(int taskId) {
        this.taskId = taskId;
    }
    
    @Override
    public void run() {
        
        List<CacheData> cacheDatas = new ArrayList<CacheData>();
        List<String> inInitializingCacheList = new ArrayList<String>();
        try {
            // check failover config
            // 拿到缓存集合中属于对应 taskId 的缓存
            for (CacheData cacheData : cacheMap.values()) {
                if (cacheData.getTaskId() == taskId) {
                    cacheDatas.add(cacheData);
                    try {
                        // 检查本地配置
                        // cacheData 当前配置中的某一个配置
                        checkLocalConfig(cacheData);
                      	// 内存缓存设置成功后
                        if (cacheData.isUseLocalConfigInfo()) {
                            cacheData.checkListenerMd5();
                        }
                    } catch (Exception e) {
                        LOGGER.error("get local config info error", e);
                    }
                }
            }
            // 远程监控
            List<String> changedGroupKeys = checkUpdateDataIds(cacheDatas, inInitializingCacheList);
            if (!CollectionUtils.isEmpty(changedGroupKeys)) {
                LOGGER.info("get changedGroupKeys:" + changedGroupKeys);
            }
			// 根据服务端提供的发生变化的 dataid 去服务端取配置信息
            for (String groupKey : changedGroupKeys) {
                String[] key = GroupKey.parseKey(groupKey);
                String dataId = key[0];
                String group = key[1];
                String tenant = null;
                if (key.length == 3) {
                    tenant = key[2];
                }
                try {
                    // 根据获取到的发生变化的datdaid 去服务端拿配置信息
                    String[] ct = getServerConfig(dataId, group, tenant, 3000L);
                    CacheData cache = cacheMap.get(GroupKey.getKeyTenant(dataId, group, tenant));
                    // 设置 content 和 md5 值
                    cache.setContent(ct[0]);
                    if (null != ct[1]) {
                        cache.setType(ct[1]);
                    }
                ...
            }
            for (CacheData cacheData : cacheDatas) {
                if (!cacheData.isInitializing() || inInitializingCacheList
                    .contains(GroupKey.getKeyTenant(cacheData.dataId, cacheData.group, cacheData.tenant))) {
                    // 校验 md5
                    cacheData.checkListenerMd5();
                    cacheData.setInitializing(false);
                }
            }
            inInitializingCacheList.clear();

            executorService.execute(this);
            ...
        }
    }
}
```

## 3.4 检查本地配置

```java
private void checkLocalConfig(CacheData cacheData) {
    final String dataId = cacheData.dataId;
    final String group = cacheData.group;
    final String tenant = cacheData.tenant;
    // 本地文件缓存
    File path = LocalConfigInfoProcessor.getFailoverFile(agent.getName(), dataId, group, tenant);
    // 检查内存缓存中是否使用本地缓存标识
    // 检查本地文件是否存在
    // 即是本地文件存在且缓存不存在，因为新配置的配置信息，会先加载在文件中
    if (!cacheData.isUseLocalConfigInfo() && path.exists()) {
        // 读取本地文件，放到 cacheData
        String content = LocalConfigInfoProcessor.getFailover(agent.getName(), dataId, group, tenant);
        final String md5 = MD5Utils.md5Hex(content, Constants.ENCODE);
        cacheData.setUseLocalConfigInfo(true);
        cacheData.setLocalConfigInfoVersion(path.lastModified());
        cacheData.setContent(content);

        LOGGER.warn(
            "[{}] [failover-change] failover file created. dataId={}, group={}, tenant={}, md5={}, content={}",
            agent.getName(), dataId, group, tenant, md5, ContentUtils.truncateContent(content));
        return;
    }

    // 内存中存在，但是本地文件没有
    if (cacheData.isUseLocalConfigInfo() && !path.exists()) {
        cacheData.setUseLocalConfigInfo(false);
        LOGGER.warn("[{}] [failover-change] failover file deleted. dataId={}, group={}, tenant={}", agent.getName(),
                    dataId, group, tenant);
        return;
    }

    // 本地内存中都有，但是内存缓存的修改时间和本地缓存不一致
    if (cacheData.isUseLocalConfigInfo() && path.exists() && cacheData.getLocalConfigInfoVersion() != path.lastModified()) {
        String content = LocalConfigInfoProcessor.getFailover(agent.getName(), dataId, group, tenant);
        final String md5 = MD5Utils.md5Hex(content, Constants.ENCODE);
        cacheData.setUseLocalConfigInfo(true);
        cacheData.setLocalConfigInfoVersion(path.lastModified());
        cacheData.setContent(content);
        LOGGER.warn(
            "[{}] [failover-change] failover file changed. dataId={}, group={}, tenant={}, md5={}, content={}",
            agent.getName(), dataId, group, tenant, md5, ContentUtils.truncateContent(content));
    }
}
```

## 3.5 远程监控

本地检查结束之后，回到长轮询程序的后续内容，远程监控：

```java
// 从 server 获取变化了的 dataID
List<String> checkUpdateDataIds(List<CacheData> cacheDatas, List<String> inInitializingCacheList) throws Exception {
    StringBuilder sb = new StringBuilder();
    for (CacheData cacheData : cacheDatas) {
        if (!cacheData.isUseLocalConfigInfo()) {
            sb.append(cacheData.dataId).append(WORD_SEPARATOR);
            sb.append(cacheData.group).append(WORD_SEPARATOR);
            if (StringUtils.isBlank(cacheData.tenant)) {
                sb.append(cacheData.getMd5()).append(LINE_SEPARATOR);
            } else {
                sb.append(cacheData.getMd5()).append(WORD_SEPARATOR);
                sb.append(cacheData.getTenant()).append(LINE_SEPARATOR);
            }
            if (cacheData.isInitializing()) {
                // 当 cacheMap 中的 cacheData 第一次出现时，它会更新
                inInitializingCacheList
                        .add(GroupKey.getKeyTenant(cacheData.dataId, cacheData.group, cacheData.tenant));
            }
        }
    }
    boolean isInitializingCacheList = !inInitializingCacheList.isEmpty();
  	// 把可能发生变化的 dataid 发送到服务端进行判断
  	// 服务端返回发生变化的 dataid
    return checkUpdateConfigStr(sb.toString(), isInitializingCacheList);
}
List<String> checkUpdateConfigStr(String probeUpdateString, boolean isInitializingCacheList) throws Exception {
    Map<String, String> params = new HashMap<String, String>(2);
    params.put(Constants.PROBE_MODIFY_REQUEST, probeUpdateString);
    Map<String, String> headers = new HashMap<String, String>(2);
    headers.put("Long-Pulling-Timeout", "" + timeout);

    // 告诉服务器如果是新数据初始化，不要挂起(直接返回)
    if (isInitializingCacheList) {
        headers.put("Long-Pulling-Timeout-No-Hangup", "true");
    }

    if (StringUtils.isBlank(probeUpdateString)) {
        return Collections.emptyList();
    }

    try {
        long readTimeoutMs = timeout + (long) Math.round(timeout >> 1);
        // 这个请求在服务器端如果对应 dataid 有数据更新、新数据初始化或者超时时候才会返回
        // 超时时间 timeout = Math.max(ConvertUtils.toInt(properties.getProperty(PropertyKeyConst.CONFIG_LONG_POLL_TIMEOUT),Constants.CONFIG_LONG_POLL_TIMEOUT), Constants.MIN_CONFIG_LONG_POLL_TIMEOUT);
        // public static final int CONFIG_LONG_POLL_TIMEOUT = 30000;
        // 默认 30000 ms
        // 实际服务端在 29.5 s 超时，为了保证在网络延迟情况下返回，一般最终时间为 29.5+
        HttpRestResult<String> result = agent
            .httpPost(Constants.CONFIG_CONTROLLER_PATH + "/listener", headers, params, agent.getEncode(),
                      readTimeoutMs);
        if (result.ok()) {
            setHealthServer(true);
            return parseUpdateDataIdResponse(result.getData());
        } else {
            setHealthServer(false);
            LOGGER.error("[{}] [check-update] get changed dataId error, code: {}", agent.getName(),
                         result.getCode());
        }
    } catch (Exception e) {
        setHealthServer(false);
        LOGGER.error("[" + agent.getName() + "] [check-update] get changed dataId exception", e);
        throw e;
    }
    return Collections.emptyList();
}
```

## 3.6 从服务端获取配置信息

长轮询线程拿到服务端发生变化的 dataid 后，会执行 getServerConfig() 方法获取配置信息：

```java
public String[] getServerConfig(String dataId, String group, String tenant, long readTimeout)
        throws NacosException {
    String[] ct = new String[2];
    if (StringUtils.isBlank(group)) {
        group = Constants.DEFAULT_GROUP;
    }
    
    HttpRestResult<String> result = null;
    try {
        Map<String, String> params = new HashMap<String, String>(3);
        if (StringUtils.isBlank(tenant)) {
            params.put("dataId", dataId);
            params.put("group", group);
        } else {
            params.put("dataId", dataId);
            params.put("group", group);
            params.put("tenant", tenant);
        }
        result = agent.httpGet(Constants.CONFIG_CONTROLLER_PATH, null, params, agent.getEncode(), readTimeout);
    } catch (Exception ex) {
        String message = String
                .format("[%s] [sub-server] get server config exception, dataId=%s, group=%s, tenant=%s",
                        agent.getName(), dataId, group, tenant);
        LOGGER.error(message, ex);
        throw new NacosException(NacosException.SERVER_ERROR, ex);
    }
    
    switch (result.getCode()) {
        case HttpURLConnection.HTTP_OK:
            // 本地缓存
            LocalConfigInfoProcessor.saveSnapshot(agent.getName(), dataId, group, tenant, result.getData());
            ct[0] = result.getData();
            if (result.getHeader().getValue(CONFIG_TYPE) != null) {
                ct[1] = result.getHeader().getValue(CONFIG_TYPE);
            } else {
                ct[1] = ConfigType.TEXT.getType();
            }
            return ct;
		...
    }
}
```

## 3.7 MD5 校验

获取到更新过的 dataid 配置信息后，后续会对 MD5 值和之前的 MD5 进行比对：

```java
// CacheData.java
public void setContent(String content) {
    this.content = content;
    this.md5 = getMd5String(this.content);
}
void checkListenerMd5() {
    // 遍历自己定义添加的 listener
    for (ManagerListenerWrap wrap : listeners) {
        // md5 值如果不一致
        if (!md5.equals(wrap.lastCallMd5)) {
            safeNotifyListener(dataId, group, content, type, md5, wrap);
        }
    }
}
private void safeNotifyListener(final String dataId, final String group, final String content, final String type,
            final String md5, final ManagerListenerWrap listenerWrap) {
        final Listener listener = listenerWrap.listener;
        
        Runnable job = new Runnable() {
            @Override
            public void run() {
                ClassLoader myClassLoader = Thread.currentThread().getContextClassLoader();
                ClassLoader appClassLoader = listener.getClass().getClassLoader();
                try {
                    if (listener instanceof AbstractSharedListener) {
                        AbstractSharedListener adapter = (AbstractSharedListener) listener;
                        adapter.fillContext(dataId, group);
                        LOGGER.info("[{}] [notify-context] dataId={}, group={}, md5={}", name, dataId, group, md5);
                    }
                    // 执行回调之前先将线程classloader设置为具体webapp的classloader，以免回调方法中调用spi接口是出现异常或错用（多应用部署才会有该问题）。
                    Thread.currentThread().setContextClassLoader(appClassLoader);
                    
                    ConfigResponse cr = new ConfigResponse();
                    cr.setDataId(dataId);
                    cr.setGroup(group);
                    cr.setContent(content);
                    configFilterChainManager.doFilter(null, cr);
                    String contentTmp = cr.getContent();
                  	// 执行自己定义的 receiveConfigInfo 方法
                    listener.receiveConfigInfo(contentTmp);
                    ...
    }
```

# 4 服务注册发现

# 5 服务端

ConfigController listener

LongPollingService onEvent





集群选举。raft. RaftCore




